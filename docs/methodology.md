---
layout: doc
title: Methodology
---

# System development and testing methodology

DFU Thermography Screening Station
Contract EMX-2026-DFUSCAN-01 · Job Order EMX-2026-DFUSCAN-JO-01
Built by eMathrix Technologies for Mendoza, Mutia and Rompe (research team)

This document describes how the station works and how it was tested, at the
level of detail a methods section needs. It is written for the research team.

**What this document does not contain.** It makes no claim about
classification accuracy, sensitivity, specificity or clinical performance.
Those depend on the research team's dataset and are theirs to measure and
report. Where a number in this document must come from the finished station
(a calibration result, a registration error), the text says so and leaves it
for the team to fill in from the station's own records. Section 13 lists what
had not yet been verified on hardware when this was written.

**What the station is.** A research instrument. It does not diagnose, grade
or treat. Wagner-Meggitt grades are assigned by a licensed clinician and enter
the study only as labels the research team supplies. The station never
assigns, stores or displays a grade.

---

## 1. Rationale

Inflamed plantar tissue runs warmer than the tissue around it before the skin
breaks. Absolute foot temperature is a poor signal because it moves with the
room, the circulation and the person. The comparison that carries information
is between the same site on the left and right foot. A difference of 2.2 °C or
more at matched sites is the flag used in this study.

Three design requirements follow:

1. Both soles are imaged in one thermal frame, at the same instant and under
   the same conditions.
2. The thermal camera is radiometric: every pixel is a temperature, not a
   colour.
3. Conditions are controlled: an enclosed cabinet, a room at 22–25 °C, and a
   15-minute rest before scanning.

## 2. Hardware

| Part | Role |
|---|---|
| Raspberry Pi 5, 8 GB | Main computer, runs everything on the device |
| Raspberry Pi AI HAT+ (Hailo-8L) | Runs the colour-image classifier |
| FLIR Lepton 3.5 on a GroupGets PureThermal 3 (USB) | Radiometric thermal camera, 160 × 120 pixels, about 57° horizontal field of view, about 8.7 frames per second |
| Raspberry Pi Camera Module 3 (CSI) | Colour camera |
| Interior LED, MOSFET-switched | Lights the soles for the colour frame only |
| Ambient temperature and humidity sensor (I2C) | Room conditions, logged with every scan |
| 7-inch touchscreen, capture button, status LED | Operator interface |

Geometry: the sensors are 420 mm from the soles. The two cameras are 40 mm
apart, centre to centre. At 420 mm one thermal pixel covers about 2.85 mm of
the sole. The cabinet is enclosed with a matte interior and a blackout shroud.

The thermal frame is the limiting frame of the system. Nothing downstream
treats the thermal image as having more detail than 160 × 120 pixels.

## 3. Capture protocol

The patient lies supine or sits with legs extended, soles facing the camera
head, and rests with feet uncovered for the acclimatisation period.

One press of the capture control (the physical button or the on-screen button;
both run the same code) starts a fixed sequence. The order is enforced in
software and each step is timestamped in the scan record.

1. **Ambient check.** The room sensor is read. If the reading is outside
   22.0–25.0 °C (inclusive), is more than 5 s old, or the sensor does not
   answer, the scan is refused.
2. **Acclimatisation check.** A timer starts when the operator enters the
   patient's reference code. Unless 900 s have passed for that code, the scan
   is refused.
3. **Flat-field correction (FFC).** The Lepton is commanded to run an FFC,
   which briefly closes its shutter, and the software waits for it to finish.
4. **Colour frame.** The interior LED is switched on, allowed 1.0 s to reach
   steady brightness, and one colour frame is taken.
5. **Settle.** The LED is switched off and the software waits 2.0 s, because
   light warms skin.
6. **Thermal frame.** One raw radiometric (Y16) frame is taken with the LED
   off.
7. **Paired record.** The two frames, the ambient reading, the step
   timestamps, the reference code and a hash of the configuration form one
   record.

There is no override for steps 1 and 2 in the software: no command-line
option, configuration value or development switch skips or widens them. The
configuration can make the ambient range narrower or the rest period longer,
never the reverse. The times in steps 4 and 5 are configuration values and
are captured by the configuration hash stored with each scan.

One trigger produces one paired colour and thermal record, or nothing. If any
step fails, no record is produced and nothing is written.

## 4. Thermal measurement

