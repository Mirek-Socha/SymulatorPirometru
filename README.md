# Pirometria — Symulator Toru Pomiarowego

**Interaktywna aplikacja edukacyjna** do demonstracji fizyki bezdotykowego pomiaru temperatury metodą pirometryczną. Modeluje pełny 6-blokowy tor pomiarowy z dwoma trybami obsługi, sześcioma środowiskami atmosferycznymi i dwoma modelami emisyjności.

> Materiał dydaktyczny — ćwiczenia laboratoryjne z metrologii temperatury  
> **Mirosław Socha** · Katedra Metrologii i Elektroniki · WEAIiIB · AGH Kraków

---

## Uruchomienie

Jeden samodzielny plik HTML — otwórz `src/symulator_pirometru.html` w przeglądarce.  
Brak instalacji, brak serwera, brak zależności lokalnych. CDN: Google Fonts + KaTeX.

---

## Dwa tryby obsługi

### 🎓 Tryb Podstawowy (domyślny)
Dla studentów bez doświadczenia z pirometrią:
- Temperatura −20 do 3480 °C (suwak + pole numeryczne)
- 4 przyciski emisyjności: Metal polerowany / Stal utleniona / Ceramika / CDC
- 3 przyciski atmosfery: Brak / Krótka 2 m / Długa 20 m
- 3 typy pirometru: Bliskie IR 0.65 µm / Środkowe 3–5 µm / Dalekie 8–14 µm
- Sygnalizacja świetlna ΔT 🟢🟡🔴
- Wbudowany przewodnik „Jak działa pirometr?" (5 sekcji z analogiami)
- Spis treści z linkami do sekcji

### 🔬 Tryb Eksperta
Pełny model fizyczny — wszystkie parametry toru:
- Temperatura 10 K – 12 000 K (suwak w K + pole °C)
- **Model emisyjności**: szara ε = const lub Hagen-Rubens ε(λ,T) z 8 materiałami
- **6 środowisk atmosferycznych**: Ziemia, Mars, Wenus, woda, NH₃ MOCVD, CO₂ tech.
- 4 suwaki atmosferyczne: droga L, wilgotność RH, CO₂ [ppm], T_atm
- Przesłona optyczna: 6 materiałów, grubość d (Beer-Lambert), temperatura (emisja własna)
- 7 typów detektorów z krzywą R(λ) na wykresie widmowym
- Dwa wykresy: widmo promieniowania + transmitancja τ(λ)
- **Tryb procesora**: Jednobarwny / Dwubarwny (ratio S₁/S₂ → T_ratio)
- Budżet błędów 3-składowy: ΔT_ε + ΔT_atm + ΔT_win
- Dokumentacja: 11 sekcji z wzorami KaTeX + bibliografia 15 pozycji
- Spis treści z linkami kotwicowymi + przycisk „Otwórz w nowym oknie" + „Drukuj / PDF"

---

## Tor pomiarowy — 6 bloków

```
[Obiekt T,ε(λ,T)] → [Atmosfera P,τ_atm(λ)] → [Przesłona τ_win(λ,d)] → [Detektor R(λ)] → [Procesor ε_zał] → T_ind
```

Równanie sygnału na wejściu detektora:

```
L_det(λ) = τ_win(λ) · [ τ_atm(λ)·ε(λ,T)·L_bb(λ,T_obj)
                        + [1−τ_atm(λ)]·L_bb(λ,T_atm) ]
          + [1−τ_win(λ)]·L_bb(λ,T_win)
```

---

## Model emisyjności

| Model | Opis | Parametry |
|---|---|---|
| **Szara** | ε(λ,T) = const | suwak ε_real |
| **Hagen-Rubens** | ε(λ,T) ≈ 0.365√(ρ/λ) − 0.0667(ρ/λ) | ρ₀ [µΩ·cm], α [K⁻¹], 8 materiałów |
| **Wielomianowy** | ε(λ) = a₀ + a₁λ + a₂λ² z danych TPRC | 9 presetów (W, Mo, Ti, Fe, Ni, C, SiC, Al₂O₃, Custom) |

Model H-R: ε maleje z λ i rośnie z T — metale emitują lepiej przy krótkich falach. Krzywa ε(λ) widoczna na wykresie widmowym (bursztynowa przerywana).

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

Model: τ(λ) = exp(−Σ αᵢ·cᵢ·L·**P/P₀**) — czynnik ciśnienia kluczowy (paradoks Marsa).

---

## Model fizyczny

**Prawa promieniowania:** Planck · Wien · Stefan–Boltzmann · Kirchhoff

**Atmosfera:** Beer–Lambert z P/P₀ · 12 pasm H₂O · 7 pasm CO₂ · 7 pasm NH₃ · 4 pasma SO₂ · model wody ciekłej

**Emisyjność metali:** Hagen-Rubens (1903) · ρ(T) = ρ₀[1+α(T−293)]

**Przesłona:** τ(λ,d) = τ_Fresnel · τ_bandpass(λ) · τ_bulk(d) + emisja własna (1−τ)·L_bb(T_win)

**Detektory (7):** Si 0.65 µm · InGaAs 1.0/1.6 µm · InSb 3.9 µm · InSb 3–5 µm · MCT 8–14 µm · Termostos 1–18 µm

