# RÖRBOCK — Pipe Bending Calculator

> **AI context + user documentation.**
> This file serves both as project README and as system context for AI-assisted development.
> When working with AI tools: reference this file for domain rules, UX principles and mental models.

---

## Syfte

Hjälpa rörmokare (och lärlingar) att räkna ut märkpositioner, kaplängder och böjgeometri — direkt i fält, utan papper.

**Källfil:** `pipe-bender.html`
**Deploy:** `index.html` + `README.md` — ingenting annat publiceras.

---

## Kärnproblem (AI: läs detta först)

Rörmokare tänker inte i matematiska variabler. De tänker i mål:

- *"Jag ska flytta röret 60mm åt sidan"*
- *"Jag ska ta mig runt ett hinder"*
- *"Röret ska passa in utan spänning"*

Appen måste alltid **översätta verkligt intent → beräkning**, aldrig tvärtom.

**Exponera aldrig:** arctan, CLR-formeln, variabelnamn (H, D, alpha) i UI-text.
**Fråga alltid efter:** mått rörmokaren redan har på tumstocken.

---

## Mental modell — hur rörmokaren tänker

| Rörmokaren säger | Appens böjtyp | Nyckelvariabler |
|---|---|---|
| "Flytta röret i sidled" | S-böj | Sidmått, Väggsträcka |
| "Runt ett hinder" | Puckel / Svanbock | Hinderhöjd, Plats |
| "Från T-koppling till H-stycke" | Bananböj | Instick, M1, Kaplängd |
| "Parallellt med grannröret" | Följeböj (ej impl.) | cc-avstånd |
| "Upp längs väggen och in i taket" | Takrör 90° | S1–S4, Gap |

---

## Flikar och funktion

### 1. Bananböj
Enkel böj mellan T-koppling (diagonal) och H-stycke (rakt).
- **Rörmokaren vill:** röret ska falla på plats utan spänning i kopplingarna
- **Kräver:** sidmått (D), höjder för kopplingar, instick, M1
- **Ger:** kaplängd, M1-position, böjvinkel, snedlängd
- Stöder friläge (D+H→α) och vinkelläge (α→H automatiskt)

### 2. Flerböj
Sekventiell märkning för flera böjar på ett rör.
- **Rörmokaren vill:** märka upp ett rör en gång, bocka i rätt ordning
- **Kräver:** sträcklängder, böjriktningar (L/R i färdriktning), vinkel
- **Ger:** M1–Mn kumulativt med böjavdrag
- U-bock-förinställning: 4 böjar R-L-L-R (rektangulärt omlopp)
- Skarv med muff: visar märken på pjäs 1 och 2 utan att grundmåtten ändras

### 3. S-böj
Parallellförskjutning — röret byter läge men fortsätter i samma riktning.
- **Rörmokaren vill:** flytta röret förbi ett hinder eller byta position
- **Kräver:** sidmått (D), väggsträcka (H) eller vinkel
- **Ger:** M1, M2, kaplängd
- Guidar live med intent-text: "Du försöker flytta röret i sidled (S-böj)"
- Saknade mått visas som nästa steg (ingen rå feltext)
- För tight geometri förklaras i verkliga termer + konkreta förslag
- Knapp: **Justera automatiskt** (flackare vinkel först, räknar om direkt)

### 4. Svanbock
Tre böjar — uppåt, vågrätt, tillbaka — för att ta sig över hinder eller höjdskillnad.
- **Rörmokaren vill:** ta sig förbi ett rör eller upp till en ny höjd
- **Kräver:** sidmått (D), stigning (H1), mittdel (H2), vinkel
- **Ger:** M1, M2, M3, kaplängd
- Om H2 < 0: appen föreslår annat mått eller vinkel

### 5. Takoffset
S-böj anpassad till takinstallation — byter höjd längs väggen.
- **Rörmokaren vill:** rör som löper längs vägg ska klättra upp till taket
- **Kräver:** rörcentrum nu (h1), rörcentrum mål (h2), sidmått, vinkel
- **Ger:** M1, M2, kaplängd, clearance-kontroll mot takböjen
- Varnar automatiskt om utrymmet är för litet

### 6. Takrör 90°
Tre 90°-böjar: längs vägg → upp vägg → längs tak → upp genom tak.
- **Rörmokaren vill:** dra ett rör från väggen upp och ut genom bjälklaget
- **Kräver:** S1 (längs vägg), S2 (upp vägg), S3 (längs tak), S4 (ovan tak), Gap
- **Ger:** M1, M2, M3, kaplängd, frontvy, sidovy, 3D-vy

---

## UX-regler (AI: följ alltid dessa)

1. **Fråga i verkliga termer** — "Hur långt ska röret flyttas?" inte "Ange D"
2. **Dölj matematiken** — visa bara resultat, aldrig formelsträngar
3. **Föreslå, varna aldrig** — "Testa 30° istället" inte "Ogiltigt värde"
4. **Mät alltid cc** — centerline till centerline, aldrig utsida
5. **Samma referensände** — märk alltid från START-änden, visualisera med pil i diagram

### Terminologi

| Teknisk term | Använd istället |
|---|---|
| D (variabelnamn) | Sidmått / Utstick |
| H (axial) | Väggsträcka / Längd längs vägg |
| α | Böjvinkel (grader) |
| arctan, CLR | — (dölj helt) |
| M1, M2, M3 | Märke 1, 2, 3 (i diagram) |

---

## Beräkningslogik

