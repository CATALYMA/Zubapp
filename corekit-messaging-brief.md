# Startovací brief: @corekit/messaging

Vlož tento dokument do nové konverzace — obsahuje vše potřebné k pokračování na tomto modulu bez nutnosti znovu vysvětlovat kontext.

## Kontext projektu

ZubApp je Doctolib-styl rezervační/komunikační platforma, budovaná paralelně pro dva obory (zubní hygienistky a veterináře). Sdílené, oborově nezávislé moduly žijí v monorepu **CoreKit** (`@corekit/*`), který obě oborové aplikace importují jako závislost. Stack: Vercel (hosting) + Supabase (Postgres, Auth), EU region (Frankfurt). Detaily: `czech-dental-platform-architecture-modularization.md` a `kickoff-checklist.md` v projektové složce ZubApp.

## Tento modul: `@corekit/messaging`

**Účel:** obousměrná komunikace mezi účtem a poskytovatelem (ordinace).

**Zodpovědnost:**
- Vlákna konverzací vázaná na konkrétní **rezervaci** (`booking_id`), ne jen na subjekt — zpráva je vždy v kontextu konkrétního termínu
- Doručení v reálném čase, historie zpráv, stav přečteno/nepřečteno

**Design rozhodnutí (z wireframů, důležité — nezjednodušovat zpět):** zprávy se nezobrazují jako samostatná záložka "Zprávy", ale jako součást objednávkového frame — na detailu rezervace, na obou stranách (pacient/majitel i recepce ordinace). Samostatný souhrnný inbox může existovat navíc, ale zdroj pravdy je vazba na rezervaci.

**Vlastněná data:** konverzace (klíčované přes `booking_id`), jednotlivé zprávy.

**Rozhraní navenek:** `sendMessage(bookingId, ...)`, `getThread(bookingId)`; event `MessageReceived`.

**Závislosti:** `@corekit/identity`, **`@corekit/booking`** (konverzace bez existující rezervace nedává smysl — tato závislost byla přidaná dodatečně po revizi wireframů).

## Wireframy (hotové)

- Detail rezervace (pacient/majitel) se zabudovanou konverzací a polem pro odpověď
- Detail rezervace na recepci se stejným vzorem zabudované konverzace

## Stav a další kroky

- YouTrack milestone: `CoreKit-5` (sdílený s `@corekit/reviews`, https://utisol1.youtrack.cloud/issue/CoreKit-5), dílčí úkol `CoreKit-20`
- Další krok: datový model konverzací vázaný na `booking_id`, zvážit Supabase Realtime pro doručení v reálném čase
- Vzor "zákazník píše poskytovateli o konkrétní rezervaci" je oborově nezávislý — funguje stejně pro zubaře i veterináře
