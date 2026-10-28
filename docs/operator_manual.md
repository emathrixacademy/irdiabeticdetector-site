---
layout: doc
title: Operator manual
---

# Operator manual

DFU Thermography Screening Station

This station is a **research instrument**. It measures skin temperature on
the soles of both feet and records a colour picture. It does not diagnose,
grade or treat. What appears on the screen is for the research record. Do not
tell a patient what a result means; that is for the study clinician.

---

## Before you start: three rules

1. **Never type a patient's name.** Use only the study reference code the
   research team gave you, for example `DFU-0001`.
2. **Do not scan outside 22–25 °C.** The station will refuse, and there is no
   way round it. Fix the room, not the station.
3. **The patient rests for 15 minutes first.** The station counts it and will
   not scan early.

---

## 1. Prepare the room

- Bring the room to **22–25 °C** and hold it there. Start the air conditioning
  well before the first patient.
- Close doors and windows. Turn fans away from the cabinet and the patient's
  feet. No draught across the feet.
- Keep sunlight, lamps, heaters and anything hot or cold away from the cabinet
  opening.
- Place the bed or chair so the patient's feet reach the cabinet opening
  comfortably. Set the cabinet height with the telescoping legs so the
  opening is level with the feet.

## 2. Start the station

1. Switch on the power. The station starts by itself and shows
   **Enter the study reference code**.
2. Check the status light. It should be off (idle).
3. **Check the station clock** on the code screen, in the lines under the code
   box, once a day before the first patient. The station is not connected to
   the internet and cannot set its own clock. If the date or time is wrong,
   stop and call the technician: every scan is stamped with it.
4. On the same lines, check **Storage free**. If the screen says storage is
   nearly full, tell the technician before scanning.
5. If any heading on the screen starts with **DEMO**, the station is running
   on stand-in pictures for training or testing. **Do not scan patients.**
   Nothing it shows is a measurement. Call the technician.

During data collection the code screen also says "Data collection: no colour
model". That is expected: scans are measured and stored in full, and the
results screen simply has no predicted class.

If the station shows a message instead of the code screen, read it. A message
that says to call the technician means do not scan.

## 3. Position the patient

The patient may lie on their back (supine) or sit upright with legs out
straight. Either way:

- **Shoes, socks, stockings and dressings off both feet**, and trouser legs
  rolled above the ankle. Do this at the start, because the rest period only
  counts with the feet uncovered.
- Feet go into the opening **heel first**, soles facing the back wall where
  the cameras are, **toes pointing up**.
- Rest the heels or ankles on the padded foot rest so the legs are relaxed.
- Feet **slightly apart, not touching**, and not touching the cabinet walls.
- Both whole soles must be inside the picture, toes to heel.
- Close the blackout shroud around the legs.

**Supine:** a pillow under the knees keeps the legs relaxed and the soles
upright. Check the feet have not rolled outward.

**Seated:** support the back so the patient is not holding their legs up by
effort. If the heels do not reach the foot rest, move the chair, not the
cabinet.

Ask the patient to keep their feet still and not rub them together.

## 4. Enter the reference code and wait

1. On the screen, tap the reference code using the keys, then tap
   **Start rest period**.
2. The screen changes to show:
   - a **thermal view** of the feet (colours show warmth),
   - a **colour view**, which stays dark because the inside light is off until
     the scan,
   - the **rest time remaining**, counting down from 15:00,
   - the **room temperature and humidity**.
3. Use the thermal view to check both soles are fully inside the picture and
   apart. Adjust the patient now, early in the rest period.

The status light blinks slowly during the rest period.

If the patient puts socks back on, walks about, or you had to move their feet
a lot, tap **Restart rest period** so the full 15 minutes is counted again.

## 5. Scan

When the countdown reaches 00:00 **and** the room is in range, the screen says
**Ready to scan**, the status light is steady and the **CAPTURE** button turns
green.

1. Ask the patient to hold still.
2. Press the **capture button on the cabinet** or tap **CAPTURE** on the
   screen. They do the same thing.
3. Wait about five seconds. You will hear the thermal camera click, the inside
   light comes on briefly for the colour picture, goes off, and then the
   thermal picture is taken. The status light blinks quickly. **Do not open
   the shroud or move the feet until the next screen appears.**

