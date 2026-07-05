# Návrh architektury: platforma pro komunikaci se zubními pacienty na českém trhu

**Připravil:** návrh řešitelské architektury
**Účel:** referenční architektura pro rezervační/komunikační vrstvu ve stylu Doctolib, přizpůsobenou českým regulatorním, plátcovským a tržním podmínkám
**Stav:** Počáteční návrh — předpoklady označené [OVĚŘIT] je třeba validovat s právním/compliance oddělením a s místními kontakty u plátců, než se přistoupí ke stavbě

---

## 1. Rozsah a positioning produktu

Systém kopíruje strategii „vrstva nad PMS, ne náhrada PMS“, která přinesla úspěch Doctolibu ve Francii a Německu:

- **Nenahrazuje** stávající systémy praxe-managementu / klinického vedení dokumentace, které už české zubní ordinace používají
- **Vlastní** pacientskou vrstvu: vyhledávání, rezervace, připomínky, komunikace a follow-up po návštěvě
- Integruje se s existujícími back-office systémy přes API/webhook, místo aby nutil ordinace migrovat svou klinickou dokumentaci

Toto rozhodnutí o rozsahu je jednoznačně nejdůležitějším architektonickým vodítkem — určuje téměř všechny integrační hranice popsané níže.

### 1.1 Konkurenční riziko: „jen vrstva" je dočasné okno, ne trvalá výhoda

