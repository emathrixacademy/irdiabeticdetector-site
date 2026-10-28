---
layout: doc
title: Acceptance test runbook
---

# Acceptance test runbook

DFU Thermography Screening Station · Annex A of Job Order EMX-2026-DFUSCAN-JO-01
Client acceptance: October 30, 2026, with the client present.

For each of the 14 lines of the acceptance form this gives the exact step, the
expected result, and the test or tool that demonstrates it.

**Read this first.** When this runbook was written, the software was complete
and tested on a laptop with stand-in drivers. It had **not** been run on the
station. Lines marked *needs the station* below cannot pass until the work in
`BLOCKERS.md` is done. The right-hand column of the summary says which blocker
applies. Do not tick a line on the strength of the laptop tests alone where
the line is about hardware.

---

## Before the client arrives

Do these the day before, and again on the morning.

1. Room at 22–25 °C and steady.
2. Station powered for at least 15 minutes.
3. Station clock correct (`date -u`).
4. `python -m src.diagnostics.check_hardware` → every line `[PASS]`.
5. `calibration/` contains `thermal.yaml`, `color.yaml`, `registration.yaml`.
6. `models/` contains the compiled `.hef` and `model_meta.json`.
7. On the station: `python tasks.py check` → all checks pass, and
   `pytest -m hardware` → all hardware tests pass.
8. One full practice scan on a volunteer, start to finish.
9. Have to hand: the reference thermometer and target (line 9), the
   registration target (line 10), a tape measure (line 3), a warm pack
   (line 11), a printed copy of `docs/methodology.md` (line 14).

Keep the terminal output of steps 4 and 7 for the acceptance record.

---

## Summary

| Line | Requirement | Demonstrated by | Ready? |
|---|---|---|---|
| A1 | Cabinet, matte interior, blackout shroud | Inspection | Fabrication; no software |
| A2 | Padded foot rest, both soles in one thermal frame | Inspection, live thermal view, refusal demo | Software ready; needs the station |
| A3 | Camera head at a fixed, repeatable 420 mm | Tape measure, repeat registration | Fabrication; no software |
| A4 | Pi 5 with Hailo-8L, cooling, storage, power | `check_hardware` | Needs the station (B6) |
| A5 | Lepton 3.5 with interface board, Camera Module 3 | `check_hardware` | Needs the station (B1, B3, B6) |
| A6 | Touchscreen, capture control, status indication | UI walk-through | Software ready; needs the station (B4, B12) |
| A7 | Ambient temperature and humidity logged with every scan | Scan, then open `scans.csv` | Software ready; needs the station (B5) |
| A8 | One trigger produces one paired colour and thermal frame | One press, one folder | Software ready; needs the station |
| A9 | Thermal calibration matches a reference thermometer | `thermal_reference` | Needs the station (B2) |
| A10 | Thermal frame registered to the colour frame | `registration_calibrate`, overlay | Needs the station (B7) |
| A11 | Left-right matching with the 2.2 °C flag | Test suite, warm-pack demo | Software ready; needs the station |
| A12 | MobileNetV2 compiled to Hailo, running on-device | Scan shows a predicted class from the `.hef` | **Blocked: no dataset, no model (B9, B10, B8)** |
| A13 | Offline logging of images, metadata and output to CSV | Scan with no network, open the folder | Software ready; needs the station |
| A14 | Methodology documentation handed over | `docs/methodology.md` | Ready, with gaps for the team to fill |

The automated test `tests/test_acceptance_e2e.py` runs the whole pipeline on
synthetic fixtures and checks every line that software alone can check (A2,
A6, A7, A8, A9, A10, A11, A12, A13, A14). It is evidence that the software
does what it should. It is not evidence about the hardware.

---

## A1. Cabinet with welded frame, matte non-reflective interior, blackout shroud

