# Mapa obrazovek — pms-hygienistky-prototyp.html

Referenční seznam pro adresování konkrétních obrazovek/sekcí v analýze a zpětné vazbě. Kódy jsou viditelné přímo v prototypu jako malý štítek u nadpisu každé obrazovky/modálu.

## Hlavní obrazovky (S)

| Kód | Obrazovka | Přístup |
|---|---|---|
| S01 | Dnešní agenda | výchozí obrazovka po otevření |
| S02 | Kalendář (+ čekací listina) | menu vlevo |
| S03 | Pacienti | menu vlevo |
| S04 | Detail pacienta | klik na pacienta v S01/S02/S03 |
| S05 | Fakturace | menu vlevo |
| S06 | Sklad materiálu | menu vlevo |
| S07 | Recenze a hodnocení | menu vlevo |
| S08 | Souhlasy a přístupy | menu vlevo |
| S09 | Auditní stopa | menu vlevo |
| S10 | Nastavení praxe (ověření, tým, indikující ZL) | menu vlevo |

### Záložky detailu pacienta (S04.x)

| Kód | Záložka |
|---|---|
| S04.1 | Přehled (anamnéza, rizikové faktory) |
| S04.2 | Provedené výkony |
| S04.3 | Plán péče a souhlasy |
| S04.4 | Recall plán |
| S04.5 | Přístupy (consent granty) |
| S04.6 | Rezervace a zprávy |

## Modály (M)

| Kód | Modál | Otevření |
|---|---|---|
| M01 | Nová rezervace | tlačítko "Nová rezervace" (S01/S02/S03) |
| M02 | Vystavit fakturu | tlačítko v S05 |
| M03 | Nový záznam z výkonu | tlačítko v S04.2 nebo rychlá akce v S01 |
| M04 | Pohled pacienta (telefon) | tlačítko "Pohled pacienta" v horní liště |
| M05 | Detail výkonu | klik na řádek v S04.2 (Provedené výkony) |

### Záložky pohledu pacienta (M04.x)

| Kód | Záložka |
|---|---|
| M04.1 | Rezervace |
| M04.2 | Moje karta |
| M04.3 | Sdílet přístup |

---

## Jak kódy používat při sběru zpětné vazby od klienta

- Kódy jsou barevně zvýrazněné štítky u nadpisu každé obrazovky/záložky (žlutý = hlavní obrazovka, modrý = záložka).
- V pravém dolním rohu je navíc **trvalý plovoucí ukazatel** "Aktuální obrazovka", který se automaticky aktualizuje při přepínání obrazovek, záložek i modálů — vidíte ho i po scrollování.
- Klik na tento ukazatel **zkopíruje aktuální kód do schránky** — lze rovnou vložit do poznámky, e-mailu nebo ticketu s klientovou připomínkou.
- Doporučený zápis zpětné vazby: `S04.2 — klient chce vidět datum poslední fluoridace přímo v tabulce`, `M03 — chybí pole pro číslo zubu u nálezu`.
