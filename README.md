# Progress – träningslogg med progressiv överbelastning

En mobil-webbapp (PWA) för att logga träningspass och få vetenskapligt grundade förslag på när du ska öka vikt eller reps. Appen räknar också ut när varje muskelgrupp är återhämtad och kan tränas igen.

**Live:** https://progress-training-7k3m.netlify.app

Gränssnittet är på engelska. Den fullständiga kravspecen finns i [SPEC.md](SPEC.md).

## Funktioner

- **Övningsbibliotek** med cirka 70 standardövningar för skivstång, hantlar, kettlebell och kroppsvikt. Övningarna är sorterade efter muskelgrupp, och du kan lägga till egna.
- **Passmallar** där vikt och reps fylls i automatiskt när du startar ett pass.
- **Loggning** av vikt, reps och RPE (1–10) per set. Kroppsviktsövningar räknas med andel av kroppsvikten plus extravikt.
- **Progressionsmotor** med dubbel progression och RPE-autoreglering:
  - Basövningar: vikten ökar när alla set klaras på RPE ≤ 8.
  - Isolations- och kroppsviktsövningar: först fler reps, sedan mer vikt.
  - Om en övning står still i 3 pass föreslås en deload på −10 %.
  - Om RPE stiger vid samma vikt varnar appen för trötthet.
  - Varje förslag visas med en kort motivering.
- **Återhämtning per muskelgrupp** (48–72 h beroende på muskelns storlek, antal set och RPE), med en varning om du startar ett pass där en muskel inte är återhämtad.
- **Veckovolym** i hårda set per muskel, jämfört med riktvärdet 10–20 set per vecka.
- **Vilotimer** som startar automatiskt när ett set bockas av, med vibration och ljud.
- **Grafer och personliga rekord:** beräknat 1RM, toppvikt och volym per övning.
- **Kropp och kondition:** logg för kroppsvikt, mått och konditionspass.
- **Offline:** appen fungerar utan nät och kan installeras på hemskärmen.
- **Backup** genom export och import av all data som en JSON-fil.

## Data och integritet

All data sparas lokalt i webbläsaren (IndexedDB). Det finns ingen server och inget konto, och ingenting skickas någonstans. Det innebär att:

- datan finns bara på den enhet och den adress där appen används
- du bör exportera en backup regelbundet under *More → Export backup*
- en ny webbadress (t.ex. vid byte av hosting) startar med en tom app, så exportera först och importera sedan

## Teknik

- [Vite](https://vitejs.dev), React 18 och TypeScript
- [Dexie](https://dexie.org) för IndexedDB
- [Recharts](https://recharts.org) för grafer
- [vite-plugin-pwa](https://vite-pwa-org.netlify.app) för service worker och manifest
- [Vitest](https://vitest.dev) för enhetstester av progressions- och återhämtningslogiken

## Kom igång

Kräver Node.js 20 eller senare.

```bash
npm install
```

```bash
npm run dev
```

Öppna sedan http://localhost:5173. Med `--host`, som redan är inställt i dev-skriptet, går appen även att nå från mobilen på samma nätverk via datorns IP-adress. Offline-läge och installation fungerar dock bara över https.

| Kommando | Vad det gör |
|---|---|
| `npm run dev` | Startar utvecklingsservern |
| `npm test` | Kör enhetstesterna |
| `npm run build` | Typkontrollerar och bygger till `dist/` |
| `npm run preview` | Visar det byggda resultatet lokalt |

> **Windows/PowerShell:** om `npx` eller `npm` blockeras av execution policy kan du använda `npx.cmd` respektive `npm.cmd` i stället.

## Publicering

Appen publiceras på Netlify. Inställningarna finns i `netlify.toml`.

```bash
npx.cmd --yes netlify-cli deploy --prod --dir dist
```

Kommandot bygger appen med `npm run build` och publicerar `dist/` till produktionsadressen. Telefoner som har appen installerad hämtar den nya versionen nästa gång appen öppnas med nät.

## Projektstruktur

```
src/
  lib/
    exercises.ts     Övningsbibliotek, muskelgrupper och standardintervall
    progression.ts   Progressionsmotorn (förslag, stagnation, trötthet)
    recovery.ts      Återhämtning och veckovolym per muskel
    stats.ts         e1RM, belastning, volym och personliga rekord
    db.ts            Databas (Dexie), historik, start/avslut av pass, backup
    workoutStart.ts  Återhämtningsvarning och deload-kontroll
    logic.test.ts    Enhetstester
  screens/           En fil per skärm (Home, ActiveWorkout, Templates …)
  components/        Delade komponenter (övningsväljare, vilotimer, grafer)
  App.tsx            Navigering med flikar
SPEC.md              Kravspec och vetenskaplig grund
```

## Vetenskaplig grund

Modellerna och källorna (Helms 2016, Schoenfeld 2016/2017, MacDougall 1995, Damas 2016) beskrivs i [SPEC.md](SPEC.md) och i appen under *More → How the app decides*.
