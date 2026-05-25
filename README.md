# Pirometria — Symulator Toru Pomiarowego

**Interaktywna aplikacja edukacyjna** do demonstracji fizyki bezdotykowego pomiaru temperatury metodą pirometryczną. Modeluje pełny tor pomiarowy z dwoma trybami obsługi i sześcioma środowiskami atmosferycznymi.

> Materiał dydaktyczny — ćwiczenia laboratoryjne z metrologii temperatury  
> **Mirosław Socha** · Katedra Metrologii i Elektroniki · WEAIiIB · AGH Kraków

---

## Uruchomienie

Jeden plik HTML — otwórz `src/symulator_pirometru.html` w przeglądarce. Brak instalacji, brak serwera.

---

## Dwa tryby obsługi

### 🎓 Tryb Podstawowy (domyślny)
- Temperatura w °C, zakres −20 do 3480 °C
- 4 presetowe przyciski emisyjności (metal / stal / ceramika / CDC)
- 3 przyciski atmosfery (brak / krótka 2m / długa 20m)
- 3 typy pirometru (bliskie IR / środkowe / dalekie IR)
- Sygnalizacja świetlna ΔT 🟢🟡🔴
- Wbudowana dokumentacja „Jak działa pirometr?" (4 sekcje z analogiami)

### 🔬 Tryb Eksperta
- Zakres temperatury: 10 K – 12 000 K
- **6 środowisk atmosferycznych** (Ziemia, Mars, Wenus, woda, NH₃ MOCVD, CO₂ tech.)
- 4 suwaki atmosferyczne (L, HR, CO₂, T_atm)
- Przesłona optyczna: 6 materiałów, grubość (Beer-Lambert), temperatura (emisja własna)
- **2 modele emisyjności**: szara ε=const / Hagen-Rubens ε(λ,T) z oporności elektrycznej
- 7 typów detektorów z wizualizacją R(λ) na wykresie
- Dwa wykresy: widmo + transmitancja τ(λ)
- Budżet błędów 3-składowy (Δε, ΔT_atm, ΔT_win)
- Dokumentacja: 11 sekcji z wzorami KaTeX + bibliografia 15 pozycji

---

## Tor pomiarowy

```
[Obiekt T,ε] → [Atmosfera P,τ(λ)] → [Przesłona τ_win(λ,d)] → [Detektor R(λ)] → [Procesor] → T_ind
```

Sygnał na detektorze:

$$L_{\rm det}(\lambda) = \tau_{\rm win}\!\cdot\!\Big[\tau_{\rm atm}\cdot\varepsilon\cdot L_{bb}(T) + (1-\tau_{\rm atm})\cdot L_{bb}(T_{\rm atm})\Big] + (1-\tau_{\rm win})\cdot L_{bb}(T_{\rm win})$$

---

## Środowiska atmosferyczne

| Środowisko | Skład | Ciśnienie | Efekt dydaktyczny |
|---|---|---|---|
| 🌍 Ziemia | H₂O + CO₂ + O₃ | 1 atm | Model referencyjny HITRAN |
| 🔴 Mars | 95.3% CO₂ | **0.006 atm** | Paradoks — gęsty CO₂ ale mała ilość gazu → przezroczysta |
| 🟡 Wenus | 96.5% CO₂ + SO₂ | **92 atm**, 465°C | Pirometr widzi tylko atmosferę |
| 💧 Woda | H₂O ciekła | 1 atm | IR blokowane po 10 cm, tylko VIS |
| 🏭 NH₃ MOCVD | NH₃ 50% + H₂ | 0.3 atm | Reaktor GaN — okno 0.9–1.1 µm |
| 🏭 CO₂ tech. | 100% CO₂ | 1 atm | Spawanie — InSb 3.9 µm w oknie |

Model uogólniony: τ(λ) = exp(−Σ αᵢ·cᵢ·L·**P/P₀**) — czynnik ciśnienia jest kluczowy.

---

## Model fizyczny

