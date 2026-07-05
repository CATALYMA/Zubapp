# PMS pro zubní hygienisty — rozsah a architektura

**Návazný dokument k:** `czech-dental-platform-architecture.md`, `czech-dental-platform-architecture-modularization.md`, `kickoff-checklist.md`, `corekit-identity-brief.md`, `corekit-booking-brief.md`
**Podkladové zdroje:** `zdroje-dentalni-hygienistka/00_INDEX.md` — stažené oficiální/profesní materiály (kvalifikační standard MZ ČR, etický kodex ADH ČR, mezinárodní srovnání kompetencí), viz sekce 5, bod 1
**Zdroj zadání:** YouTrack projekt ZUB — epiky E8 (Onboarding tenantů a předplatné), E9 (Souhlasy a přístupové granty), E10 (Multi-tenant základ), stories ZUB-43, 44, 47, 49, 53, 54
**Status:** Návrh k interní diskusi — sepsáno na základě existujícího backlogu a architektonických dokumentů, ne na základě nového živého ověření trhu

---

## 1. Klíčové zjištění, které mění rozsah

Původní architektura (`czech-dental-platform-architecture.md`) a rozjezdový plán (`kickoff-checklist.md`) stavěly na modelu **„vrstva nad existujícím PMS ordinace, ne jeho náhrada"** — DentApp cílí na zubní ordinace, kde hygienistka je personál v rámci tenantu zubaře, a klinická práce probíhá mimo hranice appky, ve vlastním PMS ordinace.

Backlog v epikách E8–E10 ale popisuje jinou situaci: **hygienista jako samostatný tenant** — sám si zakládá entitu (ZUB-43), prochází ověřením odbornosti (ZUB-44), a pacienti mezi jednotlivými nezávislými hygienisty přechází s kontinuitou dokumentace (ZUB-47, 49, 53, 54). To je jiný zákaznický segment, pro který **model „vrstva nad PMS" nefunguje** — nezávislý hygienista bez vlastní ordinace typicky žádný PMS nemá, na co by appka mohla být vrstvou. Aby mohl v appce reálně vést pacienty, appka musí PMS **být**, ne ho jen doplňovat.

Toto přesně odpovídá scénáři, který původní dokument označil jako **Fázi 5 — podmíněný „light" klinický modul** (sekce 6), ale s jiným spouštěčem, než se čekalo: nejde o reakci na konkurenci (Doctolib all-in-one), ale o vlastní zákaznický segment (nezávislý hygienista), který si o klinickou vrstvu řekl od začátku.

**Důsledek:** toto je samostatná práce vedle DentApp (vrstva pro ordinace), ne jeho rozšíření. Sdílí s DentApp stejné CoreKit moduly (identity, booking, notifications, messaging, reviews), ale přidává vlastní doménovou vrstvu, kterou DentApp nepotřebuje.

---

## 2. Rozsah produktu

**Je uvnitř:**
- Samoobslužné založení tenantu nezávislým hygienistou + ověření odbornosti
- Rezervační kalendář a recepční pohled (reuse `@corekit/booking`)
- Vedení pacienta — lehký klinický záznam (PHR): anamnéza, provedené výkony, recall plán — **ne** plný odontogram nebo zobrazovací diagnostika
- Globální identita pacienta napříč tenanty (pacient přechází mezi hygienisty se souvislou historií)
- Přístupové granty pacienta (kdo z hygienistů smí vidět jakou část dokumentace, na jak dlouho)
- Notifikace, messaging, hodnocení (reuse CoreKit beze změny)

**Je mimo (zatím):**
- Napojení na PMS zubní ordinace (to řeší DentApp, ne toto)
- Plátcovská integrace (VZP a spol.) — hygienistka bez smluvního vztahu s pojišťovnou pravděpodobně fakturuje výkony jinak než ordinace; vyžaduje samostatné ověření, ne převzetí adaptéru z DentApp beze změny
- Zobrazovací diagnostika, plný odontogram, AI kódování — mimo definici „light clinical"

---

## 3. Aktérský a datový model (návrh)

| Entita | Popis | Poznámka |
|---|---|---|
| **Tenant** | Samostatná entita hygienisty/hygienistky | Vzniká samoobslužnou registrací (ZUB-43); má stav ověření (ZUB-44) |
| **Membership** | Vazba osoba↔tenant + role | Zakladatel = správce, může zvát další personál |
| **Pacient (globální identita)** | Jedna identita napříč všemi tenanty | ZUB-53 — nezávislá na tenantu, musí mít spolehlivé ověření totožnosti proti sloučení dvou lidí |
| **PHR záznam** | Klinický záznam navázaný na globálního pacienta, atribuovaný tenantu, kde vznikl | ZUB-54 — šifrováno klíčem pacienta (podpora výmazu/crypto-shredding) |
| **Consent Grant** | Souhlas pacienta s přístupem konkrétního tenantu k (části) dokumentace | ZUB-47 — rozsah (celá/vybrané typy) + platnost (dočasná/do odvolání), časově zaznamenaný |
| **Access check** | Vyhodnocení při každém přístupu: členství v tenantu (provozní data) **+** platný grant (dokumentace) | ZUB-49 — kombinace obou, ne jen jednoho |

