# Changelog — Symulator Toru Pirometrycznego

Format: [Keep a Changelog](https://keepachangelog.com/pl/1.0.0/)

---

## [v1.7.0] — 2026-05-21 ✅ bieżąca

### Dodano
- **Przesłona optyczna** — nowy blok 3 w torze pomiarowym z 6 materiałami:
  szyba float, Plexiglas (PMMA), kwarc topiony (SiO₂), fluoryt (CaF₂),
  selenek cynku (ZnSe), german (Ge)
- Spektralne modele transmitancji: smoothstep na krawędziach UV/IR
  + gaussowskie pasma absorpcji (dane NIST/literature)
- Budżet błędów rozszerzony do **3 składowych**: Δε, ΔT_atm, ΔT_win
- Pole numeryczne do bezpośredniego wpisania temperatury w K (+ przelicznik °C)
- Preset **🪟 Plexiglas** — pomiar przez wziernik PMMA, detektor InGaAs 1.0 µm
- Krzywa τ_win(λ) na wykresie widma (przerywana, kolor per materiał)
- Dokumentacja: sekcja 8 (teoria przesłony, wzory, tabela materiałów)
- Dokumentacja: sekcja 9 (autor, wersja, repozytorium)
- Nagłówek toru: dodana pozycja "Przesłona"

### Poprawiono
- Kursor (krzyżyk hover) na wykresie widma w skali **liniowej** —
  używał błędnie formuły logarytmicznej; linia i dymek były rozsynchronizowane
- Usunięty zduplikowany blok Atmosfera w schemacie blokowym
- Schemat blokowy ma teraz poprawnie 6 bloków i 5 strzałek

---

## [v1.6.0] — 2026-05-21

### Poprawiono
- Skrót **CDC** (Ciało Doskonale Czarne) zamiast błędnego CDB (= Białe)
- Ostrzeżenia KaTeX: polskie litery i µ usunięte z trybu matematycznego

---

## [v1.5.0] — 2026-05-21

### Dodano
- Pasek widma **UV** (100–380 nm) + VIS (380–700 nm) na obu wykresach
- Zakres temperatury: **10 K – 12 000 K** (suwak w Kelwinach)
- Przełącznik skali osi λ: **liniowa / logarytmiczna**
- Grubsze strzałki w schemacie blokowym
- Formuły matematyczne renderowane przez **KaTeX**

---

## [v1.4.0] — 2026-05-21

### Dodano
- Przełącznik **motywu jasnego / ciemnego**
- Przebudowa schematu blokowego (karty HTML + animowane strzałki)
- Panel **dokumentacji** z 7 sekcjami merytorycznymi
- Responsywność: 3 układy (desktop / tablet / telefon)

### Poprawiono
- `tv()` czyta CSS vars z `body` — poprawne kolory w obu motywach
- Jawne wypełnienie tła kanwy

---

## [v1.3.0] — 2026-05-21

### Dodano
- Pasek widma widzialnego z gradientem tęczy

---

## [v1.2.0] — 2026-05-21

### Dodano
- Schemat blokowy toru pomiarowego
- Panel wyników z dekompozycją błędu
- Tooltip hover z wartościami widmowymi
- 5 presetów demonstracyjnych

---

## [v1.1.0] — 2026-05-21

### Dodano
- Model atmosfery (H₂O, CO₂, 19 pasm)
- 7 typów detektorów

---

## [v1.0.0] — 2026-05-21

### Dodano
- Pierwsza wersja symulatora — prawa Plancka, Wien, Stefan-Boltzmann