Aktualizace k 2026-07: Doctolib sám od podzimu 2025 nabízí pro vybrané obory (zatím primárně Německo) all-in-one praxe software, který jde nad rámec bookingu — klinická dokumentace s AI asistentem (návrh ICD-10 kódů), OCR karta pacienta, AI fakturace (návrh EBM/GOÄ kódů). Detailní rozbor a srovnání viz KB článek ZUB-A-1 („Referenční PMS pro zubní praxi vs. Doctolib").

Důsledek pro tuto architekturu: pozicování „vrstva, ne náhrada PMS" zůstává správný start (viz fázování v sekci 6), ale nelze ho brát jako trvalý strategický plot — je to okno, které se může zavřít, pokud Doctolib nebo konkurent přinese all-in-one model do ČR/CEE. Z toho plynou dva průřezové požadavky na architekturu níže:

- **Nezamykat rozšiřitelnost směrem ke klinické vrstvě** — datový model a integrační rozhraní stavět tak, aby šlo později přidat „light" klinické moduly (typicky recall/parodontální rizikové skóre, ne plný odontogram) bez přepisu základu (viz 3.1, 5)
- **Partnerství s PMS dodavateli mají teď vyšší strategickou hodnotu** — dokud je platforma partner, ne konkurent, integrace a distribuce jsou snazší; toto okno má časový limit

---

## 2. Požadavky specifické pro český trh

Toto jsou omezení, která odlišují tuto stavbu od obecné EU rezervační platformy a měla by formovat architekturu od prvního dne, ne se dodatečně dolepovat později.

### 2.1 Regulace a compliance
- **GDPR** platí jako ve všech členských státech EU, ale zdravotní údaje jsou „zvláštní kategorií“ podle čl. 9 — vyžaduje explicitní souhlasové toky, přísnější logování přístupu a minimalizaci dat už v návrhu
- **České očekávání ohledně rezidence dat** — GDPR sice striktně nevyžaduje hosting v tuzemsku, ale čeští zdravotničtí zákazníci a partneři z veřejného sektoru často očekávají hosting v EU regionu (ideálně v ČR nebo blízkém okolí) jako důvěryhodnostní/procurement faktor [OVĚŘIT aktuální pravidla veřejných zakázek, pokud se cílí na zakázky navázané na VZP]
- **Zákon o zdravotních službách** a související pravidla vedení dokumentace určují, co se počítá za zdravotnickou dokumentaci a jak dlouho se musí uchovávat — relevantní, pokud platforma ukládá jakékoli klinické poznámky, ne jen čistá rezervační/kontaktní data
- **Česká stomatologická komora** — ověření registrace/licence zubaře může být užitečný signál důvěryhodnosti (např. ověřený odznak praktikujícího), ale nejde o veřejně potvrzenou API integraci [OVĚŘIT]

### 2.2 Plátci a pojišťovací krajina
- České zdravotnictví je založené na pojištění, ne na jednom plátci — sedm zdravotních pojišťoven (**VZP** je dominantní s cca 60% podílem trhu, dále ZP MV ČR, OZP, ČPZP, VoZP, RBP a Zaměstnanecká pojišťovna Škoda)
- Rozsah krytí zubní péče se výrazně liší podle pojišťovny a plánu (základní péče je hrazena, mnoho běžných výkonů vyžaduje doplatek) — integrace fakturace/pojištění je podstatně komplexnější než na jednoplátcovém nebo čistě soukromém trhu
- **Architektonický důsledek:** nestavět jeden natvrdo zakódovaný modul „pojišťovací fakturace“. Navrhnout abstrahovanou **vrstvu integrace plátců** s adaptérem po vzoru pluginu pro každou pojišťovnu, protože každá bude pravděpodobně mít jiný formát nároků a jiný cyklus aktualizací. Počítat s vícefázovým rolloutem (začít s VZP, postupně přidávat další), ne s jednorázovou velkou integrací [OVĚŘIT aktuální dostupnost API/EDI u každé pojišťovny — pravděpodobně to bude vyžadovat přímé oslovení, ne veřejnou dokumentaci]

### 2.3 Jazyk a lokalizace
- Český jazyk uživatelského rozhraní je povinný, ne volitelný — nejde o „hezké mít navíc“ lokalizaci, je to primární tržní jazyk
- Diakritika (ě š č ř ž ý á í é) musí být korektně ošetřena všude — vyhledávání, SMS šablony a jakákoliv logika porovnávání textu (např. fuzzy vyhledávání jména zubaře) s tím musí explicitně počítat, ne jen spoléhat na obecnou podporu Unicode
- SMS připomínky (klíčová retenční funkce) mají limit počtu znaků na zprávu, který se dál zmenšuje s diakritikou, pokud není kódování ošetřeno pečlivě (GSM-7 vs. UCS-2) — na toto je třeba upozornit brzy, protože to ovlivňuje náklady na SMS providera na zprávu

---

## 3. Architektura na vysoké úrovni

### 3.1 Přehled komponent

**Pacientská vrstva**
- Pacientská webová aplikace (vyhledávání, rezervace, správa termínů)
- Pacientská mobilní aplikace (iOS/Android) — pro v1 volitelná, lze spustit web-first vzhledem k nižší počáteční uživatelské základně než ve Francii/Německu
- Handlery notifikačních kanálů (SMS, e-mail, push)

**Vrstva pro ordinaci**
- Dashboard ordinace (agenda, seznam kontaktů pacientů, inbox zpráv)
- Správa personálu/týmu (přístup více uživatelů per ordinace, role-based oprávnění pro zubaře vs. hygienistku vs. recepci)
- Základní reporting (míra no-show, objem rezervací, hodnocení)

**Základní backendové služby**
- **Booking service** — logika kalendáře dostupnosti, prevence dvojitého zápisu, správa slotů
- **Identity & auth service** — oddělené identity domény pro pacienty vs. personál ordinace; založené na OAuth2/OIDC
- **Notification service** — šablonované SMS/e-mail/push, s českými šablonami a diakriticky bezpečným kódováním
- **Messaging service** — obousměrná komunikace pacient–ordinace, ukládaná po konverzacích
- **Payer integration service** — abstrahovaná vrstva adaptérů pro každou pojišťovnu (viz 2.2)
- **PMS integration service** — odchozí webhook/API konektory na existující české zubní PMS produkty (např. cokoliv, co na lokálním trhu dominuje — je potřeba revize krajiny dodavatelů před stavbou, protože partneři pro PMS integraci se budou lišit od Francie/Německa). Datový model navrhnout s vyhrazeným místem pro budoucí „light" klinická data (recall interval, rizikové skóre) — i když se v Fázi 1–3 nepoužije, ať rozšíření nevyžaduje redesign schématu (viz 1.1)
- **Review/reputation service** — sběr a zobrazení hodnocení po návštěvě
- **Admin/reporting service** — interní provozní nástroje, ne pro pacienty/ordinaci

**Datová vrstva**
- Úložiště pacientských záznamů (PII + data přiléhající ke zdravotní kategorii — šifrovaná at rest, s logováním přístupu)
- Úložiště rezervací/rozvrhu
- Úložiště zpráv
- Úložiště auditního logu (append-only, odolné proti manipulaci — potřebné pro účely GDPR accountability)

**Integrační vrstva**
- SMS/e-mail brána (SMS provideři na českém trhu mají typicky lepší doručitelnost/ceny než globální obecní provideři — stojí za to lokální dodavatele porovnat)
- Platební brána (pro jakékoli doplatky nebo platby za soukromé služby)
- Synchronizace kalendáře (export Google/Outlook/iCal pro personál ordinace)
- Adaptéry pojišťoven (dle 2.2)

### 3.2 Navržený tok požadavku (rezervace → návštěva → follow-up)

1. Pacient vyhledá zubaře → **Booking service** dotáže dostupnost z **PMS integration service** (nebo z vlastního kalendáře platformy, pokud ordinace nemá napojený PMS)
2. Rezervace potvrzena → **Notification service** odešle potvrzení a naplánuje úlohu s připomínkou
3. Připomínka se spustí (typicky 24–48 h před termínem) → SMS/e-mail odeslaný přes **Notification service**
4. Návštěva proběhne v ordinaci — klinická práce probíhá zcela ve vlastním PMS ordinace, mimo hranice této platformy
5. Po návštěvě → **Notification service** spustí follow-up zprávu a žádost o hodnocení přes **Review/reputation service**
6. Pokud se týká fakturace/pojištění → **Payer integration service** zpracuje podání nároků asynchronně, oddělené od rezervačního toku, aby výpadek plátce nikdy neblokoval rezervaci

---

## 4. Nefunkční požadavky

- **Dostupnost:** rezervace je klíčová hodnota produktu — cílit na vysokou dostupnost (99,9 %+) konkrétně pro booking service, i když admin/reporting nástroje mohou mít nižší SLA
- **Rezidence dat:** výchozí hosting v EU regionu; silně zvážit datacentrum v ČR nebo střední Evropě vzhledem k lokálním očekáváním důvěryhodnosti ve zdravotnictví-adjacentním procurementu [OVĚŘIT konkrétní požadavky při jednání o partnerství s veřejným sektorem nebo pojišťovnami]
- **Auditovatelnost:** každý přístup ke zdravotně-adjacentním datům pacienta by měl být zalogovaný v append-only auditní stopě — jde jak o požadavek GDPR accountability, tak o praktickou funkci budující důvěru u ordinací hodnotících platformu
- **Přístup ke škálovatelnosti:** oddělit vrstvu integrace plátců a PMS integrace od hlavního rezervačního toku pomocí asynchronního messagingu (fronty), aby pomalá nebo nedostupná integrace třetí strany nikdy nezhoršila základní rezervační zážitek
- **Lokalizace od začátku, ne dodatečně:** postavit systém řetězců/šablon jako od začátku vědomý lokalizace, místo natvrdo zakódovaného českého textu, protože expanze na Slovensko (velmi podobná plátcovská/regulatorní struktura) je přirozeným dalším trhem

---

## 5. Navržený technologický přístup (implementačně neutrální)

Tato sekce záměrně nepředepisuje konkrétní frameworky — jde o hranice a integrační filozofii, ne o konkrétní tech stack. Nicméně obecné doporučení:

- **API-first návrh** pro každou interní službu — to je to, co umožňuje vzor adaptéru per PMS/plátce
- **Event-driven architektura** mezi rezervačním tokem a notifikační/plátcovskou/hodnoticí službou, aby downstream selhání neblokovala hlavní rezervaci
- **Oddělené identity domény** pro pacienty a personál ordinace od začátku — sloučení později je nákladná migrace
- **Feature-flag integrace plátců per pojišťovna**, aby šlo spustit podporu jen pro VZP a přidávat další bez redeploye základních služeb
- **Rozšiřitelné hranice, ne uzavřené moduly** — booking/identity/notification služby navrhnout s explicitními extension pointy (např. rozhraní pro budoucí „clinical light" service), aby reakce na konkurenční tlak (viz 1.1) byla otázkou přidání modulu, ne přepisu jádra

---

## 6. Doporučené fázování rolloutu

1. **Fáze 1 — základní rezervace + notifikace**, bez integrace plátců, ruční/CSV synchronizace PMS podle potřeby pro pilotní ordinace
2. **Fáze 2 — adaptér VZP** (největší pojišťovna, nejvyšší návratnost vynaloženého úsilí)
3. **Fáze 3 — adaptéry dalších pojišťoven**, prioritizované podle poptávky ordinací
4. **Fáze 4 — hlubší integrace PMS** s těmi českými PMS dodavateli, kteří mají největší instalovanou základnu [vyžaduje studii krajiny dodavatelů — nemám k dispozici spolehlivá aktuální data o tom, které PMS produkty na českém trhu konkrétně dominují]
5. **Fáze 5 — podmíněná, spouštěná konkurencí:** „light" klinický modul (recall/rizikové skóre pacienta, ne plný odontogram) — aktivovat pouze pokud nastane spouštěč popsaný v sekci 1.1 a 7 (Doctolib nebo jiný all-in-one hráč expanduje do ČR/CEE), ne jako výchozí plán

---

## 7. Otevřené body vyžadující přímé ověření

Vlajkuji explicitně místo hádání, protože tyto body zásadně ovlivňují architektonická rozhodnutí:

- Které PMS/systémy praxe-managementu aktuálně dominují na českém zubařském trhu (potřeba pro prioritizaci integračních partnerů)
- Aktuální dostupnost API/EDI od VZP a dalších pojišťoven pro podání nároků
- Zda je pro zubní obor relevantní nějaká česká národní e-health infrastruktura (obdoba interakcí Doctolibu s národními zdravotními systémy ve Francii)
- Aktuální výklad Úřadu pro ochranu osobních údajů (ÚOOÚ) k prosazování GDPR konkrétně pro zdravotně-adjacentní rezervační data, na rozdíl od plné klinické dokumentace
- **Sledovat jako spouštěč k přehodnocení (viz 1.1, Fáze 5):** zda a kdy Doctolib nebo jiný all-in-one hráč (klinika + fakturace) expanduje do ČR/CEE — při potvrzení je třeba rychle revidovat scope a případně urychlit „light" klinický modul

Tento dokument je třeba brát jako výchozí architekturu k internímu review, ne jako finalizovanou specifikaci — položky plátců a PMS krajiny výše jsou nejvyšší prioritou mezer k uzavření před závazkem k detailnímu technickému návrhu.
