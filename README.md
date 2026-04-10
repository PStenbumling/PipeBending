# RORBOCK - Pipe Bending Calculator

Denna app ar ett en-filsverktyg for praktisk rorbojning (koppar/stal) med fokus pa markning, kaplangd och montagelogik.

Kallfil: pipe-bender.html
Publicerad startsida: index.html

## Syfte

Minska fel vid matning, markning och bojning genom att:
- Rakna ut kaplangd och markpositioner konsekvent.
- Visa stegvis arbetsordning.
- Hantera verktygsskillnader (Fromax/manuell) och empiriska avdrag.
- Visualisera geometri i 2D och 3D dar det hjalper montoren.

## Flikar och funktion

1. Bananboj
- Enkel offsetboj mellan T-koppling och H-stycke.
- Stoder fri geometri (D+H -> alfa) och vinkellage (alfa -> H).
- Ger kaplangd, M1, bojvinkel, snedlangd och arbetsordning.

2. Flerboj
- Sekventiell markning for flera bojar i samma ror.
- Kumulativ kompensation med bojavdrag per boj.
- Stod for stocklangd och skarvmuff-laning utan att flytta grundmarkerna.

3. S-boj
- Parallelforskjutning med tva bojar.
- Varningar om orimlig geometri (for kort axial stracka).

4. Svanbock
- Tre bojar med mittdel och hojdkrav.
- Varningar for negativ/for kort mittdel.

5. Takoffset
- S-boj mot takkrav (h1 till h2).
- Geometrikontroller och varningar for tranga lagen.

6. Takror 90 grader
- Tre 90-gradersbojar upp genom tak.
- Frontvy, sidovy, isometrisk/perspektiv och 3D-preview.

## Berakningslogik (harledning)

### Bananboj - huvudformel

Kaplangd:

```
kapa = M1 + L + instA - bendDeduction
```

Dar:

```
bendDeduction = CLR * (1 - cos(alfa))
```

Och geometri:

```
H_rise     = hojdB - (hojdA + tBygg)
freeBefore = M1 - instB
riseOfDiag = H_rise - freeBefore
alfa       = atan(D / riseOfDiag)
L          = sqrt(D^2 + riseOfDiag^2)
```

Viktig referens:
- M1 mats fran H-stycke-anden (overkant), inte fran T-sidan.

### Flerboj - markalgoritm

```
mark[0] = segment[0]
mark[i] = mark[i-1] + segment[i] - bend_deduction, i > 0
total   = mark[last]
```

Prioritet for bojavdrag:
1. Manuell override (om ifylld)
2. Empiriskt varde vid Fromax + 90 grader
3. CLR-formel for ovriga fall

### Stocklangd och skarv

```
overflow     = total - stock
coupling_net = coupling_length - 2 * insertion_per_side
pipe2_length = overflow - coupling_net
```

Regel:
- Skarvlosning far inte skriva om grundmarkerna, bara visa hur markerna mappas till pjas 1/pjas 2.

## Fasta verktygsdata

Fromax CLR-tabell:

| Diameter | CLR |
|---|---|
| 6 | 20 |
| 8 | 28 |
| 10 | 35 |
| 12 | 43 |
| 15 | 53 |
| 18 | 72 |
| 22 | 110 |

Empiriskt 90-graders bojavdrag:

| Diameter | Avdrag |
|---|---|
| 12 | 19 mm |
| 15 | 27 mm (uppskattat, ej uppmatt) |

## Spårbarhet: krav -> implementation

Nedan visas hur krav kan harledas till konkreta delar i appen.

| Krav | UI-beteende | Kodansvar |
|---|---|---|
| D far ej andras av inside/outside | Mätlage andrar bara visad hojdtext | calculate(...), measureMode-logik |
| M1 ska vara entydig referens | Texter och arbetsordning visar H-stycke-ande/overkant | Bananboj input + workflow/formeltext |
| Fjädring ska vara justerbar | Falt "Fjädringstillägg" i Bananboj | calculate(...), springback-inlasning |
| 90 grader Fromax ska vara empirisk | Auto-avdrag i Flerboj/S-boj/Svanbock/Takoffset | MB_EMPIRICAL_90 + respektive calculate-funktion |
| 15 mm empirisk osakerhet ska synas | Röd varningstext i Flerboj avdragsbadge | mbUpdateDeductBadge() |
| L/R ska tolkas i färdriktning | Textforandring i riktningskolumn + legend i diagram | mb-seg-header + mbDrawDiagram() |
| Overflow ska hanteras utan att forstora markerna | Varnar och visar pjas 1/pjas 2 med skarv | mbCheckOverflow() |

## Releasehistorik

Version v0.4.3:
- Refactor: central `projectPoint()` funktion för konsekvent z-inversion i side-view
- Side-view (Visa från sidan): 45° roterad vänster för bättre perspektiv
- Side-view: spegelvänd perspektiv (X och Y inverterade) för "bakom"-vy
- Versionstring i header uppdaterad till v0.4.3

Version v0.4.2:
- Auto-select travel-mode (Följ röret framåt) vid beräkna märken
- Isometrisk zoomfix: borttagen BIAS och NORMAL_BOOST för korrekt bounding box
- Riktningsmedveten centrering: route midpoint istället för bbox center
- Side-view depth direction: negerad z i isoProj för korrekt djupframställning

Version v0.4.1:
- Isometrisk flerboj-vy har separat visualisering for riktningslogik.
- Segment i isometrisk vy ar fargkodade efter nasta bojriktning (L/R).
- Nytt U-bocklage med forval R-L-R for snabbstart.
- Toggle for referenssystem i isometrisk vy:
	- Folj roret framat (rormokarlogik)
	- Visa fran sidan (visuell/debug)
- Follow-pipe camera i travel-lage.
- Auto-zoom fix i isometrisk vy:
	- padding runt extents
	- minimum spread
	- centrering pa START->SLUT
- Centerlinje-referens och netto sidoforskjutning visas i isometrisk vy.

Version v0.4:
- Enhetlig M1-referens i Bananboj: H-stycke-ande (overkant).
- Nytt falt for justerbar fjadringstillagg (grader).
- Arbetsordning fortydligad med bojriktning (mot/fran vagg nar underlag finns).
- Marktabell i Bananboj visar fjadring som faktisk parameter.
- Flerboj visar tydlig varning att 15 mm empiriskt varde ar uppskattat.

## Spårning av ändringar

For att halla track pa allt i projektet anvands denna rutin:
- Versionsnummer i appheadern (index.html och pipe-bender.html).
- Releasehistorik i denna README (detta avsnitt).
- En commit per logisk andring (kort, tydligt meddelande).
- Git-tag per release (exempel: v0.4.1).

## Verifiering och anvandning

Lokal korning:
- Oppna index.html i webblasare, eller
- Starta enkel server om behov finns for testning.

Praktisk verifiering pa jobb:
- Verifiera alltid med provmontering innan slutkap vid tranga toleranser.
- Kalibrera mot dagens material/verktyg om flera identiska bojar ska goras.
- Anvand H-stycke-ande som konsekvent nollreferens for M1 i Bananboj.

## Tekniskt

- En fil: HTML + CSS + JS i samma dokument.
- Ingen build, inga runtime-ramverk.
- SVG-rendering sker via JavaScript.
- 3D-preview anvander Zdog med fallback-laddning.
