# Changelog — Pirometria: Symulator Toru Pomiarowego

Format: [Keep a Changelog](https://keepachangelog.com/pl/1.0.0/)  
Repozytorium: https://github.com/Mirek-Socha/SymulatorPirometru

---

## [v2.4.1] — 2026-05-26 ✅ bieżąca

### Poprawiono — pirometria dwubarwna (bugfix po v2.4.0)
- **Puste wyniki T_ratio/ΔT_ratio** — `compute(p)` zamiast `computeRatio(p)`
  w głównej pętli update; `res.ratio` nigdy nie istniało → pole wyników puste.
  Naprawka: `ratioMode ? computeRatio(p) : compute(p)`
- **R₂(λ) niewidoczna na wykresie** — normalizacja `Math.max(...R2.filter())`
  dawała inne skalowanie niż R₁; zamieniono na `R/det2.Rp` → jednolita skala [0,1]
- **KaTeX `unicodeTextInMathMode`** — polskie znaki w formułach sekcji 8
  (`niezależne`, `arepsilon` z uszkodzonymi `\x0b`/`\x0c` z heredoc)
  → zamienione na ASCII + poprawne sekwencje LaTeX
- **`ratioMode is not defined`** — deklaracja `let ratioMode = false`
  zniknęła przy zamianie wersją przez `sed`; przywrócona

---

## [v2.4.0] — 2026-05-26 ✅ bieżąca

### Dodano — pirometria dwubarwna (stosunkowa)

**Nowy tryb procesora:** panel „⚙ Tryb procesora" z przełącznikiem
Jednobarwny / Dwubarwny (Ratio Pyrometry), dostępny w Trybie Eksperta.

#### Fizyka
- Mierzy stosunek S(λ₁)/S(λ₂) — eliminuje ε dla ciała szarego
- Inwersja: bisekcja 80 iter. na stosunku sygnałów (zakres 1–15000 K)
- Pełny sygnał S₁ i S₂ wyliczane przez kompletny tor (τ_atm, τ_win, ε(λ,T))
- Procesor ratio zakłada τ=1, ε=1 do inwersji (ε skraca się przy szarym)
- ΔT_ratio pokazuje błąd gdy ε(λ₁) ≠ ε(λ₂) — ciało selektywne

#### UI
- Selektor λ₂ (drugi detektor) niezależny od λ₁
- 3 przyciski typowych par przemysłowych (Si/InGaAs, InGaAs 1.0/1.6, InSb/window)
- Blok wyników: S₁/S₂, T_ratio [K/°C], ΔT_ratio [K], ε(λ₁)/ε(λ₂)
- Krzywa R₂(λ) na wykresie widmowym — zielona przerywana
- Legenda: wpis R₂(λ) det.2
- Schemat blokowy: blok Procesor pokazuje tryb i nazwę λ₂

#### Dokumentacja (sekcja 8 — nowa, sekcje 8–11 → 9–12)
- Zasada fizyczna: całki S₁, S₂ i ich stosunek
- Przybliżenie Wiena dla wąskopasmowych detektorów (wzór analityczny)
- Błąd przy ciele selektywnym: wzór ΔT_ratio = f(ε₁/ε₂)
- Tabela 4 typowych par przemysłowych z zakresami T i zastosowaniami
- Warunki stosowalności i ograniczenia (lista + warn-box)
- Info-box: eksperyment z Al₂O₃ (model wielomianowy) vs grafit
- Spis treści zaktualizowany (12 sekcji)

---

## [v2.3.1] — 2026-05-25 ✅ bieżąca

### Poprawiono
- **ε(λ) w tooltipie** — dymek hover pokazuje teraz wartość emisyjności
  z aktywnego modelu (szary/H-R/wielomianowy) z etykietą nazwy modelu/materiału
- **Numer wersji w nagłówku** — drobna etykieta `v2.3.x` obok tytułu
  (IBM Plex Mono, wyszarzona, wyrównana do dolnej krawędzi tytułu)

---

## [v2.3.0] — 2026-05-25 ✅ bieżąca

### Dodano — model emisyjności wielomianowy

Trzeci model emisyjności: **ε(λ) = a₀ + a₁·λ + a₂·λ²** (λ [µm])
oparty na danych literaturowych z Touloukian & DeWitt TPRC Vol.7/8.

#### Panel UI „∿ Wielomianowy"
- Selektor 9 materiałów z etykietami źródeł
- Trzy pola numeryczne: a₀, a₁ [µm⁻¹], a₂ [µm⁻²]
- Pola są zablokowane (readonly) dla presetów; edytowalne dla trybu Custom
- Nota literaturowa i zakres ważności (λ i T) dla każdego materiału

#### Materiały (dopasowania do danych tabelarycznych):
| Materiał | Źródło | Zachowanie ε(λ) |
|---|---|---|
| Wolfram (W) | De Vos (1954), CRC | maleje z λ, minimum ~1.2 µm |
| Molibden (Mo) | TPRC Vol.7, Cagran 2005 | podobny do W |
| Tytan (Ti) | TPRC Vol.7 | szybko spada w IR |
| Żelazo (Fe) | Ward (1956), TPRC Vol.7 | monotoniczny spadek |
| Nikiel (Ni) | Ward (1956), TPRC Vol.7 | podobny do Fe |
| Grafit (C) | TPRC Vol.8 | prawie płaski ~0.85 |
| SiC | NIST, TPRC Vol.8 | wysoki i płaski ~0.92 |
| Al₂O₃ | TPRC Vol.8 | ROŚNIE z λ (ceramika!) |
| Custom | — | pola a₀, a₁, a₂ edytowalne |

#### Dydaktyczny efekt Al₂O₃:
ε(1 µm) ≈ 0.28 vs ε(8 µm) ≈ 0.88 — pirometr InGaAs drastycznie zaniży T,
natomiast MCT 8-14 µm może dać poprawny wynik. Odwrócone zachowanie względem metali.

#### Implementacja:
- `POLY_MATERIALS` — dict 9 materiałów z a0,a1,a2,lam_range,T_range,source,note
- `polyEmissivity(lam_um, mat)` — oblicza ε, clamp [0.01,0.99]
- `epsSpectral()` — rozszerzone o gałąź 'poly'
- `onPolyMat(key)`, `onPolyCoeff()` — handlery UI
- Krzywa ε(λ) na wykresie — bursztynowa przerywana (jak H-R)
- Suwak ε_real zablokowany w trybie Poly

#### Dokumentacja:
- Sekcja 3 Expert: nowa podsekcja „Model wielomianowy" z tabelą 8 materiałów,
  wzorem, info-boxem Al₂O₃, ostrzeżeniem o ograniczeniach

---

## [v2.2.3] — 2026-05-25 ✅ bieżąca

### Poprawiono
- **Równanie toru (sekcja 1)** — rozbite na 3 wyrównane linie (`aligned` KaTeX);
  każdy składnik fizyczny (obiekt, atmosfera, przesłona) w osobnej linii z podpisem;
  nie wychodzi poza kontener ani w panelu, ani w oknie drukowania
- **KaTeX font-size**: 1rem → 0.93rem — lepsza czytelność przy wąskich kontenerach
- **`overflow-x:auto` zachowane w `@media print`** — długie wzory scrollują się
  zamiast ucinać przy druku do PDF

---

## [v2.2.2] — 2026-05-25 ✅ bieżąca

### Poprawiono — dokumentacja wbudowana

**Sekcja 1 — Model toru pomiarowego:**
- Pełne równanie L_det(λ) uwzględnia teraz τ_win (była pomijana)
- ε(λ,T) zamiast ε (zaznaczono że to funkcja, nie stała)
- Tabela 6 bloków toru z opisem modelu i sterowników każdego bloku

**Sekcja 8 — Całkowanie sygnału i inwersja temperatury:**
- Grid: N=480, λ 0.4–16 µm → N=700, λ 0.10–16 µm (zgodnie z kodem)
- Bisekcja: 64 iter., T ∈ [200,6500] K → 72 iter., T ∈ [1,15000] K
- Dekompozycja błędu: 2 składowe → **3 składowe** (ΔT_ε + ΔT_atm + ΔT_win)
- Wzory z pełnymi warunkami dla każdej składowej
- Tabela znaczenia i typowych znaków składowych
- Info-box: wyjaśnienie dlaczego inwersja używa ε_zał=const nawet w modelu H-R
- Wzór sumy całkowania dyskretnej (metoda prostokątów)

**Sekcja 9 — Słownik symboli:**
- Dodano: ε(λ,T), ρ₀, α (H-R), τ_win(λ), T_win, ΔT_ε, ΔT_atm, ΔT_win
- Dodano: N=700 (siatka), P/P₀ (Beer-Lambert)
- Uszczegółowiono opisy istniejących symboli

**Sekcja 10 — Informacje:**
- Wersja zaktualizowana do v2.2.2

---

## [v2.2.1] — 2026-05-25 ✅ bieżąca

### Dodano — spis treści dokumentacji
- **Spis treści Eksperta** (2 kolumny) — 11 pozycji z podtytułami i linkami
  kotwicowymi `#doc-s1` … `#doc-s11`; pojawia się na początku panelu dokumentacji
- **Spis treści Beginner** — 5 pozycji z linkami kotwicowymi `#beg-s1` … `#beg-s5`
- CSS `.toc`: cyjanowa lewa ramka, layout 2-kolumnowy (desktop) / 1-kol. (telefon)
- Kotwice `id` dodane do wszystkich sekcji `<h2>` w obu trybach

### Poprawiono — numeracja sekcji Beginner
- Sekcja „3b. Środowiska" → poprawnie **4. Środowiska**
- Sekcja „4. Jak czytać wykresy?" → **5. Jak czytać wykresy?**

### Poprawiono — komentarze sekcji w HTML
- Stare komentarze `<!-- 6. DETEKTOR -->`, `<!-- 7. SYGNAŁ -->`, `<!-- 9. AUTOR -->`
  doprowadzone do zgodności z numeracją sekcji dokumentacji

---

## [v2.2.0] — 2026-05-25 ✅ bieżąca

### Dodano — model emisyjności spektralnej Hagena-Rubensa

Dotychczasowy model zakładał ciało szare: ε(λ,T) = const.
Nowy model Hagena-Rubensa opisuje rzeczywistą emisyjność metali jako
funkcję długości fali i temperatury, wyprowadzoną z teorii elektronowej.

#### Model fizyczny
Wzór Hagena-Rubensa (dla λ >> głębokość wnikania, metale):

  ε(λ,T) ≈ 0.365·√(ρ(T)/λ) − 0.0667·(ρ(T)/λ)

gdzie ρ(T) = ρ₀·[1 + α·(T − 293 K)] [µΩ·cm], λ [µm].

Ref: Hagen & Rubens (1903); Seifter et al. (2003); Arnold, Appl.Opt. 23 (1984).

Kluczowe właściwości fizyczne modelu H-R:
- ε maleje z długością fali (∝ 1/√λ dla dużych λ) — odwrotnie niż intuicja
- ε rośnie z temperaturą (ρ(T) rośnie liniowo)
- Efekt dydaktyczny: pirometr Si 0.65 µm i MCT 8-14 µm widzą inną ε
  tego samego obiektu → różne ΔT przy tym samym ε_zał

#### Nowy panel UI „🌑 Model emisyjności" (Tryb Eksperta)
- Przełącznik: [━ Szara ε=const] / [╲ Hagen-Rubens ε(λ,T)]
- Selektor 8 materiałów (Fe, Fe₂O₃, Ti, Ni, Cu, Al, W, Custom)
  z tabelarycznymi danymi ρ₀ i α (źródło: CRC Handbook, Kaye & Laby)
- Suwaki ρ₀ [µΩ·cm] i α [1/K] aktywne dla trybu Custom
- Suwak ε_real zablokowany w trybie H-R (ε wyznaczane z modelu)
- Nota o materiale i ostrzeżenie o błędzie ε_zał ≠ ε(λ)

#### Krzywa ε(λ) na wykresie widmowym
- Bursztynowa przerywana krzywa ε(λ) w górnej strefie wykresu (razem z R(λ))
- Mapowanie: ε ∈ [0,1] → pełna strefa Rzone (ryEps)
- Pojawia się tylko w trybie H-R; etykieta „ε(λ)" po lewej stronie
- Pozycja w legendzie widma: „ε(λ) H-R"

#### Refaktoring compute()
- eps_real (scalar) → epsSpectral(lam, T_K, p) zwracająca ε(λ,T)
- Tablica Eps[] dodana do wyników compute() dla rysowania krzywej
- Dekompozycja błędów (sig_eps, sig_atm) używa epsSpectral()

---

## [v2.1.1] — 2026-05-24 ✅ bieżąca

### Poprawiono
- **Suwaki RH i CO₂ nie wpływały na transmitancję atmosfery** — błąd logiczny
  w `atmTau()`: `env.rh_pct`/`co2_ppm` z `ENVS.earth` wygrywały nad suwakami
  (`!== undefined` zawsze true). Dla Ziemi używane są teraz wartości z suwaków;
  dla pozostałych środowisk — wartości z ENVS (narzucone fizycznie)
- **Suwak T_atm resetował się do 60°C po wybraniu Wenus** — `max="60"` w HTML
  clampowało wartość 465°C. `setEnv()` dynamicznie rozszerza `min`/`max` suwaka
  przed ustawieniem wartości; `initSliders()` respektuje bieżący zakres środowiska
- Większe wykresy: specCanvas `clamp(200px, 54vh, 460px)`,
  tauCanvas `clamp(100px, 22vh, 200px)`
- Kolumna wykresów `position:sticky` w trybie Eksperta — wykresy widoczne
  podczas scrollowania listy suwaków

---

## [v2.1.0] — 2026-05-24

### Dodano — 6 środowisk atmosferycznych
- Panel **🌐 Środowisko / atmosfera** w trybie Eksperta z 6 środowiskami:
  - 🌍 **Ziemia** — model HITRAN (H₂O 12 pasm, CO₂ 7 pasm, O₃); suwaki RH/CO₂ aktywne
  - 🔴 **Mars** — 95.3% CO₂, P = 0.006 atm; paradoks Beer-Lamberta (gęsty CO₂,
    ale mała kolumnowa ilość gazu → atmosfera zaskakująco przejrzysta)
  - 🟡 **Wenus** — 96.5% CO₂ + 150 ppm SO₂, P = 92 atm, T_atm = 465°C;
    absorpcja totalna, pirometr widzi wyłącznie emisję atmosfery
  - 💧 **Woda ciekła** — specjalny model α(λ); blokada IR po kilku cm,
    jedyne okno: VIS 0.4–0.7 µm (detektor Si)
  - 🏭 **NH₃ MOCVD** — reaktory wzrostu GaN; NH₃ 50% + H₂ 50%, P = 0.3 atm;
    pasma N-H przy 2.97 i 6.15 µm, okno pirometryczne 0.9–1.1 µm (InGaAs)
  - 🏭 **CO₂ techniczny** — spawanie MIG/MAG; 100% CO₂, P = 1 atm;
    4.26 µm zablokowane, InSb 3.9 µm pracuje w oknie między pasmami CO₂
- **Uogólniony model Beer-Lambert** z czynnikiem ciśnienia P/P₀:
  τ(λ) = exp(−Σ αᵢ·cᵢ·L·P/P₀)
- Modele pasm absorpcyjnych: NH₃ (7 pasm), SO₂ (4 pasma), woda ciekła (α tabelaryczne)
- Presety środowiskowe w menu rozwijanym (5 nowych pozycji)
- Hint sugerowanego detektora per środowisko
- Dokumentacja Expert **sekcja 5**: „Środowiska atmosferyczne" — pełna fizyka z KaTeX,
  tabele porównawcze, przykłady liczbowe, info-boxy z ćwiczeniami
- Dokumentacja Beginner **sekcja 3b**: „Środowiska" — analogie i ćwiczenie interaktywne
- Zakres temperatury Beginner: **−20 do 3480 °C** (krok 50°C, siatka dokładna)

### Poprawiono — WebKit / Safari / iPadOS
- `color-scheme: dark/light` na `.theme-dark/.theme-light` — natywne kontrolki
  w odpowiednim motywie
- `color-scheme` na `:root` aktualizowany przez JS przy `toggleTheme()`
- `-webkit-appearance:none` na `button` i `select`
- `-webkit-text-fill-color` na polach numerycznych (Safari ignoruje `color`)
- Jawne `background:var(--surf/--bg)` na `.ctrl-col`, `.beg-panel`, `.ctrl-panel`,
  `.plots-col`, `.res-col`, `.main-layout` — brak dziedziczenia z `body.theme-X`
- `classList.toggle(name, force)` → jawne `add()`/`remove()` (bug Safari WebKit)
- `querySelectorAll('.env-btn')` zamiast `getElementById` — odporne na duplikaty DOM
- Usunięty zduplikowany panel środowiska

### Poprawiono — UI/UX
- Nagłówek: `flex:1; min-width:0` na `hdr-brand`, `flex-shrink:0` na `hdr-actions`
  — eliminuje overflow i nakładanie przy wąskim ekranie, `overflow:hidden` usunięte

---

## [v2.0.0] — 2026-05-23

### Przełomowe — dwa tryby obsługi

#### 🎓 Tryb Podstawowy (domyślny)
- Uproszczony interfejs dla studentów II roku bez doświadczenia z pirometrią
- Temperatura w °C (suwak + pole numeryczne, krok 1°C)
- 4 przyciski emisyjności: Metal polerowany / Stal utleniona / Ceramika / CDC
- 3 przyciski atmosfery: Brak / Krótka (2 m) / Długa (20 m)
- 3 typy pirometru: Bliskie IR 0.65 µm / Środkowe IR 3–5 µm / Dalekie IR 8–14 µm
- Sygnalizacja świetlna ΔT 🟢🟡🔴 z dynamicznym hintem tekstowym
- Dokumentacja „Jak działa pirometr?" — 4 sekcje z analogiami i ćwiczeniami

#### 🔬 Tryb Eksperta
- Pełny interfejs wersji v1.x + nowe funkcje v2.x
- Przełącznik w nagłówku: wyróżniony żółty przycisk

### Dodano
- Nowa nazwa aplikacji: **„Pirometria"** (było „Symulator Pirometryczny")
- Presety: menu rozwijane `☰ Presety ▾` zamiast przycisków w rzędzie
- Presety beginner: CDC wzorzec / Stal w piecu / Ceramika
- `<body class="theme-dark beginner">` — klasy inicjalne w HTML (brak migotania)
- Dokumentacja: tryb Podstawowy (bo) i Eksperta (xo) w jednym panelu

