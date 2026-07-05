# Modulární architektura ZubApp — návrh pro znovupoužitelnost mezi aplikacemi

**Návazný dokument k:** `czech-dental-platform-architecture.md`
**Účel:** Přepracovat rozdělení modulů tak, aby šly jednotlivě vyvíjet, verzovat a znovu nasadit i v jiných aplikacích (např. jiná zdravotnická vertikála, jiný obor s rezervačním systémem), aniž by se ZubApp musel forkovat jako celek.
**Status:** Návrh k interní diskusi — validovat před tím, než se do modularizace investuje reálná implementační kapacita.
**Jméno sdíleného projektu: CoreKit.** Vrstva 1 (generic core) se vyvíjí a verzuje jako samostatný projekt/monorepo s tímto jménem — záměrně nezávislé na názvu firmy i na kterémkoli oboru (zubaři, veterináři, střelnice...), protože právě to je modul, který má obsluhovat všechny obory najednou. Balíčky uvnitř se pojmenovávají `@corekit/identity`, `@corekit/notifications`, `@corekit/booking`, `@corekit/messaging`, `@corekit/reviews`. Toto jméno se od teď používá napříč vývojem i dokumentací konzistentně.

---

## 1. Princip

Původní architektura dělí systém na moduly podle *funkce* (booking, notifikace, platby...). Pro znovupoužitelnost je potřeba přidat druhý rozměr: podle *míry doménové specifičnosti*. Ne každý modul je stejně "přenositelný" — autentizace je univerzální, adaptér na VZP je specifický pro české zdravotnictví, číselník zubařských výkonů je specifický jen pro ZubApp.

Navrhuji rozdělit moduly do tří vrstev:

1. **Generic core** — bez doménové znalosti, použitelné v jakékoli rezervační/klientské aplikaci (veterinář, fyzioterapie, kadeřnictví...)
2. **Healthcare platform vrstva** — specifické pro české zdravotnictví, ale ne pro zubařství konkrétně — znovupoužitelné v jiné budoucí zdravotnické aplikaci
3. **ZubApp-specifická konfigurace** — zůstává natvrdo v ZubApp, vystavené jako konfigurace/plugin nad vrstvami 1 a 2

Cílem je, aby se vrstvy 1 a 2 daly zabalit jako samostatné, verzované balíčky/služby, které si "půjčí" i budoucí druhá aplikace — bez copy-paste kódu.

---

## 2. Klasifikace stávajících modulů

### Vrstva 1 — Generic core (balíček **CoreKit**)

#### `@corekit/identity` — Identity & account/subject model

- **Účel:** správa účtů a klíčový vztah účet↔subjekt péče (jeden účet, 0..N navázaných subjektů — dítě u zubaře, zvíře u veterináře).
- **Zodpovědnost:** registrace a přihlášení (e-mail/telefon/OAuth), role a oprávnění (majitel účtu vs. personál poskytovatele, s rozlišením práv v rámci personálu), CRUD subjektů péče a jejich navázání na účet, přístupová pravidla — kdo smí vidět/upravovat čí subjekt.
- **Vlastněná data:** účty, subjekty péče (jen obecná pole — jméno, datum narození/vznik vztahu; oborová rozšíření žijí mimo tento modul), role, oprávnění.
- **Rozhraní navenek:** dotazy typu "kdo jsem", "jaké subjekty spravuji"; eventy `AccountCreated`, `SubjectLinked`, `SubjectUnlinked`.
- **Závislosti:** žádné — je to nejnižší vrstva, na které stojí všechno ostatní.
- **Proč reusable:** neobsahuje žádnou oborovou logiku; je to přesně to místo, kde by se dřív skrytě propsal předpoklad "účet = subjekt", kdyby se nestavělo takhle obecně.

#### `@corekit/notifications` — Notification engine

- **Účel:** doručování šablonovaných zpráv přes SMS, e-mail a push.
- **Zodpovědnost:** šablonovací systém s korektní diakritikou, volba SMS kódování (GSM-7 vs. UCS-2 kvůli ceně za zprávu), plánování odeslání (např. připomínka 24–48 h před termínem), sledování stavu doručení. Zahrnuje i uživatelské nastavení kanálů (zapnout/vypnout SMS/e-mail/push) a předstihu připomínky — nastavení platí per účet, napříč všemi subjekty.
- **Vlastněná data:** šablony (konfigurovatelné per obor/tenant), log odeslaných zpráv, stav doručení.
- **Rozhraní navenek:** `send(subjectId, templateKey, params)`; eventy `NotificationSent`, `NotificationFailed`.
- **Závislosti:** `@corekit/identity` (kontakt a subjekt), volitelně `@corekit/booking` (naplánování připomínky k termínu).
- **Proč reusable:** čistě technický problém doručení zprávy, nulová oborová znalost — dobrý první test, že modularizace skutečně funguje.