### Bananböj

```
kapa = M1 + L + instA - bendDeduction
bendDeduction = CLR × (1 - cos α)

H_rise     = hojdB - (hojdA + tBygg)
freeBefore = M1 - instB
riseOfDiag = H_rise - freeBefore
α          = arctan(D / riseOfDiag)
L          = sqrt(D² + riseOfDiag²)
```

M1 mäts alltid från H-styckets ände (överkant).

### Flerböj

```
mark[0] = segment[0]
mark[i] = mark[i-1] + segment[i] - bend_deduction   (i > 0)
total   = mark[last]
```

Böjavdrag-prioritet:
1. Manuell override
2. Empiriskt: `MB_EMPIRICAL_90 = { 12: 19mm, 15: 27mm }` vid Fromax + 90°
3. Fallback: `CLR × (1 - cos θ)`

### Stocklängd och skarv

```
overflow     = total - stock_length
coupling_net = coupling_length - (2 × insertion_per_side)
pipe2_length = overflow - coupling_net
```

Skarv ändrar aldrig grundmarkerna — bara hur de fördelas på pjäs 1 och 2.

---

## Verktygsdata

### Fromax No.55 — CLR-tabell

| Ø mm | CLR mm |
|---|---|
| 6 | 20 |
| 8 | 28 |
| 10 | 35 |
| 12 | 43 |
| 15 | 53 |
| 18 | 72 |
| 22 | 110 |

### Empiriskt böjavdrag 90°

| Ø mm | Avdrag |
|---|---|
| 12 | 19 mm (uppmätt) |
| 15 | 27 mm (uppskattat) |

### tBygg

`22mm` = avstånd från stamrör cc till T-grenens anslutningsyta (mun). Används i höjdberäkning för bananböj.

---

## Tekniskt

- **En fil:** HTML + CSS + JS i `pipe-bender.html`. Ingen build.
- **Deploy:** kopieras till `index.html`, pushas med `README.md`.
- **SVG-rendering:** genereras med JavaScript, ingen extern lib (utom Zdog för 3D-preview).
- **Geometri:** alltid centerline (cc). Inside/outside påverkar bara visning, aldrig beräkning.

---

## Roadmap / identifierade gap

Funktioner som saknas baserat på rörmokarpraktik:

- **Puckel-kalkylator** — ange hinderhöjd, få markeringar för 3-böjars brygga
- **Parallella rör (följeböj)** — cc-avstånd → extra rörängd för ytterröret
- **Radiatoranslutning** — mall med standardmått per radiatorhöjd
- **Väggbricka-snabbval** — 60 cc förvalt, genväg i S-böj och bananböj
- **Alternativvinkel-förslag** — om vald vinkel ger ogiltigt resultat, föreslå närmaste som fungerar

---

## Releasehistorik

### v0.5.3
- Auditfix: Bananböj använder empiriskt böjavdrag vid Fromax + 90° (`MB_EMPIRICAL_90`) istället för CLR-formel
- Auditfix: S-böj `Rör före M1` korrigerad label/hint (ej sammanblandad med `Längd längs vägg`)
- Auditfix: S-böj kräver endast sidmått för beräkning; `Rör före M1` är frivillig och kan vara tom (= 0)
- UX-fix: S-böj varningar visar bara åtgärder användaren faktiskt kan göra (vinkel/sidmått)

### v0.5.1
- S-böj uppgraderad till guidat assistentläge (intent, live-guidning, förslag + auto-fix)
- Realtidsfeedback vid ändring av S-böj-indata (inte bara via Beräkna-knapp)
- Ny knapp: **Justera automatiskt** för snabbaste praktiska fix i trånga geometrier

### v0.5.0
- Pedagogiska intro-kort i alla sex flikar (rörmokarperspektiv, vardagsspråk)
- Riktningsvy flerböj: förenklad 2D-planvy, korrekt L/R-rotation för SVG
- U-bock förinställning: 4 böjar R-L-L-R (korrekt bypass-form)
- Perspektivvy Takrör: 2-punktsperspektiv (VPL/VPR) ersätter kameramatris
- README omskriven som AI-kontext + domändokumentation

### v0.4.4
- Fix: version display i header

### v0.4.3
- Refactor: central `projectPoint()` för konsekvent z-inversion i sidovy
- Sidovy: 45° roterad för bättre perspektiv

### v0.4.2
- Auto-select travel-mode vid beräkning
- Isometrisk zoomfix: korrekt bounding box
- Riktningsmedveten centrering

### v0.4.1
- Isometrisk flerböj-vy med L/R-färgkodning
- U-bockläge (R-L-R förinställning)
- Toggle travel/side-vy

### v0.4.0
- Enhetlig M1-referens (H-stycke-ände)
- Justerbart fjädringstillägg
- Förtydligad arbetsordning

---

## Spårbarhet

| Krav | UI-beteende | Kodansvar |
|---|---|---|
| D ändras ej av inside/outside | Mätläge ändrar bara höjdtext | `calculate()`, `measureMode` |
| M1 entydig referens | Text visar H-stycke-ände/överkant | Bananböj input + workflow |
| 90° Fromax empirisk | Auto-avdrag i alla böjtyper | `MB_EMPIRICAL_90` |
| Skarv ändrar ej grundmarker | Pjäs 1/2 visas separat | `mbCheckOverflow()` |
| L/R i färdriktning | Legend + riktningsdiagram | `mbDrawIsoDiagram()` |