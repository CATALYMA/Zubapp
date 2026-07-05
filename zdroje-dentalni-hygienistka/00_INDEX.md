# Zdroje: výukové a metodické materiály pro zubní hygienistky

Podklady stažené 2026-07-04 pro design obrazovek DentApp / PMS pro nezávislé hygienisty (viz `pms-pro-zubni-hygienisty.md`, ZUB-82). Většina zdrojů je uložena jako extrahovaný text/markdown (ne originální PDF binárka) — nástroje v tomto prostředí neumí stahovat syrové binární soubory, jen textový obsah stránek/PDF, což je ale pro práci s obsahem prakticky výhodnější (prohledávatelné, citovatelné).

## Soubory

1. **01_MZCR_Kvalifikacni_standard_Dentalni_hygienistka.md** — oficiální kvalifikační standard MZ ČR (Věstník 12/2019): kompetence DH, povinné předměty a hodinové dotace, obsah odborné praxe po ročnících, struktura logbooku výkonů. **Nejdůležitější zdroj pro datový model appky.**
2. **02_ADHCR_Profesni_a_eticky_kodex.md** — etický kodex ADH ČR **+ kapitola 4 "Profesní standardy výkonu povolání"**, která definuje 4 standardní typy návštěvy hygienistky (vstupní vyšetření, základní čištění, rozšířené/hloubkové čištění, recall) s časovou dotací, indikacemi a kroky. **Přímo odpovídá na otevřenou otázku ze story "Definice obsahu light clinical záznamu" v `pms-pro-zubni-hygienisty.md` (sekce 5, bod 1).**
3. **03_UK_BP_Cerna_Vzdelavani_a_kompetence_DH_shrnuti.md** — vlastní shrnutí bakalářské práce (UK, 2023): čtyřúrovňový model kompetencí DH v ČR (bez dohledu/s indikací/pod dohledem/pod vedením ZL) + mezinárodní srovnání kompetencí (co smí DH v Dánsku, Nizozemsku, UK, Švédsku atd.). Užitečné pro návrh rolí/oprávnění a jako inspirace pro budoucí mezinárodní rozšíření.
4. **04_VOSZ_Usti_anatomie_dutiny_ustni.md** — doplňkové materiály anatomie dutiny ústní (nízká přímá relevance, spíš pro copy k anatomickým schématům).
5. **05_ADHCR_Seznam_skol_dentalni_hygiena.md** — kompletní seznam 14 škol vzdělávajících DH v ČR (mapa potenciálních partnerů/zdrojů obsahu).

## Co jsem NEstáhl a proč

- **118 "metodických postupů" na szsvzs.cz** — po ověření jde primárně o materiály pro obor zubní technik (kreslení/modelování zubů, protetika), ne pro dentální hygienistky; stáhl jsem jen 2 ukázkové pro anatomický kontext.
- **Učebnice Mazánek: *Stomatologie pro dentální hygienistky a zubní instrumentářky*** — komerční kniha (Grada, ~400 Kč), nelze legálně stáhnout; doporučuji koupit, pokud bude potřeba hlubší klinický obsah.
- **Bakalářské práce s "výukovými testy/listy" (MUNI, 257.cz)** — nalezeny, ale extrakce plného textu se nepodařila (timeout/nedostupné); odkazy níže pro případné ruční stažení.

## Nalezené, ale nestažené odkazy (k ručnímu doplnění)

- Výukové testy pro DH – parodontologie (MUNI): https://is.muni.cz/th/kpl0j/VYUKOVE_TESTY_PRO_DENTALNI_HYGIENISTKY_-_PARODONTOLOGIE.pdf?studium=616837
- Výukové listy pro DH (bakalářská práce): https://2nifwg0.257.cz/th/o40qc/Vyukove_listy_pro_dentalni_hygienistky_Archive.pdf
- ADH ČR – Legislativa a stanoviska: https://www.asociacedh.cz/stanoviska
- LKS (časopis České stomatologické komory) – odborné články: https://www.lks-casopis.cz

## Doporučený další krok

Kapitola 4 v souboru `02_...` (typy návštěv) + kompetenční tabulka v `01_...` (co smí DH sama vs. na indikaci) dávají dohromady prakticky hotový podklad pro definici PHR záznamu, který story v `pms-pro-zubni-hygienisty.md` označuje jako otevřený bod.
