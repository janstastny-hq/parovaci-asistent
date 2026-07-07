# Párovací asistent (prototyp)

Interaktivní prototyp nástroje, který e‑shopařům pomáhá správně **zařadit produkt do
kategorie Heureky**, **pojmenovat ho podle vzoru** a doplnit **povinné i doporučené
parametry** do XML feedu.

> Postaveno z reálných One Admin UI (Ant Design 6) komponent. Jde o prototyp pro
> design review — ne produkční nasazení (viz [Omezení](#omezení-a-další-kroky)).

## Co umí

- **Vyhodnocení kategorie** z obecného názvu sortimentu (např. „sekačka", „bazén
  Marimex") — vrátí nejlepší shodu + ~8 dalších pravděpodobných kategorií.
- **Tolerance překlepů a psaní bez diakritiky** („sekcka", „bazeny" → správná kategorie).
- **Správná struktura názvu** (vzor + příklad) a upozornění na **EAN** u knižních kategorií.
- **Povinné parametry** (s ukázkou XML) a **doporučené parametry** (kompaktní štítky).
- **Volitelný generátor názvu** — vyplníte části vzoru a nástroj složí hotový `PRODUCTNAME`.
- **Kopírování** kategorie (do `CATEGORYTEXT`) i sestaveného názvu.
- **CZ / EN** přepínač.

## Spuštění

Aplikace načítá `proto-data.json` přes `fetch`, takže **nejde otevřít přímo přes `file://`** —
je potřeba ji servírovat. Stačí jakýkoli statický server, např.:

```bash
python3 -m http.server 8765
# otevři http://localhost:8765/
```

`index.html` je hlavní nástroj, `generator.html` je samostatný sandbox generátoru názvu.

## Nasazení na GitHub Pages

Repozitář funguje na GitHub Pages bez buildu — `index.html` se servíruje z kořene.
V *Settings → Pages* zvolte branch a kořenovou složku. Aplikace pak běží na
`https://<user>.github.io/<repo>/`.

## Struktura

```
index.html          hlavní nástroj (Párovací asistent)
generator.html      samostatný sandbox generátoru názvu
proto-data.json     data (3 684 kategorií, pravidla, parametry) — sestavený artefakt
scripts/
  generate-proto-data.py   regenerace proto-data.json ze zdrojových .txt
```

## Technologie

- **Ant Design 6 + React 19** načítané jako ESM přes [esm.sh](https://esm.sh) (import‑mapy),
  JSX bez build kroku přes [htm](https://github.com/developit/htm).
- Vyhledávací logika je portovaná z původního Python nástroje (`hledat_kategorie.py`):
  kmenování, synonyma, skórování shody, fuzzy (Levenshtein) tolerance překlepů.

## Data

`proto-data.json` se generuje ze čtyř zdrojových souborů (nejsou součástí repa):
`kategorie.txt`, `pravidla.txt`, `parametry.txt`, `parametry-v2.txt`.

```bash
python3 scripts/generate-proto-data.py /cesta/ke/zdrojovym/txt
```

Skript opravuje chybu původního parseru — oddělovač kategorie/pravidla bere jako
**nejlevější** „ – / - / — " v řádku (původní parser dělil na prvním „ - ", což
komolilo pravidla obsahující pomlčku v závorce).

## Omezení a další kroky

- **Závislost na CDN** (esm.sh) — bez internetu se Ant nenačte. Pro produkci zbundlovat lokálně.
- **Celá data najednou** (~1,4 MB) — pro produkci nahradit API (`/search`, lazy parametry).
- **`parametry.txt`** pokrývá jen ~96 kategorií → fuzzy přiřazení může u ostatních dát nepřesnost.
- Prototyp nemodifikuje původní Python nástroj; ten má stejný parser bug, který je tu opravený.
