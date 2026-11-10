# Träningsapp – kravspec (v1)

## Översikt
En mobil-webbapp (PWA) för att logga träningspass och styra progressiv överbelastning med vetenskaplig grund.
Användaren är nybörjare (< 1 års regelbunden styrketräning) och har både styrka och muskeltillväxt som mål.

- **Gränssnittets språk:** engelska
- **Plattform:** PWA som går att installera på hemskärmen och fungerar offline
- **Lagring:** lokalt i webbläsaren (IndexedDB). Inget konto och ingen server. Backup sker med export/import av en JSON-fil.
- **Teknik:** Vite, React, TypeScript, Dexie (IndexedDB), Recharts och vite-plugin-pwa

## Loggning
| Typ | Fält |
|---|---|
| Styrka (fria vikter) | vikt (kg), reps och RPE 1–10 per set (halva steg tillåtna) |
| Kroppsvikt | extravikt (kg, 0 = bara kroppen, negativ = assisterad), reps och RPE. Den effektiva belastningen är kroppsvikt × kroppsviktsfaktor + extravikt. |
| Kondition | aktivitet, tid (min), distans (km), snittpuls och RPE. Ingen automatisk progression i v1. |
| Kropp | datum, kroppsvikt, midja, bröst, arm och lår |

Ett pass som pågår sparas löpande, så inget går förlorat om appen stängs.

## Övningsbibliotek
- Inbyggt bibliotek med standardövningar för skivstång, hantlar, kettlebell och kroppsvikt.
- Varje övning har:
  - **primära** och **sekundära** muskelgrupper
  - typ: **compound** (basövning) eller **isolation**
  - utrustning
  - standardviktsteg
  - kroppsviktsfaktor (bara för kroppsviktsövningar)
- Övningarna listas och filtreras efter muskelgrupp och region:
  - Chest
  - Back: lats, upper back, lower back
  - Shoulders: front, side, rear
  - Arms: biceps, triceps, forearms
  - Legs: quads, hamstrings, glutes, calves
  - Core: abs
- Användaren kan lägga till egna övningar och ändra viktsteget per övning.

## Passmallar
- En mall har ett namn och en lista med övningar. Varje övning i listan har antal set, repintervall (min–max) och vilotid.
- När ett pass startas från en mall fylls den föreslagna vikten och de föreslagna repsen i för varje övning, med en kort motivering.
- Det går att lägga till övningar under ett pågående pass.

## Progressionsmotor
Den bygger på dubbel progression med RPE-autoreglering. RPE räknas om till reps i reserv: RIR = 10 − RPE (Helms m.fl. 2016).

Underlaget är det senaste passet med samma övning: målvikt, målreps, utförda set och högsta RPE.

| Utfall | Basövning (compound) | Isolationsövning |
|---|---|---|
| Alla set når målreps, högsta RPE ≤ 6 | Öka vikten med 2 × viktsteg | Öka repsen med 2 (eller vikten om taket nås) |
| Alla set når målreps, högsta RPE ≤ 8 | Öka vikten med 1 viktsteg | Öka repsen med 1. Vid repMax: öka vikten och börja om på repMin |
| Alla set når målreps, RPE 8,5–9 | Öka repsen med 1 mot repMax. Vid repMax: öka vikten och börja om på repMin | Öka repsen med 1. Vid repMax: öka vikten och börja om på repMin |
| Alla set når målreps, RPE ≥ 9,5 | Samma vikt och samma reps (befäst) | Samma vikt och samma reps |
| Något set under målreps | Samma vikt. Räknas som ett misslyckat pass | Samma vikt |

- **Stagnation:** om det beräknade 1RM inte har förbättrats på 3 pass i rad sänks vikten med 10 % (avrundat till viktsteget), och upptrappningen börjar om.
- **Trötthet:** om samma vikt och samma reps ger en RPE som stigit med ≥ 1,5 under de 3 senaste passen visas en varning om deload.
- **Global deload:** om ≥ 2 övningar har stagnerat eller visar trötthet de senaste 14 dagarna visas ett förslag om en lättare vecka, med cirka 60 % av volymen och samma vikt.
- **Beräknat 1RM:** Epley med RIR, `e1RM = belastning × (1 + (reps + RIR) / 30)`.
- **Viktsteg:** användaren har mikroskivor. Standardsteg:
  - skivstång, överkropp: 1 kg
  - skivstång, underkropp: 2 kg
  - hantlar och kettlebell: 2 kg
  - extravikt på kroppsviktsövningar: 1,25 kg

  Viktsteget kan ändras per övning.
- **Utan historik:** appen föreslår repMin och ber användaren välja en startvikt som känns som RPE 6–7.
- **Motivering:** varje förslag får en kort förklaring, till exempel "+1 kg: all sets hit 8 reps at RPE ≤ 7".

## Återhämtning per muskelgrupp
Muskelproteinsyntesen och den neuromuskulära återhämtningen efter ett hårt pass tar ungefär 48–72 timmar. Det beror på muskelns storlek, volymen och hur nära failure seten gjordes (MacDougall 1995, Damas 2016, Schoenfeld 2016). Modellen:

1. **Effektiva set per muskel:** varje set räknas som 1,0 för en primär muskel och 0,5 för en sekundär. Värdet viktas sedan efter RPE:
   - RPE ≥ 8: 1,0
   - RPE 7: 0,85
   - RPE 6: 0,7
   - RPE ≤ 5: 0,5
   - RPE saknas: 0,9
2. **Grundtid:**
   - stora muskler (quads, hamstrings, glutes, lower back): 72 h
   - medelstora (chest, lats, upper back, front delts): 60 h
   - små (side/rear delts, biceps, triceps, forearms, calves, abs): 48 h
3. **Återhämtningstid för ett pass:** `grundtid × clamp(0,5 + effektiva set / 10, 0,6, 1,4)`. 5 hårda set ger alltså grundtiden, och fler set förlänger den.
4. **Klar igen:** för varje muskel används det senaste av alla pass inom de senaste 7 dagarna. Appen visar en procentsats och tidpunkten då muskeln är klar igen.
5. **Statusfärger:**
   - grön: ≥ 100 %
   - gul: 75–99 %
   - röd: < 75 %
6. **Varning vid passstart:** när ett pass startas varnar mallen om någon av dess primära muskler inte är återhämtad.

## Volym
- Antalet set per muskelgrupp och vecka räknas på samma sätt som de effektiva seten ovan, men bara set med RPE ≥ 6.
- Riktvärdet är 10–20 set per vecka (Schoenfeld m.fl. 2017). Under 10 visas som låg volym och över 20 som hög.

## Extrafunktioner
- **Vilotimer:** startar automatiskt när ett set bockas av. Standardvila är 180 s för basövningar och 90 s för isolationsövningar, och den kan ändras per mall. Vibration och ljud när vilan är slut. Timern bygger på tidsstämplar, så den fungerar även när appen ligger i bakgrunden.
- **Grafer per övning:** beräknat 1RM, toppvikt och volym (vikt × reps) över tid.
- **Personliga rekord (PR):** bästa beräknade 1RM, tyngsta vikt och mest volym i ett set. Nya PR markeras när passet avslutas.
- **Kropp:** kroppsvikt och mått loggas över tid och visas i en graf. Den senaste kroppsvikten används för kroppsviktsövningar.
- **Kondition:** separat logg med graf över distans och tempo.
- **Backup:** export och import av all data som en JSON-fil.

## Utanför v1
- molnsynk och konto
- progression för kondition
- maskiner och kabelövningar i standardbiblioteket (de kan läggas till som egna övningar)
- periodiseringsprogram
