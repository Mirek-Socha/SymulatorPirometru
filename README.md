# Pirometria — Symulator Toru Pomiarowego

**Interaktywna aplikacja edukacyjna** do demonstracji fizyki bezdotykowego pomiaru temperatury metodą pirometryczną. Modeluje pełny tor pomiarowy z dwoma trybami obsługi.

> Materiał dydaktyczny — ćwiczenia laboratoryjne z metrologii temperatury  
> **Mirosław Socha** · Katedra Metrologii i Elektroniki · WEAIiIB · AGH Kraków

---

## Uruchomienie

Jeden plik HTML — otwórz `src/symulator_pirometru.html` w przeglądarce. Brak instalacji, brak serwera.

---

## Dwa tryby obsługi

### 🎓 Tryb Podstawowy (domyślny)
Dla studentów bez doświadczenia z pirometrią. Uproszczony interfejs:
- Temperatura w °C (suwak + pole numeryczne)
- 4 presetowe przyciski emisyjności (metal / stal / ceramika / CDC)
- 3 przyciski atmosfery (brak / krótka / długa droga)
- 3 typy pirometru (bliskie IR / środkowe / dalekie IR)
- Jeden wykres widma z 3 krzywymi
- Wyniki z sygnalizacją świetlną 🟢🟡🔴
- Wbudowana dokumentacja „Jak działa pirometr?" (4 sekcje)

### 🔬 Tryb Eksperta (przycisk w nagłówku)
Pełen model fizyczny:
- Zakres temperatury: 10 K – 12 000 K
- 4 suwaki atmosferyczne (L, HR, CO₂, T_atm)
- Przesłona optyczna z 6 materiałami, grubością i temperaturą
- 7 typów detektorów
- Dwa wykresy: widmo + transmitancja τ(λ)
- Budżet błędów 3-składowy (Δε, ΔT_atm, ΔT_win)
- Dokumentacja z 10 sekcjami i bibliografią (15 pozycji)

---

## Tor pomiarowy

```
[Obiekt T,ε] → [Atmosfera τ_atm(λ)] → [Przesłona τ_win(λ)] → [Detektor R(λ)] → [Procesor] → T_ind
```

Sygnał na detektorze:

$$L_{\mathrm{det}}(\lambda) = \tau_{\mathrm{win}}(\lambda)\cdot\left[\tau_{\mathrm{atm}}(\lambda)\cdot\varepsilon\cdot L_{bb}(\lambda,T) + [1-\tau_{\mathrm{atm}}(\lambda)]\cdot L_{bb}(\lambda,T_{\mathrm{atm}})\right] + [1-\tau_{\mathrm{win}}(\lambda)]\cdot L_{bb}(\lambda,T_{\mathrm{win}})$$

---

## Model fizyczny

### Prawa promieniowania

| Prawo | Wzór | Zastosowanie |
|-------|------|-------------|
| Planck | $L_{bb}(\lambda,T)=\frac{2hc^2}{\lambda^5(e^{hc/\lambda k_BT}-1)}$ | Widmo CDC |
| Wien | $\lambda_{\max}T=2{,}898\times10^{-3}$ m·K | Szczyt emisji |
| Stefan–Boltzmann | $M=\sigma T^4$ | Pirometr całkujący |
| Kirchhoff | $\varepsilon=\alpha$ | Emisja = absorpcja |

### Atmosfera — Beer–Lambert (uproszczony HITRAN)
12 pasm H₂O · 7 pasm CO₂ · rozpraszanie Rayleigha

### Przesłona optyczna — 6 materiałów

| Materiał | Zakres [µm] | τ_Fresnel | d_ref |
|----------|------------|-----------|-------|
| Szyba float | 0.32–2.7 | 0.91 | 4 mm |
| Plexiglas (PMMA) | 0.35–2.8 | 0.92 | 5 mm |
| Kwarc topiony | 0.15–4.5 | 0.945 | 5 mm |
| Fluoryt (CaF₂) | 0.13–9.5 | 0.944 | 2 mm |
| Selenek cynku (ZnSe) | 0.55–16 | 0.70 | 3 mm |
| German (Ge) | 1.8–16 | 0.46 | 3 mm |