**Radiometry.** The PureThermal board offers a colourised video stream and a
raw 16-bit (Y16) stream. Only the Y16 stream is used. Before any measurement
the software reads the Lepton's own settings and requires TLinear radiometry
to be enabled at 0.01 K resolution; otherwise it refuses to run. With TLinear
at that resolution each pixel value is a temperature in centikelvin:

    temperature (°C) = raw / 100 − 273.15

**Frame validity.** A thermal frame is rejected, with no substitute, if it is
older than 0.5 s, was taken during an FFC or within 1.0 s after one, was taken
before any FFC had completed, is all zero, contains a saturated or zero pixel,
is not 160 × 120 16-bit data, or contains any temperature outside 0–60 °C. A
frame rejected for age or FFC timing is discarded and a new frame is read from
the sensor, up to five frames; any other fault stops the scan. No frame is
ever repaired, interpolated or replaced with stored data.

**Calibration against a reference.** Each unit is checked against a reference
thermometer with the tool described in `docs/calibration.md`. A target of
known temperature is placed at the centre of the thermal field; the tool
averages a 9 × 9 pixel area over 9 frames and reports the difference from the
reference reading. The result, and any single-point offset the technician
chooses to apply, is stored on the station. The correction has the form
`corrected = gain × measured + offset`; the tool only ever writes a gain of
1.0 and an offset. The station will not scan without a calibration file.

> To be completed by the research team from `calibration/thermal_reference_log.yaml`
> on the delivered unit: reference instrument and its stated accuracy; target
> used; reference temperature; uncorrected Lepton reading; offset applied, if
> any; residual error after correction; date.

FLIR specifies the Lepton 3.5's radiometric accuracy as the larger of ±5 °C or
5 % in high-gain mode. The asymmetry measure in section 7 is a difference
between two regions of the same frame, so an offset common to the whole frame
cancels. Non-uniformity across the frame does not cancel, and is a limitation
(section 12).

## 5. Registration of the colour frame to the thermal frame

The colour frame is mapped onto the thermal field of view with a homography
(a 3 × 3 projective transform), found once per unit from a calibration target:
a flat, light-coloured plate with a 5 × 7 grid of round holes, held in the
sole plane over a warm, dark surface. Each hole is warm in the thermal frame
and dark in the colour frame. Hole centres are located in both frames as
intensity-weighted centroids, matched by grid position, and the homography is
fitted by least squares over all 35 points.

Registration quality is the root-mean-square distance, in thermal pixels,
between each colour-frame centre mapped through the homography and its
thermal-frame centre. Calibration fails, and nothing is stored, if this
exceeds 1.0 thermal pixel.

> To be completed by the research team from `calibration/registration.yaml`:
> reprojection error on the delivered unit and the calibration date.

The colour frame is resampled down to the thermal field of view. The thermal
frame is never resampled or enlarged for analysis. Two colour images result:
one at 160 × 120, used for the overlay, and one at 640 × 480 covering the same
field, used as the classifier input.

**Registration does not affect the temperature measurement.** Temperatures
are read from the thermal frame alone. Registration determines only which
part of the colour frame the classifier sees and how the overlay is drawn.

## 6. Segmentation and left-right assignment

**Plantar mask.** A pixel is treated as foot if it is more than 2.0 °C above a
reference, taken as the warmer of the ambient sensor reading and the frame's
own background (its 20th-percentile temperature). The binary image is cleaned
with a 3 × 3 morphological opening and closing, used only to group pixels into
connected components; a pixel that is not itself above the threshold is never
added to a mask. Components smaller than 800 pixels are ignored.

**Refusals.** The scan is refused if the frame does not contain exactly two
feet, or if a foot touches the edge of the frame. A refused scan is not stored.

**Orientation and scale.** For each foot, the long axis is the first principal
axis of its mask pixels. The forefoot is taken as the wider end. Position
along the foot is expressed as a fraction of the foot's own length (0 at the
toe tips, 1 at the heel) and position across it as a fraction of its own width
(0 at the hallux side, 1 at the lateral side). The right foot is mirrored, so
both feet are described in one coordinate frame regardless of their size,
position or tilt in the image.

**Left and right.** The station does not assume the left foot is on the left
of the image. It proposes a side for each foot from its shape (toes shorten
from the hallux outward, so the toe tips lean toward the hallux side), and
the operator must confirm or correct the proposal on the touchscreen before
the scan is analysed or stored. Each record states whether the stored
assignment was the confirmed proposal (`geometry_confirmed`) or the operator's
correction (`operator_override`), and what the proposal was.

## 7. Asymmetry method

