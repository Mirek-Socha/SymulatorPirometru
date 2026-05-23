# Pyrometry — Measurement Chain Simulator

**Interactive educational application** for demonstrating the physics of non-contact temperature measurement by pyrometry. Models the full measurement chain with two operation modes.

> Educational material — laboratory exercises in temperature metrology  
> **Mirosław Socha** · Department of Metrology and Electronics · WEAIiIB · AGH University of Krakow

---

## Running the application

Single HTML file — open `src/symulator_pirometru.html` in a browser. No installation, no server.

---

## Two operation modes

### 🎓 Basic Mode (default)
For students new to pyrometry:
- Temperature in °C (slider + numeric field)
- 4 emissivity preset buttons (polished metal / oxidised steel / ceramics / BB)
- 3 atmosphere buttons (none / short / long path)
- 3 pyrometer types (near IR / mid IR / far IR)
- Single spectrum chart with 3 curves
- Traffic-light result indicator 🟢🟡🔴
- Built-in guide "How does a pyrometer work?" (4 sections)

### 🔬 Expert Mode (button in header)
Full physical model:
- Temperature range: 10 K – 12 000 K
- 4 atmosphere sliders (L, RH, CO₂, T_atm)
- Optical window: 6 materials with thickness and temperature
- 7 detector types
- Two charts: spectrum + atmospheric transmittance τ(λ)
- 3-component error budget (Δε, ΔT_atm, ΔT_win)
- Documentation with 10 sections and bibliography (15 references)

---

## Measurement chain

```
[Object T,ε] → [Atmosphere τ_atm(λ)] → [Window τ_win(λ)] → [Detector R(λ)] → [Processor] → T_ind
```

---

## Branches

| Branch | Language | Version |
|--------|----------|---------|
| `main` | 🇵🇱 Polish | v2.0.0 |
| `en/english-translation` | 🇬🇧 English | v2.0.0-en |

---

## Licence

Educational material — AGH University of Krakow.
