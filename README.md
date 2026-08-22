# Maker Calc

Webová mini-appka na výpočet ceny 3D tisku. Výchozí hodnoty jsou laděné
pro **Bambu Lab X2D**, ale v nastavení si je přepíšeš pro jakoukoli tiskárnu.
Odpovídá na otázku „kolik mě to vlastně stálo, než jsem to zase dal zadarmo“.

## Koncept

Dva režimy, protože kamarádům se počítá jinak než platícím zákazníkům:

- **Pro kámoše 🍺** — počítá se jen materiál a elektřina. Žádná amortizace,
  spotřební díly, zmetkovitost ani vlastní čas.
- **Na prodej 💰** — počítá se úplně všechno včetně vlastního času a
  zmetkovitosti.

Přepnutí režimu skryje/zobrazí příslušné vstupy, ať to není matoucí. Přepočet
je živý při psaní, žádné tlačítko „Spočítat“.

## Repozitář

Vlastní git repozitář, veřejný na GitHubu jako projekt **maker-calc**:
<https://github.com/wineorvibecoder/maker-calc>

Živá verze běží na GitHub Pages — dá se otevřít rovnou z mobilu v dílně:
<https://wineorvibecoder.github.io/maker-calc/>

## Tech stack

Vanilla HTML/CSS/JS v **jednom souboru**, bez frameworků, bez build kroku,
bez externích requestů (fonty i ikonky jsou systémové/inline SVG). Funguje
i po otevření `index.html` přímo z disku.

| Soubor | Obsah |
|---|---|
| `index.html` | celá appka — struktura, styly i logika výpočtu v jednom souboru |

## Jak se používá

1. Otevřít `index.html` v prohlížeči (dvojklik stačí, server není potřeba).
2. Nahoře zvolit režim — **Pro kámoše** / **Na prodej**.
3. Vyplnit gramy filamentu a dobu tisku (obojí vyčteno ze sliceru).
4. Sazby a konstanty (elektřina, amortizace, zmetkovitost…) sedí
   v **⚙️ Nastavení** — jednou se nastaví a pak už na ně není potřeba sahat.
5. Vpravo (na mobilu dole) se průběžně dopočítává rozpis položek,
   **cena za kus** a **cena za celý plate**.

Zadané hodnoty i zvolený režim se ukládají do `localStorage` (klíč
`makerCalc.v1`), takže se defaulty nemusí přepisovat pokaždé znovu.
Tlačítko **Reset na výchozí** v nastavení vrátí původní hodnoty.

## Vstupy

Pole jsou rozdělená podle jednoduchého pravidla: **množství z konkrétního
tisku zůstává na hlavní obrazovce, sazby a konstanty jsou v nastavení.**

### Hlavní obrazovka — mění se u každého tisku

| Pole | Výchozí | Poznámka |
|---|---|---|
| Počet kusů na plate | `1` | dělí se jím celková cena |
| Filament (g) | — | ze sliceru |
| Cena filamentu (Kč/kg) | `350` | vepředu schválně — liší se s každou cívkou i materiálem |
| Doba tisku (h) | — | ze sliceru, desetinné číslo |
| Můj čas (h) | `0.25` | jen „Na prodej“; 15 min na přípravu a sundání, u složitějšího kusu víc |

### ⚙️ Nastavení — sazby a konstanty

**Tiskárna a energie** (platí v obou režimech):

| Pole | Výchozí | Poznámka |
|---|---|---|
| Příkon tiskárny (W) | `200` | orientační průměr, jde zpřesnit wattmetrem |
| Cena elektřiny (Kč/kWh) | `6` | |

**Náklady pro režim „Na prodej“** (v režimu Pro kámoše se skryjí):

| Pole | Výchozí | Poznámka |
|---|---|---|
| Cena tiskárny (Kč) | `19000` | |
| Životnost tiskárny (h) | `4000` | odhad pro amortizaci |
| Spotřební díly (Kč/h tisku) | `4` | viz rozpad níže |
| Spotřební materiál na tisk (Kč) | `1` | jednou za celý plate, ne za kus |
| Zmetkovitost (%) | `10` | |
| Můj čas (Kč/h) | `500` | hodinová sazba |

