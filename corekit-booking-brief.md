# Startovací brief: @corekit/booking

Vlož tento dokument do nové konverzace — obsahuje vše potřebné k pokračování na tomto modulu bez nutnosti znovu vysvětlovat kontext.

## Kontext projektu

ZubApp je Doctolib-styl rezervační/komunikační platforma, budovaná paralelně pro dva obory (zubní hygienistky a veterináře). Sdílené, oborově nezávislé moduly žijí v monorepu **CoreKit** (`@corekit/*`), který obě oborové aplikace importují jako závislost. Stack: Vercel (hosting) + Supabase (Postgres, Auth), EU region (Frankfurt). Detaily: `czech-dental-platform-architecture-modularization.md` a `kickoff-checklist.md` v projektové složce ZubApp.

## Tento modul: `@corekit/booking`

**Účel:** správa dostupnosti a rezervací — pro obě strany, pacienta/majitele i ordinaci.

**Zodpovědnost:**
- Definice slotů, prevence dvojitého zápisu
- Vytvoření/zrušení/přesun rezervace, vazba na `subject_id` (ne na účet přímo)
- **Recepční/praxi-facing pohled** — agenda ordinace (kdo přijde kdy, za koho). Objednávání má probíhat vždy skrz appku, včetně napojení na recepční část ordinace, ne mimo ni.

**Vlastněná data:** kalendáře poskytovatelů/zdrojů, sloty, rezervace.

**Rozhraní navenek:** `getAvailability`, `createBooking`, `cancelBooking`; eventy `AppointmentBooked`, `AppointmentCancelled`.

**Závislosti:** `@corekit/identity` (subjekt), volitelně `@corekit/notifications`. `@corekit/messaging` na tomto modulu závisí (zprávy jsou vázané na konkrétní rezervaci) — booking sám na messaging nezávisí.

**Vazba na PMS (Vrstva 3, mimo CoreKit):** booking se pluginově integruje s oborovou PMS vrstvou — čte dostupnost a zapisuje rezervace přes PMS adaptér, pokud ho ordinace/klinika má napojený, jinak používá vlastní kalendář. Tahle vazba musí zůstat **přerušovaná/pluginová** — booking core je psaný proti obecnému rozhraní adaptéru (`checkAvailability`/`syncBooking`), nikdy proti konkrétnímu PMS produktu. Konkrétní adaptéry (zubařský PMS vs. veterinární informační systém) jsou oborové a nejsou součástí CoreKitu.

## Otevřená otázka

Multi-resource booking (např. místnost + lékař zvlášť) zatím není ověřený jako nutnost pro v1 — rozhodnout před návrhem datového modelu kalendáře.

## Wireframy (hotové)

- Výběr termínu (patient-facing, s výběrem "rezervace pro" subjekt)
- Potvrzení rezervace
- Moje rezervace (pacient/majitel)
- Recepce — dnešní agenda (praxi-facing, seznam rezervací s indikátorem zpráv)
- Detail rezervace na recepci (info o subjektu, zabudované zprávy, akce potvrdit/přesunout)

## Stav a další kroky

- YouTrack milestone: `CoreKit-4` (https://utisol1.youtrack.cloud/issue/CoreKit-4), dílčí úkoly `CoreKit-18`, `CoreKit-19`
- Další krok: datový model kalendáře/slotů/rezervací, API kontrakt, rozhodnout multi-resource otázku
- Recepční pohled je stejný vzor bez ohledu na obor — liší se jen kdo (hygienistka vs. vet. technik) a nad čím (pacient vs. zvíře) agendu vidí
