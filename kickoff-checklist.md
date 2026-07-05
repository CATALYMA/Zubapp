# Kde a jak začít — checklist pro rozjezd (hybridní strategie: DentApp první, CoreKit hranice od začátku)

**Návazný dokument k:** `czech-dental-platform-architecture-modularization.md`, `modularization-veterinary-check.md`
**Jméno sdíleného projektu:** CoreKit (viz `czech-dental-platform-architecture-modularization.md`, sekce 1 a 2) — používat konzistentně ve vývoji, repozitářích i dalších dokumentech.
**Stack:** Vercel (hosting) + Supabase (Postgres, Auth) — poznámky níže odkazují na tento stack, ale checklist samotný je postupový plán, ne technický spec.

**Strategie (rozhodnuto po revizi):** Nestavíme DentApp a VetApp souběžně. Staví se **jen DentApp** jako jediná reálně běžící appka — ale kód od začátku rozdělený podle CoreKit modulů (`@corekit/identity`, `@corekit/booking`...) s hranicemi, které jsme navrhli. VetApp se teď nestaví. Až přijde na řadu, extrakce má být přesun složek/balíčků, ne přepis datového modelu — proto hranice existují od první řádky kódu, i když je zatím jediný konzument. Veterinární perspektiva (wireframy, `modularization-veterinary-check.md`) slouží jako validační cvičení návrhu, ne jako druhá appka k dodání.

---

## Krok 0 — nastavení (než se napíše první řádek modulu)

- [ ] **Založit monorepo s názvem CoreKit** uvnitř DentApp repozitáře (ne jako samostatný sdílený repozitář — na to je zatím jen jeden konzument). Balíčky `@corekit/identity`, `@corekit/notifications`, `@corekit/booking`, `@corekit/messaging`, `@corekit/reviews` jako oddělené workspace balíčky s vynucenými hranicemi (dependency-cruiser/ESLint boundaries), i když je importuje jen DentApp. Doporučený nástroj: **Turborepo**.
- [ ] **Jeden Supabase projekt** (EU region, Frankfurt), multi-tenant schéma od začátku (`tenant_id` všude), i když zatím obsluhuje jen jednoho tenanta (DentApp) — až přibude VetApp, nepřidává se nový projekt, jen nový tenant.
- [ ] **Jeden Vercel projekt (DentApp)** pro teď. Druhý Vercel projekt pro VetApp se založí, až se na VetApp reálně začne pracovat — ne dřív.

---

## Krok 1 — `@corekit/identity`: account/subject model (základní kámen)

- [ ] Rozhodnout přesnou podobu vztahu účet↔subjekt (viz `modularization-veterinary-check.md`): jeden účet, 0..N subjektů péče, žádný předpoklad "účet = subjekt" natvrdo v kódu.
- [ ] Postavit na Supabase Auth (přihlášení, session) + vlastní tabulka subjektů napojená na účet.
- [ ] Vertical-specific rozšíření subjektu (dětský pacient vs. zvíře) řešit jako oddělené doplňkové záznamy navázané na stejný obecný subjekt, ne jako různé typy v jedné tabulce s hromadou nepoužitých sloupců.
- [ ] Ověřit přístupová pravidla: kdo smí vidět/upravovat čí subjekty (rodič vidí dítě, majitel vidí zvíře, personál ordinace vidí jen své pacienty/klienty).
- [ ] Zatím bez UI — cíl je mít funkční a otestovaný datový základ, na kterém stojí všechno další.

---

## Krok 2 — `@corekit/notifications`

- [ ] Šablony pro SMS/e-mail/push s korektní diakritikou a vědomím GSM-7 vs. UCS-2 kódování (cena SMS).
- [ ] Navázat na subjekt, ne na účet přímo (notifikace "za dítě" nebo "za zvíře" musí jít na kontakt účtu, ale obsahově se týkat subjektu).
- [ ] Napsáno bez oborových předpokladů, i když teď má jen jednoho konzumenta (DentApp) — ověřeno myšlenkově proti veterinárnímu wireframu, ne postavením druhé appky.

---

## Krok 3 — `@corekit/booking` (jádro)

- [ ] Kalendář, sloty, prevence dvojitého zápisu — navázané na `subject_id`.
- [ ] Zatím bez napojení na PMS/veterinární informační systém — jen vlastní kalendář ordinace.
- [ ] Ověřit, jestli je potřeba multi-resource booking (např. místnost + lékař zvlášť) hned, nebo to jde odložit.

---

## Krok 4 — `@corekit/messaging` a `@corekit/reviews` (nižší priorita)

- [ ] Až po funkčním bookingu v DentApp. Nejsou blokující pro první spuštění.

---

## Krok 5 — Oborová vrstva (Vrstva 3, jen DentApp teď)

- [ ] Číselník výkonů, role zubař/hygienistka.
- [ ] PMS integrace — až v pozdější fázi, podle původního rollout plánu.

