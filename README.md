# Pirometria — Symulator Toru Pomiarowego

**Interaktywna aplikacja edukacyjna** do demonstracji fizyki bezdotykowego pomiaru temperatury metodą pirometryczną. Modeluje pełny 6-blokowy tor pomiarowy z **trzema trybami obsługi**, sześcioma środowiskami atmosferycznymi, czterema modelami emisyjności, efektem wnękowym i modelowaniem odbić promieniowania otoczenia.

> Materiał dydaktyczny — ćwiczenia laboratoryjne z metrologii temperatury  
> **Mirosław Socha** · Katedra Metrologii i Elektroniki · WEAIiIB · AGH Kraków

---

## Uruchomienie

Jeden samodzielny plik HTML — otwórz `src/symulator_pirometru.html` w przeglądarce.  
Brak instalacji, brak serwera, brak zależności lokalnych. CDN: Google Fonts + KaTeX.

---

## Trzy tryby obsługi

Segment przełącznika w nagłówku: **Podstawowy → PRO → Ekspert**. Każdy kolejny tryb jest nadzbiorem poprzedniego.

### 🎓 Tryb Podstawowy (domyślny)
Dla studentów bez doświadczenia z pirometrią:
- Temperatura −20 do 3480 °C (suwak + pole numeryczne)
- 4 przyciski emisyjności: Metal polerowany / Stal utleniona / Ceramika / CDC
- 3 przyciski atmosfery: Brak / Krótka 2 m / Długa 20 m
- 3 typy pirometru: Bliskie IR 0.65 µm / Środkowe 3–5 µm / Dalekie 8–14 µm
- Sygnalizacja świetlna ΔT 🟢🟡🔴
- Wbudowany przewodnik „Jak działa pirometr?" (5 sekcji z analogiami)

### ⚙️ Tryb PRO
Pełny tor pomiarowy bez elementów najbardziej zaawansowanych:
- Temperatura 10 K – 12 000 K (suwak w K + pole °C)
- **6 środowisk atmosferycznych**: Ziemia, Mars, Wenus, woda, NH₃ MOCVD, CO₂ tech.
- 4 suwaki atmosferyczne: droga L, wilgotność RH, CO₂ [ppm], T_atm
- **Odbicia promieniowania otoczenia**: (1−ε)·L_bb(T_otocz) z polem T_otocz
- Przesłona optyczna: 6 materiałów, grubość d (Beer-Lambert), temperatura (emisja własna)
- 7 typów detektorów z krzywą R(λ) na wykresie widmowym
- Dwa wykresy: widmo promieniowania + transmitancja τ(λ), z wizualizacją przesterowania (>100%)
- Budżet błędów 4-składowy: ΔT_ε + ΔT_refl + ΔT_atm + ΔT_win
- **Zwijane panele**: kliknięcie tytułu panelu zwija/rozwija jego zawartość
- Dokumentacja PRO: pełny opis toru z wzorami KaTeX

### 🔬 Tryb Eksperta
PRO + elementy najbardziej zaawansowane:
- **Model emisyjności** (siatka 2×2): szara · Hagen-Rubens ε(λ,T) · wielomianowy TPRC · **Fresnel ε(θ,λ,T)**
- **Fresnel**: diagram biegunowy ε(θ) live-update, porównanie dwóch materiałów, tooltip hover; model Drudego n,k z ρ(T)
- **Efekt wnękowy**: ε_eff = ε/[ε+(1−ε)·s/S], suwak s/S, diagram przekroju wnęki z wizualizacją
- **Tryb procesora**: Jednobarwny / Dwubarwny (ratio S₁/S₂ → T_ratio)
- Dokumentacja Ekspert: wszystkie modele emisyjności z wyprowadzeniami, bibliografia [1]–[20]

---

## Eksport danych

Trzy przyciski w nagłówku aplikacji:

| Przycisk | Format | Zawartość |
|---|---|---|
| **⬇ CSV** | CSV UTF-8 | Nastawy + Wyniki + Widmo widmowe (tabela λ/L_bb/L_emit/…/ε) |
| **⬇ PNG** | PNG | Kompozyt: widmo + τ(λ) + legenda z ikonami + stopka z parametrami |
| **⬇ SVG** | SVG wektor | Właściwy SVG z `<path>`, `<linearGradient>` UV-VIS-IR, wykres τ(λ), legenda |

