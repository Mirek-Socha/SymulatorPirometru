# Changelog — Pirometria: Symulator Toru Pomiarowego

Format: [Keep a Changelog](https://keepachangelog.com/pl/1.0.0/)  
Repozytorium: https://github.com/Mirek-Socha/SymulatorPirometru

---

## [v3.0.0] — 2026-06-11 ✅ bieżąca

### Dodano — trzeci tryb interfejsu: PRO

Architektura trybów rozszerzona z 2 do 3 poziomów:

| Tryb | Zakres |
|---|---|
| **Podstawowy** | uproszczony interfejs (5 kontrolek) |
| **PRO** (nowy) | pełny tor pomiarowy: atmosfera, środowiska, przesłona, detektory, dekompozycja błędów |
| **Ekspert** | PRO + pirometria dwubarwna + modele emisyjności (H-R, wielomian TPRC) |

- Segment 3 przycisków w nagłówku zastępuje pojedynczy przełącznik
- Nowa klasa CSS `.eo` (expert-only); `.xo` oznacza odtąd „PRO i wyżej"
- Higiena stanu: wyjście z trybu Ekspert automatycznie wyłącza tryb dwubarwny
  i przywraca model ε = const (obliczenia nie są sterowane ukrytymi panelami)

### Zmieniono — dokumentacja rozdzielona na PRO i EKSPERT

- Dokumentacja EKSPERT = dokumentacja PRO + sekcje zaawansowane (rozszerzenie)
- Sekcja 8 (Pirometria dwubarwna) widoczna tylko w trybie Ekspert
- Fragmenty modeli H-R i wielomianowego w sekcji 3 — tylko Ekspert
- Spis treści ukrywa wpisy sekcji eksperckich w trybie PRO
- `buildDocsHTML()` (drukowanie/nowe okno) obsługuje 3 tryby

### Refaktoryzacja — fundament pod modele powierzchni

- `expertMode` (boolean) → `uiMode` ('beginner' | 'pro' | 'expert')
- `epsSpectral()` przepisana na **rejestr `EPS_MODELS`** — dodanie nowego
  modelu emisyjności (Fresnel ε(θ), cienkie warstwy, efekt wnękowy) to
  nowy wpis w rejestrze, bez zmian w `compute()`


## [v2.9.0] — 2026-05-28 ✅ bieżąca

### Dodano — interaktywny schemat blokowy

Schemat blokowy toru pomiarowego (tryb Expert) jest teraz interaktywny:
powiązane krzywe na wykresie widmowym reagują na kursor i kliknięcia.

#### Opcja A — hover: podświetlenie krzywych
Najechanie kursorem na blok schematu powoduje, że krzywe powiązane
z tym blokiem pozostają jasne, a pozostałe przygasają do 10% jasności.
Ramka bloku zmienia kolor na cyjanową.

| Blok | Podświetlane krzywe |
|---|---|
| 1 · Obiekt | L_bb CDC, ε·B(λ), ε(λ) H-R/poly |
| 2 · Atmosfera | L_atm (emisja atm.), τ_atm(λ) |
| 3 · Przesłona | τ_win(λ), emisja własna przesłony |
| 4 · Detektor | R(λ), R₂(λ), sygnał L_det |
| 5 · Procesor | sygnał L_det |
| 6 · Wynik | T_ind(λ) |

#### Opcja B — klik: przypięcie podświetlenia
Kliknięcie bloku zamraża podświetlenie — ramka zmienia kolor
na bursztynową (pin). Ponowne kliknięcie odpina i wraca do normalnego widoku.

#### Opcja C — tooltip na strzałce
Najechanie na animowaną strzałkę między blokami pokazuje tooltip z:
- nazwą sygnału fizycznego przepływającego przez to połączenie
- aktualną wartością (np. τ̄ = 0.847)
- numerem bloku źródłowego

### Poprawiono (przeniesione z v2.8.1)
- Tooltip dymka: L_atm, L_win_emit, R₂+T_ratio (warunkowo)
- Schemat: środowisko + T_atm (blok 2), tryb proc. (blok 5), T_ratio (blok 6)
- Legenda: wpisy warunkowe (przesłona, ε(λ), R₂)
- Siatka spektralna N = 700 → 1400 punktów


## [v2.8.0] — 2026-05-27 ✅ bieżąca

### Zmieniono — Refactoring Faza 4: Konfiguracja

Wyodrębniono `const CONFIG` — obiekt z nazwanymi stałymi zamiast magic numbers.

```javascript
const CONFIG = {
  BISECT_ITER:       72,    // iteracje bisekcji — pomiar jednoobarwny
  BISECT_ITER_RATIO: 80,    // iteracje bisekcji — pirometria dwubarwna
  BISECT_T_MIN:      1,     // [K] dolna granica zakresu
  BISECT_T_MAX:      15000, // [K] górna granica zakresu
  RZONE_FRAC:  0.22,  // strefa R(λ)/ε(λ) = górne 22% canvasu
  PL_FRAC:     0.072, // margines lewy adaptacyjny
  PL_MIN:      36,    // [px] min margines lewy
  PL_MAX:      54,    // [px] max margines lewy
  PR: 12, PT: 14, PB: 40,  // [px] pozostałe marginesy
  R_OVERDRAW:  1.05,  // cap normalizacji R(λ)
};
```

9 zamian w kodzie (23 referencje `CONFIG.*`).
Brak zmian funkcjonalnych.


## [v2.7.0] — 2026-05-27 ✅ bieżąca

### Zmieniono — Refactoring Faza 3: Modularyzacja

Kod JavaScript podzielony na 8 wyraźnie oznaczonych modułów logicznych.
Każdy moduł opatrzony separatorem `/* ═══ MODULE N ═══ */` widocznym
w podglądzie kodu i panelu funkcji edytora.

| Moduł | Linia JS | Funkcje | Opis |
|---|---|---|---|
| MODULE 1 — PHYSICS | 4 | planck, atmTau, rhoT, hagenRubens, polyEmissivity, epsSpectral, epsArray | Obliczenia fizyczne, czyste funkcje |
| MODULE 1b — OPTICS | 433 | windowTau, detResp, invertTemp | Przesłona optyczna i detektor |
| MODULE 2 — COMPUTE | 533 | compute, computeSignalForDet, invertTempRatio, computeRatio | Silnik obliczeniowy |
| MODULE 3 — CANVAS | 729 | resizeCanvas, setupCanvas, drawVisibleBar, drawGrid, drawSpec, drawTau | Rysowanie wykresów |
| MODULE 4 — DOCS | 1219 | buildDocsHTML, openDocsWindow, printDocs | Dokumentacja wbudowana |
| MODULE 5 — UI EXPERT | 1383 | setEpsModel, onPolyMat, setRatioMode, setEnv, setScale, … | Handlery Tryb Eksperta |
| MODULE 5b — UI BEGINNER | 1655 | begOnT, begOnTNum, begSetEps, begSetAtm, begSetDet | Handlery Tryb Podstawowy |
| MODULE 6 — UPDATE LOOP | 1741 | scheduleUpdate, doUpdate, updateChain, updateResults | Pętla aktualizacji |
| MODULE 7 — INIT | 1972 | initSliders | Inicjalizacja |
| MODULE 8 — PRESETS & THEME | 2047 | loadPreset, toggleTheme, toggleMode, toggleDocs | Presety i UI globalne |

Brak zmian funkcjonalnych — wyłącznie komentarze/separatory.


## [v2.6.0] — 2026-05-27 ✅ bieżąca

### Zmieniono — Refactoring Faza 2: State Management

Zgrupowanie 8 rozproszonych zmiennych globalnych w jeden obiekt `appState`:

```
// PRZED (8 osobnych let):
let currentEnv      = 'earth';
let currentEpsModel = 'grey';
let hrMaterialKey   = 'fe';
let currentPolyMat  = 'w';
let ratioDet2Key    = 'band_100';
let ratioMode       = false;
let logScale        = false;
let isDark          = true;

// PO (jeden obiekt):
const appState = {
  env:       'earth',
  epsModel:  'grey',
  hrMat:     'fe',
  polyMat:   'w',
  det2Key:   'band_100',
  ratioMode: false,
  logScale:  false,
  isDark:    true,
};
```

Zamieniono 52 referencje w kodzie (np. `currentEnv` → `appState.env`).  
Zostawiono jako osobne: `expertMode`, `hoverFrac`, `lastResult`, `rafId` (zmienne runtime/wewnętrzne).

Brak zmian funkcjonalnych — logika aplikacji bez zmian.



## [v2.5.0] — 2026-05-27 ✅ bieżąca

### Zmieniono — Refactoring Faza 1: konsolidacja kodu

#### Deduplikacja CSS i JavaScript
- **CSS:** 4 bloki `<style>` → 2 bloki (usunięto 2 identyczne duplikaty, 1624 linii)
- **JS:** 6 bloków `<script>` → 1 blok (usunięto 4 puste + 1 duplikat, 2111 linii)
- Rozmiar pliku: 431 KB → 216 KB (**−50%**)
- Liczba linii: 8718 → 4361 (**−50%**)
- Brak zmian funkcjonalnych — wszystkie funkcje zachowane

#### Naprawiono przy okazji — błędy istniejące od v2.4.x
- **Zduplikowany interfejs na dole strony** — zdegenerowany `</html>` +
  wklejony wzór `S₁/S₂` po tagu zamykającym powodował, że przeglądarka
  renderowała drugi pełny dokument jako treść body; usunięto nadmiarowy fragment
- **KaTeX nie renderował formuł** — skrypty `katex.min.js` i `auto-render.min.js`
  (bloki `<script defer src=...>` z pustym body) były traktowane przez skrypt
  konsolidacji jako "puste" i usunięte; przywrócono oba tagi
- **Inicjalizacja KaTeX** — przeniesiono wywołanie `renderMathInElement()` z
  atrybutu `onload` tagu `<script defer>` do `window.addEventListener('load', ...)`
  w głównym bloku JS; gwarantuje wykonanie po wszystkich skryptach `defer`
- **`\text{pe\l{}ny tor}`** — `\l{}` (LaTeX-owe ł) nieobsługiwane przez KaTeX;
  zamieniono na `\text{pelny tor}`

---

## [v2.4.3] — 2026-05-26

### Poprawiono
- **KaTeX blok Wien S₁/S₂** — `\\approx` (podwójny backslash) i `\night` (CR+ight)
  w formule `S₁/S₂ ≈ f(λ₁,λ₂,T)`; teraz renderuje poprawnie

---

## [v2.4.2] — 2026-05-26

### Poprawiono — KaTeX sekcja 8 (formuły Wiena i ΔT_ratio)
- **T_ratio ≈ wzór** — `\x07pprox` (BEL+pprox ze zdegenerowanego `\approx`)
  zastąpione poprawnym `\approx`
- **\right** — `\night` (carriage-return+ight ze zdegenerowanego `\right`)
  zastąpione poprawnym `\right`
- Oba bloki przepisane bezpiecznie — wzory renderują się poprawnie w KaTeX

---

## [v2.4.1] — 2026-05-26

### Poprawiono — pirometria dwubarwna (bugfix po v2.4.0)
- **Puste wyniki T_ratio/ΔT_ratio** — `compute(p)` zamiast `computeRatio(p)`
  w głównej pętli update; `res.ratio` nigdy nie istniało → pole wyników puste.
  Naprawka: `ratioMode ? computeRatio(p) : compute(p)`
- **R₂(λ) niewidoczna na wykresie** — normalizacja `Math.max(...R2.filter())`
  dawała inne skalowanie niż R₁; zamieniono na `R/det2.Rp` → jednolita skala [0,1]
- **KaTeX `unicodeTextInMathMode`** — polskie znaki w formułach sekcji 8
  (`niezależne`, zdegenerowane `\x0b`/`\x0c` z heredoc) → ASCII + poprawne sekwencje
- **`ratioMode is not defined`** — deklaracja `let ratioMode = false`
  zniknęła przy zamianie wersją przez `sed`; przywrócona

---

## [v2.4.0] — 2026-05-26

### Dodano — pirometria dwubarwna (stosunkowa)

**Nowy tryb procesora:** panel „⚙ Tryb procesora" z przełącznikiem
Jednobarwny / Dwubarwny (Ratio Pyrometry), dostępny w Trybie Eksperta.

#### Fizyka
- Mierzy stosunek S(λ₁)/S(λ₂) — eliminuje ε dla ciała szarego
- Inwersja: bisekcja 80 iter. na stosunku sygnałów (zakres 1–15000 K)
- Pełny sygnał S₁ i S₂ wyliczane przez kompletny tor (τ_atm, τ_win, ε(λ,T))
- ΔT_ratio pokazuje błąd gdy ε(λ₁) ≠ ε(λ₂) — ciało selektywne

#### UI
- Selektor λ₂ (drugi detektor) niezależny od λ₁
- 3 przyciski typowych par przemysłowych (Si/InGaAs, InGaAs 1.0/1.6, InSb/window)
- Blok wyników: S₁/S₂, T_ratio [K/°C], ΔT_ratio [K], ε(λ₁)/ε(λ₂)
- Krzywa R₂(λ) na wykresie widmowym — zielona przerywana

#### Dokumentacja (sekcja 8 — nowa, sekcje 8–11 → 9–12)
- Zasada fizyczna, przybliżenie Wiena, błąd przy ciele selektywnym
- Tabela 4 typowych par przemysłowych z zakresami T i zastosowaniami

---

## [v2.3.1] — 2026-05-25

### Poprawiono
- **ε(λ) w tooltipie** — dymek hover pokazuje wartość emisyjności z aktywnego modelu
- **Numer wersji w nagłówku** — etykieta `vX.Y.Z` obok tytułu (IBM Plex Mono)

---

## [v2.3.0] — 2026-05-25

### Dodano — model emisyjności wielomianowy
Trzeci model: **ε(λ) = a₀ + a₁·λ + a₂·λ²** (λ [µm]), dane TPRC Vol.7/8.
9 materiałów: W, Mo, Ti, Fe, Ni, Grafit, SiC, Al₂O₃, Custom.

---

## [v2.2.3] — 2026-05-25

### Poprawiono
- Równanie toru wieloliniowe (KaTeX `aligned`), fix overflow przy druku PDF

---

## [v2.2.2] — 2026-05-25

### Poprawiono — dokumentacja wbudowana
- Tor z τ_win, budżet 3-składowy, siatka N=700, słownik 26 pozycji

---

## [v2.2.1] — 2026-05-25

### Dodano
- Spis treści z linkami kotwicowymi, druk PDF, dokumentacja w nowym oknie

---

## [v2.2.0] — 2026-05-25

### Dodano — model Hagena-Rubensa ε(λ,T)
8 materiałów (Fe, Fe₂O₃, Ti, Ni, Cu, Al, W, Custom), krzywa ε(λ) na wykresie.

---

## [v2.1.1] — 2026-05-24

### Poprawiono
- Fix: RH/CO₂ suwaki, T_atm zakres Wenus, większe wykresy, sticky plots

---

## [v2.1.0] — 2026-05-24

### Dodano — 6 środowisk atmosferycznych
Ziemia, Mars, Wenus, woda, NH₃ MOCVD, CO₂ tech. + Beer-Lambert z P/P₀.

---

## [v2.0.0] — 2026-05-23

### Przełomowe — dwa tryby obsługi
Tryb Podstawowy (studenci) + Tryb Eksperta, nowa nazwa „Pirometria".

---

## [v1.0.0 – v1.10.0] — 2026-05-21–23

- v1.10.0: Zakresy Gaussów ±5σ, adaptacyjna siatka R(λ)
- v1.8.x: Przesłona optyczna z Beer-Lambert i emisją własną
- v1.7.0: Przesłona, budżet 3-składowy
- v1.5.0: Atmosfera, dokumentacja KaTeX, responsywność
- v1.0.0: Pierwsza wersja — Planck, Wien, S-B, widmo CDC

Pełna historia wersji v1.x: dostępna w git log.
