# Startovací brief: @corekit/reviews

Vlož tento dokument do nové konverzace — obsahuje vše potřebné k pokračování na tomto modulu bez nutnosti znovu vysvětlovat kontext.

## Kontext projektu

ZubApp je Doctolib-styl rezervační/komunikační platforma, budovaná paralelně pro dva obory (zubní hygienistky a veterináře). Sdílené, oborově nezávislé moduly žijí v monorepu **CoreKit** (`@corekit/*`), který obě oborové aplikace importují jako závislost. Stack: Vercel (hosting) + Supabase (Postgres, Auth), EU region (Frankfurt). Detaily: `czech-dental-platform-architecture-modularization.md` a `kickoff-checklist.md` v projektové složce ZubApp.

## Tento modul: `@corekit/reviews`

**Účel:** sběr a zobrazení hodnocení po proběhlé návštěvě/službě.

**Zodpovědnost:**
- Vyžádání hodnocení po dokončené rezervaci
- Ukládání a agregace skóre
- Základní moderace

**Vlastněná data:** jednotlivá hodnocení, agregované skóre poskytovatele.

**Rozhraní navenek:** `requestReview`, `submitReview`; event `ReviewSubmitted`.

**Závislosti:** `@corekit/booking` (potřebuje vědět, že se návštěva uskutečnila — reaguje na `AppointmentCompleted`/obdobný event).

**Proč reusable:** plně obecné, funguje na jakoukoli službu vázanou na termín — nejméně rizikový modul z celého CoreKitu.

## Wireframy (hotové)

- "Žádost o hodnocení" — prompt po návštěvě (jméno subjektu, poskytovatel, datum)
- "Formulář hodnocení" — hvězdičkové skóre + textový komentář

## Stav a další kroky

- YouTrack milestone: `CoreKit-5` (sdílený s `@corekit/messaging`, https://utisol1.youtrack.cloud/issue/CoreKit-5), dílčí úkol `CoreKit-21`
- Další krok: datový model hodnocení, pravidla moderace, agregační logika skóre poskytovatele
- Nízká priorita v rollout plánu — až po funkčním bookingu v obou aplikacích
