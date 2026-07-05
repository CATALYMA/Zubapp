# Startovací brief: koordinace mezi CoreKit moduly

Na rozdíl od pěti modulových briefů (`corekit-identity-brief.md`, `corekit-notifications-brief.md`, `corekit-booking-brief.md`, `corekit-messaging-brief.md`, `corekit-reviews-brief.md`) tenhle dokument **nepatří jednomu modulu** — je pro chat, kde se řeší, jak moduly spolupracují, a kde se kontroluje, že se pět modulových chatů v čase nerozešlo v protichůdných rozhodnutích.

## Kontext projektu

ZubApp je Doctolib-styl rezervační/komunikační platforma, budovaná paralelně pro dva obory (zubní hygienistky a veterináře). Sdílené moduly žijí v monorepu **CoreKit** (`@corekit/*`). Detaily: `czech-dental-platform-architecture-modularization.md` a `kickoff-checklist.md` v projektové složce ZubApp.

## Graf závislostí mezi moduly

```
identity (bez závislostí — základ)
   ↑                    ↑
booking - - -> PMS integrace (oborové, mimo CoreKit)
   ↑        ↑
messaging  reviews

notifications: závisí na identity, volitelně na booking
```

- **identity** — základ, na kterém stojí všechno ostatní. Nezávisí na ničem.
- **booking** — závisí na identity (potřebuje vědět, pro jaký subjekt rezervuje). Navíc se **pluginově integruje s PMS vrstvou** (Vrstva 3, mimo CoreKit) — booking čte dostupnost a zapisuje rezervace přes PMS adaptér, pokud ho daná ordinace má napojený, jinak používá vlastní kalendář. Tato vazba je **přerušovaná/pluginová, ne pevná CoreKit závislost** — booking core nesmí být napsaný proti konkrétnímu PMS produktu, jen proti obecnému rozhraní adaptéru, které si každý obor implementuje samostatně (zubařský PMS vs. veterinární informační systém).
- **notifications** — závisí na identity (kontakt), volitelně na booking (naplánování připomínky k termínu).
- **messaging** — závisí na identity **a na booking** (konverzace je vázaná na `booking_id`, ne jen na subjekt — rozhodnutí z revize wireframů).
- **reviews** — závisí na booking (potřebuje vědět, že se návštěva uskutečnila).

**Vizuální konvence v diagramu:** plná čára/box = CoreKit balíček s pevnou závislostí; přerušovaná čára/box = oborová PMS integrace mimo CoreKit, napojená přes plugin rozhraní, ne přes hard-coded závislost.

## Jak moduly spolupracují — event tabulka

Toto je hlavní "reuse povrch" mezi moduly (viz `czech-dental-platform-architecture-modularization.md`, sekce 3 — event schema registry). Kdykoliv se mění tvar události, promítá se to do všech konzumentů níže:

| Událost | Producent | Konzument(i) |
|---|---|---|
| `AccountCreated` | identity | — |
| `SubjectLinked` / `SubjectUnlinked` | identity | booking (dostupné subjekty pro rezervaci), notifications (kontaktní údaje) |
| `AppointmentBooked` | booking | notifications (potvrzení + naplánování připomínky), messaging (založení vlákna vázaného na rezervaci) |
| `AppointmentCancelled` | booking | notifications (zrušení naplánované připomínky) |
| `AppointmentCompleted` | booking | reviews (vyžádání hodnocení), notifications (follow-up zpráva) |
| `MessageReceived` | messaging | notifications (upozornění na novou zprávu) |
| `ReviewSubmitted` | reviews | — (interní agregace skóre poskytovatele) |
| (synchronní dotaz, ne event) `checkAvailability` / `syncBooking` | booking | PMS integrace (oborový adaptér, Vrstva 3) — jen pokud ordinace/klinika PMS napojený má |

## Na co dávat pozor při koordinaci modulových chatů

- **Změna v jednom modulu, která mění event kontrakt nebo závislost, se musí promítnout do briefu závislého modulu.** Příklad, který se už stal: revize wireframů přidala závislost `messaging → booking` (zprávy vázané na `booking_id`) — to je potřeba mít v obou briefech (messaging i booking), ne jen v jednom.
- **Identity je jediný modul bez závislostí** — pokud se v identity chatu objeví rozhodnutí, které mění tvar účet↔subjekt vztahu, dotýká se to potenciálně všech čtyř zbylých modulů. Takové rozhodnutí patří probrat tady, ne jen v identity chatu.
- **Sémantické verzování** — pokud modul vydá breaking change (major verze balíčku), ostatní moduly/aplikace na něj nemusí hned přejít. Sledovat to tady, ne v jednotlivých modulových chatech.

## Stav

- Aktuální diagram závislostí byl vygenerovaný v konverzaci, kde vznikl tento brief — při zásadní změně architektury ho požádejte Claude přegenerovat.
- YouTrack: všechny moduly jsou milestones v projektu CoreKit (https://utisol1.youtrack.cloud/projects/CoreKit) — `CoreKit-2` (identity), `CoreKit-3` (notifications), `CoreKit-4` (booking), `CoreKit-5` (messaging + reviews).
