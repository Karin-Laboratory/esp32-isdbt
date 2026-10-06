# esp32-isdbt

Experimental ISDB-T / SDR receiver research using ESP32 raw I/Q capture.

## Goal

Explore whether ESP32-C3/C5 Wi-Fi RF hardware can be repurposed as an SDR front-end for Japanese terrestrial digital television (ISDB-T), including both 1seg and full-seg experiments.

This project explicitly records both successful and unsuccessful experiments. A clean negative result is useful evidence.

## Current hypothesis

ESP-SDR can expose raw I/Q samples from supported ESP32 Wi-Fi radios, including ESP32-C3 and ESP32-C5. Software may accept tuning values far outside the normal Wi-Fi bands, but that does **not** prove PLL lock, usable RF gain, or actual reception there.

The first question is therefore deliberately small:

> Can an ESP32-C3 directly observe a real 6 MHz-wide ISDB-T terrestrial TV signal in the UHF band?

If direct reception fails, the fallback path is to use an external mixer / upconverter and move the selected UHF TV channel into a usable 2.4 GHz IF for the ESP32.

## Scope

- ESP32-C3 first
- ESP32-C5 later for comparison
- Raw I/Q capture with ESP-SDR or closely related techniques
- UHF ISDB-T RF observation
- 1seg and 13-segment/full-seg experiments
- External upconversion if direct UHF reception is not practical
- PC-side demodulation first
- ESP32-side processing only after the RF path is proven

## Milestones

1. Reproduce raw I/Q capture on ESP32-C3.
2. Establish a 2.4 GHz known-signal baseline.
3. Attempt direct tuning in the terrestrial TV UHF band.
4. Decide PASS / FAIL using repeatable evidence.
5. If direct UHF fails, test an upconverter into the 2.4 GHz range.
6. Capture a complete ~6 MHz ISDB-T channel and export I/Q to a PC.
7. Demodulate ISDB-T on the PC and recover MPEG-TS.
8. Investigate moving parts of the demodulation pipeline onto ESP32.
9. Stretch goal: ESP32-originated MPEG-TS suitable for downstream tools such as Mirakurun.

## Evidence policy

Each experiment should record:

- board and module name
- ESP32 revision
- firmware commit
- antenna / RF wiring
- center frequency
- configured bandwidth
- sample rate
- gain settings
- nearby known transmitters or signal generator settings
- raw capture or screenshot when useful
- exact commands
- result
- PASS / FAIL / INCONCLUSIVE
- interpretation separated from observation

Do not treat a visible waterfall feature as proof of reception until it tracks a known RF source or otherwise has a reproducible signature.

## Safety / legal

This repository is for passive reception and RF experimentation. Do not transmit on frequencies for which you are not authorized.

## Upstream / references

Primary starting point:

- ESP-SDR: https://github.com/ESPARGOS/esp-sdr

Other related ESP32 raw-I/Q and RF experiments should be documented in `docs/PLAN.md` as they are validated.

## Status

Repository initialized. First target: ESP32-C3 direct-UHF feasibility test.