Model: `τ(λ,d) = τ_Fresnel · τ_bandpass(λ) · τ_bulk(λ,d_ref)^(d/d_ref)` + emisja własna `(1−τ)·L_bb(T_win)`

### Detektory — 7 typów

| Typ | Detektor | Zakres |
|-----|----------|--------|
| Pasmowy Si | 0.65 µm | [0.555, 0.745] µm |
| Pasmowy InGaAs | 1.0 µm | [0.851, 1.149] µm |
| Pasmowy InGaAs | 1.6 µm | [1.388, 1.812] µm |
| Pasmowy InSb | 3.9 µm | [3.156, 4.644] µm |
| Okienkowy InSb | 3–5 µm | — |
| Okienkowy MCT | 8–14 µm | — |
| Całkujący termostos | 1–18 µm | — |

Zakresy Gaussów = ±5σ (fizycznie poprawne).

### Inwersja temperatury
Bisekcja 72 iteracje, zakres 1–15 000 K, zbieżność < 0.01 K.

### Budżet błędów
$$\Delta T = \Delta T_\varepsilon + \Delta T_{\mathrm{atm}} + \Delta T_{\mathrm{win}}$$

---

## Funkcje interfejsu

| Funkcja | Opis |
|---------|------|
| Skala λ | Liniowa / logarytmiczna (przełącznik) |
| R(λ) na wykresie | Adaptacyjna siatka 500 pkt w zakresie czułości |
| Pasek UV+VIS | Gradient 380–700 nm na osi λ |
| Tooltip hover | λ [µm/nm], L_bb, ε·B, τ_atm, τ_win, Sygnał, R(λ) |
| Motywy | Ciemny / jasny |
| Responsywność | Desktop · tablet · telefon |
| Presety | Menu rozwijane (3 basic + 6 expert) |

---

## Dokumentacja wbudowana

**Tryb Podstawowy:** 4 sekcje w języku potocznym z analogiami i ćwiczeniami interaktywnymi.

**Tryb Eksperta:** 10 sekcji z wzorami KaTeX + bibliografia 15 pozycji [1–10 naukowe z DOI, 11–15 polskie/online].

---

## Nomenklatura

| Skrót | Znaczenie |
|-------|-----------|
| **CDC** | Ciało Doskonale Czarne (ang. *blackbody*, BB) |
| **CDB** | Ciało Doskonale Białe — NIE to samo! |

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

| Gałąź | Język |
|-------|-------|
| `main` | 🇵🇱 Polski |
| `en/english-translation` | 🇬🇧 English (v1.7.1, wymaga aktualizacji) |

---

## Historia wersji

| Wersja | Kluczowe zmiany |
|--------|----------------|
| **v2.0.0** | Tryb Podstawowy + Eksperta, nowa nazwa „Pirometria", menu presetów, dokumentacja dwujęzyczna, R(λ) na wykresie, poprawki zakresu Gaussów |
| v1.10.0 | Fizycznie poprawne zakresy Gaussów (±5σ), adaptacyjna siatka R(λ) |
| v1.9.0 | Literatura z DOI (15 pozycji), pola numeryczne T/ε |
| v1.8.x | Przesłona: grubość Beer-Lambert + temperatura + emisja własna |
| v1.7.x | Przesłona optyczna, budżet błędów 3-składowy |
| v1.5.0 | Pierwsza pełna wersja z atmosferą i dokumentacją |

Pełna historia: [CHANGELOG.md](CHANGELOG.md)

---

## Licencja

Materiał dydaktyczny — AGH Akademia Górniczo-Hutnicza w Krakowie. Użytek edukacyjny.
