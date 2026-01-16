# BOSS GX‑10 TSL Format Specification

This document describes the JSON‑based Tone Studio LiveSet (TSL) file format used by the BOSS GX‑10 multi‑effect processor. A TSL file encapsulates one or more user patches with complete parameter sets and is intended for exchange and backup via BOSS Tone Studio.

## Top‑level structure
A TSL file is a JSON object with these keys:

- **name** – The live‑set name.
- **formatRev** – Format revision (e.g. `"0000"`).
- **device** – Device model (always `"GX‑10"`).
- **data** – An array of patch arrays; each element in `data` is itself an array containing a single patch object.

## Patch object
Each patch object contains:
- **memo** – A free‑form description of the patch.
- **paramSet** – A dictionary of parameter arrays keyed by descriptive names.

### Common parameters
The `User_patch%common` entry holds 62 hex strings. The first 16 bytes encode the patch name in ASCII (padded or truncated to fit); the remaining bytes store various flags.

### LED parameters
`User_patch%led` is a 5‑byte array controlling the colour and status of the footswitch LEDs.

### Assign parameters
Eight assign blocks (`User_patch%assign(1)` … `User_patch%assign(8)`) map control targets to hardware knobs/switches. Each is a 5‑byte array.

### Effect chain
The `User_patch%efct` array (62 bytes) defines the signal routing. Bytes 0–11 store chain metadata; bytes 12–61 map signal positions to effect slots.

### Effect items
Up to 20 effect slots (`User_patch%fxItem(1)` … `User_patch%fxItem(20)`) may be present. Each is a 179‑byte array:
- **byte 0** – Effect type ID (`00` = disabled).
- **byte 1** – On/off flag (`01` = active).
- **bytes 2–178** – Effect‑specific parameters, often encoded as three‑byte groups `[value_msb, value_lsb, address]`.

#### Example effect type IDs (from file analysis)

| Hex | Dec | Slot              | Category (hypothesis)      |
|----:|----:|--------------------|-----------------------------|
| 02  |  2  | fxItem 5, 12, 15  | Compressor / EQ             |
| 04  |  4  | fxItem 7          | Distortion / Overdrive      |
| 08  |  8  | fxItem 13         | Modulation                  |
| 09  |  9  | fxItem 1          | Chorus / Flanger            |
| 0C  | 12  | fxItem 17         | Delay                       |
| 0E  | 14  | fxItem 2          | Reverb / Ambience           |
| 10  | 16  | fxItem 14         | Equalizer                  |
| 15  | 21  | fxItem 4          | Amp simulator               |
| 20  | 32  | fxItem 9          | Special effect              |
| 32  | 50  | fxItem 6, 18      | Special effect              |
| 3F  | 63  | fxItem 3          | Special effect              |

## Encoding conventions
All arrays use two‑character hex strings. Multi‑byte parameters are typically big‑endian and may be grouped as `[value_msb, value_lsb, address]`, where the last byte denotes the parameter address within the effect.

## Practical example
The following `fxItem(1)` fragment shows an active effect (type `09`, on) followed by parameter triples:


