# Domáce Questy

Domáce úlohy za body, body za vreckové, čas na počítači a iné odmeny. Jeden HTML súbor, bez buildu, funguje ako PWA (dá sa „nainštalovať“ na plochu mobilu aj počítača).

## 1. Nasadenie na GitHub Pages (5 minút)

Repo tvojej stránky je `branislavstark-cloud.github.io`. Skopíruj celý obsah tohto priečinka do podpriečinka, napr. `questy/`:

```
branislavstark-cloud.github.io/
└── questy/
    ├── index.html
    ├── manifest.json
    ├── sw.js
    ├── icon-192.png
    └── icon-512.png
```

Commit, push, a o minútu beží na **https://branislavstark-cloud.github.io/questy/**.

Bez ďalšieho nastavovania appka funguje, ale dáta žijú len v prehliadači, kde ju otvoríš (localStorage). Mobil a počítač by mali každý svoj stav. Na zdieľanie treba krok 2.

## 2. Synchronizácia medzi zariadeniami (Firebase, zadarmo, ~10 minút)

Appka nepoužíva Firebase SDK, len REST + streamovanie, takže stačí Realtime Database a pár pravidiel.

1. Choď na https://console.firebase.google.com → **Add project** (názov ľubovoľný, Google Analytics vypni).
2. V ľavom menu **Build → Realtime Database → Create database**. Región vyber `europe-west1` (Belgicko). Režim: **locked mode** (pravidlá nastavíme sami).
3. Záložka **Rules** → nahraď obsah týmto a klikni **Publish**:

```json
{
  "rules": {
    "families": {
      "$key": {
        ".read": "$key.length >= 16",
        ".write": "$key.length >= 16",
        ".validate": "newData.hasChildren(['kids','tasks','rewards']) || data.exists()"
      }
    }
  }
}
```

4. Záložka **Data** → skopíruj URL databázy, vyzerá ako `https://nazov-default-rtdb.europe-west1.firebasedatabase.app`.
5. V appke: **Rodič → PIN 1234 → Nastavenia → Synchronizácia**. Vlož URL, klikni **Vygenerovať** (kľúč rodiny) a **Pripojiť**. Terajší stav sa nahrá do databázy.
6. Na ďalšom zariadení buď naskenuj QR kód z nastavení, alebo otvor odkaz „Pripojiť ďalšie zariadenie“. Stav sa stiahne a odvtedy sa všetko premieta v reálnom čase.

### Bezpečnostný model (povedané na rovinu)

Databáza je prístupná každému, kto pozná kľúč rodiny (20 náhodných znakov, ~10^31 možností, prakticky neuhádnuteľný, ale nie je to prihlásenie). Kto má odkaz alebo QR, má plný prístup vrátane rodičovskej časti (PIN je len bariéra v rozhraní, nie kryptografická ochrana, dieťa so znalosťou DevTools ho obíde). Pre domácnosť to stačí; nedávaj kľúč nikomu mimo rodiny a odkaz neposielaj cez verejné kanály. Ak by unikol: **Odpojiť**, **Vygenerovať** nový kľúč, **Pripojiť**, a nový QR na ostatné zariadenia.

Free tier (Spark) má 1 GB úložiska a 10 GB/mesiac prenosu. Táto appka má dáta v desiatkach kB; limit neohrozíš.

## 3. Inštalácia ako appka

- **Android / Chrome**: menu ⋮ → „Pridať na plochu“ / „Inštalovať aplikáciu“.
- **iPhone / Safari**: Zdieľať → „Pridať na plochu“.
- **Počítač / Chrome, Edge**: ikonka inštalácie v adresnom riadku.

Service worker cacheuje appku, takže sa otvorí aj offline; zmeny sa dosynchronizujú po pripojení.

## Ako to funguje

- **Deti** vidia dnešné questy, ťuknú „hotovo“, a úloha čaká na schválenie (⏳). Môžu to ťuknutím zrušiť.
- **Rodič** (PIN, východiskový 1234, zmeň si ho) schváli alebo zamietne. Po schválení body pribudnú.
- **Obchod**: dieťa si kúpi odmenu, body sa odpočítajú hneď, rodič ju vidí v „Odmeny na vydanie“ a označí ako vydanú (alebo vráti body).
- **Levely**: podľa celkovo získaných bodov (50 → 120 → 210 …). Séria 🔥 = po sebe idúce dni s aspoň jednou schválenou úlohou.
- **Úlohy**: denné (s výberom dní), týždenné, jednorazové; pre všetkých alebo vybrané deti.
- **± body**: rodič môže dať bonus alebo odpočet mimo zoznamu.
- **Záloha**: export/import JSON v nastaveniach.

## Úpravy

Všetko je v `index.html`. Ukážkové deti Ema a Tomáš, úlohy a odmeny sú vo funkcii `seedState()`; v appke ich ale jednoduchšie prepíšeš cez rodičovskú časť. Farby a písma sú v `:root` na začiatku `<style>`.