**VetApp (odloženo, nestaví se teď):** až se na ni skutečně začne pracovat, bude potřeba číselník druhů zvířat/výkonů a **[OVĚŘIT]** jaký praxe-management software veterinární ordinace v ČR skutečně používají.

---

## Krok 6 — Payer integrace (odložit na konec)

- [ ] Zubní: adaptér na VZP (podle fáze 2 původního rollout plánu).
- [ ] Veterinární: odloženo spolu s celou VetApp. Až bude aktuální, nejdřív **[OVĚŘIT]** penetraci pojištění mazlíčků v ČR.

---

## Ověření volby stacku (Vercel + Supabase)

Ověřeno proti aktuální dokumentaci obou platforem (ne jen z paměti) — celkový verdikt: **rozumná volba, sedí i na navrženou multi-tenant architekturu, se dvěma body k pohlídání.**

**Co funguje dobře:**

- **EU regiony jsou u obou k dispozici** — Supabase nabízí Frankfurt, Dublin, Londýn, Paříž, Curych, Stockholm; Vercel má vlastní compute regiony ve Frankfurtu, Dublinu, Paříži, Stockholmu. Je potřeba je explicitně zvolit při zakládání projektu (výchozí Vercel region je USA).
- **RLS multi-tenancy v Supabase přesně sedí** na navržený model účet↔subjekt z Kroku 1 — přístupová pravidla na úrovni řádků v Postgresu jsou přesně nástroj, který tenhle vztah potřebuje vynutit.
- **Async/queue požadavek z architektury je pokrytý.** Původní architektura chtěla oddělit payer/PMS integraci od bookingu přes frontu (aby výpadek pojišťovny nezablokoval rezervace). Vercel má na to nativní **Queues** (trvalé zprávy, at-least-once doručení, replikace do tří zón) a **Workflows** (běhy s `sleep()` na libovolně dlouhou dobu — přesně pro naplánování připomínky 24–48 h před návštěvou). Dřív jsem to považoval za riziko (Vercel funkce mají defaultní limit běhu 300 s), teď je vidět, že platforma na to má řešení.
- **Compliance základ je solidní u obou:** SOC 2 Type II a ISO 27001 na Vercelu i Supabase, GDPR DPA (Data Processing Addendum) k dispozici u obou. Supabase navíc nabízí HIPAA add-on (americký zákon, sem se přímo nevztahuje, ale ukazuje, že mají hotová technická opatření — šifrování at rest/in transit, Point-in-Time Recovery, síťová omezení), která jsou dobrým vzorem i pro požadavky GDPR čl. 32 (zabezpečení zpracování) na zdravotní údaje.

**Na co si dát pozor, než se nasadí produkce s citlivými daty:**

- **Plná garance, že data nikdy neopustí EU, není samozřejmá u žádné z platforem na běžném tarifu.** Vercel v dokumentaci výslovně píše, že data mohou být přenášena i mimo EU (kryto SCC/DPA) a u nové služby Queues sami uvádí, že "striktní data residency zatím není podporovaná" (při výpadku regionu se zprávy mohou dočasně uložit jinam). Pokud budete cílit na zakázky s veřejným sektorem nebo pojišťovnami, kde už dřívější dokument flagoval očekávání striktního EU/ČR hostingu, bude to potřeba řešit přes Enterprise tier, nebo využít toho, že Supabase je open-source a jde nakonec i samohostovat.
- **Zapnout "High Compliance" režim na Supabase i bez potřeby HIPAA** — vynucuje Point-in-Time Recovery, SSL enforcement a síťová omezení, což je přesně to, co by mělo být zapnuté i kvůli GDPR čl. 9 datům, i když se HIPAA jako takové netýká.
- **Vercel funkce nejsou vhodné pro dlouhotrvající zpracování napřímo** (limit 300 s, rozšiřitelný na 800 s na Pro/Enterprise) — pro cokoliv delšího nebo asynchronního použít Queues/Workflows, ne přímo funkci s dlouhým timeoutem.

**Zdroje:** [Supabase — Available regions](https://supabase.com/docs/guides/platform/regions), [Supabase — Security](https://supabase.com/security), [Supabase — HIPAA Projects](https://supabase.com/docs/guides/platform/hipaa-projects), [Vercel — Regions](https://vercel.com/docs/regions), [Vercel — Security & Compliance](https://vercel.com/docs/security/compliance), [Vercel — Queues concepts](https://vercel.com/docs/queues/concepts), [Vercel — Functions limits](https://vercel.com/docs/functions/limitations), [Vercel — Cron Jobs](https://vercel.com/docs/cron-jobs)

---

## Co hlídat průběžně

- Nepřidávat do Kroku 1–4 (sdílené moduly) nic, co je specifické jen pro jeden obor — jakmile se to objeví, patří to do Kroku 5.
- Před psaním kódu v Kroku 1 stojí za to udělat krátký technický spec (datový model, přístupová pravidla) — bez něj hrozí, že se "účet = subjekt" předpoklad propíše do kódu znovu, i když víme, že neplatí.