This is deterministic arithmetic. It involves no trained model and no tuned
parameter other than the published threshold.

**Regions.** Eight plantar regions are defined as fixed polygons in the
normalised foot frame:

| Region | Along the foot (toe = 0, heel = 1) | Across the foot (hallux side = 0) |
|---|---|---|
| Hallux | 0.00–0.18 | 0.00–0.35 |
| Lesser toes | 0.00–0.18 | 0.35–1.00 |
| 1st metatarsal head | 0.18–0.38 | 0.00–0.35 |
| Central metatarsal heads | 0.18–0.38 | 0.35–0.70 |
| 5th metatarsal head | 0.18–0.38 | 0.70–1.00 |
| Medial midfoot | 0.38–0.70 | 0.00–0.50 |
| Lateral midfoot | 0.38–0.70 | 0.50–1.00 |
| Heel | 0.70–1.00 | 0.00–1.00 |

These boundaries are proportional defaults chosen by the builder, not a
published anatomical standard. They are configuration values. **The research
team's clinician should review them** and, if they are changed, report the
values used. The configuration hash stored with every scan identifies which
region map produced it.

Each thermal pixel of a foot belongs to exactly one region, decided by where
the pixel's centre falls. Region membership is computed on the native thermal
grid; no temperature is resampled.

**Per-region difference.** For each region, the mean temperature of its pixels
is computed on each foot, and

    ΔT = mean(left) − mean(right)

A positive value means the left region is warmer.

**Flag.** A region is flagged when |ΔT| ≥ 2.2 °C. The threshold is inclusive.
In the implementation the comparison is |ΔT| ≥ 2.2 − 10⁻⁶ °C. The 10⁻⁶ term
exists only because 2.2 cannot be represented exactly in binary floating
point, so two readings exactly 2.20 °C apart can subtract to
2.1999999999999993. It is ten thousand times smaller than the sensor's 0.01 °C
resolution and cannot change the outcome of any distinguishable measurement.

**Regions that cannot be compared.** If a region has fewer than 5 pixels on
either foot (for example an amputated toe, or skin too close to background
temperature to fall inside the mask), it is reported as not comparable: no ΔT
is given, it is not flagged, and both pixel counts are recorded. No value is
estimated in its place.

**Difference map.** For display only, both feet are also placed on a common
40 × 100 grid, each cell taking the nearest real thermal pixel, and the right
is subtracted from the left. No measurement is taken from this map.

## 8. Colour-image classifier

**Model.** MobileNetV2 with ImageNet-pretrained weights and a new final layer
sized to the number of classes in the research team's dataset (transfer
learning). By default the pretrained feature layers are frozen and only the
new layer is trained.

**Input.** The 640 × 480 registered colour image is resized to 224 × 224
without cropping, converted to RGB, scaled to 0–1 and normalised with the
ImageNet channel means (0.485, 0.456, 0.406) and standard deviations (0.229,
0.224, 0.225). One software module performs this for both training and
on-device inference, so the two cannot differ.

**Deployment.** The trained model is exported to ONNX with a fixed input
shape; the export is accepted only if ONNX Runtime reproduces the PyTorch
outputs to within 10⁻⁴. It is then quantised and compiled for the Hailo-8L
with the Hailo Dataflow Compiler, using colour frames captured by this station
as the calibration set, and runs on the device through HailoRT.

**Output.** A class index, a class name and a confidence (the largest softmax
probability). On the station, classes have neutral names (`class_0`,
`class_1`, …). The output is stored and displayed as a "predicted class". It
is not a Wagner grade and is not treated as ground truth anywhere in the
system. The mapping from class names to the clinician's labels is kept with
the training outputs, off the station.

**Relationship to the asymmetry result.** The two are reported side by side.
They are not combined into a single score or verdict, and the classifier
output cannot change which regions are flagged.

> To be completed by the research team: the dataset (number of patients and
> images per class), the labels used and who assigned them, the training
> settings actually used, and all performance results. No model had been
> trained on real data when this document was written (section 13).

## 9. Validation design: patient-level cross-validation

Several images of the same foot are near-duplicates. If images of one patient
fall on both sides of a train/test split, test performance is inflated and
does not estimate performance on new patients. The pipeline therefore splits
by patient:

- Every image carries a `patient_id` (the anonymised reference code). A
  dataset manifest with any row lacking one is rejected outright.
- Evaluation is by grouped k-fold cross-validation (default k = 5). All images
  of a patient are in the test fold or in the training folds for a given
  split, never both. Patients of each class are distributed across the folds
  as evenly as their number allows.
- The software verifies that no patient appears on both sides, once when the
  folds are built and again immediately before each fold is used.
- Loss weights for class imbalance, when used, are computed from the training
  side of each fold only.
- Splits and training are seeded and repeatable.

**The number of patients is the sample size.** Thirty images of ten patients
are ten samples, not thirty. The training report states patient counts
alongside every figure.

**Data quality is reported, not corrected.** Before training, the pipeline
writes a report of images and patients per class, images per patient, missing
metadata, missing files, duplicated paths and patients carrying more than one
label, with warnings for small classes and class imbalance. It does not drop,
impute, rebalance or resample. A class represented by a handful of patients
is reported as such.

### 9.1 Interim model on public photographs

Before the station has collected its own images, a first classifier may be
trained on public colour photographs of diabetic feet that carry Wagner
grades, so that the on-device classifier path can be demonstrated. If the
research team reports anything about that model, the following apply.

- **It is a stand-in.** Public photographs differ from station images in
  camera, lighting, distance and view (typically close-ups of a wound, not a
  fixed view of both soles). Performance on public data says little about
  performance on station images. The model is to be retrained on station
  captures.
- **Labels.** Only images whose source label is a stated Wagner grade of 0, 1
  or 2 are used, through an explicit per-dataset table confirmed by the
  research team. Every other label (grades 3 to 5, "normal" or "healthy",
  ungraded ulcers, other grading systems) is excluded and listed. No grade is
  inferred or converted.
- **Grouping is a proxy, true patient separation cannot be guaranteed on this
  public data.** Public datasets usually carry no patient identifier, and one
  person's foot often appears in several images. Where a dataset documents a
  subject identifier it is used. Otherwise images are linked into groups when
  a 64-bit perceptual hash differs by no more than a set number of bits, or
  when their MobileNetV2 feature vectors (centred on the dataset mean) exceed
  a set cosine similarity; each connected set of images is treated as one
  "patient" for cross-validation. Images are never given one identifier each.
  This keeps near-duplicates and visually similar shots of a foot together,
  but two different-looking photographs of one patient can still fall in
  different groups. Cross-validation figures on public data may therefore be
  optimistic, and should be reported with this limitation and with the number
  of groups, not presented as patient-level results.
- **Duplicates.** Byte-identical files are reduced to one, and the removed
  files are listed.
- **Quantisation.** Until station frames exist, the model is quantised for the
  Hailo-8L using held-out public images. It is to be recompiled with at least
  64 station frames.

> To be completed by the research team if this model is reported: each
> dataset's name, address, licence and retrieval date; the confirmed label
> table; images and groups per label; images excluded and why; the grouping
> thresholds used; and the per-fold results as produced. All of these are
> written by the intake and training tools into their report files.

## 10. Data recorded for each scan

Each scan is stored on the device as a folder
`data/<date>_<reference code>/<scan time>/` containing:

| File | Content |
|---|---|
| `color.png` | Colour frame as captured |
| `color_registered.png` | Colour frame on the thermal field of view (classifier input) |
| `thermal_raw_y16.npy` | Raw Lepton counts (centikelvin), 160 × 120 |
| `thermal_temp_c.npy` | Temperatures in °C after calibration, 160 × 120 |
| `overlay.png` | Thermal false colour blended with registered colour |
| `meta.json` | Reference code, timestamps of every capture step, ambient temperature and humidity, configuration hash, software version, left-right assignment and its source, segmentation threshold, registration error and coverage, per-region means, ΔT, flags and pixel counts, classifier output |

A row with the same results is appended to `data/scans.csv`, one row per scan.
The file is append-only. A scan folder and its row are written together or not
at all.

The only patient identifier stored is the anonymised reference code supplied
by the research team. The software rejects an entry that looks like a name.
All times are UTC.

The station is offline. The software opens no network connection, and an
automated test fails the build if it ever does.

## 11. Software verification

The software was developed with an automated test suite that runs without any
hardware, using stand-in ("fake") drivers that replay stored frames. The suite
runs before every change is accepted, together with static type checking and
linting.

**Synthetic test scenes.** The tests use generated scenes, not patient data:
two plantar-shaped regions at 30.00 °C on a 23.00 °C background, with the heel
of one raised by a set amount, quantised to centikelvin as the sensor would
report it, with a matching colour image. Scenes were generated with heel
differences of 0.00, 1.50, 2.19, 2.20, 2.21 and 3.50 °C.

**What the tests establish**, among other things:

- The temperature conversion formula, and the rejection of each kind of
  invalid thermal frame.
- That a frame is never substituted: the frame returned is bit-for-bit the
  frame the (stand-in) sensor produced.
- The ambient gate at and around both limits, the acclimatisation timer, and
  the absence of any bypass option.
- The order of the capture sequence, that the colour frame is lit and the
  thermal frame is taken dark after the settle delay, and that the LED is
  switched off even when a camera fails.
- That a failed capture or a failed write leaves no partial record.
- For the full pipeline on the synthetic scenes: the measured heel ΔT equals
  the generated value to within 10⁻⁹ °C; 2.19 °C is not flagged, 2.20 °C and
  2.21 °C are flagged, and the symmetric scene raises no flag.
- That the result follows anatomy, not image position: swapping the feet in
  the image, or rotating them, gives the same answer.
- Registration on a synthetic target with a known transform, and rejection
  above the error limit.
- That no patient is shared between training and testing in any fold, across
  several fold counts and seeds, and that a deliberately leaked patient is
  caught.
- That no part of the code names, stores or displays a classifier output as a
  grade.
- That a complete scan makes no network call.

**What the tests do not establish.** They show the software does what it was
written to do on idealised inputs. They do not show that the Lepton reads true
temperatures, that real feet segment as cleanly as synthetic ones, or anything
about clinical validity. Those require the hardware checks in section 13 and
the research team's own study.

The acceptance procedure for the delivered station is in
`docs/acceptance_test.md`.

## 12. Limitations

1. **Sensor accuracy.** The Lepton 3.5's absolute accuracy is modest (section
   4). The study's measure is a within-frame difference, which removes a
   common offset but not variation across the field of view. A flat-field
   uniformity check against a uniform-temperature surface is advisable.
2. **Resolution.** One thermal pixel covers about 2.85 mm. Region means are
   robust to this; small hot spots are averaged with their surroundings. The
   smallest regions (toes) contain the fewest pixels.
3. **Emissivity and viewing angle.** Temperatures assume the emissivity the
   Lepton is configured for. Skin is close to 0.98; the sensor's default
   setting should be confirmed and reported. Curved surfaces viewed obliquely
   (the sides of the toes and heel) read slightly low.
4. **Mask threshold.** Skin within 2.0 °C of the background is outside the
   mask and is not measured. Very cold toes in a patient with poor circulation
   may be partly or wholly excluded. Pixel counts per region are recorded so
   this can be detected.
5. **Region map.** The region boundaries are proportional defaults pending
   clinical review (section 7). They assume a complete foot; deformity or
   partial amputation shifts the proportions.
6. **Foot orientation.** The long axis comes from the mask's principal axis,
   which on test shapes leans about 2° toward the hallux. The lean is the same
   on both feet, so matched regions still correspond, but region boundaries
   are approximate at the level of a pixel or two.
7. **Left-right assignment** relies on the operator's confirmation. The shape
   rule can fail on unusual feet.
8. **Registration** is exact only in the plane where the target was held. With
   40 mm between cameras, a point 30 mm nearer or farther shifts by about one
   thermal pixel. The colour camera may not cover the full height of the
   thermal frame; the covered fraction is recorded per scan. This affects the
   classifier input and the overlay, not the temperature measurement.
9. **Protocol dependence.** The ambient gate and rest period control the main
   confounders but not all of them (recent activity, footwear, room draughts,
   time since the feet were uncovered before the timer started).
10. **Classifier.** Its performance is a property of the dataset. With few
    patients in a class, estimates are unstable and should be reported with
    the patient counts. Quantisation for the Hailo-8L changes outputs slightly
    relative to the floating-point model; the two should be compared on the
    same images.
11. **Clock.** Timestamps come from the station's clock, which is not
    network-synchronised because the station is offline.

## 13. Status at the time of writing

This document was written when the software was complete and tested off the
device, and before the following had been done. None of these results exist
yet, and none have been assumed.

- The PureThermal driver, the Camera Module 3 driver, the GPIO devices, the
  ambient sensor driver and the Hailo runner had not been run on the station's
  hardware.
- The reference-thermometer check had not been performed.
- Registration had not been calibrated on the real unit.
- No model had been trained on real data, and none had been compiled for the
  Hailo-8L. No performance figure exists.
- The region map had not been reviewed by a clinician.

`BLOCKERS.md` in the repository lists each item and what completes it.
`DECISIONS.md` lists every default the builder chose that the research team or
the owner should review.