---

## Tor pomiarowy — 6 bloków

```
[Obiekt T,ε + odbicia] → [Atmosfera P,τ_atm(λ)] → [Przesłona τ_win(λ,d)] → [Detektor R(λ)] → [Procesor ε_zał] → T_ind
```

Równanie sygnału na wejściu detektora:

```
L_obj(λ) = ε(λ,T)·L_bb(λ,T_obj) + [1−ε(λ,T)]·L_bb(λ,T_otocz)   ← emisja + odbicia otoczenia

L_det(λ) = τ_win(λ) · [ τ_atm(λ)·L_obj(λ)
                        + [1−τ_atm(λ)]·L_bb(λ,T_atm) ]
          + [1−τ_win(λ)]·L_bb(λ,T_win)
```

---

## Modele emisyjności

| Model | Opis | Parametry |
|---|---|---|
| **Szara** | ε(λ,T) = const | suwak ε_real |
| **Hagen-Rubens** | ε(λ,T) ≈ 0.365√(ρ/λ) − 0.0667(ρ/λ) | ρ₀ [µΩ·cm], α [K⁻¹], 8 materiałów |
| **Wielomianowy** | ε(λ) = a₀ + a₁λ + a₂λ² z danych TPRC | 9 presetów (W, Mo, Ti, Fe, Ni, C, SiC, Al₂O₃, Custom) |
| **Fresnel ε(θ,λ,T)** | Emisyjność kierunkowa z równań Fresnela | kąt θ 0–85°, 6 materiałów; model Drudego n,k z ρ(T) |

Efekt wnękowy komponuje się z każdym modelem: `ε_eff = ε / [ε + (1−ε)·s/S]`

---

## Środowiska atmosferyczne

| Środowisko | Skład | Ciśnienie | Efekt dydaktyczny |
|---|---|---|---|
| 🌍 Ziemia | H₂O + CO₂ + O₃ | 1 atm | Model referencyjny wg HITRAN |
| 🔴 Mars | 95.3% CO₂ | **0.006 atm** | Paradoks Beer-Lamberta: gęsty CO₂, ale mała ilość gazu |
| 🟡 Wenus | 96.5% CO₂ + SO₂ | **92 atm**, 465°C | Atmosfera dominuje sygnał — pomiar niemożliwy |
| 💧 Woda | H₂O ciekła | — | IR blokowane po kilku cm, jedyne okno: VIS 0.65 µm |
| 🏭 NH₃ MOCVD | NH₃ 50% + H₂ | 0.3 atm | Reaktory GaN — okno pirometryczne 0.9–1.1 µm |
| 🏭 CO₂ tech. | 100% CO₂ | 1 atm | Spawanie MIG/MAG — InSb 3.9 µm między pasmami CO₂ |

---

## Model fizyczny

**Prawa promieniowania:** Planck · Wien · Stefan–Boltzmann · Kirchhoff  
**Stałe:** CODATA 2018 (h = 6.62607×10⁻³⁴ J·s, c = 2.99792×10⁸ m/s, k_B = 1.38065×10⁻²³ J/K)

**Atmosfera:** Beer–Lambert z P/P₀ · 12 pasm H₂O · 7 pasm CO₂ · 7 pasm NH₃ · 4 pasma SO₂ · model wody ciekłej · emisja własna gazu (1−τ)·L_bb(T_atm)

**Emisyjność metali:** Hagen-Rubens (1903) · ρ(T) = ρ₀[1+α(T−293)] · 8 materiałów  
**Emisyjność kierunkowa:** Fresnel (równania zespolone) + Drude IR: n≈k≈√(2998·λ/ρ) · 6 materiałów  
**Efekt wnękowy:** model Gouffé — ε_eff = ε/[ε+(1−ε)·s/S]

**Przesłona:** τ(λ,d) = τ_Fresnel · τ_bandpass(λ) · τ_bulk(d) + emisja własna (1−τ)·L_bb(T_win)