### Prawa promieniowania
Planck · Wien · Stefan–Boltzmann · Kirchhoff

### Atmosfera — Beer–Lambert (uproszczony HITRAN [9])
12 pasm H₂O · 7 pasm CO₂ · 7 pasm NH₃ · 4 pasma SO₂ · woda ciekła

### Przesłona — model trójskładowy
`τ(λ,d) = τ_Fresnel · τ_bandpass(λ) · τ_bulk(λ,d_ref)^(d/d_ref)` + emisja własna `(1−τ)·L_bb(T_win)`

### Detektory — 7 typów, zakresy ±5σ
Si 0.65 µm · InGaAs 1.0/1.6 µm · InSb 3.9 µm · InSb 3–5 µm · MCT 8–14 µm · Termostos 1–18 µm

### Inwersja temperatury
Bisekcja 72 iteracje, 1–15 000 K, zbieżność < 0.01 K

### Budżet błędów
ΔT = ΔT_ε + ΔT_atm + ΔT_win

---

## Funkcje interfejsu

| Funkcja | Opis |
|---|---|
| Skala λ | Liniowa / logarytmiczna |
| R(λ) na wykresie | Adaptacyjna siatka 500 pkt — gładka nawet dla Gaussa 0.65 µm |
| Pasek UV+VIS | Gradient na osi λ obu wykresów |
| Tooltip hover | λ, L_bb [W·m⁻²·sr⁻¹·m⁻¹], ε·B, τ_atm, τ_win, sygnał, R(λ) |
| Model ε(λ) | Szara / Hagen-Rubens; 8 materiałów z danymi ρ₀,α; krzywa ε(λ) na wykresie |
| Motywy | Ciemny / jasny (Safari/iPad: `color-scheme` + `color-scheme` na `:root`) |
| Responsywność | Desktop · tablet · telefon |
| Presety | Menu rozwijane (3 basic + 6 expert + 5 środowiskowych) |

---

## Dokumentacja wbudowana

**Tryb Podstawowy:** 4+1 sekcji — „Jak działa pirometr?", emisyjność, atmosfera, środowiska, wykresy  
**Tryb Eksperta:** 11 sekcji z KaTeX — tor pomiarowy, Planck, emisyjność, atmosfera, **środowiska** (nowe!), przesłona, detektory, sygnał+inwersja, słownik, informacje, literatura  
**Bibliografia:** 15 pozycji [1–10 naukowe z DOI, 11–15 polskie/online]

---

## Struktura repozytorium

```
SymulatorPirometru/
├── src/
│   └── symulator_pirometru.html   # standalone — HTML + CSS + JS
├── CHANGELOG.md
├── README.md
└── .gitignore
```

## Gałęzie

| Gałąź | Język | Wersja |
|---|---|---|
| `main` | 🇵🇱 Polski | v2.2.0 |
| `en/english-translation` | 🇬🇧 English | v2.0.0-en (wymaga aktualizacji) |

---

## Historia wersji

| Wersja | Kluczowe zmiany |
|---|---|
| **v2.1.1** | 6 środowisk atmosferycznych (Mars, Wenus, woda, NH₃, CO₂), nowy atmTau z P/P₀, dokumentacja sekcja 5, naprawa WebKit/Safari/iPad, zakres T beginner do 3480°C |
| v2.0.0 | Tryb Podstawowy + Eksperta, „Pirometria", menu presetów |
| v1.10.0 | Poprawne zakresy Gaussów ±5σ, adaptacyjna siatka R(λ) |
| v1.9.0 | Literatura z DOI, pola numeryczne T/ε |
| v1.8.x | Przesłona: grubość + temperatura + emisja własna |
| v1.7.x | Przesłona optyczna, budżet błędów 3-składowy |
| v1.5.0 | Pierwsza pełna wersja z atmosferą i dokumentacją |

Pełna historia: [CHANGELOG.md](CHANGELOG.md)

---

## Licencja

Materiał dydaktyczny — AGH Akademia Górniczo-Hutnicza w Krakowie. Użytek edukacyjny.
