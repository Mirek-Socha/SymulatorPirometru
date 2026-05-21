# Changelog — Symulator Toru Pirometrycznego

Wszystkie istotne zmiany w projekcie. Format: [Keep a Changelog](https://keepachangelog.com/pl/1.0.0/).

---

## [v1.5.0] — 2026-05-21 ✅ bieżąca

### Poprawiono
- Skrót **CDC** (Ciało Doskonale Czarne) zamiast błędnego CDB (które oznacza Białe)
- Ostrzeżenia KaTeX: polskie litery (`ł`, `ż`) i znak `µ` usunięte z trybu matematycznego
- Wszystkie formuły w `\text{}` używają teraz czystego ASCII

---

## [v1.4.0] — 2026-05-21

### Dodano
- Pasek widma **UV** (100–380 nm) przed paskiem VIS na obu wykresach
- Zakres temperatury: **10 K – 12 000 K** (suwak w Kelwinach)
- Skala osi λ: **przełącznik liniowa / logarytmiczna** (domyślnie liniowa)
- Grubsze strzałki w schemacie blokowym (stroke-width 3.5, groty 22×17 px)
- Formuły w dokumentacji renderowane przez **KaTeX** (LaTeX)

### Zmieniono
- Zakres spektralny: 0.10–16 µm (wcześniej 0.40–16 µm)
- Grid spektralny N=700 punktów (wcześniej 480)
- Algorytm inwersji: bisekcja 72 iteracje, zakres 1–15 000 K

---

## [v1.3.0] — 2026-05-21

### Dodano
- Pasek widma **widzialnego** (380–700 nm) z gradientem tęczy na obu wykresach
- Etykieta "VIS", "IR →", tick 0.38 µm i 0.70 µm

---

## [v1.2.0] — 2026-05-21

### Dodano
- Przełącznik **motywu jasnego / ciemnego** (przycisk 🌙/☀️)
- Przebudowa schematu blokowego: 5 kart HTML z animowanymi strzałkami
- Panel **dokumentacji** (przycisk 📖) z 7 sekcjami merytorycznymi
- Responsywność: 3 układy (≥1100px / 700–1099px / <700px)

### Poprawiono
- `tv()` czyta CSS vars z `body` zamiast `html` — poprawne kolory w obu motywach
- Jawne wypełnienie tła kanwy — widoczne w motywie jasnym
- `globalAlpha` zamiast konkatenacji hex dla przezroczystości

---

## [v1.1.0] — 2026-05-21

### Dodano
- Schemat blokowy toru pomiarowego (SVG)
- Panel wyników z dekompozycją błędu (Δε, Δatm)
- Tooltip hover ze spektralnymi wartościami L(λ), τ(λ), R(λ)
- 5 presetów demonstracyjnych

---

## [v1.0.0] — 2026-05-21

### Dodano
- Pierwsza wersja interaktywnego symulatora
- Model Plancka, Wien, Stefan-Boltzmann
- Uproszczony model atmosfery (H₂O, CO₂)
- 7 typów detektorów
- Wykresy: widmo spektralne + transmitancja τ(λ)