**Detektory (7):** Si 0.65 µm · InGaAs 1.0/1.6 µm · InSb 3.9 µm · InSb 3–5 µm · MCT 8–14 µm · Termostos 1–18 µm

**Inwersja temperatury:** bisekcja 72 iter. · zakres 1–15 000 K · zbieżność < 0.01 K · siatka N=1400 · λ ∈ [0.10, 16] µm

**Budżet błędów:** ΔT = ΔT_ε + ΔT_refl + ΔT_atm + ΔT_win (suma teleskopowa, 4 składowe)

---

## Struktura repozytorium

```
SymulatorPirometru/
├── docs/
│   └── PRZEWODNIK_PROGRAMISTYCZNY.md   # przewodnik implementacji i architektury
├── src/
│   └── symulator_pirometru.html        # standalone — HTML + CSS + JS (~5300 linii)
├── CHANGELOG.md
├── README.md
└── .gitignore
```

## Dokumentacja dla programistów

Dla osób analizujących projekt od strony implementacyjnej dostępny jest osobny materiał:

- **[Przewodnik programistyczny](docs/PRZEWODNIK_PROGRAMISTYCZNY.md)** — opis architektury aplikacji, zarządzania stanem, silnika obliczeniowego, renderowania Canvas/SVG, eksportu danych oraz zastosowanych wzorców projektowych.

> Wersja angielska gałęzi `en/english-translation` jest zsynchronizowana z `main` (v3.4.0-en): Fresnel, efekt wnękowy, zwijane panele, eksport CSV/PNG/SVG, pełny audit tłumaczenia.

## Gałęzie

| Gałąź | Język | Wersja |
|---|---|---|
| `main` | 🇵🇱 Polski | **v3.4.0** |
| `en/english-translation` | 🇬🇧 English | **v3.4.0-en** ✅ |

---

## Historia wersji

| Wersja | Kluczowe zmiany |
|---|---|
| **v3.4.0** | Eksport CSV (nastawy+wyniki+widmo), PNG kompozytowy (spec+τ+legenda+stopka), SVG wektorowy (path, linearGradient UV-VIS-IR, τ(λ), legenda) |
| **v3.3.0** | Efekt wnękowy ε_eff=ε/[ε+(1−ε)·s/S] z diagramem przekroju; zwijane panele (click nagłówka); diagram biegunowy Fresnela: tooltip hover + porównanie 2 materiałów |
| **v3.2.1** | Audyt merytoryczny: stała Drudego DRUDE_A=2997.91 (błąd jednostek), Tatm_K w dekompozycji, bibliografia [16]–[20], opis ε=const w PRO, NH₃/SO₂ w dok. s4, stałe CODATA 2018 |
| **v3.2.0** | Model Fresnela ε(θ,λ,T): emisyjność kierunkowa; diagram biegunowy live-update; model Drudego n,k z ρ(T) |
| v3.1.1 | Poprawki UI: kontrast przycisków trybu, reorganizacja paneli, pole T_otocz, wizualizacja przesterowania (>100%) i warstwy odbić |
| v3.1.0 | Modelowanie odbić promieniowania otoczenia (1−ε)·L_bb(T_otocz), składowa błędu ΔT_refl (tryb PRO) |
| v3.0.0 | Trzeci tryb UI — PRO między Podstawowym a Ekspertem; rozdzielenie dokumentacji PRO/Ekspert; rejestr EPS_MODELS |
| v2.9.0 | Pełne tłumaczenie EN (gałąź en); interaktywny schemat blokowy |
| v2.4.0 | Pirometria dwubarwna: S₁/S₂, T_ratio, ΔT_ratio, 3 pary przemysłowe |
| v2.3.0 | Model wielomianowy ε(λ) z TPRC: 9 materiałów |
| v2.2.0 | Model emisyjności Hagena-Rubensa ε(λ,T), 8 materiałów |
| v2.1.0 | 6 środowisk atmosferycznych, Beer-Lambert z P/P₀ |
| v2.0.0 | Tryb Podstawowy + Eksperta, menu presetów |

Pełna historia: [CHANGELOG.md](CHANGELOG.md)

---

## Licencja

Materiał dydaktyczny — AGH Akademia Górniczo-Hutnicza w Krakowie. Użytek edukacyjny.
