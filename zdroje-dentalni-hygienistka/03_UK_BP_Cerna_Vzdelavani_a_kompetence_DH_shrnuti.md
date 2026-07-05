# Shrnutí: Vzdělávání a kompetence dentální hygienistky (bakalářská práce)

Zdroj: Nikola Černá, *Vzdělávání a kompetence dentální hygienistky* (Education and Competences of a Dental Hygienist), bakalářská práce, Univerzita Karlova, 3. lékařská fakulta, Stomatologická klinika, Praha, květen 2023. Vedoucí práce: Mgr. Petra Křížová, DiS.
Plné znění (veřejně dostupné k akademickému využití): https://dspace.cuni.cz/bitstream/handle/20.500.11956/183468/130362464.pdf?sequence=1

**Toto je vlastní shrnutí klíčových zjištění pro účely návrhu ZubApp/DentApp, nikoli doslovný přepis práce — pro citace nebo přebírání textu použijte prosím originál.**

---

## Proč je to relevantní pro DentApp

Práce srovnává vzdělávání a kompetence DH v ČR s 10+ zeměmi (Dánsko, Finsko, Nizozemsko, Norsko, Portugalsko, Slovensko, Spojené království, Švédsko, Švýcarsko + širší mezinárodní přehled). Pro produkt je to užitečné zejména kvůli:
- **čtyřúrovňovému modelu kompetencí v ČR** – přímý podklad pro návrh rolí/oprávnění v appce (co smí DH dělat sama, co jen na indikaci, co pod dohledem),
- mezinárodnímu srovnání (relevantní, pokud appka někdy cílí i mimo ČR, nebo jen pro kontext "co je v ČR neobvykle omezené").

## Čtyři úrovně kompetencí DH v ČR (dle vyhlášky č. 55/2011 Sb.)

1. **Bez odborného dohledu a bez indikace ZL:** motivace a instruktáž ústní hygieny, stanovení úrovně ústní hygieny, zdravotně-výchovná činnost v prevenci, výzkumná činnost, manipulace se zdravotnickými prostředky.
2. **Bez odborného dohledu, v souladu s diagnózou ZL:** vyšetření dutiny ústní, hodnocení měkkých/tvrdých tkání, otisky a modely, odstranění plaku/kamene supra- i subgingiválně, leštění zubů, aplikace desenzibilizačních/remineralizačních/antiseptických prostředků, odstranění pigmentací.
3. **Pod odborným dohledem ZL:** práce jako zubní instrumentářka, aplikace povrchové anestezie, bělení zubů, zhotovování RTG snímků, výměna gumiček fixních ortodontických aparátů.
4. **Pod přímým vedením ZL:** pečetění fisur.

**Poznámka:** Odborný dohled ZL nevyžaduje fyzickou přítomnost v budově – lékař musí být jen v případě potřeby dostupný pro pomoc (stanovisko ČSK ZS 1/2020). Roste trend samostatných ordinací DH jako OSVČ bez trvalé přítomnosti ZL. → **Relevantní pro model "provozovna" v ZubApp: DH může mít vlastní ordinaci nezávisle na zubní ordinaci, propojenou jen "na dálku" s indikujícím lékařem.**

## Mezinárodní srovnání kompetencí (výběr zemí, zjednodušeno)

| Země | Lokální anestezie | RTG snímky | Bělení zubů | Extrakce | Výplně | Screening rakoviny DÚ |
|---|---|---|---|---|---|---|
| ČR | jen povrchová | ne (od 2011 nově možné dle novely) | ano (pod dohledem) | ne | ne | ne |
| Dánsko | infiltrační | ne | ne | ne | ne | ne |
| Finsko | infiltrační | ano | ano | ne | ne | ne |
| Nizozemsko | infiltrační+svodná | ano | ano | ne | ano (malé kazy) | – |
| Norsko | infiltrační | ano | ne (jen ZL) | ne | ne | ne |
| Portugalsko | ne | ano | ano | ne | ne | ne |
| Slovensko | jen povrchová | ne | ano | ne | ne | ne |
| Spojené království | infiltrační+svodná | ano | ano (na předpis) | ne | ne | ano |
| Švédsko | infiltrační+svodná | ano | ano | ne | ne | ne |
| Švýcarsko | infiltrální | ano | ano | ne | ne | ne |

*(Zjednodušeno z tabulek a textu práce, kap. 1.4.2; pro přesné znění a zdroje jednotlivých zemí viz originál.)*

Zajímavost: Nizozemsko rozlišuje od 1. 7. 2020 "dentální hygienistku" a "registrovanou dentální hygienistku" – ta druhá smí pracovat zcela bez indikace/dohledu zubního lékaře a podléhá samostatné disciplinární odpovědnosti. Spojené království rozlišuje "dental hygienist" (preventivní role) a "dental therapist" (širší kompetence, blíž zubnímu lékaři, hlavně pro pacienty do 18 let).

## Vzdělávací systém v ČR (doplňuje seznam škol – viz `05_ADHCR_Seznam_skol_dentalni_hygiena.md`)

- 3 vysoké školy (Praha – 3. LF UK, Brno – MUNI, Opava – Slezská univerzita) + 11 vyšších odborných škol.
- Studium na VŠ hrazeno MŠMT; státní VOŠ cca 3 000 Kč/rok; soukromé školy 42–70 000 Kč/rok (příp. celkem za studium).
- Přijímací zkoušky: typicky biologie + chemie (VŠ), na VOŠ často test manuální zručnosti + biologie/somatologie, pohovor.
- Ukončení: obhajoba práce + zkoušky z parodontologie a preventivní stomatologie (+ cizí jazyk na VOŠ).

## Historie oboru (stručně)

Vznik v USA 1913 (Dr. Alfred Fones, první hygienistka Irene Newman, 1917 první licence). V Evropě první Norsko (1924), pak Anglie (1943), postupně další státy. V ČR obor vzniká 1995–1996 (inspirace švýcarským modelem), první výuka na Soukromé VOŠZ v Praze a VOŠZ a SŠZ v Ústí nad Labem.

---

**Poznámka pro design DentApp:** čtyřúrovňový model kompetencí (výše) je přímo použitelný jako datový model "typů úkonů" s příznakem požadované úrovně dohledu/indikace zubního lékaře – to pak určuje, jaké UI/workflow (žádost o indikaci, potvrzení lékařem, apod.) appka pro daný úkon potřebuje.
