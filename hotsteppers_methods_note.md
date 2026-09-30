# Hotsteppers paired-sensor feasibility trial — methods note

**Date:** 26 September 2026 · **Site:** University of Lagos campus, Lagos, Nigeria
**Timezone:** Africa/Lagos (UTC+1) · **AirCasting session:** "Run4CleanAir Unilag 3"

## Objective

Test whether a temperature logger with no positioning capability (Maxim/Analog Devices
iButton) can be combined with a GPS-equipped air-quality monitor (AirBeam 3) to produce a
single geolocated record of heat and particulate exposure along a walking route, using
timestamps alone as the link.

## Instrumentation and pairing

| Pair | iButton | AirBeam | Walk window | Duration | AirBeam readings |
|------|---------|---------|-------------|----------|------------------|
| 28 | No. 28 (AE00000080851A41) | 94e686f5c338 | 07:49:04 – 08:51:02 | 62.0 min | 3,683 |
| 29 | No. 29 (BA0000008083F241) | 0cb815a91e2c | 07:39:38 – 09:01:37 | 82.0 min | 4,771 |

iButtons logged temperature and relative humidity at 60-second intervals; AirBeams logged
PM1, PM2.5, PM10 and GPS position at approximately 1-second intervals. Both iButtons logged
continuously from about 07:15 to about 10:09, giving roughly 90 minutes of margin around each
AirBeam session.

## Collection mode

One collector ran and the other walked, which is why the two sessions differ in length. The GPS
traces are consistent with this: set 1 moved at a median 5.0 km/h with bursts reaching 9.4 km/h
sustained over a full minute and spent 17.5% of its time above 6.5 km/h; set 2 held a steady
4.6 km/h, exceeded 6.5 km/h for only 1.3% of the walk, and was stationary for 7.3% of it. Set 1's
running was intermittent rather than sustained.

## Time synchronisation

**The two devices were not set to a common reference clock before this trial.** The match still
came out at 100%, which indicates the clocks happened to be close enough over a session of about
an hour, but this was not established by procedure and must not be relied on. Setting both devices
from a single time source is the first recommendation for the actual campaign.

What was done:

1. iButton logging began well before, and ended well after, each AirBeam session, so the AirBeam
   window sat entirely inside the iButton window.
2. Each iButton was assigned to one specific AirBeam as a fixed pairing.

## Data processing

1. Each AirBeam reading was matched to the nearest iButton reading within a 30-second
   tolerance (`pandas.merge_asof`, `direction="nearest"`).
2. GPS quality control: the great-circle step distance and implied speed were computed between
   consecutive fixes. Steps implying more than 4 m/s on a walking survey were flagged
   (`gps_ok = False`) and excluded from route length and mapping. Rows were retained, not deleted.
3. No filtering of temperature or humidity: every iButton reading is shown and counted exactly as
   recorded. The `heat_ok` column from the source export is carried through the data files but is
   not applied to any figure in this report.
4. The AirBeam's onboard temperature and humidity channels were not used in any analysis. Heat
   comes exclusively from the iButtons; particulates exclusively from the AirBeams.

## Results

- **Match rate: 100%.** 8,454 of 8,454 AirBeam readings matched an iButton reading
  (pair 28: 3,683/3,683; pair 29: 4,771/4,771).
- **Clock drift: none corrected.** `timestamp_shift_seconds` is zero throughout. 62 and 79
  minutes coincided exactly to the second before any tolerance was applied.
- **Inter-unit agreement:** r = 0.94 across 173 shared minutes; mean difference 0.44 °C,
  SD 0.50 °C, maximum 2.45 °C. Pre-walk, co-located: r = 0.97 (23 minutes).
  During the walks: r = 0.76, SD 0.33 °C.
- **Route length:** 5,100 m (pair 28) and 5,843 m (pair 29) after GPS filtering; 10.94 km
  combined. Mean walking speed 1.37 and 1.19 m/s.
- **Temperature range across the session, all readings:** 27.1–29.6 °C (pair 28, mean 27.6 °C,
  RH 85%) and 27.1–29.1 °C (pair 29, mean 27.4 °C, RH 84%). Setting aside the first few minutes of
  each session, while both loggers were settling, the spread over the rest of the route was 1.5 °C
  and 1.0 °C. Quoted to 0.1 °C; the DS1923 is specified to roughly ±0.5 °C, so further decimals
  would be false precision.