- **Step.** Inspect the cabinet with the client. Check the frame welds, look
  inside with a torch for any shiny surface, close the shroud around a pair of
  legs and look for light leaks from inside.
- **Expected.** Welded frame; every interior surface matte; no visible light
  with the shroud closed.
- **Demonstrated by.** Inspection. No software is involved.
- **Supporting check.** With the shroud closed and the interior LED off, the
  colour view on the positioning screen is dark.

## A2. Padded foot rest, both soles in one thermal frame

- **Step 1.** Inspect the padded foot rest.
- **Step 2.** Seat a volunteer, enter a reference code, and look at the
  thermal view on the positioning screen.
- **Expected.** Both soles, toes to heel, are inside the one thermal picture
  with margin on every side.
- **Step 3 (refusal).** After the rest period, move one foot so it leaves the
  picture and press capture.
- **Expected.** The station refuses with "Foot out of view" and saves nothing.
- **Demonstrated by.** The live view and the refusal on the station.
  Software: `tests/test_segmentation.py` (two feet found; one, none, three or
  touching feet refused; a foot cut off at any of the four edges refused) and
  `tests/test_acceptance_e2e.py`.

## A3. Camera head at a fixed, repeatable 420 mm

- **Step.** Measure from the camera head to the sole plane (the face of the
  foot rest where the soles sit) with a tape measure. Remove and refit
  anything that is meant to be removable, and measure again.
- **Expected.** 420 mm both times, within the tolerance agreed with the
  client.
- **Demonstrated by.** The tape measure. No software is involved.
- **Supporting check.** `working_distance_mm: 420` in the config; the loader
  refuses a value outside 400–450. A repeat of the registration calibration
  (A10) that gives the same result shows the geometry has not moved.

## A4. Pi 5 with Hailo-8L, cooling, storage, power

- **Step.** Run on the station:

  ```bash
  python -m src.diagnostics.check_hardware
  hailortcli fw-control identify
  vcgencmd measure_temp
  df -h /
  ```

- **Expected.** `[PASS] Hailo-8L accelerator (AI HAT+): identified as
  Hailo-8L`; the identify command names a Hailo-8L; the temperature is well
  below throttling with the cooler running; free storage is ample.
- **Demonstrated by.** `src/diagnostics/check_hardware.py`. Its reporting is
  tested in `tests/test_check_hardware.py`; the checks themselves only mean
  something on the station.

## A5. Lepton 3.5 with interface board and Camera Module 3 integrated

- **Step.** In the same `check_hardware` output, read the camera lines. Then
  run `v4l2-ctl --list-devices` and `rpicam-hello --list-cameras`.
- **Expected.**
  `[PASS] Camera Module 3 (CSI): imx708 sensor listed`,
  `[PASS] PureThermal 3 on USB: found at /dev/videoN`,
  `[PASS] Lepton Y16 stream and TLinear radiometry: Y16 frame 160x120`.
- **Demonstrated by.** `check_hardware`, and on the station
  `pytest -m hardware tests/test_purethermal.py tests/test_color_camera.py`.
- **Note.** The station uses only the raw Y16 stream. It refuses the
  colourised video stream, and refuses to measure unless the Lepton itself
  reports TLinear radiometry. `tests/test_purethermal.py` covers both rules.

## A6. Touchscreen, capture control and status indication

- **Step 1.** Power on. The app starts by itself, full screen.
- **Step 2.** Walk through one scan by touch only: reference code, rest
  countdown, capture, left/right confirmation, results, next patient.
- **Step 3.** Do a second scan using the **physical capture button** in place
  of the on-screen button.
- **Step 4.** Watch the status LED through both scans.
- **Expected.** Every screen works by touch. The physical button does exactly
  what the on-screen button does. The LED is off when idle, blinks slowly
  during the rest period, is steady when ready, blinks fast while capturing,
  and gives a short flash once a second after a refusal.
