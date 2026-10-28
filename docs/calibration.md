---
layout: doc
title: Calibration manual
---

# Calibration manual

Per-unit calibration for the DFU Thermography Screening Station. Everything
written by these procedures goes in `calibration/`, which is never committed
to git. Keep a copy of that folder with the station's records.

The station will not start a real scan until these three files exist:

| File | Written by | Section |
|---|---|---|
| `calibration/thermal.yaml` | `python -m src.diagnostics.thermal_reference` | 1 |
| `calibration/color.yaml` | `python -m src.diagnostics.color_lock` | 2 |
| `calibration/registration.yaml` | `python -m src.diagnostics.registration_calibrate` | 3 |

Do them in this order: thermal reference check, colour camera lock,
registration. Section 4 says how often to repeat each. Before any of them,
run the hardware self-check and fix anything it reports:

```bash
python -m src.diagnostics.check_hardware
```

Let the station run for at least 15 minutes before calibrating, so the Lepton
has reached a steady temperature, and have the room at 22–25 °C as for a scan.

---

## 1. Thermal reference check

This compares the Lepton against a thermometer you trust. Until it has been
done, the sensor is not known to be working, and the station will not scan.

### 1.1 What you need

- **A reference thermometer** with a current calibration, ideally a contact
  probe. Note its make, model and stated accuracy for the study record.
- **A target of known, steady temperature** that fills the centre of the
  thermal picture. The tool reads a 9 × 9 pixel square at the centre, about
  26 mm across at the sole plane, so make the target at least 60 mm across.
  A good target is a matte-black metal block, or a sealed water-filled
  container with matte-black tape on its face, warmed to roughly skin
  temperature (30–35 °C) and left to settle.
- The target face must be **matte and dark**. A shiny surface reflects the
  room and reads wrongly however good the sensor is.

### 1.2 Procedure

1. Place the target in the sole plane, at the centre of the cabinet opening,
   facing the camera head. Close the shroud.
2. Start the app (`python -m src.app`), enter any valid reference code and use
   the thermal view to check the target covers the middle of the picture.
   Close the app again.
3. Measure the target's surface with the reference thermometer and note the
   reading.
4. Straight away, run:

   ```bash
   python -m src.diagnostics.thermal_reference
   ```

   and type the reference reading when asked.
5. The tool checks that the Lepton reports TLinear radiometry, runs a
   flat-field correction, averages the centre of 9 frames and prints:

   ```
   Reference thermometer:         34.20 C
   Lepton, uncalibrated:          33.65 C   (error -0.55 C)
   Lepton, calibration in use:    33.65 C   (error -0.55 C)
   Spread inside the 9x9 px target area: 0.12 C over 9 frames
   Tolerance: 1.00 C   ->   PASS
   ```

   (The numbers above only show the layout. They are not results from any
   unit.)

### 1.3 Reading the result

- **Spread** is the difference between the warmest and coolest pixel in the
  target area. If it is more than a few tenths of a degree the target is not
  uniform or does not fill the area; fix that before trusting the error.
- **PASS** means the reading with the calibration in use is within
  `thermal.reference_tolerance_c` of the reference. On the first pass with no
  calibration file, the tool writes `calibration/thermal.yaml` with no
  correction (gain 1, offset 0).
- **FAIL** means it is outside. First check the target, the reference
  thermometer and the placement, and repeat. If the error is steady and
  repeatable, store a single-point correction:

  ```bash
  python -m src.diagnostics.thermal_reference --apply-offset
  ```

  then **run the check once more without `--apply-offset`** to confirm the
  corrected reading passes.
- The tool will not write an offset larger than 10 °C. An error that big means
  something is wrong with the sensor, the target or the reference.
- If the tool says the Lepton does not report TLinear radiometry, stop. The
  sensor's values cannot be read as temperatures. Check the PureThermal
  firmware and the Lepton's radiometry setting.

Every check, pass or fail, is added to
`calibration/thermal_reference_log.yaml`. Keep that file: it is the evidence
for acceptance line 9 and for the study's methods section.

### 1.4 Limits of this check

It is a check at one temperature at the centre of the picture. It corrects an
offset. It does not show whether the error changes with temperature or across
the picture. For the study record it is worth repeating at a second
temperature (for example near 25 °C and near 35 °C) and noting both errors,
and viewing a uniform-temperature surface that fills the whole picture to see
how even the reading is from centre to edge.

---

## 2. Colour camera lock (focus, exposure, white balance)

Colour pictures are only comparable from scan to scan if the camera's settings
do not change. This step finds the settings once, under the interior LED, and
stores them. From then on nothing is on auto.

### 2.1 Procedure

1. Put something foot-like in the sole plane: a hand, or a skin-tone card.
   The settings are chosen for whatever the camera sees, so an empty black
   cabinet gives the wrong exposure. Close the shroud.
2. Run:

   ```bash
   python -m src.diagnostics.color_lock
   ```

3. The tool sets manual focus for the working distance (1000 ÷ 420 mm ≈ 2.38
   dioptres), switches the interior LED on, lets auto-exposure and auto white
   balance settle for a few seconds, reads the values they settled on,
   switches the LED off and writes `calibration/color.yaml`.

### 2.2 Check the result

Take a test scan and open its `color.png`:

- **Sharpness.** Skin creases should be crisp at 420 mm. If the picture is
  soft, edit `lens_position_dioptres` in `calibration/color.yaml` in steps of
  0.1 and test again. Larger numbers focus closer.
- **Brightness.** Skin should not be blown out to white or lost in shadow. If
  it is, repeat the procedure with a better stand-in for a foot.
- **Colour.** A red object should look red. If red and blue look swapped,
  tell the developer: the colour channel order is wrong for this camera
  software version.