**Architektonický dopad na CoreKit:** `@corekit/identity`, jak je popsaný v `corekit-identity-brief.md`, řeší vztah **účet↔subjekt v rámci jednoho tenantu**. Globální identita pacienta napříč nezávislými tenanty (ZUB-53) je nad rámec původního návrhu modulu — buď je potřeba `@corekit/identity` rozšířit o cross-tenant identitu, nebo tuto vrstvu postavit jako nadstavbu nad ním, ne uvnitř. Doporučuji toto rozhodnout explicitně před psaním datového modelu, protože retrofit později je nákladný (stejný typ rizika, jaký `modularization-veterinary-check.md` popsal u vztahu účet↔subjekt).

---

## 4. Mapování na existující YouTrack backlog

| Story | Pokrývá | Chybí/otevřeno |
|---|---|---|
| ZUB-43 | Self-service registrace tenantu | — |
| ZUB-44 | Ověření hygienisty | Konkrétní zdroj ověření (registr Komory? [OVĚŘIT]) není specifikován |
| ZUB-47 | Pacient uděluje grant | UI/flow pro odvolání grantu není popsán |
| ZUB-49 | Vyhodnocení grantu při přístupu | Technický mechanismus (middleware/RLS politika) není specifikován |
| ZUB-53 | Globální identita pacienta | Mechanismus ověření totožnosti proti duplicitě/sloučení není popsán |
| ZUB-54 | PHR jako patient-centric záznam | Konkrétní obsah „light clinical" záznamu (co se smí/musí zaznamenat) není definován |

Epiky E8/E9/E10 tedy scope pokrývají, ale jde o roztříštěné kusy bez zastřešujícího produktového rámce a bez několika klíčových implementačních stories.

---

## 5. Navrhované nové stories

1. **Definice obsahu „light clinical" záznamu** — co přesně PHR obsahuje (anamnéza, provedené výkony, recall interval), co záměrně ne (odontogram, zobrazování) — potřeba pro ZUB-54, aby nedošlo k postupnému rozšiřování rozsahu bez rozhodnutí. **Podklad již k dispozici:** `zdroje-dentalni-hygienistka/02_ADHCR_Profesni_a_eticky_kodex.md` kap. 4 definuje 4 standardní typy návštěvy (vstupní vyšetření, základní čištění, rozšířené/hloubkové čištění, recall) s kroky a časovou dotací; `01_MZCR_Kvalifikacni_standard_Dentalni_hygienistka.md` k tomu dává čtyřúrovňový model kompetencí (co smí DH sama vs. jen na indikaci/pod dohledem ZL) — použitelné jako výchozí bod pro schéma PHR a pro rozlišení výkonů podle nutné indikace.
2. **Mechanismus ověření totožnosti pacienta** (proti duplicitní/sloučené identitě) — podkladová práce pro ZUB-53.
3. **Technický návrh vyhodnocení přístupu (membership + grant)** — RLS politika nebo middleware — podkladová práce pro ZUB-49.
4. **Zdroj ověření odbornosti hygienisty** — rozhodnout, zda jde o manuální doložení dokladu, nebo napojení na registr (Česká stomatologická komora / obdobný rejstřík) — podkladová práce pro ZUB-44, sdílí otevřený bod se sekcí 2.1 `czech-dental-platform-architecture.md`.
5. **Fakturace/úhrada výkonů hygienisty bez plátcovské integrace v1** — rozhodnout výchozí model (přímá platba pacientem) pro fázi 1, než se řeší pojišťovny.
6. **Recepční/praxe-facing pohled pro samostatného hygienistu** — reuse vzoru z `@corekit/booking`, ale ověřit, že jednočlenný tenant (hygienistka bez recepce) pohled skutečně potřebuje ve stejné podobě jako ordinace.

---

## 6. Fázování

1. **Fáze A** — tenant onboarding + ověření (ZUB-43, 44) + booking (reuse CoreKit) bez PHR — hygienista může fungovat, ale bez cross-tenant kontinuity dokumentace.
2. **Fáze B** — globální identita pacienta + PHR (ZUB-53, 54) — jádro diferenciace oproti „jen booking".
3. **Fáze C** — consent grant + access enforcement (ZUB-47, 49) — nutné před tím, než PHR reálně obsahuje citlivá data sdílená mezi tenanty; z hlediska GDPR čl. 9 by fáze B a C měly jít prakticky současně, ne s odstupem (PHR bez fungujícího grant mechanismu = neoprávněný přístup k zdravotním datům jiného tenantu).
4. **Fáze D** — fakturace/úhrada výkonů (mimo scope tohoto dokumentu, samostatné rozhodnutí).

---

## 7. Otevřené body k ověření

- Zdroj ověření odbornosti hygienisty (registr Komory? [OVĚŘIT], jak zmíněno v `czech-dental-platform-architecture.md` sekce 2.1)
- Zda nezávislá hygienistka může legálně fakturovat výkony bez vazby na ordinaci/zubaře — právní otázka mimo technický rozsah tohoto dokumentu
- Vztah k rozhodnutí v `kickoff-checklist.md` („staví se jen DentApp teď") — tento dokument popisuje druhý paralelní produkt/segment; je potřeba explicitní rozhodnutí, zda a kdy se do něj investuje vedle DentApp, ne jen tichý předpoklad
- Zda `@corekit/identity` rozšířit o cross-tenant identitu, nebo stavět nadstavbu mimo modul (viz sekce 3)