- **PM2.5:** mean 18.4 and 19.9 µg/m³; above 15 µg/m³ for 80.6% and 69.2% of readings.
  Maximum 172 µg/m³ (pair 29, 08:57:55, 6.517853 N 3.389207 E), with PM10 322 and PM1 138
  µg/m³ in the same seconds.

## Data completeness

The iButtons lost nothing: both logged continuously at exactly 60-second intervals with no gap
anywhere (175 and 174 readings across roughly three hours).

The AirBeams dropped short stretches. Set 1 recorded 3,683 readings across 3,719 seconds of wall
clock, missing 36 seconds in 36 gaps of one to two seconds (1.0% of the session). Set 2 recorded
4,771 across 4,920 seconds, missing 149 seconds in 131 gaps (3.0%), the longest a 12-second break
at 08:15:32. These seconds were never recorded by the instrument; nothing was lost in the join.

Every iButton reading is accounted for. Logger 28 recorded 175 readings (07:16–10:10) and logger 29
recorded 174 (07:15–10:08). Of these, 62 and 82 fall inside the respective AirBeam session and are all
plotted and counted exactly as recorded. The remainder — 34 before and 79 after
for logger 28, 25 before and 67 after for logger 29 — were taken while the AirBeam was not recording and
therefore have no GPS position; they appear in the data files and in the full-record chart but cannot be
mapped.

The temperature and humidity traces span slightly less time than PM2.5 for two reasons. First,
the iButton writes on the minute, so its first reading inside an AirBeam session arrives up to 60 s
after that session starts and its last can fall up to 60 s before it ends: set 1's in-window
readings run 07:50:01 – 08:51:01 against an AirBeam window of 07:49:04 – 08:51:02, and set 2's run
07:40:01 – 09:01:01 against 07:39:38 – 09:01:37. No reading is excluded.

## Limitations

1. **Thermal lag.** The iButton's sealed steel case has significant thermal mass and responds
   over minutes. Recorded range across a full hour of varied campus environment was only
   1.0–1.5 °C, considerably less than true air temperature variation between sunlit and shaded
   ground. The heat record is valid as a route-level average, not as a microscale measurement.
2. **GPS spikes.** Pair 28 produced 29 position jumps above 4 m/s (maximum 75.6 m/s), inflating
   raw route length from 5,100 m to 5,601 m. Pair 29 produced 6 (maximum 4.6 m/s).
3. **Start and end excursions.** Both units recorded sharp temperature excursions at the same three
   moments: 07:23–07:39 (before departure), 07:47–07:52 (start of route) and 09:46–09:56 (around
   collection), reaching 34.06 °C at 07:33. Readings that high are not plausible for Lagos air at
   that hour, so the loggers were measuring something other than open air during those minutes. The
   cause is not established by this data and no field log of device handling was kept, so it can
   only be inferred. All of these readings are retained and shown.
4. **No reference calibration.** Sensor accuracy was not characterised against a calibrated
   reference. Recorded temperatures are consistent with September climatology for Lagos
   (23.8–27.2 °C, mean RH 87%, climate-data.org 1991–2021) but this is a plausibility check,
   not a calibration.
5. **Scope.** One morning, one campus, two device pairs. Demonstrates method feasibility only.

## Recommendations for the full campaign

1. Retain the clock protocol unchanged — it performed without error.
2. Add a deliberate thermal sync marker at the start and end of each route so alignment can be
   verified from the data rather than assumed.
3. Reconsider the mobile heat sensor: use a faster aspirated sensor for walking transects and
   redeploy iButtons to fixed points, where slow response is an advantage.
4. Standardise shielding and mounting: radiation shield with airflow, breathing height, on a
   boom clear of the carrier's body; document the rig photographically.
5. Batch-calibrate all loggers against a reference before and after the campaign.
6. Repeat routes across times of day before attributing differences to places.

## Files

- `sensor_readings_combined.csv` — original merged export, 8,664 rows, unchanged.
- `paired_clean.csv` — one row per AirBeam reading with matched iButton values and QC flags.
- `routes.geojson` — walked tracks as LineStrings plus PM2.5 peak points.