- **Demonstrated by.** The walk-through. Software: `tests/test_ui_qt.py`
  (the whole workflow through the real widgets), `tests/test_ui_controller.py`
  (button parity: identical device actions and results from either button, and
  the same refusals), `tests/test_gpio.py` (LED and button on GPIO).

## A7. Ambient temperature and humidity logged with every scan

- **Step.** After the scans in A6, open `data/scans.csv` and the `meta.json`
  of one scan. Compare with a room thermometer.
- **Expected.** Every row has `ambient_temp_c` and `ambient_humidity_pct`, and
  they agree with the room. `meta.json` has the same values and the time they
  were read.
- **Refusal demo (optional).** If the room can be taken outside 22–25 °C, or
  the sensor warmed in the hand, press capture: the station refuses with
  "Room too warm" and saves nothing. There is no way to override this.
- **Demonstrated by.** The files. Software: `tests/test_fusion_storage.py`
  (ambient columns in every row), `tests/test_gates.py` (the gate at and
  around both limits, a stale or failed sensor is a fault and not a pass, and
  no bypass exists), `tests/test_sht31.py` (the sensor driver and its
  checksum).

## A8. One trigger produces one paired colour and thermal frame

- **Step.** Note the number of scan folders under `data/`. Press capture once
  and complete the scan. Count again and open the new folder.
- **Expected.** Exactly one new folder, containing both `color.png` and the
  thermal files (`thermal_raw_y16.npy`, `thermal_temp_c.npy`), plus
  `color_registered.png`, `overlay.png` and `meta.json`. `meta.json` lists a
  timestamp for every step of the sequence, and the colour and thermal times
  are a few seconds apart from the one trigger.
- **Demonstrated by.** The folder. Software: `tests/test_sequencer.py` (one
  trigger gives one paired record; the sequence order and delays; a camera
  fault or a failed write leaves no files at all).

## A9. Thermal calibration matches a reference thermometer

- **Step.** Follow `docs/calibration.md` section 1 with the client watching:
  place the target, read the reference thermometer aloud, run

  ```bash
  python -m src.diagnostics.thermal_reference
  ```

  and enter the reading.
- **Expected.** The tool prints the reference, the Lepton reading, the error
  and `PASS`, with the error inside the tolerance agreed with the client
  (`thermal.reference_tolerance_c`, 1.0 °C unless changed). The result is
  added to `calibration/thermal_reference_log.yaml`.
- **Demonstrated by.** `src/diagnostics/thermal_reference.py`. Its arithmetic
  and file handling are tested in `tests/test_thermal_reference.py`, and the
  conversion `raw / 100 − 273.15` in `tests/test_thermal_convert.py`. Whether
  this unit agrees with a thermometer can only be shown on the station.
- **If it fails.** Do not adjust numbers to make it pass. See
  `docs/calibration.md` section 1.3.

## A10. Thermal frame registered to the colour frame

- **Step.** Follow `docs/calibration.md` section 3: hold the perforated target
  in the sole plane over the warm backing and run

  ```bash
  python -m src.diagnostics.registration_calibrate
  ```

  Open `calibration/registration_check.png`. Then do a scan and look at
  `overlay.png` in its folder.
- **Expected.** `Reprojection error N px (limit 1.00 px): PASS`. In the check
  picture every warm spot sits on its hole, corners included. In the scan
  overlay the thermal feet sit on the colour feet.
- **Demonstrated by.** `src/diagnostics/registration_calibrate.py`. Software:
  `tests/test_registration.py` (hole detection and fit against a known
  transform, calibration refused above the limit, colour mapped down to the
  thermal grid with the thermal frame untouched).

## A11. Left-right matching with the 2.2 °C flag

- **Step 1 (the rule itself).** Run, with the client watching:

  ```bash
  pytest tests/test_asymmetry.py -v
  ```

