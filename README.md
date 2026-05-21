# Symulator Toru Pirometrycznego

**Interaktywna aplikacja edukacyjna** do demonstracji fizyki bezdotykowego pomiaru temperatury metodą pirometryczną. Modeluje pełny tor pomiarowy: obiekt → atmosfera → przesłona optyczna → detektor → procesor → wynik.

> Materiał dydaktyczny do ćwiczeń laboratoryjnych z metrologii temperatury  
> **Mirosław Socha** · Katedra Metrologii i Elektroniki · WEAIiIB · AGH Kraków

---

## Uruchomienie

Aplikacja jest **jednym samodzielnym plikiem HTML** — wystarczy otworzyć `src/symulator_pirometru.html` w przeglądarce. Nie wymaga serwera ani instalacji.

Zewnętrzne zasoby ładowane przez CDN (wymagane połączenie z internetem):
- [Google Fonts](https://fonts.google.com) — typografia (IBM Plex Sans/Mono, Playfair Display)
- [KaTeX 0.16](https://katex.org) — renderowanie wzorów matematycznych

---

## Tor pomiarowy

```
┌──────────┐   ┌───────────┐   ┌───────────┐   ┌──────────┐   ┌──────────┐   ┌────────┐
│  Obiekt  │ → │ Atmosfera │ → │ Przesłona │ → │ Detektor │ → │ Procesor │ → │ Wynik  │
│  T, ε    │   │ L, HR, CO₂│   │ τ_win(λ)  │   │  R(λ)    │   │ ε_zał.   │   │ T_ind  │
└──────────┘   └───────────┘   └───────────┘   └──────────┘   └──────────┘   └────────┘
```

Sygnał na detektorze opisany ogólnym równaniem transportu promieniowania:

$$L_{\mathrm{det}}(\lambda) = \tau_{\mathrm{win}}(\lambda) \cdot \left[ \tau_{\mathrm{atm}}(\lambda)\cdot\varepsilon\cdot L_{bb}(\lambda,T) + \left(1-\tau_{\mathrm{atm}}(\lambda)\right)\cdot L_{bb}(\lambda,T_{\mathrm{atm}}) \right]$$

---

## Model fizyczny

### Prawa promieniowania termicznego

| Prawo | Wzór | Zastosowanie |
|-------|------|--------------|
| **Plancka** | $L_{bb}(\lambda,T) = \frac{2hc^2}{\lambda^5} \cdot \frac{1}{e^{hc/\lambda k_B T}-1}$ | Widmo CDC |
| **Wiena** | $\lambda_{\max} \cdot T = 2{,}898 \times 10^{-3}\ \mathrm{m{\cdot}K}$ | Maksimum emisji |
| **Stefana–Boltzmanna** | $M = \sigma T^4$ | Pirometr całkujący |
| **Kirchhoffa** | $\varepsilon(\lambda,T) = \alpha(\lambda,T)$ | Emisyjność = absorpcyjność |

### Model atmosfery — Beer–Lambert (uproszczony HITRAN)

- **H₂O**: 12 pasm absorpcji (0.72–13.4 µm)
- **CO₂**: 7 pasm (dominujące: 4.26 µm)
- **Rozpraszanie Rayleigha** w zakresie widzialnym

### Przesłony optyczne — 6 materiałów

| Materiał | Zakres [µm] | τ_peak | Uwagi |
|----------|------------|--------|-------|
| Szyba float (SiO₂) | 0.32 – 2.7 | 0.91 | pasma OH 1.4 / 1.9 µm |
| Plexiglas (PMMA) | 0.35 – 2.8 | 0.92 | pasma C-H 1.15 / 1.41 / 1.69 / 2.18 µm |
| Kwarc topiony | 0.15 – 4.5 | 0.945 | słabe pasmo OH 2.72 µm |
| Fluoryt (CaF₂) | 0.13 – 9.5 | 0.944 | brak istotnych pasm |
| Selenek cynku (ZnSe) | 0.55 – 16 | 0.70 | n ≈ 2.4, straty Fresnela |
| German (Ge) | 1.8 – 16 | 0.46 | blokuje VIS/UV; n ≈ 4 |

Model transmitancji: smoothstep na krawędziach UV/IR + gaussowskie pasma absorpcji.

### Detektory — 7 typów

| Typ | Detektor | Zakres / centrum |
|-----|----------|-----------------|
| Pasmowy (gauss) | Si | 0.65 µm |
| Pasmowy (gauss) | InGaAs | 1.0 µm |
| Pasmowy (gauss) | InGaAs | 1.6 µm |
| Pasmowy (gauss) | InSb | 3.9 µm |
| Okienkowy | InSb | 3 – 5 µm |
| Okienkowy | MCT / bolometr | 8 – 14 µm |
| Całkujący | Termostos | 1 – 18 µm |

### Inwersja temperatury

Pirometr wyznacza T_ind szukając temperatury, dla której model uproszczony (τ=1, ε=ε_zał) daje ten sam sygnał co pełny tor:

$$\int_0^\infty R(\lambda)\cdot\varepsilon_{\mathrm{zal}}\cdot L_{bb}(\lambda,T_{\mathrm{ind}})\,\mathrm{d}\lambda = S_{\mathrm{zmierzony}}$$

Algorytm: **bisekcja 72 iteracje**, zakres 1–15 000 K, zbieżność < 0.01 K.

### Budżet błędów — 3 niezależne składowe

$$\Delta T = \underbrace{\Delta T_\varepsilon}_{\text{emisyjność}} + \underbrace{\Delta T_{\mathrm{atm}}}_{\text{atmosfera}} + \underbrace{\Delta T_{\mathrm{win}}}_{\text{przesłona}}$$

---

## Funkcje aplikacji

| Funkcja | Opis |
|---------|------|
| 🌡️ Zakres T | 10 K – 12 000 K (suwak + pole numeryczne, przelicznik °C) |
| 📊 Skala osi λ | Liniowa (kształt z podręcznika) / logarytmiczna (UV+VIS+IR razem) |
| 🌈 Pasek widma | UV (100–380 nm) + VIS (380–700 nm) na osi λ obu wykresów |
| 🌙 Motyw | Ciemny / jasny (przełącznik) |
| 📖 Dokumentacja | Panel boczny z 9 sekcjami, wzory KaTeX |
| 🔬 Tooltip hover | Wartości L(λ), ε·B(λ), τ_atm(λ), τ_win(λ), R(λ) w dowolnym punkcie |
| 📱 Responsywność | 3 układy: desktop (≥1100 px) · tablet · telefon |

### Presets demonstracyjne

| Preset | Scenariusz | Kluczowy efekt |
|--------|-----------|---------------|
| Idealny | CDC, τ=1, ε_zał=ε_real | ΔT = 0 — punkt odniesienia |
| Błąd ε | Stal polerowana ε=0.15, założono ε=0.90 | Duże ΔT_ε — błąd emisyjności |
| Atmosfera | 25 m, HR=80%, CO₂×2 | Widoczna absorpcja 4.26 µm CO₂ |
| Niska T | 200°C, detektor MCT 8–14 µm | Pomiar niskotemperaturowy |
| Całkujący | Termostos, zła ε | Czułość T⁴ na emisyjność |
| 🪟 Plexiglas | Wziernik PMMA, InGaAs 1.0 µm | ΔT_win ≈ −210 K, pasma C-H |

---

## Nomenklatura

| Skrót | Pełna nazwa | Angielski odpowiednik |
|-------|------------|----------------------|
| **CDC** | Ciało Doskonale Czarne | blackbody (BB) |
| **CDB** | Ciało Doskonale Białe | whitebody |

> ⚠️ CDC ≠ CDB — nie mylić!

---

## Struktura repozytorium

```
SymulatorPirometru/
├── src/
│   └── symulator_pirometru.html   # standalone aplikacja (HTML + CSS + JS)
├── CHANGELOG.md                    # historia wersji
├── README.md
└── .gitignore
```

---

## Historia wersji

| Wersja | Najważniejsze zmiany |
|--------|---------------------|
| **v1.7.1** | Poprawka: zanikanie wykresów w Opera (jawna wysokość CSS canvas) |
| v1.7.0 | Przesłona optyczna, budżet błędów 3-składowy, pole numeryczne T |
| v1.6.0 | Poprawki KaTeX, CDC zamiast CDB |
| v1.5.0 | Pasek UV+VIS, zakres 10–12000 K, skala log/lin, KaTeX |
| v1.4.0 | Motyw jasny/ciemny, dokumentacja, responsywność |
| v1.0–1.3 | Pierwsza wersja, model atmosfery, schematy, widmo VIS |

Pełna historia: [CHANGELOG.md](CHANGELOG.md)

---

## Licencja

Materiał dydaktyczny do użytku edukacyjnego — AGH Akademia Górniczo-Hutnicza w Krakowie.
