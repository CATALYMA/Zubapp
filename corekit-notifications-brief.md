# Startovací brief: @corekit/notifications

Vlož tento dokument do nové konverzace — obsahuje vše potřebné k pokračování na tomto modulu bez nutnosti znovu vysvětlovat kontext.

## Kontext projektu

ZubApp je Doctolib-styl rezervační/komunikační platforma, budovaná paralelně pro dva obory (zubní hygienistky a veterináře). Sdílené, oborově nezávislé moduly žijí v monorepu **CoreKit** (`@corekit/*`), který obě oborové aplikace importují jako závislost. Stack: Vercel (hosting) + Supabase (Postgres, Auth), EU region (Frankfurt). Detaily: `czech-dental-platform-architecture-modularization.md` a `kickoff-checklist.md` v projektové složce ZubApp.

## Tento modul: `@corekit/notifications`

**Účel:** doručování šablonovaných zpráv přes SMS, e-mail a push.

**Zodpovědnost:**
- Šablonovací systém s korektní diakritikou
- Volba SMS kódování (GSM-7 vs. UCS-2) kvůli ceně za zprávu
- Plánování odeslání (např. připomínka 24–48 h před termínem)
- Sledování stavu doručení
- Uživatelské nastavení kanálů (zapnout/vypnout SMS/e-mail/push) a předstihu připomínky — nastavení platí per účet, napříč všemi subjekty

**Vlastněná data:** šablony (konfigurovatelné per obor/tenant), log odeslaných zpráv, stav doručení, uživatelské preference kanálů.

**Rozhraní navenek:** `send(subjectId, templateKey, params)`; eventy `NotificationSent`, `NotificationFailed`.

**Závislosti:** `@corekit/identity` (kontakt a subjekt), volitelně `@corekit/booking` (naplánování připomínky k termínu).

## Technická poznámka ke stacku

Vercel Functions mají limit běhu 300 s (rozšiřitelný na 800 s). Pro plánování připomínek s libovolným zpožděním (24–48 h dopředu) použít **Vercel Workflows** (`sleep()` na libovolně dlouhou dobu) nebo **Vercel Queues** (delayed delivery až 7 dní), ne přímo funkci s dlouhým timeoutem.

## Wireframy (hotové)

- "Nastavení notifikací" — toggly SMS / e-mail / push, volba předstihu připomínky (24 h / 48 h)

## Stav a další kroky

- YouTrack milestone: `CoreKit-3` (https://utisol1.youtrack.cloud/issue/CoreKit-3), dílčí úkoly `CoreKit-15` až `CoreKit-17`
- Další krok: šablonovací engine, výběr SMS providera pro český trh, implementace plánování přes Queues/Workflows
- Dobrý první test modularizace — nulová oborová znalost, mělo by fungovat identicky pro oba obory bez úprav