- **Expected.** All pass. The named cases show a heel difference of 2.19 °C is
  not flagged, 2.20 °C is flagged, 2.21 °C is flagged, and a symmetric pair of
  feet raises no flag, through the full pipeline.
- **Step 2 (on the station).** Scan a volunteer normally and note the table.
  Then hold a warm pack against one heel for a minute, remove it, and scan
  again. Confirm left and right carefully both times.
- **Expected.** Second scan: the heel on the warmed side reads clearly warmer
  than the other, the difference appears in the table with the correct sign
  (left minus right), and the heel row shows FLAG if the difference is 2.2 °C
  or more. Regions away from the heel are little changed.
- **Step 3 (anatomy, not position).** Ask the volunteer to cross their ankles
  so the feet swap sides in the picture, and scan. Tap the correct foot as the
  left foot.
- **Expected.** The results are still labelled by the patient's true left and
  right.
- **Demonstrated by.** `tests/test_asymmetry.py`, `tests/test_segmentation.py`
  and the two scans.
- **Note for the client.** The flag is plain arithmetic on the thermal frame:
  mean of a region on the left foot minus the mean of the same region on the
  right. No model is involved in it.

## A12. MobileNetV2 compiled to Hailo, running on-device

- **Step.** With `inference.backend: hailo` in the config, do a scan. Look at
  the results screen and at `meta.json`.
- **Expected.** The results show "Colour model predicted class: class_N
  (confidence X %)". `meta.json` has `"model_id": "hailo:<file>.hef"`. On the
  station, `pytest -m hardware tests/test_inference.py` passes.
- **Demonstrated by.** `src/inference/hailo_runner.py` on the device.
  Software: `tests/test_inference.py` (the runner's call flow, the shared
  preprocessing), `tests/test_training_model.py` (training, export and the
  PyTorch-to-ONNX match), `tests/test_output_naming.py` (the output is a
  predicted class and never a grade).
- **Status: blocked.** This line needs a model trained on the research team's
  dataset and compiled with the Hailo Dataflow Compiler. Neither existed when
  this runbook was written (`BLOCKERS.md` B9, B10, B8). The station will not
  start in `hailo` mode without the `.hef`. Agree with the client beforehand
  how this line is to be handled if their dataset is not ready by October 30.
- **What this line does not promise.** Any level of accuracy. The contract
  excludes it, and it depends on the client's dataset.

## A13. Offline logging of images, metadata and output to CSV

- **Step.** Confirm the station has no network: Wi-Fi off, no Ethernet cable
  (`ip addr` shows no address beyond loopback). Do a scan. Open the scan
  folder and `data/scans.csv`.
- **Expected.** The scan completes with no network. The folder holds the
  images, the raw and temperature arrays and `meta.json`. `scans.csv` has one
  new row with the reference code, timestamps, ambient readings, config hash,
  every region's left mean, right mean, difference and flag, the predicted
  class and its confidence. No row or file contains a patient name.
- **Demonstrated by.** The files. Software: `tests/test_fusion_storage.py`
  (folder contents, every CSV column, append-only, a scan folder and its row
  written together or not at all, a full scan with all network functions
  blocked, and no network library imported anywhere in the station code).

## A14. Methodology documentation handed over

- **Step.** Hand the client `docs/methodology.md` (printed and as a file),
  with `docs/operator_manual.md` and `docs/calibration.md`.
- **Expected.** The client has the document. Point out the boxed notes where
  they are to enter results from this unit and from their own dataset, and
  section 12 (limitations).
- **Demonstrated by.** The documents. `tests/test_docs.py` checks the
  methodology covers every required topic, quotes the protocol values in use,
  and makes no accuracy claim.

---

## After the test

- Copy `calibration/` and the terminal output from the day to the acceptance
  record.
- Any line that did not pass: write down exactly what was observed. Do not
  mark a line passed on the basis of a laptop test.
- Scans of volunteers made during the test are research data on the device.
  Remove or keep them as the client directs.