### Poprawiono
- `toggleTheme()`: `classList.toggle` zamiast `className=` (zachowuje klasy trybu)
- Duplikaty ID (`r_Tind`, `r_Tobj` itd.): panel Expert ma sufiks `_x`
- Adaptacyjny `pL` osi Y: skaluje się z zoom przeglądarki (`W*0.072`)
- Canvas: `offsetHeight` zamiast `getAttribute` (poprawne po `display:none`)
- `requestAnimationFrame` w `applyMode()` — canvas po odkryciu z `display:none`

---

## [v1.10.0] — 2026-05-23

### Poprawiono
- **Zakresy Gaussów** — były 3–5× za szerokie; nowe: `range = [lc-5σ, lc+5σ]`
  gdzie σ = bw/2.355 (R przy granicach ≈ 3.7×10⁻⁶, praktycznie zero):
  `band_065`: [0.45,1.00] → [0.555,0.745]  
  `band_100`: [0.70,1.40] → [0.851,1.149]  
  `band_160`: [1.20,2.10] → [1.388,1.812]  
  `band_390`: [3.00,5.00] → [3.156,4.644]
- **R(λ) na wykresie widma** — adaptacyjna siatka 500 pkt w zakresie czułości
  (gęsta w `[dmin,dmax]`, rzadka poza) — gładka nawet dla Gaussa 0.65 µm