**Inwersja temperatury:** bisekcja 72 iter. · zakres 1–15 000 K · zbieżność < 0.01 K · siatka N=700 · λ ∈ [0.10, 16] µm

**Budżet błędów:** ΔT = ΔT_ε + ΔT_atm + ΔT_win (3 niezależne składowe)

---

## Funkcje interfejsu

| Funkcja | Opis |
|---|---|
| Skala osi λ | Liniowa / logarytmiczna |
| Krzywa R(λ) | Adaptacyjna siatka, gładka dla wąskich Gaussów (Si 0.65 µm) |
| Krzywa ε(λ) | Widoczna w trybie Hagen-Rubens — bursztynowa przerywana |
| Pasek UV+VIS | Gradient tęczowy na osi λ obu wykresów |
| Tooltip hover | λ, L_bb, ε·B, τ_atm, τ_win, sygnał, R(λ) |
| Presety | Menu rozwijane: 3 basic + 6 expert + 5 środowiskowych |
| Spis treści | Linki kotwicowe w obu panelach dokumentacji |
| Nowe okno | Dokumentacja w osobnym oknie przeglądarki |
| Drukuj / PDF | Eksport przez systemowy dialog druku |
| Motywy | Ciemny / jasny — pełna obsługa Safari/iPadOS (`color-scheme`) |
| Responsywność | Desktop · tablet · telefon |

---

## Dokumentacja wbudowana

**Tryb Podstawowy (5 sekcji):** Co to jest pirometr? · Emisyjność · Atmosfera · Środowiska · Jak czytać wykresy?

**Tryb Eksperta (11 sekcji + KaTeX):**
1. Model toru pomiarowego (równanie L_det z 3 składnikami)
2. Prawo Plancka — Wien, Stefan-Boltzmann
3. Emisyjność i prawo Kirchhoffa — model H-R z tabelą materiałów
4. Model transmitancji atmosferycznej — HITRAN, pasma H₂O/CO₂
5. Środowiska atmosferyczne — 6 środowisk, fizyka, eksperymenty
6. Przesłona optyczna — Beer-Lambert, emisja własna, tabela materiałów
7. Modele detektora R(λ) — Gauss, prostokąt, szerokopasmowy
8. Całkowanie sygnału i inwersja T — bisekcja, dekompozycja 3-składowa
9. Słownik symboli — 26 pozycji
10. Informacje o aplikacji
11. Literatura — 15 pozycji z DOI

---

## Struktura repozytorium

```
SymulatorPirometru/
├── src/
│   └── symulator_pirometru.html   # standalone — HTML + CSS + JS (~3800 linii)
├── CHANGELOG.md
├── README.md
└── .gitignore
```

## Gałęzie

| Gałąź | Język | Wersja |
|---|---|---|
| `main` | 🇵🇱 Polski | **v2.4.3** |
| `en/english-translation` | 🇬🇧 English | v2.0.0-en *(wymaga aktualizacji)* |

---

## Historia wersji

| Wersja | Kluczowe zmiany |
|---|---|
| **v2.4.3** | Fix KaTeX Wien S₁/S₂ block (double backslash, \night) |
| v2.4.2 | Fix KaTeX sekcja 8: \approx i \right zdegenerowane escape |
| v2.4.1 | Bugfix: computeRatio(), R₂(λ) normalizacja, KaTeX Polish chars |
| v2.4.0 | Pirometria dwubarwna: S₁/S₂, T_ratio, ΔT_ratio, 3 pary przemysłowe, sekcja 8 doc |
| v2.3.1 | ε(λ) w tooltipie, numer wersji w nagłówku |
| v2.3.0 | Model wielomianowy ε(λ) z TPRC: 8 materiałów (W, Mo, Ti, Fe, Ni, C, SiC, Al₂O₃) |
| v2.2.3 | Równanie toru wieloliniowe (KaTeX aligned), fix overflow przy druku PDF |
| v2.2.2 | Aktualizacja dokumentacji: tor z τ_win, budżet 3-składowy, słownik 26 poz. |
| v2.2.1 | Spis treści z linkami, numeracja sekcji Beginner poprawiona |
| v2.2.0 | Model emisyjności Hagena-Rubensa ε(λ,T), 8 materiałów, krzywa ε(λ) |
| v2.2.1* | Dokumentacja w nowym oknie / druk PDF — `buildDocsHTML()` |
| v2.1.1 | Fix: RH/CO₂ suwaki, T_atm zakres Wenus, większe wykresy, sticky plots |
| v2.1.0 | 6 środowisk atmosferycznych, Beer-Lambert z P/P₀, Safari/iPad fixes |
| v2.0.0 | Tryb Podstawowy + Eksperta, nazwa „Pirometria", menu presetów |
| v1.10.0 | Zakresy Gaussów ±5σ, adaptacyjna siatka R(λ) |
| v1.8.x | Przesłona: grubość (Beer-Lambert) + temperatura (emisja własna) |
| v1.7.0 | Przesłona optyczna, budżet błędów 3-składowy |
| v1.5.0 | Pierwsza pełna wersja: atmosfera, dokumentacja KaTeX, responsywność |

Pełna historia: [CHANGELOG.md](CHANGELOG.md)

---

## Licencja

Materiał dydaktyczny — AGH Akademia Górniczo-Hutnicza w Krakowie. Użytek edukacyjny.