## 6. Confirm left and right

The screen shows the thermal picture with the two feet marked **A** and **B**
and asks: **Which is the patient's LEFT foot?**

- It tells you which one it thinks is the left foot, from the shape.
- **Look at the patient**, not just the picture, and tap **Foot A is the LEFT
  foot** or **Foot B is the LEFT foot**.

Take a moment over this. Normally the patient's left foot appears on the right
of the picture, because the cameras face the patient. If the station's guess
is wrong, tap the correct one; it records that you corrected it.

If the picture is plainly bad (a foot moved, something was in the way), tap
**Discard and scan again**. Nothing is saved.

## 7. Results

The results screen shows:

- the thermal picture with eight regions outlined on each foot,
- the colour picture,
- a table with the temperature of each region on the left and right foot and
  the difference between them,
- **FLAG** beside any region where the two feet differ by 2.2 °C or more,
- a line giving the colour model's predicted class, or "Colour model: not
  installed" while the station is collecting data without one.

The scan is already saved when this screen appears.

Then tap:

- **Scan again** for another scan of the same patient. No new rest period is
  needed if the feet stayed uncovered and still.
- **Next patient** when you are finished with this patient.

**What to say to the patient.** The screen is a research record, not a result
for the patient. A FLAG is a temperature difference, not a diagnosis, and the
predicted class is a computer model's output, not a grade. Refer any question
about what it means to the study clinician.

## 8. If the station refuses to scan

| Message | What to do |
|---|---|
| Room too warm / Room too cold | Adjust the room. Wait for the reading on the screen to come into 22–25 °C. |
| Rest period not finished | Wait for the countdown. |
| Foot out of view | Move the feet so both soles are fully inside the thermal view. Scan again. |
| Both soles are needed | Check both feet are in, apart and not touching, and nothing else warm (a hand, a hot-water bottle) is inside. Scan again. |
| Feet not clear | Check toes point up and soles face the cameras. Scan again. |
| Feet could not be compared | Reposition and scan again. |
| Thermal camera busy / Thermal picture not usable | Scan again. |
| Room sensor not responding / Thermal camera not responding / Equipment problem | Try once more. If it repeats, stop and call the technician. |
| Station needs calibration / Thermal camera not ready to measure / Station settings problem | **Do not scan.** Call the technician. |
| Scan could not be saved | **Do not scan.** The storage may be full. Call the technician. |
| Reference code not accepted | Type the code exactly as given. Never a name. |

When a scan is refused, **nothing is saved**. There is no half-finished scan
to tidy up.

The status light flashes briefly once a second when there is a problem.

## 9. Status light

| Light | Meaning |
|---|---|
| Off | Idle, waiting for a reference code |
| Slow blink | Rest period running, or waiting for the room temperature |
| Steady on | Ready to scan |
| Fast blink | Scanning. Keep still. |
| Short flash once a second | A scan was refused or something is wrong. Read the screen. |

## 10. Between patients: cleaning

Follow your facility's infection-control policy. As a minimum:

- Wipe the **padded foot rest** and any part of the cabinet the patient
  touched with the disinfectant wipe your facility uses for patient-contact
  surfaces. Let it dry completely before the next patient; a wet or freshly
  wiped foot rest is cold and can chill the heels.
- **Do not wipe, spray or touch the two camera lenses** on the back wall. If a
  lens looks dirty, tell the technician.
- Do not spray liquid into the cabinet. Wipe only.
- Replace the shroud if it becomes soiled.

Take extra care with any patient who has an open wound, in line with your
facility's wound-care precautions.

## 11. Shutting down

1. Finish with the last patient and tap **Next patient** so the code screen is
   showing.
2. Tap **Exit** on the code screen to close the station app, then choose
   Shutdown from the desktop menu. Wait until the Pi's green light stops
   blinking.
3. Switch off the power.

Do not pull the power while the station is running. It can damage the storage
card. A scan in progress when the power is cut is not saved, but earlier scans
are safe.

## 12. Where the scans are

On the station's storage, in the `data` folder: one folder per scan, and one
list of all scans called `scans.csv`. Nothing is sent anywhere. Copying the
data off the station is done by the research team under their own data
procedures, not by the operator.
