# Experimental plan

## Phase 0: Reproduce the known-good SDR path

Before touching ISDB-T frequencies, prove the capture chain works.

### Test 0A: Build and flash

- Build the chosen ESP-SDR revision for ESP32-C3.
- Record toolchain and ESP-IDF versions.
- Record board/module identity and flash/PSRAM details.
- Save the exact upstream commit SHA.

PASS:
- firmware boots normally
- raw I/Q capture command completes
- captured data size matches expectation

### Test 0B: Known 2.4 GHz signal

Use a known Wi-Fi source or controlled RF source.

PASS:
- changing the known transmitter/channel causes a repeatable corresponding change in captured spectrum

This establishes that later failures are not simply firmware, USB/UART, host, plotting, or antenna mistakes.

---

## Phase 1: Direct UHF experiment on ESP32-C3

### Objective

Determine whether an ESP32-C3 can receive meaningful RF energy in the Japanese terrestrial ISDB-T UHF band directly, without an external mixer.

### Important distinction

A frequency-setting API accepting a value is **not** evidence of RF tuning.

We need evidence for:

1. LO / PLL behavior
2. RF signal response
3. reproducibility

### Test 1A: Noise-floor sweep

Sweep selected frequencies from the normal 2.4 GHz region downward toward UHF.

Record:

- requested center frequency
- actual observable response
- gain setting
- capture statistics
- spectrum image / raw sample file where useful

Look for discontinuities that may indicate PLL limits or fixed-frequency behavior.

### Test 1B: Known UHF source

Preferred order:

1. laboratory RF signal generator, if available
2. legal low-power / shielded test source
3. strong known broadcast signal as a passive source

PASS:
- moving the known source produces a frequency-correlated, repeatable response in the capture

FAIL:
- requested frequency changes do not produce corresponding RF response
- response remains tied to ~2.4 GHz behavior
- no measurable response to a strong known UHF source under otherwise valid conditions

INCONCLUSIVE:
- unexplained peaks with no controlled correlation

### Test 1C: ISDB-T channel observation

After a local physical TV channel is selected, look for the characteristic approximately 6 MHz-wide OFDM occupancy.

PASS requires more than a rectangle in a waterfall. At least one of:

- channel center changes track real broadcast channels
- antenna disconnect/reconnect changes the candidate signal as expected
- attenuation changes produce corresponding level changes
- a reference SDR sees the same channel at the same time

---

## Phase 2: Upconverter fallback

If Phase 1 fails, do not force the C3 RF front-end outside its useful range.

Translate one UHF TV channel into the ESP32-C3 2.4 GHz receive region.

Concept:

```
UHF ISDB-T
   |
   v
preselector / filter
   |
   v
mixer + LO
   |
   v
~2.4 GHz IF
   |
   v
ESP32-C3 raw I/Q
```

Requirements:

- at least 6 MHz usable instantaneous bandwidth
- LO stability adequate for OFDM experiments
- filtering sufficient to distinguish desired conversion product
- gain chosen to avoid ESP32 front-end overload

First objective is spectrum visibility, not decoding.

---

## Phase 3: PC-side ISDB-T demodulation

Once continuous or sufficiently useful I/Q is available:

1. frequency correction
2. sample-rate conversion / decimation
3. symbol timing / guard interval handling
4. OFDM synchronization
5. FFT
6. TMCC extraction
7. segment extraction
8. QPSK / 16QAM / 64QAM demapping as applicable
9. deinterleaving
10. Viterbi decoding
11. Reed-Solomon
12. MPEG-TS recovery

Do this on the PC first.

The ESP32 is not considered the demodulator until the RF path and reference demodulation are proven.

---

## Phase 4: Move processing onto ESP32

Possible candidates for offload or incremental migration:

- decimation
- channel filtering
- AGC assistance
- coarse frequency correction
- guard-interval correlation
- symbol framing
- selected FFT / DSP stages if practical

ESP32-C5 should be evaluated here because newer peripheral paths and BitScrambler-style techniques may offer materially better continuous-processing options than C3.

---

## Repository layout proposal

```
.
├── README.md
├── docs/
│   ├── PLAN.md
│   ├── REFERENCES.md
│   └── hardware/
├── firmware/
│   ├── c3/
│   └── c5/
├── host/
│   ├── capture/
│   └── analysis/
├── experiments/
│   └── YYYYMMDD-name/
└── captures/
    └── README.md
```

Large raw I/Q files should not be committed blindly. Store metadata and hashes, and use releases or external storage only when needed.

---

## First experiment definition

**EXP-001: ESP32-C3 direct UHF feasibility**

Required outputs:

- exact board
- upstream firmware SHA
- build instructions
- known-good 2.4 GHz baseline
- one or more UHF test frequencies
- raw/spectrum evidence
- PASS / FAIL / INCONCLUSIVE
- short conclusion describing what the result actually proves
