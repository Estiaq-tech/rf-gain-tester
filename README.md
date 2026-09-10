# Automated RF Gain-vs-Frequency Tester (LabVIEW)

A LabVIEW automated test system that sweeps an RF gain block across frequency,
measures gain, applies pass/fail limits, and produces traceable per-unit results
and a running test log. Built as a portfolio project to demonstrate test &
measurement automation practice.

The device under test (DUT) is a **simulated RF gain block** so the project runs
with no instruments. The measurement is isolated behind a single SubVI, so the
same sweep/limits/reporting logic drives real hardware once the simulated DUT is
swapped for VISA/SCPI instrument calls (see [Roadmap](#roadmap)).

## Demo

[![Watch the demo](https://img.youtube.com/vi/N4l0R24y6hk/hqdefault.jpg)]
 
(https://youtu.be/N4l0R24y6hk)

A walkthrough in three parts: a FAIL run in slow motion (Highlight Execution)
showing the state machine step through Init → Run → Idle; full-speed runs with
changed limits (pass + fail); and the traceable master test log.

---

## What it does

1. **Configurable sweep** — reads start / stop / step frequency and computes the
   point count; runs a 900–2400 MHz sweep by default.
2. **Measurement** — at each frequency, calls `Measure_Gain.vi`, which models a
   realistic amplifier response (flat passband ~20 dB, rolling off to ~17 dB at
   the band edges) plus measurement noise.
3. **Pass/Fail** — compares every measured point against min/max gain limits,
   flags out-of-limit points, and rolls up to a single unit verdict.
4. **Logging & traceability** — saves the full sweep to a per-unit CSV named by
   serial number, and appends a timestamped record (serial, verdict, fail count)
   to a master `test_log.csv`.

## Architecture

The final application (`Sweep_Test_statemachine.vi`) is built as a **state
machine**: a `While` loop + `Case` structure driven by a typedef'd state enum
(`States.ctl`), with the state and the error cluster carried through shift
registers, and a stop condition gated on both completion and error status.

```
[ Init ] --> [ Run ] --> [ Idle ] --> stop
                |
   sweep -> measure -> evaluate limits -> write results -> append log
                |
   error cluster threaded through file I/O (ordered, fail-safe)
```

This makes the control flow explicit and safe to extend — a new test step is a
new state, not a rewrite.

## Repository contents

| File | Description |
|------|-------------|
| `Sweep_Test_statemachine.vi` | **Main VI** — the finished state-machine tester |
| `Measure_Gain.vi` | Simulated DUT: frequency in → gain (dB) out, with noise |
| `States.ctl` | Typedef enum of machine states (Init, Run, Idle) |
| `First_Light.vi` | Phase 1 learning VI (live noisy waveform) |
| `Sweep_Test.vi` | Basic sweep engine (pre-refactor) |
| `Sweep_Test_pass_fail.vi` | Sweep + limits/pass-fail stage |
| `Sweep_Test_logging.vi` | Sweep + limits + file logging stage |
| `DUT-00x.csv` | Example per-unit sweep results (frequency, gain) |
| `test_log.csv` | Example master test log |
| `limits.csv` | Per-band limit spec (reference for the data-driven-limits roadmap item; not yet read by the VI, which uses flat min/max controls) |

The intermediate `Sweep_Test_*.vi` files are kept to show the project's
build-up; the state-machine VI is the one to open.

## How to run

Requires **LabVIEW 2021+ (Community Edition works)**.

1. Open `Sweep_Test_statemachine.vi`.
2. On the front panel set **Start/Stop/Step (MHz)**, **Gain Min/Max (dB)**, and a
   **Serial Number**.
3. Run. The gain curve plots, the PASS/FAIL verdict and fail count update, a
   `DUT-<serial>.csv` is written, and a row is appended to `test_log.csv`.

Tighten the gain limits to see a unit correctly fail.

## Example output

`test_log.csv`:

```
7.9.2026 16.45,DUT-004,PASS,0
7.9.2026 16.46,DUT-005,FAIL,136
7.9.2026 16.52,DUT-005,PASS,0
```

`DUT-001.csv` (frequency MHz, gain dB):

```
900.000,16.885
902.000,17.195
904.000,17.099
```

## Skills demonstrated

- Automated measurement sequencing and configurable sweeps
- Limit-based pass/fail evaluation with a rolled-up unit verdict
- State-machine architecture with typedef'd state enum
- Error-cluster propagation for ordered, fail-safe file I/O
- Data logging and serial-number traceability
- Modular design (measurement isolated behind one SubVI for hardware swap-in)

## Roadmap

- **Real instruments** — replace `Measure_Gain.vi` internals with VISA/SCPI calls
  to a signal generator + power meter (`gain = P_out − P_in`); the rest of the
  application is unchanged.
- **Per-band limits from file** — load `limits.csv` (per-band min/max) instead of
  flat min/max controls, for data-driven test specs.
- **HTML/PDF report** — generate a formatted per-unit report alongside the CSV.

---

*Simulated-hardware portfolio project. Not affiliated with NI.*