Když se cokoli v nastavení liší od výchozí hodnoty, svítí na tlačítku
**Nastavení** tečka — ať je poznat, že se počítá s něčím přenastaveným.

**Odkud se bere 4 Kč/h za spotřební díly:**
kalená tryska ~400–600 Kč / ~300–500 h ≈ 1–1,5 Kč/h · PEI deska
~1500–2000 Kč / ~500–800 h ≈ 2–3 Kč/h · drobné opotřebení ~0,5–1 Kč/h.

**Co je „spotřební materiál“:** trocha Perlanu na odmaštění desky
(~0,2–0,5 Kč) a pár stříků IPA (~0,5 Kč). Počítá se **jednou za plate**,
protože desku odmašťuješ jednou, ne po každém kusu.

## Vzorce

**Pro kámoše:**

```
Materiál   = (filament_g / 1000) × cena_filamentu
Elektřina  = (příkon_W / 1000) × doba_tisku_h × cena_elektřiny
Celkem     = Materiál + Elektřina
Cena/kus   = Celkem / počet_kusů
```

**Na prodej:**

```
Materiál           = (filament_g / 1000) × cena_filamentu
Elektřina          = (příkon_W / 1000) × doba_tisku_h × cena_elektřiny
Amortizace         = (cena_tiskárny / životnost_h) × doba_tisku_h
Spotřební díly     = díly_Kč_na_h × doba_tisku_h
Spotřební materiál = flat cena (jednou za plate)
Můj čas            = čas_Kč_na_h × čas_h
Mezisoučet         = Materiál + Elektřina + Amortizace + Spotřební díly
                     + Spotřební materiál + Můj čas
Zmetkovitost       = Mezisoučet × (zmetkovitost_% / 100)
Celkem             = Mezisoučet + Zmetkovitost
Cena/kus           = Celkem / počet_kusů
```

### Kontrolní příklad

100 g filamentu, 3 h tisku, 2 kusy na plate, ostatní hodnoty výchozí:

| Položka | Pro kámoše | Na prodej |
|---|---|---|
| Materiál | 35,00 Kč | 35,00 Kč |
| Elektřina | 3,60 Kč | 3,60 Kč |
| Amortizace tiskárny | — | 14,25 Kč |
| Spotřební díly | — | 12,00 Kč |
| Spotřební materiál | — | 1,00 Kč |
| Můj čas | — | 125,00 Kč |
| Zmetkovitost | — | 19,09 Kč |
| **Celkem za plate** | **38,60 Kč** | **209,94 Kč** |
| **Cena za kus** | **19,30 Kč** | **104,97 Kč** |

Hezky je z toho vidět, že u malých tisků nedělá největší položku filament,
ale vlastní čas.

## Poznámky k implementaci

- Vše se počítá lokálně v prohlížeči, nic nikam neodchází.
- U každé položky se ukazuje i její podíl na celkové ceně (barevný proužek
  v barvách filamentu) — je hned vidět, kam peníze tečou.
- Dokud nejsou vyplněné gramy a doba tisku, místo nul se zobrazí výzva
  k vyplnění.
- Prázdné pole = 0; počet kusů minimálně 1; nulová životnost tiskárny dá
  nulovou amortizaci, ne `Infinity`.
- Odsazení textu v inputech se dopočítává podle šířky jednotky, aby se
  hodnota nikdy nedotkla popisku „Kč/kWh“. Po otevření nastavení se přepočítá
  znovu — skrytá pole mají do té doby nulové rozměry.
- Nastavení je nativní `<dialog>` otevřený přes `showModal()`, takže Esc
  a fokus řeší prohlížeč sám. Zavírá se i křížkem a klikem na pozadí.
- `color-scheme: dark` — aby prohlížeč kreslil scrollbary a spinnery tmavě.
- Starý klíč `localStorage` (`bambuX2D.v1`) se při prvním spuštění zmigruje
  na `makerCalc.v1`, ať uložené hodnoty nezmizí.

## Stav

Hotovo, funkční. Ověřeno v prohlížeči — čísla sedí s kontrolním příkladem,
konzole čistá, na mobilu (375 px) jednosloupcový layout bez horizontálního
scrollu.
