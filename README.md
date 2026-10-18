# CrossFit Coach — nativní iOS appka (SwiftUI)

Toto je nativní přepis webové PWA appky CrossFit Coach do SwiftUI. Obsahuje
stejnou databázi 100 cviků, stejný generátor tréninků (periodizace, autoregulace
podle RPE, segmentace dlouhých metconů), lokální persistenci, in-app timer a
reálnou Strava integraci.

**Důležité:** tento kód byl napsán bez přístupu k Xcode/Swift compileru — nebyl
tedy nikde zkompilován ani odzkoušen. Soubory jsou pečlivě provázané 1:1 podle
ověřené JS logiky, ale první build v Xcode téměř jistě ukáže pár drobných
chyb (chybějící import, překlep, typová neshoda). Zkopíruj mi prosím text
chyby z Xcode a já to doladím.

## 1. Vytvoření Xcode projektu

1. Otevři Xcode → **File → New → Project… → iOS → App**.
2. Product Name: `CrossFitCoach`, Interface: **SwiftUI**, Language: **Swift**.
   Nezaškrtávej Core Data ani testy (nejsou potřeba).
3. Ulož projekt **do kořene tohoto repozitáře** (vedle tohoto README) — nebo
   projekt vytvoř kamkoliv a poté níže popsané soubory jen přetáhni do jeho
   `CrossFitCoach/` adresáře.
4. Xcode vytvoří vlastní `CrossFitCoachApp.swift` a `ContentView.swift` —
   **oba smaž** (nahrazují je soubory z tohoto repozitáře).

## 2. Import zdrojových souborů

Přetáhni do Xcode (do skupiny `CrossFitCoach`, se zaškrtnutým „Copy items if
needed" a cílovým target `CrossFitCoach`) celou strukturu složek z tohoto
adresáře:

```
CrossFitCoach/
├── CrossFitCoachApp.swift
├── Models/
│   ├── Enums.swift
│   ├── Exercise.swift
│   ├── Profile.swift
│   └── Workout.swift
├── Data/
│   └── ExerciseDatabase.swift      (auto-generováno, 100 cviků)
├── Services/
│   ├── WorkoutGenerator.swift
│   ├── Storage.swift
│   ├── TimerEngine.swift
│   ├── StravaService.swift
│   ├── ShareCardRenderer.swift
│   └── AppStore.swift
└── Views/
    ├── RootTabView.swift
    ├── Shared/
    │   ├── UIComponents.swift
    │   ├── ActivityShareSheet.swift
    │   ├── ShareCardView.swift
    │   └── ExerciseDetailView.swift
    ├── Today/
    │   ├── TodayView.swift
    │   ├── TodayComponents.swift
    │   ├── TimerSheetView.swift
    │   └── SwapSheetView.swift
    ├── History/
    │   └── HistoryView.swift
    ├── Library/
    │   └── LibraryView.swift
    └── Profile/
        └── ProfileView.swift
```

Zachovej tuto adresářovou strukturu i ve Xcode (skupiny podle složek) — žádný
soubor na to není technicky závislý, ale usnadní to orientaci.

## 3. Nastavení cíle (deployment target)

- **iOS Deployment Target: 16.0 nebo vyšší** — appka používá `ImageRenderer`
  (export sdílecí karty) a `NavigationStack`, které jsou dostupné od iOS 16.
  V Xcode: target `CrossFitCoach` → **General → Minimum Deployments → iOS 16.0**.

## 4. Custom URL scheme pro Strava OAuth

Appka používá `ASWebAuthenticationSession` a po přihlášení na Strava se
vrací zpět do appky přes vlastní URL scheme `crossfitcoach://`.

1. Target `CrossFitCoach` → **Info** tab → **URL Types** → **+**.
2. **URL Schemes**: `crossfitcoach`
3. Identifier: `com.yourcompany.crossfitcoach` (libovolné, jen pro přehled).

Bez tohoto kroku se po přihlášení na Strava appka nevrátí zpět (OAuth
callback se neotevře).

## 5. Strava Cloudflare Worker

Appka používá stejný princip jako webová verze CrossFit Coach: malý Cloudflare
Worker, který drží Strava `client_secret` a appce přeposílá OAuth token
exchange / refresh / vytvoření aktivity. Pokud ho ještě nemáš nasazený, je
potřeba ho nasadit (viz `strava-worker/` ve webovém repozitáři CrossFit
Coach, případně si ho necháš vygenerovat znovu). V appce (záložka **Profil →
Strava**) pak zadáš URL svého workeru (např.
`https://strava-worker.<tvuj-subdomain>.workers.dev`) a klikneš **Připojit
Strava**.

## 6. Build & run

- Vyber simulátor nebo svoje fyzické zařízení a stiskni ⌘R.
- Appka při prvním spuštění vygeneruje dnešní trénink z výchozího profilu.
  V **Profil** si buď vyplň vlastní údaje, nebo klikni **Testovací profil**
  pro načtení stejného test profilu, co byl použitý ve webové verzi (Muž,
  95 kg, Advanced, Front Squat 110 / Clean 100 / Snatch 60 kg).

## Co dělat, když build selže

Toto je první kompilace celého kódu — očekávej drobné chyby. Nejčastější
kandidáti a jak je hlásit:

- **„Cannot find type X in scope"** → nejspíš soubor nebyl přidaný do
  cílového targetu (zaškrtni ho v File Inspector → Target Membership).
- **Chyba v `ExerciseDatabase.swift`** (nejdelší, auto-generovaný soubor) →
  zkopíruj přesné znění chyby a číslo řádku, dohledám a opravím.
- **Cokoliv jiného** → zkopíruj celou chybovou hlášku z Xcode (červený
  text + soubor + řádek) a pošli mi ji, oprava proběhne naslepo podle textu.

## Rozdíly oproti webové verzi

- Persistence je přes JSON soubory v `Application Support` (místo
  `localStorage`), chová se ale identicky (profil, historie, PR log, Strava
  auth, motiv).
- Timer používá systémové zvuky (`AudioServicesPlaySystemSound`) místo
  Web Audio API — zvukové odezvy jsou tedy jednodušší pípnutí, ne generované
  tóny, ale fáze/rozvržení (práce/odpočinek/varování) jsou 1:1 stejné.
- Export sdílecí karty vykresluje stejnou 1080×1350 kartu nativně přes
  SwiftUI `ImageRenderer` (místo Canvas API) a nabízí nativní share sheet
  (uložit do Fotek, AirDrop, zprávy…).
