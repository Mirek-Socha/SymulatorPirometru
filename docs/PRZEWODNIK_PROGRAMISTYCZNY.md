# Przewodnik Programistyczny — SymulatorPirometru

> **Cel dokumentu:** Materiał szkoleniowy dla programistów — opis architektury, wzorców projektowych i implementacji aplikacji `SymulatorPirometru` (v3.4.0).
> Przeznaczony dla studentów znających podstawy JavaScript, HTML i fizyki promieniowania.
>
> 🇬🇧 **English version:** zostanie dodana po synchronizacji gałęzi [`en/english-translation`](https://github.com/Mirek-Socha/SymulatorPirometru/tree/en/english-translation) z aktualnym stanem `main`.

---

## 1. Panorama projektu

Symulator jest **samodzielną aplikacją webową** zamkniętą w jednym pliku HTML
(`src/symulator_pirometru.html`, ~5 300 linii, ~310 kB).
Nie ma serwera, pliku konfiguracyjnego ani menedżera pakietów — wystarczy otworzyć plik w przeglądarce.
Jedyne zależności zewnętrzne to **KaTeX** (renderowanie wzorów matematycznych) i **Google Fonts**, ładowane z CDN.

Taka architektura — zwana *single-file app* — to świadomy wybór dydaktyczny:
student widzi cały kod w jednym miejscu, bez kompilacji i transpilacji.
Ceną jest brak modularności plikowej; programista zastępuje ją **modułami logicznymi**
wyraźnie oznaczonymi komentarzami w stylu `MODULE N — NAZWA`.

### Stos technologiczny

| Warstwa | Technologia | Uzasadnienie wyboru |
|---|---|---|
| Markup | HTML5 / CSS3 | Brak frameworka — zero narzutu startowego |
| Logika | Vanilla JavaScript (ES2020) | `const/let`, `Array.from`, optional chaining (`?.`), spread `{...obj}` |
| Wzory matematyczne | KaTeX 0.16 | Renderowanie LaTeX w przeglądarce bez MathJax |
| Wykresy | Canvas 2D API | Pełna kontrola pikseli, brak zewnętrznej biblioteki wykresów |
| Eksport wektorowy | SVG generowany ręcznie jako string | Niezależność od biblioteki, lekki wynik |
| Typografia | IBM Plex Mono/Sans + Playfair Display | Styl naukowy/instrumentalny |

---

## 2. Architektura — dekompozycja modułowa

Cały JavaScript zawarty jest w jednym bloku `<script>` (linie ~2528–6122),
podzielonym na **10 logicznych modułów** oddzielonych komentarzami-separatorami (`MODULE N`).
Jest to wzorzec *namespace by convention* — moduły nie są obiektami ani klasami,
ale grupami powiązanych funkcji.

```
MODULE 1   — PHYSICS      Czyste funkcje fizyczne (Planck, Beer-Lambert, ε-modele)
MODULE 1b  — OPTICS       Przesłona optyczna i detektor
MODULE 2   — COMPUTE      Silnik obliczeniowy — integracja sygnału, inwersja T
MODULE 3   — CANVAS       Rysowanie wykresów (Canvas 2D + SVG)
MODULE 4   — DOCS         Dokumentacja wbudowana (KaTeX, HTML)
MODULE 5   — UI EXPERT    Handlery interfejsu — Tryb Eksperta
MODULE 5b  — UI BEGINNER  Handlery interfejsu — Tryb Podstawowy
MODULE 6   — UPDATE LOOP  Pętla aktualizacji i renderowania
MODULE 7   — INIT         Inicjalizacja suwaków i event listenerów
MODULE 8   — PRESETS & THEME  Presety, motyw, tryb, dokumentacja
```

**Kluczowa zasada separacji:** Moduły 1, 1b i 2 są **bezstanowe** — nie czytają ani nie piszą do DOM.
Pobierają dane przez parametr `p` i zwracają wyniki jako czyste obiekty JavaScript.
To sprawia, że można je testować w izolacji (np. w Node.js) bez przeglądarki.

---

## 3. Stan aplikacji — dwa obiekty stanu

Aplikacja zarządza stanem przez **dwa odrębne obiekty globalne**:

```javascript
// Stan parametrów pomiarowych (co mierzy symulator)
const state = {
  Tobj: 1473,          // temperatura obiektu [K]
  eps_real: 0.70,      // rzeczywista emisyjność
  path: 2,             // droga atmosferyczna [m]
  rh: 50,              // wilgotność względna [%]
  co2: 400,            // stężenie CO₂ [ppm]
  Tatm: 20,            // temperatura atmosfery [°C]
  detKey: 'band_065',  // typ detektora (klucz słownika DETS)
  eps_assumed: 0.70,   // emisyjność zakładana przez procesor
  windowKey: 'none',   // typ przesłony optycznej
  windowTemp: 25,      // temperatura przesłony [°C]
  windowThick: 5,      // grubość przesłony [mm]
  Tsurr: 20,           // temperatura otoczenia [°C]
  reflOn: false        // odbicia otoczenia włączone?
};

// Stan UI (jak aplikacja wygląda i zachowuje się)
const appState = {
  env: 'earth',        // środowisko atmosferyczne
  epsModel: 'grey',    // model emisyjności: 'grey'|'hr'|'poly'|'fresnel'
  hrMat: 'fe',         // materiał Hagen-Rubens
  polyMat: 'w',        // materiał wielomianowy (wolfram)
  det2Key: 'band_100', // drugi detektor (pirometria dwubarwna)
  ratioMode: false,    // tryb dwubarwny?
  logScale: false,     // skala logarytmiczna?
  isDark: true,        // motyw ciemny?
  theta: 0,            // kąt obserwacji [°] (model Fresnela)
  fresnelMat:  'fe',   // materiał modelu Fresnela (klucz HR_MATERIALS)
  fresnelMat2: '',     // drugi materiał do porównania na diagramie ('' = brak)
  cavityOn: false,     // efekt wnękowy?
  cavitySS: 0.1        // parametr wnęki s/S
};
```

**Dlaczego dwa obiekty, a nie jeden?**
Separacja odpowiada dwóm domenom: `state` to parametry *fizyczne* (co obliczamy),
`appState` to parametry *interfejsu* (jak to pokazujemy).
Dzięki temu można wywołać `compute(state)` bez zmiany czegokolwiek w UI,
i odwrotnie — zmienić `appState.logScale` bez ponownego obliczania sygnału.

---

## 4. Pętla renderowania — wzorzec RAF debounce

Każda zmiana parametru kończy się wywołaniem `scheduleUpdate()`.
To kluczowa funkcja, która implementuje wzorzec **RAF debounce**
(debouncing przez requestAnimationFrame):

```javascript
let rafId = null;

function scheduleUpdate() {
  if (rafId) return;         // jeśli ramka już zaplanowana — nie duplikuj
  rafId = requestAnimationFrame(() => {
    rafId = null;
    doUpdate();              // wykonaj faktyczną aktualizację
  });
}
```

**Dlaczego to ważne?**
Gdy użytkownik przesuwa suwak, przeglądarka generuje dziesiątki zdarzeń na sekundę.
Bez debouncingu `doUpdate()` wywoływałoby się przy każdym zdarzeniu, przeciążając CPU.
`requestAnimationFrame` gwarantuje co najwyżej jedno wywołanie na klatkę animacji (~60 Hz) —
zbędne eventy są automatycznie porzucane przez warunek `if (rafId) return`.

Funkcja `doUpdate()` realizuje pełny cykl:

```javascript
function doUpdate() {
  setupCanvas();                        // 1. Dopasuj rozmiar canvasa do okna
  const p = {...state};                 // 2. Utwórz KOPIĘ stanu (immutability)
  const res = appState.ratioMode        // 3. Oblicz wyniki
    ? computeRatio(p)
    : compute(p);
  lastResult = res;                     // 4. Zapisz cache (dla eksportu)
  drawSpec(res, p);                     // 5. Narysuj widmo na Canvas
  drawTau(res);                         // 6. Narysuj transmitancję τ(λ)
  if (appState.epsModel === 'fresnel')
    drawPolarPlot();                    // 7. (opcjonalnie) Diagram Fresnela
  if (appState.cavityOn)
    drawCavityDiagram();                // 8. (opcjonalnie) Diagram wnęki
  updateChain(res, p);                  // 9. Odśwież schemat blokowy
  updateResults(res, p);               // 10. Odśwież liczby wyników
  updateLegend(res, p);                // 11. Odśwież legendę
}
```

Zwróć uwagę na `{...state}` — spread operator tworzy **płytką kopię** obiektu stanu.
Funkcja `compute()` nigdy nie modyfikuje oryginału,
co eliminuje trudne do debugowania efekty uboczne.

---

## 5. Moduł fizyki (MODULE 1) — czyste funkcje obliczeniowe

### 5.1 Stałe CODATA 2018 i siatka widmowa

```javascript
const h_  = 6.62607e-34;   // stała Plancka [J·s]
const cL  = 2.99792e8;     // prędkość światła [m/s]
const kB  = 1.38065e-23;   // stała Boltzmanna [J/K]
const c2  = h_ * cL / kB;  // druga stała promieniowania = 1.43878e-2 m·K

const LAM_MIN = 0.10e-6;   // 100 nm
const LAM_MAX = 16e-6;     // 16 µm
const N = 1400;            // liczba punktów siatki
const DL = (LAM_MAX - LAM_MIN) / N;  // krok całkowania [m]

// Prekomputowana tablica długości fal — tworzona raz przy załadowaniu strony
const lams = Array.from({length: N+1}, (_, i) => LAM_MIN + i * DL);
```

Tablica `lams` jest tworzona **raz** przy starcie — optymalizacja,
dzięki której nie alokujemy 1401 elementów przy każdym obliczeniu.
Siatka 1400 punktów to kompromis między dokładnością numeryczną a czasem obliczeń w JavaScript.

### 5.2 Funkcja Plancka

```javascript
function planck(lam, T_K) {
  if (T_K < 1) return 0;
  const x = c2 / (lam * T_K);      // hν/kT
  return (2 * h_ * cL * cL) /
         (Math.pow(lam, 5) * (Math.exp(Math.min(x, 700)) - 1));
}
```

Parametr `Math.min(x, 700)` zapobiega przepełnieniu `Math.exp()` dla krótkich fal
i niskich temperatur (np. λ = 0.5 µm, T = 10 K daje x ≈ 2880 —
wartość po wykładniku przekraczałaby `Number.MAX_VALUE`).
To wzorzec *numerical clamping* — zachowanie poprawności numerycznej kosztem
minimalnej zmiany wartości w ekstremalnych warunkach.

### 5.3 Model transmitancji atmosferycznej (Beer-Lambert)

Funkcja `atmTau(lam_m, pathLen, rh_pct, co2_ppm, env_override)` implementuje
uproszczony model pochłaniania molekularnego oparty na prawie Beer-Lamberta.
Dla każdego pasma absorpcyjnego stosowany jest model Gaussa lub trapezoidalny:

```javascript
// Przykład: fragment pasm H₂O (z 12 pasm wg HITRAN-lite)
const h2o = [
  {c:1.38, bw:0.08, s:0.50},  // centrum [µm], szerokość [µm], siła
  {c:1.87, bw:0.10, s:1.00},
  {c:2.70, bw:0.30, s:2.50},
  // ...
];
// Całkowita transmitancja: iloczyn wkładów składowych
let tau = 1.0;
// dla każdego pasma: tau *= Math.exp(-absorpcja)
```

Transmitancja jest **iloczynem** wkładów składowych — bezpośrednie zastosowanie
prawa Beer-Lamberta w wersji multiplikatywnej:

```
τ = exp(−Σᵢ αᵢ · L)
```

Wkłady H₂O, CO₂, NH₃, SO₂ i Rayleigh obliczane są niezależnie i mnożone.
Ciśnienie środowiska `env.P_atm` skaluje długości optyczne —
modeluje to zmniejszoną absorpcję na Marsie (0.006 atm) lub zwiększoną na Wenus (92 atm).

Specjalny wariant dla **wody ciekłej** używa modelu α(λ) z empirycznej bazy danych —
absorpcja jest tak duża, że po kilku cm w zakresie IR sygnał praktycznie zanika.

### 5.4 Modele emisyjności — wzorzec tablicy strategii (Strategy Pattern)

Cztery modele emisyjności zaimplementowane są jako **rejestr strategii**:

```javascript
const EPS_MODELS = {
  grey: {
    label: 'ε = const (ciało szare)',
    fn: (lam_um, T_K, p) => p.eps_real
  },
  hr: {
    label: 'Hagen-Rubens ε(λ,T)',
    fn: (lam_um, T_K, p) => {
      const mat = HR_MATERIALS[appState.hrMat] || HR_MATERIALS.fe;
      const rho  = rhoT(mat, T_K);        // ρ(T) = ρ₀[1+α(T−293)]
      return hagenRubens(lam_um, rho);    // ε ≈ 0.365√(ρ/λ) − ...
    }
  },
  poly: {
    label: 'Wielomianowy TPRC',
    fn: (lam_um, T_K, p) => polyEmissivity(lam_um,
          POLY_MATERIALS[appState.polyMat] || POLY_MATERIALS.w)
  },
  fresnel: {
    label: 'Fresnel ε(θ,λ,T)',
    fn: (lam_um, T_K, p) => fresnelEmissivity(lam_um, T_K,
          appState.theta, appState.fresnelMat)
  }
};

function epsSpectral(lam_m, T_K, p) {
  const model = EPS_MODELS[appState.epsModel] || EPS_MODELS.grey;
  const eps0 = model.fn(lam_m * 1e6, T_K, p);
  if (appState.cavityOn)
    return cavityBoost(eps0, appState.cavitySS);  // efekt wnękowy Gouffé
  return eps0;
}
```

**Wzorzec Strategy** polega na tym, że wywołujący kod (`epsSpectral`) nie wie,
który konkretny algorytm jest uruchamiany — dowiaduje się tego przez klucz `appState.epsModel`.
Dodanie nowego modelu wymaga tylko wpisania go do słownika `EPS_MODELS`,
bez modyfikacji kodu silnika obliczeniowego.

### 5.5 Model Hagena-Rubensa i Fresnela

Emisyjność metali wynika z ich przewodności elektrycznej.
Wzór Hagena-Rubensa (1903) łączy oporność elektryczną z emisyjnością w podczerwieni:

```
ε(λ, T) ≈ 0.365·√(ρ/λ) − 0.0667·(ρ/λ) + 0.006·(ρ/λ)^(3/2)
```

gdzie ρ to oporność [µΩ·cm], λ w [µm].
Oporność zmienia się z temperaturą liniowo: `ρ(T) = ρ₀·[1 + α·(T − 293)]`.

Model Fresnela rozszerza to o **emisyjność kierunkową** ε(θ).
Współczynnik załamania zespolonego `ñ = n − ik` wyznaczany jest z modelu Drudego IR:

```
n ≈ k ≈ √(D_A · λ / ρ)
```

gdzie stała `D_A = 2997.91` (jednostki µm·µΩcm, wynikająca z prawa Wiedemanna-Franza).
Następnie stosowane są pełne równania Fresnela dla polaryzacji s i p,
uwzględniające kąt padania θ.

---

## 6. Silnik obliczeniowy (MODULE 2) — przepływ danych

Funkcja `compute(p)` jest **sercem symulatora**.
Realizuje całkowanie numeryczne metodą prostokątów po siatce 1401 punktów widmowych:

```
Dla każdego λᵢ w [0.10, 16] µm:
  1. L_bb(λ,T_obj)                                    ← prawo Plancka
  2. ε(λ,T) · L_bb                                    → L_emit (emisja własna)
  3. (1−ε) · L_bb(T_surr)                             → L_refl (odbicia otoczenia)
  4. τ_atm(λ)·L_obj + (1−τ_atm)·L_bb(T_atm)         → propagacja przez atmosferę
  5. τ_win(λ)·L_atm + (1−τ_win)·L_bb(T_win)          → przez przesłonę
  6. R(λ) · L_det                                     → odpowiedź detektora
  7. signal += R(λ) · L_det · Δλ                      ← całkowanie prostokątów

Po pętli:
  8. T_ind = invertTemp(signal, det, ε_zał)           ← inwersja bisekcją
  9. Dekompozycja błędu (4 składowe)
```

Równanie sygnału na wejściu detektora:

```
L_det(λ) = τ_win(λ) · [ τ_atm(λ) · L_obj(λ) + (1−τ_atm) · L_bb(λ, T_atm) ]
          + (1−τ_win(λ)) · L_bb(λ, T_win)
```

### 6.1 Inwersja temperatury — algorytm bisekcji

Procesor pirometru zna tylko sygnał elektryczny S, nie zna temperatury.
Musi rozwiązać równanie odwrotne: znaleźć `T_ind` takie,
że model daje sygnał równy zmierzonemu.
Używana jest **metoda bisekcji** (bisection search):

```javascript
function invertTemp(target, det, eps_a) {
  if (target <= 0) return 1;
  let lo = CONFIG.BISECT_T_MIN;  // 1 K
  let hi = CONFIG.BISECT_T_MAX;  // 15 000 K

  for (let it = 0; it < CONFIG.BISECT_ITER; it++) {  // 72 iteracje
    const Tm = (lo + hi) / 2;
    let s = 0;
    for (let i = 0; i <= N; i++)
      s += detResp(lams[i], det) * eps_a * planck(lams[i], Tm) * DL;
    if (s < target) lo = Tm; else hi = Tm;
  }
  return (lo + hi) / 2;
}
```

72 iteracje bisekcji na przedziale [1, 15 000] K dają zbieżność ΔT < 0.01 K
(zakres / 2⁷² ≈ 3·10⁻¹⁸ K). Wewnętrzna pętla iteruje po N=1400 punktach —
łączna złożoność jednej inwersji to ~100 000 operacji zmiennoprzecinkowych,
co w nowoczesnej przeglądarce zajmuje < 1 ms.

### 6.2 Dekompozycja budżetu błędów

Funkcja `compute()` oblicza cztery składowe błędu przez **teleskopową różnicę**
sygnałów przy stopniowym włączaniu efektów:

```
sig_eps   = ∫ R(λ) · ε(λ) · L_bb(λ, T_obj) dλ     ← tylko emisyjność
sig_refl  = sig_eps  + wkład odbić otoczenia        ← + odbicia
sig_atm   = sig_refl + wkład atmosfery              ← + atmosfera
signal    = sig_atm  + wkład przesłony              ← + przesłona (= pełny)

ΔT_ε    = T(sig_eps)  − T_obj        ← błąd emisyjności
ΔT_refl = T(sig_refl) − T(sig_eps)   ← przyrost z odbić
ΔT_atm  = T(sig_atm)  − T(sig_refl)  ← przyrost z atmosfery
ΔT_win  = T(signal)   − T(sig_atm)   ← przyrost z przesłony
```

Suma `ΔT = ΔT_ε + ΔT_refl + ΔT_atm + ΔT_win` jest łącznym błędem wskazania.

---

## 7. Rysowanie wykresów (MODULE 3) — Canvas 2D

### 7.1 Architektura renderowania

Aplikacja używa **HTML5 Canvas 2D** zamiast zewnętrznej biblioteki (Chart.js, D3.js).
Daje to pełną kontrolę nad renderowaniem, ale wymaga ręcznego obliczania współrzędnych pikseli.

Mapowanie długości fali → współrzędna X:

```javascript
const lx = appState.logScale
  ? u => pL + W2 * Math.log(u / lamMin) / Math.log(lamMax / lamMin)  // log
  : u => pL + W2 * (u - lamMin) / (lamMax - lamMin);                 // liniowe
```

Funkcja `lx()` jest zdefiniowana lokalnie wewnątrz funkcji rysującej i zamknięta w *closure* —
pozwala przekazywać transformację bez globalnego stanu.

### 7.2 Obsługa DPI (Retina / HiDPI)

Canvas na ekranach o wysokiej gęstości pikseli wymaga jawnego skalowania:

```javascript
function resizeCanvas(c) {
  const dpr = window.devicePixelRatio || 1;  // 2.0 na Retina, 1.0 na zwykłym
  const w = c.offsetWidth || 600;
  const h = c.offsetHeight || 200;
  const needW = Math.round(w * dpr);
  const needH = Math.round(h * dpr);
  if (c.width !== needW || c.height !== needH) {
    c.width  = needW;
    c.height = needH;
    c.getContext('2d').scale(dpr, dpr);  // skaluj kontekst
  }
}
```

**Mechanizm:** Element `<canvas>` ma dwa rozmiary — *pikselowy* (atrybuty `width`/`height`)
i *CSS* (przez styl). Na ekranie Retina CSS 1px = 2 piksele fizyczne.
Bez skalowania kontekstu rysunek byłby rozmyty.
Po `ctx.scale(dpr, dpr)` wszystkie współrzędne logiczne są automatycznie przeliczane.

### 7.3 Interaktywny tooltip — hover na Canvas

Canvas nie ma wbudowanego systemu zdarzeń dla poszczególnych elementów.
Tooltip implementowany jest ręcznie przez nasłuchiwanie `mousemove`
i odwrotne mapowanie pozycji myszy na długość fali:

```javascript
specCanvas.addEventListener('mousemove', e => {
  const rect = specCanvas.getBoundingClientRect();
  const mx = e.clientX - rect.left;      // pozycja myszy w CSS px
  const frac = (mx - pL) / W2;           // frakcja [0..1] w obszarze wykresu

  // Odwrotne mapowanie frac → λ (log lub liniowe)
  const lamHov = appState.logScale
    ? lamMin * Math.pow(lamMax / lamMin, frac)
    : lamMin + (lamMax - lamMin) * frac;

  const idx = Math.round(frac * N);       // indeks w tablicy wyników
  // ... wypełnienie HTML tooltipa wartościami res.Lbb[idx], res.tau[idx] itd.
});
```

Wynik `lastResult` (cache z ostatniego `doUpdate()`) jest odczytywany synchronicznie —
nie ma potrzeby ponownego obliczania przy każdym ruchu myszy.

---

## 8. System motywów CSS (dark/light)

Motywy zaimplementowane są przez **CSS Custom Properties** i klasy na `<body>`:

```css
.theme-dark  { --bg: #07070a; --amber: #f59e0b; --cyan: #06b6d4; ... }
.theme-light { --bg: #f5f2ea; --amber: #b45309; --cyan: #0e7490; ... }
```

Przełączenie motywu to pojedyncza operacja DOM + aktualizacja Canvas:

```javascript
function toggleTheme() {
  appState.isDark = !appState.isDark;
  document.body.classList.toggle('theme-dark', appState.isDark);
  document.body.classList.toggle('theme-light', !appState.isDark);
  // Safari fix: zaktualizuj color-scheme na :root
  document.documentElement.style.colorScheme = appState.isDark ? 'dark' : 'light';
  scheduleUpdate();  // przerysuj Canvas z nowymi kolorami
}
```

Canvas odczytuje zmienne CSS przez helper `tv(name)`:

```javascript
function tv(name) {
  return getComputedStyle(document.body).getPropertyValue(name).trim();
}
// Użycie: ctx.strokeStyle = tv('--amber');
```

Dzięki temu **jeden kod rysowania** obsługuje oba motywy — zmiana klasy CSS
automatycznie zmienia wszystkie kolory przy następnym przerysowaniu.

---

## 9. Trzy tryby UI — wzorzec progressive disclosure

Interfejs ma trzy poziomy złożoności: **Podstawowy → PRO → Ekspert**.
Jest to wzorzec *progressive disclosure* — użytkownik widzi tylko to,
co jest mu potrzebne na danym poziomie wiedzy.

Implementacja opiera się na klasach CSS na `<body>`:

```javascript
function setUiMode(mode) {
  document.body.classList.remove('beginner', 'pro', 'expert');
  document.body.classList.add(mode);

  // Zabezpieczenie: ukryte funkcje nie mogą sterować obliczeniami
  if (mode !== 'expert') {
    if (appState.ratioMode) setRatioMode(false);
    if (appState.epsModel !== 'grey') setEpsModel('grey');
  }
  if (mode === 'beginner') {
    if (state.reflOn) setRefl(false);
  }
  requestAnimationFrame(() => {
    resizeCanvas(specCanvas);
    resizeCanvas(tauCanvas);
    scheduleUpdate();
  });
}
```

W CSS panele dla trybów wyższych są ukryte w niższym trybie:

```css
.expert-only { display: none; }
body.expert .expert-only { display: block; }
```

**Wzorzec bezpieczeństwa stanu:** Gdy użytkownik cofa się z Expert do PRO,
kod jawnie wyłącza zaawansowane funkcje (`ratioMode`, modele ε).
Zapobiega to sytuacji, w której ukryty interfejs nadal wpływa na obliczenia.

---

## 10. System presetów — wzorzec Data-Driven Configuration

Presety to słowniki konfiguracyjne zawierające pełny stan `state`.
Dodanie nowego presetu nie wymaga zmiany żadnego kodu logicznego:

```javascript
const PRESETS = {
  ideal:      {Tobj:1473, eps_real:1.00, path:0,  rh:0,  detKey:'band_065', ...},
  emissivity: {Tobj:1473, eps_real:0.15, path:0,  rh:0,  detKey:'band_065', ...},
  atmosphere: {Tobj:1073, eps_real:0.85, path:25, rh:80, detKey:'window_35', ...},
  mars_basic: {Tobj:523,  eps_real:0.90, path:10, Tatm:-55, ...},
  venus_basic:{Tobj:738,  eps_real:0.95, path:1,  Tatm:465, ...},
};

function loadPreset(name) {
  const p = PRESETS[name] || BEG_PRESETS[name];
  if (!p) return;
  Object.assign(state, p);   // nadpisz stan prestem (selektywne nadpisanie)
  initSliders();             // zsynchronizuj UI ze stanem
  scheduleUpdate();
}
```

`Object.assign(state, p)` kopiuje tylko właściwości z presetu,
nie dotykając kluczy, których preset nie zawiera.

---

## 11. Eksport danych — trzy formaty

### 11.1 Eksport CSV

Funkcja `exportCSV()` generuje plik z trzema sekcjami: Nastawy, Wyniki, Widmo.
Tablica `rows` budowana jest programowo, a na końcu serializowana:

```javascript
const csv = rows.map(r =>
  r.map(c => String(c).includes(',')
    ? '"' + String(c).replace(/"/g, '""') + '"'  // escape cudzysłowów (RFC 4180)
    : c
  ).join(',')
).join('\r\n');

// BOM UTF-8: '\uFEFF' gwarantuje poprawne otwarcie w Excelu
downloadBlob('\uFEFF' + csv, fname, 'text/csv;charset=utf-8');
```

Tabela widma zawiera max ~700 wierszy (co `floor(N/700)` punkt) —
kompromis między rozdzielczością a rozmiarem pliku.

### 11.2 Eksport PNG — kompozyt offscreen canvas

PNG realizowany jest przez **offscreen canvas** — niewidoczny element canvas
tworzony w pamięci, na który naklejane są inne canvas plus ręcznie rysowana legenda:

```javascript
const off = document.createElement('canvas');
off.width = W;  off.height = H;
const ctx = off.getContext('2d');
ctx.drawImage(specC, 0, 0, Ws, Hs, 0, 0, W, Hs);    // 1. widmo
ctx.drawImage(tauC,  0, 0, Wt, Ht, 0, yTau, W, Ht); // 2. τ(λ)
// ... legenda i stopka rysowane programowo
off.toBlob(blob => downloadBlob(blob, fname, 'image/png'), 'image/png', 0.95);
```

### 11.3 Eksport SVG — generowanie wektora jako string

SVG generowany jest przez **konkatenację stringów** zamiast API DOM —
pełna kontrola nad każdym atrybutem bez narzutu tworzenia obiektów:

```javascript
const els = [], defs = [];

// Gradient UV-VIS jako liniowy gradient SVG
defs.push(`<linearGradient id="gradVIS" ...><stop .../></linearGradient>`);

// Krzywa widmowa jako SVG <path>
const d = arr.map((v,i) =>
  (i===0 ? `M${lx(lam[i])},${ly(v)}` : `L${lx(lam[i])},${ly(v)}`)
).join(' ');
els.push(`<path d="${d}" stroke="${color}" stroke-width="2"/>`);

// Złożenie dokumentu SVG
const svg = [
  `<?xml version="1.0" encoding="UTF-8"?>`,
  `<svg xmlns="http://www.w3.org/2000/svg" width="${W}" height="${H}">`,
  `  <defs>`, ...defs, `  </defs>`,
  ...els,
  `</svg>`
].join('\n');
```

---

## 12. Pirometria dwubarwna (ratio pyrometry)

Tryb ratio implementuje **pirometrię stosunkową** — pomiar temperatury
bez znajomości emisyjności przez iloraz sygnałów z dwóch pasm spektralnych:

```javascript
function computeRatio(p) {
  const base = compute(p);   // standardowy wynik jednobarwny (λ₁)

  // Oblicz sygnał S₂ dla drugiego detektora (ten sam tor fizyczny)
  const sig2 = computeSignalForDet(det2, p, env, Tobj_K, Tatm_K, Twin_K, thick);
  const ratio_meas = sig2 > 0 ? sig1 / sig2 : null;  // S₁/S₂

  // Inwersja stosunkowa: znajdź T takie że S₁(T)/S₂(T) = ratio_meas
  // Uproszczenie: τ=1, ε=1 (ε wyskakuje dla ciała szarego)
  const T_ratio_K = invertTempRatio(ratio_meas, det1, det2, ...);

  return { ...base, ratio: { T_ratio_K, T_ratio_C, ... } };
}
```

**Klucz fizyczny:** Dla ciała szarego ε(λ₁) = ε(λ₂) = ε,
więc iloraz S₁/S₂ = f(T) nie zależy od ε.
Inwersja stosunkowa szuka T dla uproszczonego modelu z τ=1, ε=1.

---

## 13. Wzorce projektowe — zestawienie

| Wzorzec | Gdzie zastosowany | Zysk |
|---|---|---|
| **Strategy** | `EPS_MODELS` — rejestr modeli emisyjności | Dodanie nowego modelu = 1 wpis w słowniku |
| **Observer / Event-driven** | Każdy suwak/checkbox → `scheduleUpdate()` | Luźne powiązanie UI ↔ logika |
| **RAF Debounce** | `scheduleUpdate()` z `requestAnimationFrame` | Płynność UI, brak przeciążenia CPU |
| **Data-Driven Config** | `PRESETS`, `DETS`, `ENVS`, `WINDOWS` | Rozszerzanie przez dane, nie kod |
| **Immutable parameter passing** | `{...state}` w `doUpdate()` | Brak efektów ubocznych w obliczeniach |
| **Progressive Disclosure** | Tryby Podstawowy/PRO/Ekspert | UX dopasowany do poziomu użytkownika |
| **Single Source of Truth** | `state` + `appState` | Jeden punkt mutacji stanu |
| **Offscreen Canvas** | Eksport PNG | Komponowanie obrazu bez widocznego artefaktu |
| **Template Literal SVG** | `exportSVG()` | Wektorowy eksport bez biblioteki |
| **Namespace by Convention** | Komentarze `MODULE N` | Czytelność bez kosztu modularności ES6 |

---

## 14. Wskazówki praktyczne dla programistów

### Jak dodać nowy model emisyjności?

1. Napisz funkcję `fn(lam_um, T_K, p)` zwracającą wartość ε ∈ [0,1].
2. Dodaj wpis do słownika `EPS_MODELS`.
3. Dodaj opcję `<option>` do selektora HTML (tryb Expert).
4. Ewentualnie dodaj panel parametrów (analogicznie do panelu `#epsPanel_hr`).

### Jak dodać nowe środowisko atmosferyczne?

Wpisz nowy klucz do słownika `ENVS` z polami:
`P_atm`, `T_atm`, `L_max`, `L_def`, `label`, `note`, `det_hint`
i opcjonalnymi składnikami gazowymi (`h2o_pure`, `nh3_ppm`, `so2_ppm`).
Przycisk środowiska jest generowany automatycznie przez inicjalizację.

### Jak debugować obliczenia z konsoli przeglądarki?

Otwórz konsolę (F12) i wpisz:

```javascript
compute({...state});           // pełny wynik obliczeń dla aktualnego stanu
lastResult.dT;                 // aktualny błąd temperatury [K]
lastResult.dT_eps;             // składowa: błąd emisyjności
planck(1e-6, 1273);            // Planck dla λ=1µm, T=1000°C
atmTau(3.9e-6, 10, 50, 400);  // τ w λ=3.9µm, L=10m, RH=50%, CO₂=400ppm
appState;                      // aktualny stan UI
state;                         // aktualne parametry pomiarowe
```

### Wzorzec inicjalizacji

Cała inicjalizacja jest opóźniona do zdarzenia `load` — gwarantuje,
że DOM jest w pełni zbudowany przed próbą odczytu elementów przez JavaScript:

```javascript
window.addEventListener('load', () => {
  setupCanvas();
  initSliders();
  setEnv('earth');
  onPolyMat('w');
  scheduleUpdate();
  initChainInteraction();
  drawPolarPlot();
  drawCavityDiagram();
  initPolarHover();
  initCollapsiblePanels();
  // KaTeX renderuje się przez onload="renderDocsKatex()" na CDN script tag
});
```

Kolejność wywołań ma znaczenie: `initSliders()` musi być przed `scheduleUpdate()`,
bo slider ustawia wartości `state`, które są potem odczytywane przez `compute()`.

---

---

## 15. Zmiany wersji

Pełna historia zmian dostępna w [`CHANGELOG.md`](../CHANGELOG.md).
Kluczowe wersje z perspektywy architektonicznej:

| Wersja | Zmiana architektoniczna |
|---|---|
| v3.4.0 | `exportCSV()`, `exportSpectrum()`, `exportSVG()` — trzy formaty eksportu |
| v3.3.0 | `cavityBoost()` w `epsSpectral()`, `initCollapsiblePanels()`, hover na polarCanvas |
| v3.2.0 | `EPS_MODELS.fresnel` z `fresnelNK()` + `fresnelEps()`, `drawPolarPlot()` |
| v3.2.1 | Stała `DRUDE_A=2997.91` (naprawa jednostek), `Tatm_K` w dekompozycji |
| v3.0.0 | 3 tryby UI (`.bo`/`.xo`/`.eo`), `EPS_MODELS` rejestr, higiena stanu przy zmianie trybu |
| v2.4.0 | `computeRatio()`, `invertTempRatio()` — pirometria dwubarwna |
| v2.3.0 | `POLY_MATERIALS` + `polyEmissivity()` — model wielomianowy TPRC |
| v2.2.0 | `HR_MATERIALS` + `hagenRubens()` + `rhoT()` — model Hagena-Rubensa |
| v2.8.0 | Obiekt `CONFIG` — wszystkie stałe numeryczne w jednym miejscu |

*Mirosław Socha · Katedra Metrologii i Elektroniki · WEAIiIB · AGH Kraków*