#### `@corekit/messaging` — Messaging service

- **Účel:** obousměrná komunikace mezi účtem a poskytovatelem (ordinace).
- **Zodpovědnost:** vlákna konverzací vázaná na konkrétní rezervaci (`booking_id`), ne jen na subjekt — zpráva je vždy v kontextu konkrétního termínu. Doručení v reálném čase, historie zpráv, stav přečteno/nepřečteno.
- **Design rozhodnutí (z wireframů):** zprávy se nezobrazují jako samostatná záložka "Zprávy", ale jako součást objednávkového frame — na detailu rezervace, na obou stranách (pacient/majitel i recepce ordinace). Samostatný souhrnný inbox může existovat navíc, ale zdroj pravdy je vazba na rezervaci.
- **Vlastněná data:** konverzace (klíčovaná přes `booking_id`), jednotlivé zprávy.
- **Rozhraní navenek:** `sendMessage(bookingId, ...)`, `getThread(bookingId)`; event `MessageReceived`.
- **Závislosti:** `@corekit/identity`, **`@corekit/booking`** (konverzace bez existující rezervace nedává smysl).
- **Proč reusable:** vzor "zákazník píše poskytovateli o konkrétní rezervaci" nezávisí na oboru.

#### `@corekit/reviews` — Review/reputation service

- **Účel:** sběr a zobrazení hodnocení po proběhlé návštěvě/službě.
- **Zodpovědnost:** vyžádání hodnocení po dokončené rezervaci, ukládání a agregace skóre, základní moderace.
- **Vlastněná data:** jednotlivá hodnocení, agregované skóre poskytovatele.
- **Rozhraní navenek:** `requestReview`, `submitReview`; event `ReviewSubmitted`.
- **Závislosti:** `@corekit/booking` (potřebuje vědět, že se návštěva uskutečnila).
- **Proč reusable:** plně obecné, funguje na jakoukoli službu vázanou na termín.

#### `@corekit/booking` — Booking engine (jádro)

- **Účel:** správa dostupnosti a rezervací — pro obě strany, pacienta/majitele i ordinaci.
- **Zodpovědnost:** definice slotů, prevence dvojitého zápisu, vytvoření/zrušení/přesun rezervace, vazba na `subject_id` (ne na účet přímo). Zahrnuje i **recepční/praxi-facing pohled** — agendu ordinace (kdo přijde kdy, za koho), ne jen pacientský booking flow. Objednávání má probíhat vždy skrz appku, včetně napojení na recepční část ordinace, ne mimo ni.
- **Vlastněná data:** kalendáře poskytovatelů/zdrojů, sloty, rezervace.
- **Rozhraní navenek:** `getAvailability`, `createBooking`, `cancelBooking`; eventy `AppointmentBooked`, `AppointmentCancelled`.
- **Závislosti:** `@corekit/identity` (subjekt), volitelně `@corekit/notifications`. `@corekit/messaging` na něm naopak závisí (viz výše) — zprávy jsou vázané na konkrétní rezervaci.
- **Proč reusable:** kalendářová logika je oborově nezávislá, pokud oborová pravidla (např. délka slotu podle typu výkonu) zůstanou konfigurací, ne kódem zapečeným v modulu. Recepční pohled je stejný vzor bez ohledu na obor — jen se liší, kdo (dentální hygienistka vs. veterinární technik) a nad čím (pacient vs. zvíře) agendu vidí.

### Vrstva 2 — Healthcare platform (specifické pro ČR zdravotnictví, ne pro CoreKit ani pro zubařství)

#### Payer integration framework

- **Účel:** obecný plug-in mechanismus pro napojení na plátce (pojišťovny) — podání nároku, ověření krytí.
- **Zodpovědnost:** definice rozhraní `PayerAdapter`, asynchronní orchestrace zpracování přes frontu (aby výpadek pojišťovny nikdy neblokoval booking), routing požadavku na konkrétní adaptér podle plátce.
- **Vlastněná data:** nároky/claims a jejich stav, historie komunikace s plátcem.
- **Rozhraní navenek:** `submitClaim`, `checkCoverage`; eventy `ClaimSubmitted`, `ClaimSettled`.
- **Závislosti:** `@corekit/booking` (návaznost na návštěvu); konkrétní adaptéry (VZP a spol.) NEJSOU součástí CoreKitu — jsou oborové (Vrstva 3).
- **Proč jen částečně reusable:** rámec ano, obsah ne — u veterinární verze by šlo o jiný mechanismus proplacení (zpětné majiteli, ne přímé zúčtování), viz `modularization-veterinary-check.md`.

#### Audit/compliance modul

