# RÖRBOCK - Pipe Bending Calculator

Denna app är ett en-filsverktyg för praktisk rörböjning (koppar/stål) med fokus på märkning, kaplängd och montagelogik.

Källfil: pipe-bender.html
Publicerad startsida: index.html

## Syfte

Minska fel vid mätning, märkning och böjning genom att:
- Räkna ut kaplängd och markpositioner konsekvent.
- Visa stegvis arbetsordning.
- Hantera verktygsskillnader (Fromax/manuell) och empiriska avdrag.
- Visualisera geometri i 2D och 3D där det hjälper montören.

## Flikar och funktion

1. Bananböj
- Enkel offsetböj mellan T-koppling och H-stycke.
- Stöder fri geometri (D+H -> alfa) och vinkelläge (alfa -> H).
- Ger kaplängd, M1, böjvinkel, snedlängd och arbetsordning.

2. Flerböj
- Sekventiell märkning för flera böjar i samma rör.
- Kumulativ kompensation med böjavdrag per böj.
- Stöd för stocklängd och skarvmuff-lösning utan att flytta grundmarkerna.

3. S-böj
- Parallelförskjutning med två böjar.
- Varningar om orimlig geometri (för kort axial sträcka).

4. Svanbock
- Tre böjar med mittdel och höjdkrav.
- Varningar för negativ/för kort mittdel.

5. Takoffset
- S-böj mot takkrav (h1 till h2).
- Geometrikontroller och varningar för trånga lägen.

6. Takrör 90 grader
- Tre 90-gradersböjar upp genom tak.
- Frontvy, sidovy, isometrisk/perspektiv och 3D-preview.

## Beräkningslogik (härledning)

### Bananböj - huvudformel

Kaplängd:

```
kapa = M1 + L + instA - bendDeduction
```

Där:

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
- M1 mäts från H-stycke-änden (överkant), inte från T-sidan.

### Flerböj - markalgoritm

```
mark[0] = segment[0]
mark[i] = mark[i-1] + segment[i] - bend_deduction, i > 0
total   = mark[last]
```

Prioritet för böjavdrag:
1. Manuell override (om ifylld)
2. Empiriskt värde vid Fromax + 90 grader
3. CLR-formel för övriga fall

### Stocklängd och skarv

```
overflow     = total - stock
coupling_net = coupling_length - 2 * insertion_per_side
pipe2_length = overflow - coupling_net
```

Regel:
- Skarvlösning får inte skriva om grundmarkerna, bara visa hur markerna mappas till pjäs 1/pjäs 2.

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

Empiriskt 90-graders böjavdrag:

| Diameter | Avdrag |
|---|---|
| 12 | 19 mm |
| 15 | 27 mm (uppskattat, ej uppmätt) |

## Spårbarhet: krav -> implementation

Nedan visas hur krav kan härledas till konkreta delar i appen.

| Krav | UI-beteende | Kodansvar |
|---|---|---|
| D får ej ändras av inside/outside | Mätläge ändrar bara visad höjdtext | calculate(...), measureMode-logik |
| M1 ska vara entydig referens | Texter och arbetsordning visar H-stycke-ände/överkant | Bananböj input + workflow/formeltext |
| Fjädring ska vara justerbar | Fält "Fjädringstillägg" i Bananböj | calculate(...), springback-inläsning |
| 90 grader Fromax ska vara empirisk | Auto-avdrag i Flerböj/S-böj/Svanbock/Takoffset | MB_EMPIRICAL_90 + respektive calculate-funktion |
| 15 mm empirisk osäkerhet ska synas | Röd varningstext i Flerböj avdragsbadge | mbUpdateDeductBadge() |
| L/R ska tolkas i färdriktning | Textförändring i riktningskolumn + legend i diagram | mb-seg-header + mbDrawDiagram() |
| Overflow ska hanteras utan att förstöra markerna | Varnar och visar pjäs 1/pjäs 2 med skarv | mbCheckOverflow() |

## Releasehistorik

Version v0.5.0:
- Pedagogiska intro-kort tillagda i alla sex flikar (Bananböj, Flerböj, S-böj, Svanbock, Takoffset, Takrör 90°)
- Varje intro förklarar syfte, nödvändiga mått, vanliga misstag och extra funktioner — skrivet för lärlingar
- Riktningsvy flerböj: förenklad 2D-planvy ersätter komplex 3D-matrisprojektion
- Riktningsvy: korrekt L/R-rotationslogik för SVG-koordinatsystem
- U-bock förinställning uppdaterad till 4 böjar (R-L-L-R) för korrekt bypass-form
- Perspektivvy Takrör: 2-punktsperspektiv (VPL/VPR) ersätter kameramatris-approach
- Versionstring uppdaterad till v0.5.0

Version v0.4.3:
- Refactor: central `projectPoint()` funktion för konsekvent z-inversion i side-view
- Side-view (Visa från sidan): 45° roterad vänster för bättre perspektiv
- Side-view: spegelvänt perspektiv (X och Y inverterade) för "bakom"-vy
- Versionstring i header uppdaterad till v0.4.3

Version v0.4.2:
- Auto-select travel-mode (Följ röret framåt) vid beräkna märken
- Isometrisk zoomfix: borttagen BIAS och NORMAL_BOOST för korrekt bounding box
- Riktningsmedveten centrering: route midpoint istället för bbox center
- Side-view depth direction: negerad z i isoProj för korrekt djupframställning

Version v0.4.1:
- Isometrisk flerböj-vy har separat visualisering för riktningslogik.
- Segment i isometrisk vy är färgkodade efter nästa böjriktning (L/R).
- Nytt U-bockläge med förval R-L-R för snabbstart.
- Toggle för referenssystem i isometrisk vy:
	- Följ röret framåt (rörmokarlogik)
	- Visa från sidan (visuell/debug)
- Follow-pipe camera i travel-läge.
- Auto-zoom fix i isometrisk vy:
	- padding runt extents
	- minimum spread
	- centrering på START->SLUT
- Centerlinje-referens och netto sidoförskjutning visas i isometrisk vy.

Version v0.4:
- Enhetlig M1-referens i Bananböj: H-stycke-ände (överkant).
- Nytt fält för justerbar fjädringstillägg (grader).
- Arbetsordning förtydligad med böjriktning (mot/från vägg när underlag finns).
- Marktabell i Bananböj visar fjädring som faktisk parameter.
- Flerböj visar tydlig varning att 15 mm empiriskt värde är uppskattat.

## Spårning av ändringar

För att hålla track på allt i projektet används denna rutin:
- Versionsnummer i appheadern (index.html och pipe-bender.html).
- Releasehistorik i denna README (detta avsnitt).
- En commit per logisk ändring (kort, tydligt meddelande).
- Git-tag per release (exempel: v0.4.1).

## Verifiering och användning

Lokal körning:
- Öppna index.html i webbläsare, eller
- Starta enkel server om behov finns för testning.

Praktisk verifiering på jobb:
- Verifiera alltid med provmontering innan slutkap vid trånga toleranser.
- Kalibrera mot dagens material/verktyg om flera identiska böjar ska göras.
- Använd H-stycke-ände som konsekvent nollreferens för M1 i Bananböj.

## Tekniskt

- En fil: HTML + CSS + JS i samma dokument.
- Ingen build, inga runtime-ramverk.
- SVG-rendering sker via JavaScript.
- 3D-preview använder Zdog med fallback-laddning.
