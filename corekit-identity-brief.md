# Startovací brief: @corekit/identity

Vlož tento dokument do nové konverzace — obsahuje vše potřebné k pokračování na tomto modulu bez nutnosti znovu vysvětlovat kontext.

## Kontext projektu

ZubApp je Doctolib-styl rezervační/komunikační platforma, budovaná paralelně pro dva obory (zubní hygienistky a veterináře). Sdílené, oborově nezávislé moduly žijí v monorepu **CoreKit** (`@corekit/*`), který obě oborové aplikace importují jako závislost. Stack: Vercel (hosting) + Supabase (Postgres, Auth), EU region (Frankfurt). Detaily: `czech-dental-platform-architecture-modularization.md` a `kickoff-checklist.md` v projektové složce ZubApp.

## Tento modul: `@corekit/identity`

**Účel:** správa účtů a klíčový vztah účet↔subjekt péče — jeden účet, 0 až N navázaných subjektů (dítě u zubaře, zvíře u veterináře).

**Zodpovědnost:**
- Registrace a přihlášení (e-mail/telefon/OAuth) přes Supabase Auth
- Role a oprávnění — majitel účtu vs. personál poskytovatele, s rozlišením práv v rámci personálu
- CRUD subjektů péče a jejich navázání na účet
- Přístupová pravidla (RLS) — kdo smí vidět/upravovat čí subjekt

**Vlastněná data:** účty, subjekty péče (jen obecná pole — jméno, datum narození/vznik vztahu; oborová rozšíření žijí mimo tento modul), role, oprávnění.

**Rozhraní navenek:** dotazy "kdo jsem", "jaké subjekty spravuji"; eventy `AccountCreated`, `SubjectLinked`, `SubjectUnlinked`.

**Závislosti:** žádné — nejnižší vrstva, na které stojí booking, notifications, messaging i reviews.

## Klíčové architektonické rozhodnutí (nezjednodušovat)

Vztah účet↔subjekt má stejný **tvar** napříč obory (1 účet → 0..N subjektů), ale ne stejná **pravidla**:

- U zubaře je výchozí případ účet = subjekt (rezervujete sami sebe), dítě je doplněk. U veterináře účet **nikdy** není subjekt — majitel se sám k veterináři neobjednává.
- Dítě jednou dosáhne způsobilosti a "vyroste" do vlastního účtu (migrace historie, předání přístupových práv). Zvíře vlastní účet nikdy nezíská.
- Právní způsobilost dítěte se v čase mění; u zvířete rozhoduje vždy majitel.
- Dítě má vlastní číslo pojištěnce nezávislé na rodiči; zvíře ne.

**Do tohoto modulu patří jen struktura vztahu**, ne pravidla kolem ní (ta jsou oborová, Vrstva 3). Detaily: `modularization-veterinary-check.md`.

## Wireframy (hotové)

- Přihlášení / registrace
- "Moje účty péče" — dvě oborové varianty ukázané vedle sebe: dental (Vy + Anna, dcera 8 let) a vet (jen Rex a Micka, bez majitele jako subjektu)
- "Přidat subjekt" — formulář (jméno, datum narození/stáří, vztah k vám)
- "Detail subjektu" — profil + nadcházející rezervace (placeholder) + kdo má přístup

## Stav a další kroky

- YouTrack milestone: `CoreKit-2` (https://utisol1.youtrack.cloud/issue/CoreKit-2), dílčí úkoly `CoreKit-11` až `CoreKit-14`
- Technický spec (datový model, RLS politiky, API kontrakt) zatím **není napsaný** — logický další krok
- Doporučeno zapnout Supabase "High Compliance" (PITR, SSL enforcement, network restrictions) i bez potřeby HIPAA, kvůli GDPR čl. 32