- **Účel:** logování přístupu k citlivým (zdravotním) datům.
- **Zodpovědnost:** append-only log každého přístupu/změny u dat spadajících pod zvláštní kategorii GDPR čl. 9, podpora pro práva subjektu údajů (export, výmaz).
- **Vlastněná data:** auditní log.
- **Rozhraní navenek:** middleware/hook volaný ostatními moduly při přístupu k citlivým datům.
- **Závislosti:** žádné přímé, ale zapojuje se prakticky do všech modulů sahajících na zdravotní data.
- **Proč jen částečně reusable:** technicky ano všude, ale právní odůvodnění (čl. 9) platí jen pro lidské pacienty — u zvířat jde jen o dobrou praxi, ne o zákonný požadavek.

### Vrstva 3 — ZubApp-specifická konfigurace (zůstává lokálně)

- **PMS integration adaptéry** — konkrétní konektory na zubařské PMS produkty (rámec adaptéru je z vrstvy 2/1, konkrétní implementace je zubařská)
- **Číselník zubařských výkonů, role hygienistka/zubař**, texty a UI specifické pro zubní péči

---

## 3. Pravidla hranic mezi moduly

Aby šly moduly skutečně znovu použít, ne jen teoreticky oddělit:

- **API-first, žádné sdílené tabulky.** Každý modul vlastní svá data. Jiný modul k nim nesmí přistupovat přímo přes DB — jen přes REST/gRPC nebo event.
- **Event schema registry.** Klíčové doménové události (`AppointmentBooked`, `PatientRegistered`, `ClaimSubmitted`...) mají verzovaný schema kontrakt. Toto je hlavní "reuse povrch" — jiná aplikace se může napojit na stejné eventy, aniž by sdílela kód.
- **Sémantické verzování** pro každý modul/balíček zvlášť, protože různé aplikace budou časem záviset na různých verzích.
- **Multi-tenancy od začátku** u vrstev 1 a 2 — každý záznam nese `tenant_id`, konfigurace šablon a feature flagy jsou per-tenant. Díky tomu může stejná běžící instance služby obsluhovat ZubApp i budoucí druhou aplikaci bez forku nebo redeploye.

---

## 4. Strategie repozitáře a balíčkování

**Doporučení: CoreKit jako monorepo teď, extrakce až při druhém reálném konzumentovi.**

- **CoreKit** = monorepo obsahující vrstvu 1 (a časem vrstvu 2) jako samostatné, verzované balíčky (`@corekit/identity`, `@corekit/notifications`, `@corekit/booking`, `@corekit/messaging`, `@corekit/reviews`). Doporučený nástroj: Turborepo (přirozeně spolupracuje s Vercelem).
- Aplikace pro jednotlivé obory (zubní ordinace, veterinární klinika...) importují balíčky z CoreKitu jako závislost, ne kopií kódu.
- Vynutit hranice nástrojem (např. dependency-cruiser / ESLint boundaries) — modul smí importovat jen veřejné API jiného modulu, nikdy jeho vnitřní soubory.
- Teprve když vznikne aplikace, která některý balíček CoreKitu skutečně potřebuje upravit nekompatibilním způsobem, řešit to přes verzování (major verze balíčku), ne rozdělením do samostatných repozitářů. Předčasné rozdělení CoreKitu do mikroservisních repozitářů bez reálné potřeby jen zpomalí dodání bez benefitu (viz rizika níže).

---

## 5. Dopad na fázování z původního dokumentu

Do rollout plánu (`czech-dental-platform-architecture.md`, sekce 6) doporučuji vložit před fázi 2 explicitní krok:

**Fáze 1.5 — zpevnění hranic modulů:** než se postaví první konkrétní pojišťovnový adaptér (VZP), navrhnout plugin rozhraní (`PayerAdapter` interface) čistě od začátku — protože právě tento modul je nejsilnější kandidát na budoucí znovupoužití v jiné zdravotnické aplikaci. Retrofitovat čisté rozhraní později, až budou existovat 2–3 adaptéry napsané "narychlo", je podstatně dražší.

---

## 6. Rizika a kompromisy

- **Over-engineering risk:** Navrhovat pro znovupoužitelnost dřív, než existuje druhý reálný konzument, může zpomalit první dodávku bez záruky, že se investice vrátí. Doporučený postup je "navrženo pro extrakci, ale zatím needitrahováno" — čisté rozhraní a oddělená data ano, samostatné repo/CI/verzovací overhead ne.
- **Vrstva 2 je specifičtější, než vypadá.** Adaptéry na pojišťovny sice nejsou zubařské, ale jsou zdravotnické — nepůjdou znovu použít mimo zdravotnictví (např. v rezervačním systému pro kadeřnictví). Než se investuje do jejich zobecnění, stojí za to ověřit, jestli je druhá zdravotnická aplikace skutečně v plánu, nebo jestli je to jen teoretická možnost.
- **Multi-tenancy zvyšuje komplexitu od první řádky kódu** (tenant_id všude, per-tenant config) — je to správná investice, pokud je druhá aplikace pravděpodobná v horizontu cca 12–18 měsíců; pokud ne, může jít o zbytečné triení navíc.
