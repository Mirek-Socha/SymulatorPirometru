# Pyrometric Measurement Chain Simulator

**Interactive educational application** demonstrating the physics of non-contact temperature measurement by pyrometry. Models the full measurement chain: object → atmosphere → optical window → detector → processor → result.

> Educational material for laboratory exercises in temperature metrology  
> **Mirosław Socha** · Department of Metrology and Electronics · WEAIiIB · AGH University of Krakow

---

## Running the application

The application is a **single standalone HTML file** — simply open `src/symulator_pirometru.html` in a browser. No server or installation required.

External resources loaded via CDN (internet connection required):
- [Google Fonts](https://fonts.google.com) — typography (IBM Plex Sans/Mono, Playfair Display)
- [KaTeX 0.16](https://katex.org) — mathematical formula rendering

---

## Measurement chain

```
┌──────────┐   ┌────────────┐   ┌────────┐   ┌──────────┐   ┌───────────┐   ┌────────┐
│  Object  │ → │ Atmosphere │ → │ Window │ → │ Detector │ → │ Processor │ → │ Result │
│  T, ε    │   │ L, RH, CO₂│   │τ_win(λ)│   │  R(λ)    │   │  ε_ass.   │   │ T_ind  │
└──────────┘   └────────────┘   └────────┘   └──────────┘   └───────────┘   └────────┘
```

Signal at the detector is described by the general radiation transport equation:

$$L_{\mathrm{det}}(\lambda) = \tau_{\mathrm{win}}(\lambda) \cdot \left[ \tau_{\mathrm{atm}}(\lambda)\cdot\varepsilon\cdot L_{bb}(\lambda,T) + \left(1-\tau_{\mathrm{atm}}(\lambda)\right)\cdot L_{bb}(\lambda,T_{\mathrm{atm}}) \right]$$

---

## Physical model

### Thermal radiation laws

| Law | Formula | Application |
|-----|---------|-------------|
| **Planck** | $L_{bb}(\lambda,T)=\frac{2hc^2}{\lambda^5}\cdot\frac{1}{e^{hc/\lambda k_B T}-1}$ | BB spectrum |
| **Wien displacement** | $\lambda_{\max}\cdot T = 2{,}898\times10^{-3}\ \mathrm{m{\cdot}K}$ | Peak emission |
| **Stefan–Boltzmann** | $M = \sigma T^4$ | Total-radiation pyrometer |
| **Kirchhoff** | $\varepsilon(\lambda,T)=\alpha(\lambda,T)$ | Emissivity = absorptivity |

### Atmosphere model — Beer–Lambert (simplified HITRAN)

- **H₂O**: 12 absorption bands (0.72–13.4 µm)
- **CO₂**: 7 bands (dominant: 4.26 µm)
- **Rayleigh scattering** in the visible range

### Optical windows — 6 materials

| Material | Range [µm] | τ_peak | Notes |
|----------|-----------|--------|-------|
| Float glass (SiO₂) | 0.32 – 2.7 | 0.91 | OH bands 1.4 / 1.9 µm |
| Plexiglas (PMMA) | 0.35 – 2.8 | 0.92 | C-H bands 1.15 / 1.41 / 1.69 / 2.18 µm |
| Fused quartz | 0.15 – 4.5 | 0.945 | weak OH band 2.72 µm |
| Calcium fluoride (CaF₂) | 0.13 – 9.5 | 0.944 | no significant bands |
| Zinc selenide (ZnSe) | 0.55 – 16 | 0.70 | n ≈ 2.4, Fresnel losses |
| Germanium (Ge) | 1.8 – 16 | 0.46 | blocks VIS/UV; n ≈ 4 |

### Detectors — 7 types

| Type | Detector | Range / centre |
|------|----------|---------------|
| Spectral (Gaussian) | Si | 0.65 µm |
| Spectral (Gaussian) | InGaAs | 1.0 µm |
| Spectral (Gaussian) | InGaAs | 1.6 µm |
| Spectral (Gaussian) | InSb | 3.9 µm |
| Band-pass | InSb | 3–5 µm |
| Band-pass | MCT / bolometer | 8–14 µm |
| Total-radiation | Thermopile | 1–18 µm |

### Temperature inversion

The pyrometer finds T_ind such that a simplified model (τ=1, ε=ε_ass) produces the same signal as the full chain:

$$\int_0^\infty R(\lambda)\cdot\varepsilon_{\mathrm{ass}}\cdot L_{bb}(\lambda,T_{\mathrm{ind}})\,\mathrm{d}\lambda = S_{\mathrm{measured}}$$

Algorithm: **bisection, 72 iterations**, range 1–15 000 K, convergence < 0.01 K.

### Error budget — 3 independent components

$$\Delta T = \underbrace{\Delta T_\varepsilon}_{\text{emissivity}} + \underbrace{\Delta T_{\mathrm{atm}}}_{\text{atmosphere}} + \underbrace{\Delta T_{\mathrm{win}}}_{\text{window}}$$

---

## Application features

| Feature | Description |
|---------|-------------|
| 🌡️ Temperature range | 10 K – 12 000 K (slider + numeric input with °C conversion) |
| 📊 λ-axis scale | Linear (textbook spectrum shape) / logarithmic (UV+VIS+IR together) |
| 🌈 Spectrum bar | UV (100–380 nm) + VIS (380–700 nm) strip on λ-axis of both plots |
| 🌙 Theme | Dark / light (toggle button) |
| 📖 Documentation | Side panel with 9 sections, KaTeX equations |
| 🔬 Hover tooltip | L(λ), ε·B(λ), τ_atm(λ), τ_win(λ), R(λ) at any wavelength |
| 📱 Responsive | 3 layouts: desktop (≥1100 px) · tablet · phone |

### Demonstration presets

| Preset | Scenario | Key effect |
|--------|---------|-----------|
| Ideal | BB, τ=1, ε_ass=ε_real | ΔT = 0 — reference case |
| ε Error | Polished steel ε=0.15, assumed ε=0.90 | Large ΔT_ε |
| Atmosphere | 25 m, RH=80%, CO₂×2 | CO₂ absorption at 4.26 µm visible |
| Low T | 200°C, MCT 8–14 µm detector | Low-temperature measurement |
| Integrating | Thermopile, wrong ε | T⁴ sensitivity to emissivity |
| 🪟 Plexiglas | PMMA sight window, InGaAs 1.0 µm | ΔT_win ≈ −210 K, C-H bands visible |

---

## Repository structure

```
SymulatorPirometru/
├── src/
│   └── symulator_pirometru.html   # standalone app (HTML + CSS + JS)
├── CHANGELOG.md
├── README.md
└── .gitignore
```

## Branches

| Branch | Language |
|--------|----------|
| `main` | 🇵🇱 Polish |
| `en/english-translation` | 🇬🇧 English (this branch) |

---

## Version history

| Version | Key changes |
|---------|------------|
| **v1.7.1-en** | English translation |
| v1.7.1 | Fix: canvas collapse in Opera |
| v1.7.0 | Optical window, 3-component error budget, numeric T input |
| v1.6.0 | KaTeX fixes, BB abbreviation corrected |
| v1.5.0 | UV+VIS bar, 10–12000 K range, log/lin scale, KaTeX |
| v1.0–1.4 | Initial versions, atmosphere, detectors, themes |

Full history: [CHANGELOG.md](CHANGELOG.md)

---

## Licence

Educational material — AGH University of Krakow.