- Sygnał na detektorze: kolor `--signal` (#c026d3) zamiast pomarańczowego
  — wyraźnie odróżnialny od żółtego ε·B i cyjanowego T_ind
- Legenda widma uzupełniona o emisję przesłony
- Polskie i bezpłatne źródła online w bibliografii: pozycje [11]–[15]
  (Minkina, GUM, BIPM SI Brochure, HITRAN on the Web, Wikipedia PL)

---

## [v1.9.0] — 2026-05-23

### Dodano
- **Sekcja 10 Literatura** — 10 pozycji naukowych z linkami DOI
  (Planck, Wien, Boltzmann, Kirchhoff, DeWitt, Preston-Thomas,
  Palik, Michalski, HITRAN2020, Modest)
- Cytowania `[n]` przy wzorach w tekście dokumentacji

### Poprawiono
- Brakujący `</div>` panelu przesłony (detektor wypadał poza kontener)
- Brakująca kolumna D_REF dla szyby float w tabeli okien optycznych
- Temperatura T_obiektu: pole liczbowe w °C (krok 1°C) + przelicznik K
- Pola numeryczne dla ε_real i ε_assumed (krok 0.01)
- Reorganizacja dokumentacji: kolejność zgodna z torem pomiaru (9 sekcji)

---

## [v1.8.1] — 2026-05-22

### Poprawiono
- `Lwin_emit` nie było zwracane z `compute()` — `TypeError: undefined is not iterable`
  przy zmianie temperatury przesłony

---

## [v1.8.0] — 2026-05-22

### Dodano
- **Grubość przesłony** (suwak 0.5–30 mm): Beer-Lambert
  `τ_bulk(d) = τ_bulk(d_ref)^(d/d_ref)` — absorpcja skaluje się z grubością
- **Temperatura przesłony** (suwak 0–700°C): emisja własna
  `L_win_emit = (1−τ_win)·L_bb(T_win)` — prawo Kirchhoffa
- Refaktoring `windowTau()`: rozdzielenie τ_Fresnel / τ_bandpass / τ_bulk(d)
- Krzywa emisji własnej okna na wykresie widmowym
- Schemat blokowy: wyświetla d i T_win w bloku przesłony
- Dokumentacja sekcja 8: pełna teoria (Beer-Lambert, emisja własna, tabela d_ref)

---

## [v1.7.1] — 2026-05-21

### Poprawiono
- Canvas collapse w Opera/Blink — jawna wysokość CSS + `offsetHeight`

---

## [v1.7.0] — 2026-05-21

### Dodano
- **Przesłona optyczna** — blok 3 w torze z 6 materiałami:
  szyba float, PMMA, kwarc (SiO₂), fluoryt (CaF₂), ZnSe, Ge
- Budżet błędów: **3 składowe** Δε, ΔT_atm, ΔT_win
- Pole numeryczne T_obiektu w K + przelicznik °C
- Preset 🪟 Plexiglas (PMMA, InGaAs 1.0 µm)
- Krzywa τ_win(λ) na wykresie, nagłówek toru zaktualizowany

### Poprawiono
- Crosshair hover w skali liniowej (błędna formuła log)
- Zduplikowany blok Atmosfera w schemacie blokowym

---

## [v1.5.0 – v1.6.0] — 2026-05-21

- v1.6.0: Skrót CDC zamiast CDB, poprawki KaTeX
- v1.5.0: Pasek UV+VIS, zakres 10–12000 K, skala log/lin, KaTeX, responsywność

---

## [v1.0.0 – v1.4.0] — 2026-05-21

- v1.4.0: Motyw jasny/ciemny, schemat blokowy HTML, dokumentacja 7 sekcji
- v1.3.0: Pasek widma widzialnego
- v1.2.0: Schemat blokowy, panel wyników, tooltip, 5 presetów
- v1.1.0: Model atmosfery (H₂O, CO₂), 7 typów detektorów
- v1.0.0: Pierwsza wersja — Planck, Wien, Stefan-Boltzmann, widmo CDC
