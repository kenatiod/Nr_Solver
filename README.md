# Nr_Solver

Number theory research program by Ken Clements (Feb 2026).

## Overview

`Nr_Solver.py` enumerates and verifies candidates for **prime-complete products of consecutive integers**:

```
N_r(m) = m · (m+1) · ··· · (m+r-1)
```

For a fixed prime set **P = {p₁, ..., pω}**, the program finds all m such that N_r(m) is P-smooth and **prime-complete** — meaning the prime support of N_r(m) is exactly P (no missing primes, no extra primes).

The primary use case is r=2 (consecutive pairs m, m+1), which is the setting for investigating the conjecture that **n=633,555 gives the last prime-complete product n(n+1)**.

## Mathematical Framework

The program implements the **Størmer-Lehmer reduction**:

Every pair (m, m+1) of P-smooth integers arises as `m = (x−1)/2` where (x, y) solves a Pell equation:

```
x² − 2q·y² = 1
```

for some squarefree q whose prime factors are all in P. The squarefree q values are encoded by bitmasks over P (all 2^ω masks are enumerated). For each mask, the Pell equation is solved via continued fractions in PARI/GP, and the resulting candidate m values are filtered for P-smoothness and prime-completeness.

**Key quantities:**
- **ω** — number of distinct prime divisors (size of prime set P)
- **Pidx** — prime index of the greatest prime divisor of N_r(m)
- **delta = Pidx − ω** — count of "missing" primes; zero iff prime-complete
- **L = max(3, (pmax+1)//2)** — Størmer-Lehmer iterate bound per Pell equation

## Algorithm

1. Enumerate all 2^ω squarefree masks over P
2. For each mask, solve x²−2qy²=1 in PARI/GP using continued fractions
3. Generate up to L Pell iterates; each gives a candidate m = (x−1)/2
4. Filter: m and m+1 must both be P-smooth
5. Verify: prime support of m·(m+1) must equal P exactly (prime-completeness)

**Pruning options:**
- `--max_m`: skip masks where `q > 2·max_m·(max_m+1)` (provably no solution ≤ max_m)
- `--nze_pruning`: require full prime coverage in the Pell inner loop (NZE filter)

## Completeness Certification

For referee-grade runs, every output JSON records:

- `complete_enumeration: true/false` — `unprocessed_masks == 0`
- SHA256 of all artifacts (S-file, verify CSV, log)
- Python version, PARI/GP version, script SHA256
- Full command line and run parameters

## Installation

Requires Python 3.9+ and PARI/GP.

```bash
# macOS
brew install pari

# Linux
sudo apt install pari-gp
```

## Usage

```bash
python3 Nr_Solver.py [options]
```

**Key options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--r` | 2 | Consecutive integer count (2=pairs, 3=triples, ...) |
| `--start_omega` | 8 | First ω to process |
| `--end_omega` | 20 | Last ω to process |
| `--workers` | cpu_count | Parallel worker processes |
| `--outdir` | auto | Output directory |
| `--max_m` | 0 (disabled) | Upper bound for provably complete enumeration |
| `--nze_pruning` | 0 (disabled) | Enable π-complete-only filter at ω ≥ this value |
| `--full_report_omega_max` | 20 | Write full S-file + verify CSV only for ω ≤ this |
| `--no_incremental` | off | Disable incremental mode (always recompute) |
| `--gp_path` | gp | Path to PARI/GP binary |

**Example — full audit sweep (ω = 8 to 20):**

```bash
python3 Nr_Solver.py --start_omega 8 --end_omega 20 --workers 10
```

**Example — bounded enumeration (provably complete up to max_m):**

```bash
python3 Nr_Solver.py --start_omega 8 --end_omega 20 --max_m 1000000 --workers 10
```

## Output

For each (r, ω), the program writes to `outdir/r_{r}/omega_{ω:02d}/`:

| File | Description |
|------|-------------|
| `S_p{pmax}_r{r}_omega_{ω}.txt` | All candidate m values (P-smooth pairs) |
| `verify_r{r}_omega_{ω}.csv` | Per-m factorizations and prime-complete flag |
| `summary_r{r}_omega_{ω}.json` | Run metadata, SHA256 hashes, completeness flag |
| `run_r{r}_omega_{ω}.log` | Progress log with timestamps |

## Versioning

Git tags mark each released version. This repository begins at **v16**, the first version with:
- Full audit trail (SHA256, environment block, completeness flags)
- Provably correct `--max_m` early rejection
- NZE pruning option
- Incremental (ω → ω+1) enumeration mode
- Multiprocessing with retry/requeue for failed masks
- PARI/GP startup handshake verification

## Background

The primorial `p_k#` is the product of the first k primes. The program searches for m such that m and m+1 together cover all primes up to p_ω — i.e., N₂(m) = m(m+1) is prime-complete. The last known prime-complete value is n=633,555:

```
633,555 = 3³ × 5 × 13 × 19²
633,556 = 2² × 7 × 11³ × 17
```

Prime support = {2, 3, 5, 7, 11, 13, 17, 19} = P₈, omega=8, Pidx=8, delta=0.

## License

MIT License — see [LICENSE](LICENSE).
