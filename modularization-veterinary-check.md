# Ověření: půjdou moduly ZubApp použít i pro veterinární aplikaci?

**Návazný dokument k:** `czech-dental-platform-architecture-modularization.md`
**Poznámka k metodě:** živé vyhledávání nebylo v tuto chvíli dostupné (prohlížeč nepřipojen, fulltextové vyhledávače nevrátily obsah), takže tržní tvrzení níže jsou označena **[OVĚŘIT]** stejně jako v původních dokumentech — jde o odhad na základě obecné znalosti, ne o ověřený aktuální stav trhu.

---

## Závěr ve zkratce

Vrstva 1 (generic core) se dá použít téměř beze změny, ale objevuje se jeden skutečně nutný zásah do datového modelu. Vrstva 2 (healthcare platform) se přenáší jen jako *vzor/rozhraní*, ne jako obsah — veterinární péče v ČR běží mimo systém veřejného zdravotního pojištění, takže konkrétní pojišťovnové adaptéry a část odůvodnění pro audit modul neplatí stejně. Vrstva 3 se nepřenáší podle očekávání.

---

## Vrstva 1 — Generic core

| Modul | Použitelné pro veterináře? | Poznámka |
|---|---|---|
| Identity & auth | Ano, beze změny | Role-based přístup majitel/personál je stejný vzor |
| Notification engine | Ano, beze změny | Šablony, diakritika, SMS kódování — nezávislé na oboru |
| Messaging service | Ano, beze změny | Obousměrná komunikace majitel–ordinace funguje stejně |
| Review/reputation | Ano, beze změny | Plně obecné |
| **Booking engine** | **Ano, ale s jednou nutnou úpravou** | viz níže |

### Zjištění, které mění i dnešní návrh (ne jen veterinární variantu)

Booking engine i identity model v původním návrhu tiše předpokládají, že osoba, která si rezervuje a komunikuje (účet), je zároveň tou, které se péče týká (pacient). U veterináře to neplatí — jeden majitel může mít víc zvířat a rezervuje/komunikuje za ně. Řešení: oddělit **"account holder"** (kdo se přihlašuje, platí, dostává notifikace) od **"subjekt péče"** (koho se rezervace týká — konkrétní zvíře, nebo u zubaře třeba dítě rezervované rodičem). Vztah 1:N mezi účtem a subjekty.

Tohle není nová komplikace vyvolaná veterinářem — je to mezera v původním generic core, kterou stálo za to najít teď, protože ovlivňuje schéma dat v booking i identity modulu už pro ZubApp samotný (rodič rezervující dítě je stejný případ).

### Dodatečné ověření: je vztah "účet ↔ subjekt" u zubaře a veterináře opravdu stejný?

Jen částečně. **Stejný je tvar vztahu** (1 účet → 0..N subjektů péče) — to je ta obecná struktura, která patří do sdíleného generic core. **Nestejná jsou pravidla kolem něj**, a ta musí zůstat v oborové (Vrstva 3) konfiguraci, ne ve sdíleném modulu:

- **Výchozí případ:** u zubaře účet obvykle = subjekt (rezervuju sám sebe), dítě je doplněk. U veterináře účet **nikdy** není subjekt — majitel se sám k veterináři neobjednává.
- **Životní cyklus:** dítě jednou dosáhne způsobilosti a "vyroste" do vlastního účtu — vyžaduje migraci historie a předání přístupových práv. Zvíře vlastní účet nikdy nezíská; vztah končí úhynem nebo změnou vlastníka (jiný typ přechodu).
- **Právní způsobilost:** u dítěte se v čase mění, kdo smí dávat souhlas k péči. U zvířete rozhoduje vždy a natrvalo majitel.
- **Vazba na pojištění:** dítě má vlastní číslo pojištěnce nezávislé na rodiči. Zvíře žádné analogické vlastní pojištění nemá — je vázané na majitelovu pojistku.

**Závěr:** do generic core (Vrstva 1) patří jen datová struktura vztahu účet↔subjekt a rozhraní, která na ni odkazují (booking, messaging, notifikace referencují `subject_id`, ne `account_id`). Pravidla o dospívání/souhlasu (zubař) a vlastnictví/úmrtí (veterinář) jsou oborová business logika a patří do Vrstvy 3.

---

## Vrstva 2 — Healthcare platform

| Modul | Rámec/vzor přenositelný? | Obsah přenositelný? |
|---|---|---|
| Payer integration framework | Ano — plug-in mechanismus na "libovolného plátce" je obecný | **Ne** — viz níže |
| Audit/compliance (GDPR čl. 9) | Ano — logování samo o sobě neškodí | **Ne jako tvrdý požadavek** — viz níže |

### Payer integration — proč se obsah nepřenáší

Veterinární péče v ČR není hrazena z veřejného zdravotního pojištění (VZP a spol. kryjí jen péči o lidi). Existuje jen soukromé pojištění domácích mazlíčků, a to funguje jinak než VZP model, na který je framework navržený:

- **Jiný mechanismus proplacení:** pojištění mazlíčků v ČR typicky funguje na bázi zpětného proplacení majiteli (majitel zaplatí veterináři, pak žádá pojišťovnu o refundaci), ne přímého zúčtování poskytovatelem jako u VZP. To znamená, že "PayerAdapter" rozhraní navržené pro přímé podání nároku (claim submission) poskytovatelem by se muselo přizpůsobit — spíš na "vygenerování dokladu pro majitele" než na "podání nároku za pacienta".
- **[OVĚŘIT]** Penetrace pojištění mazlíčků v ČR — pokud je nízká, nemusí se payer integrace pro veterinární MVP vůbec vyplatit stavět, na rozdíl od zubařské aplikace, kde je VZP dominantní a téměř nutná.

Závěr: **rozhraní/vzor adaptéru ano, konkrétní adaptéry a možná i priorita celého modulu ne.**

### Audit/compliance modul — proč se odůvodnění nepřenáší stejně

GDPR čl. 9 (zvláštní kategorie osobních údajů, mj. zdravotní údaje) se ze své definice týká výhradně **fyzických osob**. Zdravotní záznam zvířete sám o sobě není osobní údaj — GDPR chrání jen údaje o majiteli (jméno, kontakt, platba), a to v běžném režimu čl. 6, ne v přísnějším režimu čl. 9. Modul auditního logování lze technicky ponechat beze změny (víc logování neuškodí), ale nesmí se prezentovat jako "nutné ze zákona" pro veterinární verzi stejným způsobem jako u zubařské — je to jen dobrá praxe, ne hard requirement.

---

## Vrstva 3 — ZubApp-specifická konfigurace

Podle očekávání se nepřenáší: adaptéry na zubařský PMS software, číselník zubařských výkonů a role hygienistka/zubař jsou nahrazeny vlastní sadou pro veterinární obor — konkrétní veterinární informační systémy používané v ČR **[OVĚŘIT]**, číselník veterinárních výkonů/druhů zvířat, role veterinář/vet. technik.

---

## Co by bylo potřeba ověřit živě, než se do toho jde reálně investovat

- Kolik veterinárních ordinací v ČR používá jaký praxe-management software (obdoba otázky "jaký PMS dominuje" z původního dokumentu, jen pro veterinární obor)
- Skutečná penetrace a fungování pojištění domácích mazlíčků v ČR (proplácení vs. přímé zúčtování, hlavní pojišťovny)
- Zda první verze veterinární aplikace vůbec payer integraci potřebuje, nebo jde odložit na pozdější fázi
