# Changelog — Pirometria: Symulator Toru Pomiarowego

Format: [Keep a Changelog](https://keepachangelog.com/pl/1.0.0/)  
Repozytorium: https://github.com/Mirek-Socha/SymulatorPirometru

---

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
