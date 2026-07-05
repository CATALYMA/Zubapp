# Výzkum aplikací pro zubní praxe a pacienty — export z konverzace

**Kontext:** Výzkum business analytika zaměřený na aplikace používané zubaři/hygienistkami pro praxe-management a komunikaci s pacienty, se zúžením na střední Evropu a návrh architektury pro český trh.

---

## 1. Tržní kategorie (globální/US baseline)

- **Software pro praxe-management (PMS):** Dentrix, Eaglesoft, Open Dental, Curve Dental, CareStack, tab32, Denticon
- **Komunikace/engagement s pacienty:** Weave, Solutionreach, RevenueWell, NexHealth, Lighthouse 360, Yapi, Podium
- **Klinika/zobrazování:** iTero, Carestream, Dexis; AI diagnostika — Pearl, VideaHealth, Overjet
- **Specificky pro hygienu:** parodontální grafy (obvykle zabudované v PMS), sledování CE/compliance (DentalCare.com, ADHA)
- **Teledentistry:** TeleDent, Denteractive
- **Platby:** CareCredit, Sunbit, Cherry

**Klíčová dynamika:** zavedení dodavatelé PMS historicky měli slabé nástroje pro komunikaci s pacienty, což vytvořilo samostatné odvětví (Weave, Solutionreach atd.), které tuto mezeru zaplnilo. Aktuálním trendem je konsolidace platforem (dodavatelé PMS pohlcují komunikační funkce).

## 2. All-in-one ekosystémy (klientská app + recepce + historie + rozvrh + platby)

**Skutečně nativní, jednotný kódový základ:** CareStack, MoGo — cloud-first, stavěné přímo tak, aby se předešlo fragmentaci, oblíbené u DSO (víceočních zubních skupin).

**„All-in-one“ přes integrace více dodavatelů (ne jednotný kódový základ):** Denticon, Dentrix Ascend — silný cloudový PMS, ale vrstva pacientské aplikace obvykle pochází od partnerských produktů (NexHealth, Solutionreach, Weave).

**Poznámka analytika:** marketingová tvrzení o „all-in-one“ řešení by se měla ověřit přímým dotazem na dodavatele, zda pacientská aplikace, rozvrh a platby sdílejí kódový základ, nebo jde o slepené akvizice.

## 3. Evropa — regionální krajina

V celé Evropě neexistuje jeden lídr ekvivalentní Dentrixu; trh je fragmentovaný podle země kvůli jazyku, systémům pojištění/úhrad a preferencím pro rezidenci dat vyplývajícím z GDPR.

| Platforma | Region | Poznámky |
|---|---|---|
| Dentally | Velká Británie | Nejsilnější nativní UK PMS |
| Software of Excellence | UK/Irsko/Austrálie | Vlastněno Carestream |
| Exact | Nizozemsko | Vlastněno Cegedim |
| CGM Dentalis / Z1 | Německo | CompuGroup Medical — dominantní hráč zdravotnického IT v DACH regionu |
| Charly (solutio) | Německo | Oblíbený u nezávislých praxí |
| DAMPSOFT | Německo | Starší, věrná základna, méně cloud-nativní |
| Julie Solutions | Francie | |
| Clinic Cloud | Španělsko | |
| **Doctolib** | Francie, dominantní i v Německu | Pacientská vrstva, ne plnohodnotný PMS — vrstva nad existujícími back-office systémy |

**Strukturální rozdíly oproti USA:** GDPR/rezidence dat je reálný produktový diferenciátor; integrace fakturace s národním zdravotním pojištěním (KZV v Německu, Carte Vitale/AMELI ve Francii) brání snadné přeshraniční expanzi platforem.

## 4. Zaměření na DACH region (střední Evropa)

- **CGM/Charly/DAMPSOFT** dominují back-office/fakturaci (kompatibilita s KZV je pro německé praxe nevyjednatelná)
- **Doctolib** je dominantní pacientská vrstva, expandující strategií „nenahrazovat PMS“ — nižší tření při adopci než plná výměna PMS
- Polsko/Česko/Maďarsko: mnohem méně konsolidované, menší lokální dodavatelé, tenká nebo žádná nativní vrstva pacientské aplikace; Doctolib expanduje i do některých z těchto trhů

## 5. Hodnocení „vycházející hvězdy“: Doctolib

Uváděné důvody:
- Růstová trajektorie (ne jen zavedená instalovaná základna jako u CGM/Charly)
- Postupné rozšiřování rozsahu (rezervace → telehealth → messaging), stejný vzorec, který je dovedl k dominanci ve Francii
- Přeshraniční dynamika — jedna z mála platforem, které skutečně expandují napříč hranicemi národních fakturačních systémů

**Poznámka:** míra jistoty ohledně konkrétních čísel podílu na trhu/růstu v DACH regionu je střední — doporučuje se ověřit živým vyhledáváním nebo oborovou zprávou, než se to bere jako ověřený fakt.

## 6. Doctolib — moduly a proces (diagramy viz původní konverzace)

**Moduly pro pacienty:** rezervace, pacientský účet, messaging, videokonzultace
**Moduly pro ordinaci (Doctolib Pro):** rozvrh/agenda, záznamy pacientů (na úrovni kontaktu, ne klinické), správa týmu, fakturace/hodnocení

**Průběh procesu:** pacient vyhledá a rezervuje → automatická připomínka → návštěva v ordinaci (klinická práce probíhá ve vlastním PMS ordinace, mimo hranice Doctolibu) → follow-up zpráva a žádost o hodnocení (tento recall-trigger krok je klíčovým hnacím motorem retence/růstu)

## 7. Architektura řešení pro český trh

Kompletní architektonický dokument byl vytvořen samostatně: **czech-dental-platform-architecture.md** (vygenerováno již v této relaci — importujte oba soubory do Coworku společně).

Klíčové body z daného dokumentu:
- Strategie: vrstva nad existujícím PMS české ordinace, ne jeho náhrada (stejný přístup jako u Doctolibu)
- **Nejrizikovější neznámé označené k ověření:** (1) integrace plátců — český trh má 7 pojišťoven (VZP dominantní, ~60% podíl), z nichž každá pravděpodobně bude potřebovat samostatný adaptér pro nároky; (2) které dodavatelé PMS aktuálně na českém trhu dominují
- Architektura doporučuje abstrahovaný vzor adaptér-per-pojišťovna, event-driven oddělení integrace plátců/PMS od hlavního rezervačního toku a fázovaný rollout začínající jen s VZP
- Požadavky na lokalizaci: české uživatelské rozhraní, ošetření diakritiky ve vyhledávání/SMS, dopady kódování SMS na náklady (GSM-7 vs. UCS-2)
- Platí GDPR + český zákon o vedení zdravotnické dokumentace; rezidence dat v EU/ČR je faktor důvěry pro zdravotnický procurement

---

## Známé mezery / co ověřit před tím, než se podle tohoto výzkumu bude jednat

- V této konverzaci nebylo použito živé vyhledávání na webu — všechna čísla, tvrzení o tržním podílu a aktuální stav dodavatelů je třeba ověřit proti aktuálním zdrojům
- Krajina evropských dodavatelů (zejména hráči specifičtí pro Rakousko/Švýcarsko/CEE) má nižší míru jistoty než materiál pro US/UK
- Krajina českých PMS dodavatelů a dostupnost API pojišťoven byly explicitně označeny jako neověřené a vyžadují přímé oslovení/výzkum
