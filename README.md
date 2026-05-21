# Symulator Toru Pirometrycznego

Interaktywna aplikacja edukacyjna symulująca fizyczny tor pomiaru temperatury metodą pirometryczną.

## Zawartość

Plik `symulator_pirometru.html` — standalone aplikacja HTML/JS (bez zależności zewnętrznych poza CDN).

## Model fizyczny

Tor pomiarowy składa się z 5 bloków:

```
[Obiekt] → [Atmosfera] → [Detektor] → [Procesor] → [Wynik T_ind]
```

### Prawa i wzory

- **Prawo Plancka** — spektralna luminancja CDC: `L_bb(λ,T) = 2hc²/λ⁵ · 1/(exp(hc/λk_BT)−1)`
- **Prawo Wiena** — przesunięcie maksimum: `λ_max · T = 2.898×10⁻³ m·K`
- **Prawo Stefana–Boltzmanna** — całkowita moc: `M = σ·T⁴`
- **Model atmosfery** — Beer-Lambert, 19 pasm absorpcji H₂O i CO₂ (uproszczony HITRAN)
- **Inwersja Plancka** — algorytm bisekcji (72 iteracje, zbieżność < 0.01 K)

### Typy detektorów

| Typ | Detektor | Zakres |
|-----|----------|--------|
| Pasmowy | Si | 0.65 µm |
| Pasmowy | InGaAs | 1.0 / 1.6 µm |
| Pasmowy | InSb | 3.9 µm |
| Okienkowy | InSb | 3–5 µm |
| Okienkowy | MCT/bolometr | 8–14 µm |
| Całkujący | Termostos | 1–18 µm |

## Funkcje aplikacji

- 🌡️ Zakres temperatury obiektu: 10 K – 12 000 K
- 🌫️ Model atmosfery: droga optyczna, wilgotność, CO₂, temperatura
- 📊 Wykresy widma w skali **liniowej i logarytmicznej** (przełącznik)
- 🌈 Pasek widma widzialnego (UV + VIS) na osi λ
- 🌙 Motyw ciemny / jasny
- 📖 Dokumentacja z formułami KaTeX
- 🔬 Tooltip hover z wartościami widma
- 📱 Responsywny układ (desktop / tablet / telefon)

## Presets demonstracyjne

| Preset | Opis |
|--------|------|
| Idealny | CDC, τ=1, ε_zał=ε_real → ΔT=0 |
| Błąd ε | Polerowana stal (ε=0.15) vs założona ε=0.90 |
| Atmosfera | Długa droga, wysoka wilgotność, CO₂×2 |
| Niska T | 200°C, detektor 8–14 µm |
| Całkujący | Pirometr całkujący, błąd emisyjności |

## Nomenklatura

- **CDC** = Ciało Doskonale Czarne (ang. *blackbody*, BB)
- **CDB** = Ciało Doskonale Białe (ang. *whitebody*) — NIE to samo!

## Autor

Materiał dydaktyczny do ćwiczeń laboratoryjnych z metrologii temperatury.
