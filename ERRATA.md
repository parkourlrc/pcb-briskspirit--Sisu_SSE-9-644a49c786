# Rev B2 Errata

Rev B2 has been fabricated and assembled. The findings below are based on the
current source, the fabrication archive, and bench work on the assembled board.
Treat them as mandatory review items before building another board.

## Functional Issues

### J2 MHF4L footprint shorts the RF feed

The current `3210:I-PEX_MHF4L_20579-001E` footprint does not match the intended
four-contact MHF4L pin assignment. Populating J2 can short the antenna feed to
ground.

Do not populate J2 on Rev B2. Verify feed-to-ground continuity before powering
the modem and use a validated coax launch or direct pigtail for RF work. Rev C
needs a footprint rebuilt from the exact connector drawing and checked against
the physical part.

### U6 and U14 can backfeed service VBUS

U6 and U14 are USBLC6-2SC6 rail-steering ESD arrays. An active USB pull-up can
feed their VBUS clamp pin even when the external service supply is absent. On
the Telit side this prevented the modem from returning to sleep.

The tested Rev B2 assembly has pin 5 lifted on both U6 and U14. Rev C should use
ESD protection without a VBUS steering path and must verify leakage in every RP
and modem power state.

### SIM supply can contact the holder shield

The current SIM holder can allow its VSIM contact to touch grounded metal,
intermittently shorting the SIM supply. Check continuity while inserting,
pressing, and moving the SIM before applying power. Rev C should use the
Kyocera AVX `009162006006080` or another mechanically verified holder.

### Charger can report false FULL after interruption

Bench testing found that briefly interrupting and reconnecting the charger near
a 2.69 V battery condition could make the BQ25171-Q1 report `FULL` and refuse to
resume charging, despite the battery being only partially charged. This remains
unresolved. Do not treat the charger status output alone as proof of a full
battery after an input interruption.

### C98 is below the charger auxiliary-regulator requirement

C98 is 1 uF on the TPS7B69 charger auxiliary 3.3 V output. The regulator
requires at least 2.2 uF. Retrofit C98 to 4.7 uF where practical and make 4.7 uF
the Rev C default.

## Required Rev B2 Rework

The root project now records these tested assembly changes. The historical Rev
B2 ordering archive predates them.

| References | Current value | Purpose |
| --- | --- | --- |
| R71, R73 | 1.10 kOhm, 1% | Approximately 273 mA nominal BQ25171 charge-current selections |
| R70 | 24 kOhm, 1% | Approximately eight-hour NiMH safety timer |
| R74 | 82 kOhm, 1% | Li-ion timer selection, unchanged |
| R77 | 10 kOhm | Holds `CHR_EN_N` low while the TCA8418 is in reset |

R77 must be populated. The BQ25171's internal pull-down is too weak to overcome
the TCA8418 reset-state pull-up on the shared control line.

## Mechanical and Assembly Risks

- The bent portion of the positive battery spring can wear through solder mask
  and touch the adjacent ground pour. Inspect or insulate this area on Rev B2.
  Rev C needs a copper keepout or a larger positive-contact land.
- Add a copper and via keepout beneath the moving portion of the shield spring.
- The Telit alignment marks sit too far under the 29.4 mm WWX module to be easy
  to see during placement.
- The no-silkscreen assembly process needs exposed-copper orientation marks for
  ICs, diodes, polarized capacitors, and ambiguous connectors.

## Provisional RF Tuning

These are bench values, not universal production values. Revalidate them with
the final antenna, enclosure, modem, PCB fabrication, and calibration setup.

| Position | Value | Position | Value |
| --- | --- | --- | --- |
| ZP1 | 1.0 pF | ZS1 | 2.4 nH |
| ZP2 | 3.0 pF | ZS2 | 14 pF |
| ZP3 | 27 nH | ZS3 | 0 Ohm |
| ZP4 | DNP | ZS4 | 0 Ohm |
| ZP5 | DNP | ZP7 | 1.4 pF |
| ZP9 | 10 nH | ZP11 | DNP |
| ZP6 | 15 nH | ZP8 | 6.8 nH |
| ZP10 | DNP | ZP12 | DNP |

ZS2 was validated on the prototype as 10 pF plus a 4 pF piggyback capacitor.
The schematic records the resulting 14 pF target value.