If the interior LED is ever replaced or its brightness changed, repeat this
section and then section 3.

---

## 3. Thermal-to-colour registration

The Lepton and the Camera Module 3 sit 40 mm apart and see the scene at
different scales. Registration finds the mapping (a homography) from a colour
pixel to the thermal pixel that sees the same point, so the colour frame can
be laid onto the thermal frame.

The thermal frame is the limiting frame. The colour frame is mapped **down**
onto the thermal field of view and grid. The thermal frame is never enlarged.

### 3.1 The target

A target has to give the same set of points in both cameras.

- **Plate:** a flat, rigid, light-coloured sheet with a regular grid of round
  holes, **5 rows by 7 columns**. Matte white-painted aluminium or steel sheet
  works well. A bare shiny plate reflects the room in the thermal image; paint
  it or cover it with matte tape.
- **Hole size and spacing:** holes about 18 mm in diameter on a 50 mm pitch.
  At 420 mm one thermal pixel covers about 2.85 mm, so each hole is about
  6 pixels across and the grid spans about 300 x 200 mm, inside the thermal
  field of view. Smaller holes are too few thermal pixels to locate well.
- **Warm backing:** a dark, evenly warm surface directly behind the plate, for
  example a heating pad or a warm-water bag at roughly 35 C wrapped in dark
  cloth. Through each hole the thermal camera sees the warm backing (warmer
  than the plate) and the colour camera sees dark cloth (darker than the
  plate).
- **Border:** leave at least 30 mm of plain plate around the outer holes, so no
  hole touches the edge of either image.

If you use a different grid, change `registration.target_rows` and
`registration.target_cols` in the config.

### 3.2 Procedure

1. Finish sections 1 and 2 first. Registration needs both cameras working with
   their settings fixed.
2. Hold the plate **in the sole plane**: where the patient's soles will be, at
   the 420 mm working distance, square to the camera head. A simple jig that
   sits on the foot rest is worth making, because this is repeated.
3. Put the warm backing behind the plate and wait a minute for the plate to
   settle. Close the shroud.
4. Run:

   ```bash
   python -m src.diagnostics.registration_calibrate
   ```

   The tool runs a flat-field correction, takes a colour frame with the
   interior LED on, switches the LED off, takes a thermal frame, finds the 35
   holes in each, and fits the homography.
5. Read the result. The tool prints the reprojection error in thermal pixels.

### 3.3 Pass and fail

- **Pass:** reprojection error at or below
  `registration.max_reprojection_error_px` (default 1.0 thermal pixel). The
  tool writes `calibration/registration.yaml` and
  `calibration/registration_check.png`.
- **Fail:** the tool prints the reason and **writes nothing**. The previous
  registration, if any, stays in place.

Always open `registration_check.png`. It shows the thermal picture blended
with the registered colour picture. Each warm spot should sit on its dark
hole across the whole plate, including the corners. A low number with a
visibly shifted overlay means the wrong holes were matched; repeat.

Common failures:

| Message | Usual cause |
|---|---|
| found N holes, expected 35 | Plate partly out of view, a hole at the image edge, uneven warmth, or a reflective plate |
| the target rows overlap | Plate rotated; hold it square |
| registration error above the limit | Plate bent or not flat to the cameras, or it moved between the two frames |
| the calibration image is flat | Backing not warm enough, or the plate is as warm as the backing |

### 3.4 What the registration is and is not good for

- It is exact only for points **in the plane where the plate was held**. The
  cameras are 40 mm apart, so a point 30 mm nearer or farther than that plane
  shifts by about one thermal pixel. Toes and heel are not at exactly the same
  depth, so expect about a pixel of disagreement at the extremes.
- The asymmetry measurement does **not** depend on registration. Temperatures
  are measured on the thermal frame alone. Registration is used to give the
  classifier the same field of view as the thermal frame and to draw the
  overlay.
- The Camera Module 3 is wider than the Lepton horizontally but slightly
  shorter vertically. If the colour frame does not cover the top or bottom
  rows of the thermal frame, the station records the covered fraction with the
  scan (`color_coverage_fraction` in `meta.json`) instead of inventing colour.

### 3.5 When to repeat

- After any work on the camera head (either camera removed, refitted or
  knocked).
- After changing the colour camera resolution in the config. The station
  refuses to use a registration made at a different colour size.
- At the start of each data-collection campaign, and if the overlay on the
  results screen looks shifted.

---

## 4. How often to repeat

| Calibration | Repeat |
|---|---|
| Hardware self-check | Before any calibration, after any hardware work, and whenever the station behaves oddly |
| Thermal reference check (section 1) | At installation; at the start of each data-collection campaign; after the Lepton or PureThermal board is touched or replaced; after a firmware change; and at an interval the research team sets for the study (monthly is a reasonable starting point) |
| Colour camera lock (section 2) | At installation; after the colour camera or the interior LED is touched or replaced; after changing `color.width_px` or `color.height_px` |
| Registration (section 3) | At installation; after either camera is touched; after the colour camera lock is redone; at the start of each campaign; whenever the overlay looks shifted |
| Station clock | Check daily before the first patient. The station is offline and cannot set its own clock |

A check that passes still goes in the log. A run of passing checks is what
shows the station stayed stable through the study.

## 5. Records to keep

Copy the whole `calibration/` folder to the study records after every
calibration session:

- `thermal.yaml` and `thermal_reference_log.yaml`
- `color.yaml`
- `registration.yaml` and `registration_check.png`

Note alongside them the reference thermometer used, the date, and who did the
work. The folder is deliberately kept out of the code repository because it
belongs to one physical unit.
