# BOSS GX-10 TSL File Format Specification

**VERSION:** 2.0 (Updated with actual file analysis)
---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [File Format Overview](#file-format-overview)
3. [Top-Level Structure](#top-level-structure)
4. [Data Array Structure](#data-array-structure)
5. [ParamSet Structure](#paramset-structure)
6. [Common Parameters](#common-parameters)
7. [LED Parameters](#led-parameters)
8. [Assign Parameters](#assign-parameters)
9. [Effect Chain Parameters](#effect-chain-parameters)
10. [FxItem Parameters](#fxitem-parameters)
11. [Data Encoding Conventions](#data-encoding-conventions)
12. [Practical Examples](#practical-examples)
13. [AI Generation Guidelines](#ai-generation-guidelines)

---

## Executive Summary

The BOSS GX-10 TSL (Tone Studio Liveset) file format is a JSON-based proprietary format used to store and exchange guitar effect patches. This specification is based on analysis of an actual GX-10.tsl.json file and provides detailed documentation of the complete structure.

### Key Characteristics

- **Format:** JSON text file
- **Encoding:** Hexadecimal string values for all parameters
- **Structure:** Hierarchical with fixed-size parameter arrays
- **Patches:** Multiple patches can be stored in nested array structure
- **Parameters:** ~18 active effects with 179-byte parameter blocks each

### Critical Findings

1. **All numeric values are hex-encoded strings** (e.g., "08", "0F", "1E")
2. **Fixed array sizes** for each parameter category
3. **Patch name is ASCII-encoded** in first 16 bytes of common parameters
4. **Effect chain routing** is separate from effect parameters
5. **3-byte parameter encoding** appears to be used in fxItem blocks

---

## File Format Overview

### File Extension
`.tsl.json` or `.tsl`

### MIME Type
`application/json`

### Character Encoding
UTF-8

### Structure Type
JSON with nested arrays and objects containing hex-encoded parameter data

---

## Top-Level Structure

The root JSON object contains four mandatory fields:

```json
{
  "name": "My",
  "formatRev": "0000",
  "device": "GX-10",
  "data": [[...]]
}
```

### Top-Level Fields

| Field | Type | Required | Description | Example Value |
|-------|------|----------|-------------|---------------|
| `name` | String | Yes | Liveset name | `"My"` |
| `formatRev` | String | Yes | Format revision identifier | `"0000"` |
| `device` | String | Yes | Target device identifier | `"GX-10"` |
| `data` | Array | Yes | 2D array containing patch data | `[[{...}]]` |

### Field Details

#### `name`
- **Purpose:** Identifies the liveset in BOSS Tone Studio
- **Format:** UTF-8 string
- **Length:** Variable (typically short names)
- **Example:** `"My"`, `"Rock Patches"`, `"Live Set 2024"`

#### `formatRev`
- **Purpose:** Version identifier for the TSL format
- **Format:** 4-character string
- **Known Values:** `"0000"` (current GX-10 format)
- **Future-proofing:** May change with firmware updates

#### `device`
- **Purpose:** Specifies the target device model
- **Format:** String identifier
- **Valid Values:** `"GX-10"` (other BOSS devices use different identifiers)
- **Importance:** Ensures compatibility with correct hardware

#### `data`
- **Purpose:** Contains all patch information
- **Structure:** 2-dimensional array `[[patch1], [patch2], ...]`
- **Current Implementation:** Single liveset with single patch: `[[{patch_object}]]`
- **Expandability:** Can potentially hold multiple patches in nested structure

---

## Data Array Structure

### Hierarchy

```
data (Array)
└── [0] (Array) - First liveset
    └── [0] (Object) - First patch
        ├── memo (String)
        └── paramSet (Object)
```

### Patch Object

Each patch is represented by an object with two fields:

```json
{
  "memo": "",
  "paramSet": {
    "User_patch%common": [...],
    "User_patch%led": [...],
    "User_patch%assign(1)": [...],
    ...
    "User_patch%fxItem(20)": [...]
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `memo` | String | Yes | User notes/description for the patch |
| `paramSet` | Object | Yes | All patch parameters organized by category |

---

## ParamSet Structure

The `paramSet` object contains 43 keys organized into 5 categories:

### ParamSet Key Categories

1. **Common Parameters:** `User_patch%common` (1 key)
2. **LED Parameters:** `User_patch%led` (1 key)
3. **Assign Parameters:** `User_patch%assign(1)` through `User_patch%assign(20)` (20 keys)
4. **Effect Chain:** `User_patch%efct` (1 key)
5. **Effect Items:** `User_patch%fxItem(1)` through `User_patch%fxItem(20)` (20 keys)

### Complete ParamSet Key List

```javascript
{
  "User_patch%common": [129 hex strings],
  "User_patch%led": [28 hex strings],
  "User_patch%assign(1)": [45 hex strings],
  "User_patch%assign(2)": [45 hex strings],
  ...
  "User_patch%assign(20)": [45 hex strings],
  "User_patch%efct": [62 hex strings],
  "User_patch%fxItem(1)": [179 hex strings],
  "User_patch%fxItem(2)": [179 hex strings],
  ...
  "User_patch%fxItem(20)": [179 hex strings]
}
```

### Array Size Summary

| Parameter Category | Array Size | Count | Total Bytes |
|-------------------|------------|-------|-------------|
| `common` | 129 bytes | 1 | 129 |
| `led` | 28 bytes | 1 | 28 |
| `assign(n)` | 45 bytes | 20 | 900 |
| `efct` | 62 bytes | 1 | 62 |
| `fxItem(n)` | 179 bytes | 20 | 3,580 |
| **TOTAL** | - | **43 arrays** | **4,699 bytes** |

---

## Common Parameters

**Key:** `User_patch%common`
**Size:** 129 bytes (hex strings)
**Purpose:** Global patch settings including name, master controls, and general configuration

### Structure Overview

The common parameter array contains multiple logical sections:

| Byte Range | Size | Purpose |
|------------|------|---------|
| 0-15 | 16 bytes | Patch name (ASCII encoded) |
| 16-31 | 16 bytes | Primary configuration parameters |
| 32-95 | 64 bytes | Extended parameters and reserved space |
| 96-128 | 33 bytes | Additional configuration and control settings |

### Detailed Byte Map

#### Bytes 0-15: Patch Name

```javascript
["56","69","6f","6c","69","6e","5f","6c","69","76","65","20","20","20","20","20"]
// Decodes to: "Violin_live     " (spaces pad to 16 chars)
```

- **Encoding:** ASCII character codes in hexadecimal
- **Length:** Exactly 16 characters (pad with spaces "20" if shorter)
- **Character Set:** Printable ASCII (0x20-0x7E recommended)
- **Example Decoding:**
  - `"56"` = 86 decimal = 'V'
  - `"69"` = 105 decimal = 'i'
  - `"6f"` = 111 decimal = 'o'
  - `"20"` = 32 decimal = space

**To encode a patch name:**
```python
def encode_patch_name(name):
    # Truncate or pad to 16 characters
    name = name[:16].ljust(16)
    # Convert to hex strings
    return [f"{ord(c):02X}" for c in name]
```

#### Bytes 16-31: Primary Configuration

| Byte | Hex | Dec | Purpose/Hypothesis |
|------|-----|-----|--------------------|
| 16 | 02 | 2 | Category or patch type |
| 17 | 03 | 3 | Sub-category or variation |
| 18 | 01 | 1 | Enable flag or mode |
| 19 | 01 | 1 | Enable flag or mode |
| 20 | 01 | 1 | Enable flag or mode |
| 21 | 01 | 1 | Enable flag or mode |
| 22-27 | 00 | 0 | Reserved or unused |
| 28 | 08 | 8 | Control parameter |
| 29-30 | 00 | 0 | Reserved |
| 31 | 0C | 12 | Control parameter |

#### Bytes 96-128: Extended Configuration

**Notable values in example:**
- Bytes 105-107: `04 04 04` - Repeated value (possibly channel or routing)
- Bytes 111-124: Pattern of values including `0A 04`, `0A 06`, `0A 08`, `0B 05`
  - This may represent paired parameters (type + value combinations)

### ASCII Conversion Reference

Common characters for patch names:

| Char | Hex | Dec | Char | Hex | Dec | Char | Hex | Dec |
|------|-----|-----|------|-----|-----|------|-----|-----|
| Space | 20 | 32 | 0 | 30 | 48 | A | 41 | 65 |
| ! | 21 | 33 | 1 | 31 | 49 | B | 42 | 66 |
| - | 2D | 45 | 9 | 39 | 57 | Z | 5A | 90 |
| _ | 5F | 95 | a | 61 | 97 | z | 7A | 122 |

---

## LED Parameters

**Key:** `User_patch%led`
**Size:** 28 bytes (hex strings)
**Purpose:** Controls LED display settings and color assignments

### Structure

The LED array appears to use a 2-byte pair structure (14 pairs):

```javascript
["00","0B", "00","0B", "00","0B", "00","0B", "00","0B",
 "00","0B", "00","0B", "00","0B", "00","0B", "00","0B",
 "00","00", "03","04", "02","00", "08","00"]
```

### LED Pair Analysis

| Pair Index | Bytes | Hex Values | Dec Values | Possible Meaning |
|------------|-------|------------|------------|------------------|
| 0-9 | 0-19 | 00 0B | 0, 11 | Repeated pattern - default LED state |
| 10 | 20-21 | 00 00 | 0, 0 | LED off or different state |
| 11 | 22-23 | 03 04 | 3, 4 | LED mode/color setting |
| 12 | 24-25 | 02 00 | 2, 0 | LED mode/color setting |
| 13 | 26-27 | 08 00 | 8, 0 | LED mode/color setting |

### Encoding Pattern

Each 2-byte pair likely represents:
- **Byte 1:** LED state/mode (0-255)
- **Byte 2:** LED color or brightness (0-255)

### Typical Values

- `00 0B` (0, 11): Default/inactive state
- `00 00` (0, 0): Off
- `03 04` (3, 4): Active state with specific color
- `02 00` (2, 0): Alternative state
- `08 00` (8, 0): Alternative state

**Note:** The exact meaning of LED parameters requires correlation with the GX-10's physical LED indicators and may vary based on the current effect or mode.

---

## Assign Parameters

**Keys:** `User_patch%assign(1)` through `User_patch%assign(20)`
**Size:** 45 bytes each (hex strings)
**Purpose:** Configure control assignments (expression pedal, footswitches, etc.)

### Overview

The GX-10 supports up to 20 control assignments. Each assignment maps a physical control (pedal, switch, knob) to a target parameter (effect parameter, volume, etc.).

### Active vs. Inactive Assignments

An assignment is **active** if its first byte is non-zero:

**Active Assignment Example (Assign 1):**
```javascript
["01","02","00","00","00","01","08","00","00","00","08","00","00","01","0B",...]
//  ^^                     ^^
// Enable=1              Source=1
```

**Inactive Assignment Example (Assign 4):**
```javascript
["00","00","00","00","00","00","08","00","00","00","08","00","00","01","0B",...]
//  ^^
// Enable=0
```

### Assignment Structure

The 45-byte structure can be divided into logical sections:

| Byte Range | Size | Purpose |
|------------|------|---------|
| 0 | 1 byte | Enable/Disable (00=off, 01=on) |
| 1 | 1 byte | Target parameter or effect |
| 2-4 | 3 bytes | Additional target configuration |
| 5 | 1 byte | Source type (pedal, switch, etc.) |
| 6-13 | 8 bytes | Source configuration |
| 14-44 | 31 bytes | Value ranges, curves, and additional settings |

### Example Assignment Breakdown

**Assign(1) - Active Assignment:**

```javascript
["01", "02", "00","00","00", "01", "08","00","00","00", "08","00","00", "01","0B",
 "00", "14", "01", "0C", "1E", "00","00","00","00","00", "03","0F","0F","0F",
 "00","00","00","00","00","00", "03","0F","0F","0F", "00","00","00","00","00","00"]
```

| Bytes | Hex | Dec | Parameter |
|-------|-----|-----|-----------|
| [0] | 01 | 1 | **Enable:** Assignment active |
| [1] | 02 | 2 | **Target:** Parameter ID 2 |
| [2-4] | 00 00 00 | 0,0,0 | Target modifiers |
| [5] | 01 | 1 | **Source:** Type 1 (e.g., expression pedal) |
| [6-9] | 08 00 00 00 | 8,0,0,0 | Source configuration |
| [10-12] | 08 00 00 | 8,0,0 | Source range/scaling |
| [13-14] | 01 0B | 1,11 | Additional parameters |
| [15-44] | ... | ... | Value curves and ranges |

### Common Source Types

Based on analysis, source type (byte 5) likely maps to:

| Value | Decimal | Source Type |
|-------|---------|-------------|
| 00 | 0 | Disabled/None |
| 01 | 1 | Expression Pedal |
| 02 | 2 | Control Switch 1 |
| 03 | 3 | Control Switch 2 |
| 04+ | 4+ | Other controls |

### Common Target Parameters

Target parameter (byte 1) maps to various effect parameters. Examples from active assigns:

| Value | Decimal | Target |
|-------|---------|--------|
| 00 | 0 | General/Master parameter |
| 02 | 2 | Specific effect parameter |
| 09 | 9 | Another effect parameter |

**Note:** The exact mapping requires correlation with the GX-10 manual and effect structure.

---

## Effect Chain Parameters

**Key:** `User_patch%efct`
**Size:** 62 bytes (hex strings)
**Purpose:** Defines the signal routing and effect chain order

### Structure Overview

The effect chain array has two distinct sections:

1. **Bytes 0-11:** Chain configuration metadata
2. **Bytes 12-61:** Effect routing map (maps positions to fxItem indices)

### Complete Example

```javascript
["06","04","00","04","0B","00","00","00","00","01","00","32",  // Bytes 0-11
 "08","0A","03","00","07","09","02","06","01","04","05","0B",  // Bytes 12-23
 "0C","0D","0E","0F","10","11","12","13","14","15","16","17",  // Bytes 24-35
 "18","19","1A","1B","1C","1D","1E","1F","20","21","22","23",  // Bytes 36-47
 "24","25","26","27","28","29","2A","2B","2C","2D","2E","2F",  // Bytes 48-59
 "30","31"]                                                     // Bytes 60-61
```

### Bytes 0-11: Chain Configuration

| Byte | Hex | Dec | Purpose/Hypothesis |
|------|-----|-----|--------------------|
| 0 | 06 | 6 | Number of active effects or routing mode |
| 1 | 04 | 4 | Chain configuration parameter |
| 2 | 00 | 0 | Reserved/unused |
| 3 | 04 | 4 | Chain parameter |
| 4 | 0B | 11 | Chain parameter |
| 5-8 | 00 | 0 | Reserved/unused |
| 9 | 01 | 1 | Enable/mode flag |
| 10 | 00 | 0 | Reserved |
| 11 | 32 | 50 | Maximum chain length or routing parameter |

### Bytes 12-61: Effect Routing Map

This section maps signal chain positions to `fxItem(n)` indices:

**Decoded routing from example:**

| Position | Byte | Hex | Dec | Routes to |
|----------|------|-----|-----|-----------|
| 0 | 12 | 08 | 8 | fxItem(8) |
| 1 | 13 | 0A | 10 | fxItem(10) |
| 2 | 14 | 03 | 3 | fxItem(3) |
| 3 | 15 | 00 | 0 | Empty/bypass |
| 4 | 16 | 07 | 7 | fxItem(7) |
| 5 | 17 | 09 | 9 | fxItem(9) |
| 6 | 18 | 02 | 2 | fxItem(2) |
| 7 | 19 | 06 | 6 | fxItem(6) |
| 8 | 20 | 01 | 1 | fxItem(1) |
| 9 | 21 | 04 | 4 | fxItem(4) |
| 10 | 22 | 05 | 5 | fxItem(5) |
| 11+ | 23+ | 0B-31 | 11-49 | Sequential or default mapping |

### Signal Flow Interpretation

The routing map defines the order in which effects are processed:

```
Input → fxItem(8) → fxItem(10) → fxItem(3) → [bypass] → fxItem(7) →
fxItem(9) → fxItem(2) → fxItem(6) → fxItem(1) → fxItem(4) →
fxItem(5) → ... → Output
```

### Important Notes

1. **Value `00` indicates an empty slot** or bypass in the chain
2. **Values 01-14 (hex)** map directly to fxItem(1) through fxItem(20)
3. **Sequential values 0B-31** (11-49 decimal) appear after the active effects
4. The routing is **order-dependent** - changing the sequence changes the sound
5. Not all fxItems need to be in the chain (some may be inactive)

---

## FxItem Parameters

**Keys:** `User_patch%fxItem(1)` through `User_patch%fxItem(20)`
**Size:** 179 bytes each (hex strings)
**Purpose:** Complete parameter set for individual effects

### Overview

Each fxItem represents one effect unit with all its parameters. The GX-10 supports up to 20 simultaneous effect slots.

### Active vs. Inactive Effects

An effect is **active** if byte 0 (effect type) is non-zero:

**Active Effect Examples:**
- fxItem(1): Type `09` (decimal 9)
- fxItem(2): Type `0E` (decimal 14)
- fxItem(3): Type `3F` (decimal 63)

**Inactive Effect Example:**
- fxItem(19): Type `00` (disabled)

### FxItem Structure

| Byte Range | Size | Purpose |
|------------|------|---------|
| 0 | 1 byte | Effect type ID (00 = off, 01-FF = specific effects) |
| 1 | 1 byte | On/Off state (00 = bypassed, 01 = active) |
| 2-178 | 177 bytes | Effect parameters (varies by effect type) |

### Effect Type IDs

From the analyzed file, active effects include:

| Type (Hex) | Type (Dec) | Slot | Effect Category (Hypothesis) |
|------------|-----------|------|------------------------------|
| 02 | 2 | fxItem(5,12,15) | Compressor or EQ |
| 04 | 4 | fxItem(7) | Distortion/Overdrive |
| 08 | 8 | fxItem(13) | Modulation |
| 09 | 9 | fxItem(1) | Chorus/Flanger |
| 0C | 12 | fxItem(17) | Delay |
| 0E | 14 | fxItem(2) | Reverb/Ambience |
| 10 | 16 | fxItem(14) | Equalizer |
| 15 | 21 | fxItem(4) | Amp simulator |
| 20 | 32 | fxItem(9) | Special effect |
| 24 | 36 | fxItem(10) | Special effect |
| 25 | 37 | fxItem(11) | Special effect |
| 32 | 50 | fxItem(6,18) | Special effect |
| 35 | 53 | fxItem(8) | Special effect |
| 3E | 62 | fxItem(16) | Special effect |
| 3F | 63 | fxItem(3) | Special effect |

### Parameter Encoding

#### Hypothesis: 3-Byte Parameter Structure

Analysis suggests parameters may be encoded in 3-byte groups:

```
[value_msb, value_lsb, parameter_address/type]
```

**Example from fxItem(1):**

| Param | Bytes | Hex Values | Interpretation |
|-------|-------|------------|----------------|
| 0 | 0-2 | 09 01 00 | Type=9, On=1, Reserved=0 |
| 1 | 3-5 | 08 00 02 | Value=2048 (0x0800), Address=2 |
| 2 | 6-8 | 08 08 00 | Value=2056 (0x0808), Address=0 |
| 3 | 9-11 | 03 07 07 | Value=775 (0x0307), Address=7 |
| 4 | 12-14 | 0F 0F 0D | Value=3855 (0x0F0F), Address=13 |

**Note:** This is a working hypothesis. The actual encoding may vary by effect type.

### Detailed Example: fxItem(1)

**Effect Type 9, Active:**

```javascript
["09","01","00",  // Type=9, On=1, Reserved
 "08","00","02",  // Parameter 1
 "08","08","00",  // Parameter 2
 "03","07","07",  // Parameter 3
 "0F","0F","0D",  // Parameter 4
 "08","00","01",  // Parameter 5
 "01","08","00",  // Parameter 6
 "00","00","08",  // Parameter 7
 "00","03","02",  // Parameter 8
 "08","00","03",  // Parameter 9
 "0C","08","00",  // Parameter 10
 ... continues for 179 bytes total]
```

### Common Parameter Patterns

Across multiple fxItems, certain patterns appear frequently:

| Pattern | Hex | Purpose (Hypothesis) |
|---------|-----|----------------------|
| 08 00 XX | - | Default/neutral value with address XX |
| 00 00 00 | - | Disabled or zero parameter |
| 03 02 08 | - | Common parameter combination |
| 06 04 08 | - | Common parameter combination |

### Effect-Specific Parameters

Different effect types use the 177 parameter bytes differently. For example:

**Effect Type 2 (Compressor) might use:**
- Threshold
- Ratio
- Attack
- Release
- Output level

**Effect Type 0E (Reverb) might use:**
- Room size
- Pre-delay
- Decay time
- Tone
- Mix level

**Note:** Without official documentation, exact parameter mappings require systematic testing.

---

## Data Encoding Conventions

### Hexadecimal String Format

**All numeric values are stored as uppercase hexadecimal strings:**

| Value Type | Decimal | Hex String | Notes |
|------------|---------|------------|-------|
| Zero | 0 | `"00"` | Always 2 characters |
| Small value | 8 | `"08"` | Leading zero required |
| Medium value | 50 | `"32"` | Two digits |
| Large value | 255 | `"FF"` | Maximum single-byte value |

### Multi-Byte Values

**16-bit values appear to use MSB-first (big-endian) encoding:**

Example: Value 2056 (0x0808)
```javascript
["08", "08"]  // MSB=08, LSB=08 → 0x0808 = 2056 decimal
```

Example: Value 264 (0x0108)
```javascript
["01", "08"]  // MSB=01, LSB=08 → 0x0108 = 264 decimal
```

### ASCII Text Encoding

**Patch names and text use standard ASCII:**

```javascript
// "Violin_live     " encoded:
["56","69","6f","6c","69","6e","5f","6c","69","76","65","20","20","20","20","20"]
```

**Encoding algorithm:**
```python
def text_to_hex_array(text, length):
    """Convert text to hex array with padding"""
    # Truncate or pad to specified length
    text = text[:length].ljust(length)
    # Convert each character to hex
    return [f"{ord(c):02X}" for c in text]


text_to_hex_array("Rock", 16)
# Returns: ["52","6F","63","6B","20","20","20","20","20","20","20","20","20","20","20","20"]
```

### Boolean and Enumeration Values

**Simple on/off and enumeration:**

| Meaning | Hex | Decimal | Usage |
|---------|-----|---------|-------|
| Off/Disabled | `"00"` | 0 | Effect bypass, assignment disable |
| On/Enabled | `"01"` | 1 | Effect active, assignment enable |
| Option 2 | `"02"` | 2 | Enumeration value |
| Option 3 | `"03"` | 3 | Enumeration value |

### Parameter Value Ranges

**Common parameter ranges observed:**

| Range Type | Min | Max | Example Use |
|------------|-----|-----|-------------|
| Boolean | 00 | 01 | On/off switches |
| Small enum | 00 | 0F | Effect sub-types |
| Large enum | 00 | FF | Effect types |
| Parameter value | 00 | 7F | MIDI-style 0-127 |
| Extended value | 00 | FF | Full 0-255 range |

### Byte Order (Endianness)

**For multi-byte values, MSB comes first (big-endian):**

```javascript
// 16-bit value: 0x0C08 = 3080 decimal
["0C", "08"]
//  ^     ^
// MSB   LSB
```

---

## Practical Examples

### Example 1: Creating a Minimal Valid TSL File

**Minimal patch with default parameters:**

```json
{
  "name": "Minimal",
  "formatRev": "0000",
  "device": "GX-10",
  "data": [[{
    "memo": "",
    "paramSet": {
      "User_patch%common": [/* 129 hex values */],
      "User_patch%led": [/* 28 hex values */],
      "User_patch%assign(1)": [/* 45 hex values */],
      /* ... all required keys ... */
      "User_patch%fxItem(20)": [/* 179 hex values */]
    }
  }]]
}
```

### Example 2: Setting a Patch Name

**Change patch name to "Blues Tone":**

```python
# Original common array bytes 0-15:
original = ["56","69","6f","6c","69","6e","5f","6c","69","76","65","20","20","20","20","20"]
# "Violin_live     "

# New name: "Blues Tone"
new_name = "Blues Tone"
new_bytes = [f"{ord(c):02X}" for c in new_name.ljust(16)]
# Result: ["42","6C","75","65","73","20","54","6F","6E","65","20","20","20","20","20","20"]

# Replace in common array:
param_set["User_patch%common"][0:16] = new_bytes
```

### Example 3: Enabling an Assignment

**Enable Assign(4) - Expression Pedal to Effect Parameter:**

```javascript
// Original (disabled):
"User_patch%assign(4)": ["00","00","00","00","00","00","08","00","00","00",...],

// Modified (enabled):
"User_patch%assign(4)": ["01","05","00","00","00","01","08","00","00","00",...],
//                        ^^  ^^                  ^^
//                     Enable Target            Source
//                             Parameter        Type
```

### Example 4: Activating an Effect

**Enable fxItem(10) as a Delay effect (type 12):**

```javascript
// Original (disabled):
"User_patch%fxItem(10)": ["00","00","00",...],  // Type=0 (off)

// Modified (enabled):
"User_patch%fxItem(10)": ["0C","01","00",...],  // Type=12, On=1
//                          ^^  ^^
//                        Type  Active
```

### Example 5: Building Effect Chain

**Create a chain: Comp → Overdrive → Delay → Reverb:**

Assumptions:
- fxItem(1) = Compressor (type 02)
- fxItem(2) = Overdrive (type 04)
- fxItem(3) = Delay (type 0C)
- fxItem(4) = Reverb (type 0E)

```javascript
// Effect chain routing (bytes 12-19 of efct):
"User_patch%efct": [
  "04","00","00","00","00","00","00","00","00","00","00","00",  // Config bytes
  "01","02","03","04","00","00","00","00",  // Chain: fx1→fx2→fx3→fx4→empty
  // Remaining bytes with default/sequential values
  "05","06","07","08","09","0A","0B","0C","0D","0E","0F","10",...
]
```

### Example 6: Copying a Patch

**Duplicate fxItem(1) to fxItem(20):**

```python
import json

# Load TSL file
with open('patch.tsl.json', 'r') as f:
    data = json.load(f)

# Get paramSet
param_set = data['data'][0][0]['paramSet']

# Copy fxItem(1) to fxItem(20)
param_set['User_patch%fxItem(20)'] = param_set['User_patch%fxItem(1)'].copy()

# Save modified file
with open('patch_modified.tsl.json', 'w') as f:
    json.dump(data, f, indent=2)
```

---

## AI Generation Guidelines

### For Automated TSL File Generation

When building AI systems to generate TSL files automatically, follow these principles:

#### 1. **Template-Based Approach**

Start with a valid template file and modify specific sections:

```python
import json

# Load a working template
with open('template.tsl.json', 'r') as f:
    template = json.load(f)

# Modify patch name
def set_patch_name(param_set, name):
    hex_name = [f"{ord(c):02X}" for c in name[:16].ljust(16)]
    param_set['User_patch%common'][0:16] = hex_name

# Modify effect type
def set_effect_type(param_set, fx_num, effect_type):
    param_set[f'User_patch%fxItem({fx_num})'][0] = f"{effect_type:02X}"
    param_set[f'User_patch%fxItem({fx_num})'][1] = "01"  # Enable
```

#### 2. **Validation Requirements**

Always validate generated files:

```python
def validate_tsl(data):
    """Validate TSL file structure"""
    # Check top-level keys
    required_keys = ['name', 'formatRev', 'device', 'data']
    assert all(k in data for k in required_keys), "Missing top-level keys"

    # Check device
    assert data['device'] == 'GX-10', "Wrong device type"

    # Check paramSet structure
    param_set = data['data'][0][0]['paramSet']

    # Check array sizes
    assert len(param_set['User_patch%common']) == 129, "Wrong common size"
    assert len(param_set['User_patch%led']) == 28, "Wrong LED size"

    for i in range(1, 21):
        assert len(param_set[f'User_patch%assign({i})']) == 45, f"Wrong assign({i}) size"
        assert len(param_set[f'User_patch%fxItem({i})']) == 179, f"Wrong fxItem({i}) size"

    assert len(param_set['User_patch%efct']) == 62, "Wrong efct size"

    # Validate hex format
    for key, array in param_set.items():
        for val in array:
            assert len(val) == 2, f"Value {val} not 2 chars"
            assert all(c in '0123456789ABCDEFabcdef' for c in val), f"Invalid hex: {val}"

    return True
```

#### 3. **Effect Type Database**

Maintain a database of known effect types:

```python
EFFECT_TYPES = {
    'compressor': 0x02,
    'overdrive': 0x04,
    'chorus': 0x09,
    'delay': 0x0C,
    'reverb': 0x0E,
    'equalizer': 0x10,
    'amp_sim': 0x15,
    # Add more as discovered
}

def get_effect_type(effect_name):
    return EFFECT_TYPES.get(effect_name.lower(), 0x00)
```

#### 4. **Parameter Range Management**

Track valid ranges for parameters:


```python
PARAM_RANGES = {
    'enable': (0x00, 0x01),
    'effect_type': (0x00, 0xFF),
    'level': (0x00, 0x7F),
    'time': (0x00, 0xFF),
}

def clamp_value(value, param_type='level'):
    min_val, max_val = PARAM_RANGES.get(param_type, (0, 255))
    return max(min_val, min(max_val, value))
```

#### 5. **Safe Defaults**

Use safe default values for unknown parameters:

```python
def create_default_fxitem():
    """Create a disabled fxItem with safe defaults"""
    return ["00"] * 179  # All zeros = disabled effect

def create_default_assign():
    """Create a disabled assignment"""
    # Use pattern from inactive assigns in example
    return ["00","00","00","00","00","00","08","00","00","00",
            "08","00","00","01","0B","00","14","01","0C","1E",
            "00","00","00","00","00","03","0F","0F","0F","00",
            "00","00","00","00","00","03","0F","0F","0F","00",
            "00","00","00","00","00"]
```

#### 6. **Incremental Testing**

Test changes incrementally:

1. Generate file
2. Load in BOSS Tone Studio
3. Verify it doesn't crash
4. Test sound output
5. Document parameter effects

#### 7. **Semantic Parameter Control**

Build high-level abstractions:

```python
class GX10Patch:
    def __init__(self, template_path):
        with open(template_path, 'r') as f:
            self.data = json.load(f)
        self.param_set = self.data['data'][0][0]['paramSet']

    def set_name(self, name):
        """Set patch name (max 16 chars)"""
        hex_array = [f"{ord(c):02X}" for c in name[:16].ljust(16)]
        self.param_set['User_patch%common'][0:16] = hex_array

    def enable_effect(self, slot, effect_type):
        """Enable an effect in a specific slot"""
        fx_key = f'User_patch%fxItem({slot})'
        self.param_set[fx_key][0] = f"{effect_type:02X}"
        self.param_set[fx_key][1] = "01"

    def disable_effect(self, slot):
        """Disable an effect slot"""
        fx_key = f'User_patch%fxItem({slot})'
        self.param_set[fx_key][0] = "00"
        self.param_set[fx_key][1] = "00"

    def set_effect_chain(self, fx_slots):
        """Set effect chain order

        Args:
            fx_slots: List of fxItem numbers in desired order
                     e.g., [1, 3, 5, 2] for fx1→fx3→fx5→fx2
        """
        efct = self.param_set['User_patch%efct']
        # Update routing section (bytes 12+)
        for i, fx_num in enumerate(fx_slots):
            efct[12 + i] = f"{fx_num:02X}"
        # Fill remaining with sequential or zeros
        for i in range(len(fx_slots), 50):
            efct[12 + i] = f"{i+1:02X}" if i < 20 else "00"

    def save(self, output_path):
        """Save to TSL file"""
        with open(output_path, 'w') as f:
            json.dump(self.data, f, separators=(',', ':'))

# Usage:
patch = GX10Patch('template.tsl.json')
patch.set_name("AI Blues")
patch.enable_effect(1, EFFECT_TYPES['compressor'])
patch.enable_effect(2, EFFECT_TYPES['overdrive'])
patch.enable_effect(3, EFFECT_TYPES['delay'])
patch.set_effect_chain([1, 2, 3])
patch.save('ai_generated_blues.tsl.json')
```

#### 8. **Machine Learning Considerations**

For ML-based generation:

**Feature Extraction:**
- Extract effect types as categorical features
- Extract parameter values as numerical features
- Extract chain order as sequence features

**Training Data:**
- Collect diverse TSL files from users
- Label with musical genres or styles
- Create parameter correlation matrices

**Generation Strategy:**
- Use template + modification approach (more reliable)
- OR train generative model on parameter distributions
- Always post-process with validation

**Example ML Pipeline:**

```python
import numpy as np
from sklearn.preprocessing import LabelEncoder

class TSLFeatureExtractor:
    def __init__(self):
        self.effect_encoder = LabelEncoder()

    def extract_features(self, tsl_path):
        """Extract ML features from TSL file"""
        with open(tsl_path, 'r') as f:
            data = json.load(f)

        param_set = data['data'][0][0]['paramSet']

        features = {
            'patch_name': self.decode_patch_name(param_set),
            'active_effects': [],
            'effect_chain': [],
            'parameters': []
        }

        # Extract active effects
        for i in range(1, 21):
            fx = param_set[f'User_patch%fxItem({i})']
            effect_type = int(fx[0], 16)
            if effect_type > 0:
                features['active_effects'].append(effect_type)
                # Extract parameters (simplified)
                params = [int(fx[j], 16) for j in range(2, min(50, len(fx)))]
                features['parameters'].extend(params)

        # Extract chain order
        efct = param_set['User_patch%efct']
        chain = [int(efct[i], 16) for i in range(12, 20)]
        features['effect_chain'] = [x for x in chain if x > 0]

        return features

    def decode_patch_name(self, param_set):
        """Decode patch name from hex"""
        common = param_set['User_patch%common']
        return ''.join([chr(int(x, 16)) for x in common[:16]]).strip()

# Train a simple model
def train_genre_classifier(tsl_files, labels):
    """Train a classifier to predict genre from TSL features"""
    extractor = TSLFeatureExtractor()

    X = []
    y = labels

    for tsl_file in tsl_files:
        features = extractor.extract_features(tsl_file)
        # Create feature vector
        feature_vec = features['active_effects'] + [0] * (20 - len(features['active_effects']))
        X.append(feature_vec)

    X = np.array(X)

    # Train classifier (e.g., RandomForest)
    from sklearn.ensemble import RandomForestClassifier
    clf = RandomForestClassifier()
    clf.fit(X, y)

    return clf, extractor
```


#### 9. **Error Handling**

Implement robust error handling:

```python
class TSLGenerationError(Exception):
    pass

def safe_generate_tsl(params):
    """Generate TSL with error handling"""
    try:
        # Create patch
        patch = GX10Patch('template.tsl.json')

        # Apply parameters with validation
        if 'name' in params:
            if len(params['name']) > 16:
                raise TSLGenerationError("Patch name too long")
            patch.set_name(params['name'])

        if 'effects' in params:
            for slot, effect_type in params['effects']:
                if not 1 <= slot <= 20:
                    raise TSLGenerationError(f"Invalid slot: {slot}")
                if not 0 <= effect_type <= 255:
                    raise TSLGenerationError(f"Invalid effect type: {effect_type}")
                patch.enable_effect(slot, effect_type)

        # Validate before saving
        if not validate_tsl(patch.data):
            raise TSLGenerationError("Generated TSL failed validation")

        return patch

    except Exception as e:
        print(f"Generation failed: {e}")
        # Return safe default
        return GX10Patch('template.tsl.json')
```

#### 10. **Documentation and Logging**

Log all generation actions:

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger('TSL_Generator')

def generate_with_logging(params):
    """Generate TSL with detailed logging"""
    logger.info(f"Starting TSL generation with params: {params}")

    patch = GX10Patch('template.tsl.json')

    if 'name' in params:
        logger.info(f"Setting patch name: {params['name']}")
        patch.set_name(params['name'])

    if 'effects' in params:
        for slot, effect_type in params['effects']:
            logger.info(f"Enabling effect: slot={slot}, type={effect_type}")
            patch.enable_effect(slot, effect_type)

    logger.info("TSL generation complete")
    return patch
```

---

## Appendix A: Complete Structure Reference

### JSON Schema (Informal)

```javascript
{
  "name": String,           // Liveset name
  "formatRev": String,      // "0000"
  "device": String,         // "GX-10"
  "data": [                 // Array of livesets
    [                       // Array of patches in liveset
      {
        "memo": String,     // Patch memo
        "paramSet": {
          "User_patch%common": [String × 129],
          "User_patch%led": [String × 28],
          "User_patch%assign(1)": [String × 45],
          "User_patch%assign(2)": [String × 45],
          ...
          "User_patch%assign(20)": [String × 45],
          "User_patch%efct": [String × 62],
          "User_patch%fxItem(1)": [String × 179],
          "User_patch%fxItem(2)": [String × 179],
          ...
          "User_patch%fxItem(20)": [String × 179]
        }
      }
    ]
  ]
}
```

### Size Summary Table

| Component | Count | Size Each | Total Size |
|-----------|-------|-----------|------------|
| common | 1 | 129 bytes | 129 bytes |
| led | 1 | 28 bytes | 28 bytes |
| assign | 20 | 45 bytes | 900 bytes |
| efct | 1 | 62 bytes | 62 bytes |
| fxItem | 20 | 179 bytes | 3,580 bytes |
| **Total per patch** | **43 arrays** | - | **4,699 bytes** |

### Hex Value Quick Reference

| Dec | Hex | Dec | Hex | Dec | Hex | Dec | Hex |
|-----|-----|-----|-----|-----|-----|-----|-----|
| 0 | 00 | 16 | 10 | 32 | 20 | 48 | 30 |
| 1 | 01 | 17 | 11 | 33 | 21 | 49 | 31 |
| 2 | 02 | 18 | 12 | 34 | 22 | 50 | 32 |
| 3 | 03 | 19 | 13 | 35 | 23 | 63 | 3F |
| 4 | 04 | 20 | 14 | 36 | 24 | 127 | 7F |
| 8 | 08 | 21 | 15 | 37 | 25 | 255 | FF |
| 12 | 0C | 24 | 18 | 40 | 28 | | |
| 15 | 0F | 31 | 1F | 47 | 2F | | |

---

## Appendix B: Research Notes

### Remaining Unknowns

1. **Exact effect type mappings** - Requires testing each type systematically
2. **Parameter address/function mappings** - Varies by effect type
3. **Common parameter bytes 16-128** - Many unknown functions
4. **LED parameter meanings** - Need hardware correlation
5. **Assign parameter details** - Full control assignment schema
6. **3-byte parameter encoding** - Exact interpretation needs validation

### Recommended Testing Methodology

1. **Systematic Effect Testing:**
   - Create patch in Tone Studio
   - Set effect type
   - Export TSL
   - Increment effect type
   - Repeat
   - Build effect type database

2. **Parameter Mapping:**
   - Set single parameter to known value
   - Export TSL
   - Compare hex differences
   - Document parameter location and encoding
   - Repeat for all parameters

3. **Chain Routing Validation:**
   - Create various effect orders
   - Export and analyze efct array
   - Confirm routing hypothesis

4. **Assignment Testing:**
   - Configure expression pedal assignments
   - Test different targets
   - Document assign array changes

---

## Appendix C: Code Examples

### Complete Python Utility Library

```python
"""
GX-10 TSL File Utilities
Complete library for reading, modifying, and generating TSL files
"""

import json
from typing import List, Dict, Any
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger('GX10_TSL')

class TSLFile:
    """Complete GX-10 TSL file handler"""

    def __init__(self, path: str = None):
        if path:
            self.load(path)
        else:
            self.data = self._create_empty_tsl()

    def _create_empty_tsl(self) -> Dict:
        """Create empty TSL structure"""
        return {
            "name": "New Liveset",
            "formatRev": "0000",
            "device": "GX-10",
            "data": [[{
                "memo": "",
                "paramSet": self._create_empty_paramset()
            }]]
        }

    def _create_empty_paramset(self) -> Dict:
        """Create empty paramSet with correct array sizes"""
        paramset = {
            "User_patch%common": ["00"] * 129,
            "User_patch%led": ["00"] * 28,
            "User_patch%efct": ["00"] * 62
        }

        # Add assigns
        for i in range(1, 21):
            paramset[f"User_patch%assign({i})"] = ["00"] * 45

        # Add fxItems
        for i in range(1, 21):
            paramset[f"User_patch%fxItem({i})"] = ["00"] * 179

        return paramset

    def load(self, path: str):
        """Load TSL file"""
        with open(path, 'r') as f:
            self.data = json.load(f)
        logger.info(f"Loaded TSL file: {path}")

    def save(self, path: str, pretty: bool = False):
        """Save TSL file"""
        with open(path, 'w') as f:
            if pretty:
                json.dump(self.data, f, indent=2)
            else:
                json.dump(self.data, f, separators=(',', ':'))
        logger.info(f"Saved TSL file: {path}")

    @property
    def paramset(self) -> Dict:
        """Get paramSet for first patch"""
        return self.data['data'][0][0]['paramSet']

    # Patch Name Operations
    def get_patch_name(self) -> str:
        """Get patch name"""
        common = self.paramset['User_patch%common']
        hex_name = common[0:16]
        return ''.join([chr(int(h, 16)) for h in hex_name]).strip()

    def set_patch_name(self, name: str):
        """Set patch name (max 16 chars)"""
        name = name[:16].ljust(16)
        hex_array = [f"{ord(c):02X}" for c in name]
        self.paramset['User_patch%common'][0:16] = hex_array
        logger.info(f"Set patch name: {name.strip()}")

    # Effect Operations
    def get_effect_type(self, slot: int) -> int:
        """Get effect type for slot"""
        if not 1 <= slot <= 20:
            raise ValueError(f"Slot must be 1-20, got {slot}")
        fx = self.paramset[f'User_patch%fxItem({slot})']
        return int(fx[0], 16)

    def is_effect_active(self, slot: int) -> bool:
        """Check if effect is active"""
        return self.get_effect_type(slot) > 0

    def enable_effect(self, slot: int, effect_type: int):
        """Enable effect in slot"""
        if not 1 <= slot <= 20:
            raise ValueError(f"Slot must be 1-20, got {slot}")
        if not 0 <= effect_type <= 255:
            raise ValueError(f"Effect type must be 0-255, got {effect_type}")

        fx_key = f'User_patch%fxItem({slot})'
        self.paramset[fx_key][0] = f"{effect_type:02X}"
        self.paramset[fx_key][1] = "01"
        logger.info(f"Enabled effect: slot={slot}, type={effect_type}")

    def disable_effect(self, slot: int):
        """Disable effect in slot"""
        if not 1 <= slot <= 20:
            raise ValueError(f"Slot must be 1-20, got {slot}")

        fx_key = f'User_patch%fxItem({slot})'
        self.paramset[fx_key][0] = "00"
        self.paramset[fx_key][1] = "00"
        logger.info(f"Disabled effect: slot={slot}")

    def get_active_effects(self) -> List[tuple]:
        """Get list of active effects [(slot, type), ...]"""
        active = []
        for slot in range(1, 21):
            effect_type = self.get_effect_type(slot)
            if effect_type > 0:
                active.append((slot, effect_type))
        return active

    # Chain Operations
    def get_effect_chain(self) -> List[int]:
        """Get effect chain order"""
        efct = self.paramset['User_patch%efct']
        chain = []
        for i in range(12, 62):
            slot = int(efct[i], 16)
            if slot > 0 and slot <= 20:
                chain.append(slot)
            elif slot == 0:
                break
        return chain

    def set_effect_chain(self, slots: List[int]):
        """Set effect chain order"""
        efct = self.paramset['User_patch%efct']

        # Validate slots
        for slot in slots:
            if not 1 <= slot <= 20:
                raise ValueError(f"Invalid slot in chain: {slot}")

        # Update chain routing
        for i, slot in enumerate(slots):
            efct[12 + i] = f"{slot:02X}"

        # Fill remaining with sequential or default
        for i in range(len(slots), 50):
            efct[12 + i] = f"{i+1:02X}" if i < 20 else "00"

        logger.info(f"Set effect chain: {slots}")

    # Assignment Operations
    def is_assign_active(self, assign_num: int) -> bool:
        """Check if assignment is active"""
        if not 1 <= assign_num <= 20:
            raise ValueError(f"Assign must be 1-20, got {assign_num}")
        assign = self.paramset[f'User_patch%assign({assign_num})']
        return int(assign[0], 16) != 0

    def enable_assign(self, assign_num: int, target: int, source: int):
        """Enable assignment"""
        if not 1 <= assign_num <= 20:
            raise ValueError(f"Assign must be 1-20, got {assign_num}")

        assign_key = f'User_patch%assign({assign_num})'
        self.paramset[assign_key][0] = "01"  # Enable
        self.paramset[assign_key][1] = f"{target:02X}"  # Target
        self.paramset[assign_key][5] = f"{source:02X}"  # Source
        logger.info(f"Enabled assign {assign_num}: target={target}, source={source}")

    def disable_assign(self, assign_num: int):
        """Disable assignment"""
        if not 1 <= assign_num <= 20:
            raise ValueError(f"Assign must be 1-20, got {assign_num}")

        assign_key = f'User_patch%assign({assign_num})'
        self.paramset[assign_key][0] = "00"
        logger.info(f"Disabled assign {assign_num}")

    # Validation
    def validate(self) -> bool:
        """Validate TSL structure"""
        try:
            # Check top-level
            assert all(k in self.data for k in ['name', 'formatRev', 'device', 'data'])
            assert self.data['device'] == 'GX-10'
            assert self.data['formatRev'] == '0000'

            # Check paramSet
            ps = self.paramset
            assert len(ps['User_patch%common']) == 129
            assert len(ps['User_patch%led']) == 28
            assert len(ps['User_patch%efct']) == 62

            for i in range(1, 21):
                assert len(ps[f'User_patch%assign({i})']) == 45
                assert len(ps[f'User_patch%fxItem({i})']) == 179

            # Check hex format
            for key, array in ps.items():
                for val in array:
                    assert len(val) == 2, f"Invalid length: {val}"
                    assert all(c in '0123456789ABCDEFabcdef' for c in val)

            logger.info("Validation passed")
            return True

        except AssertionError as e:
            logger.error(f"Validation failed: {e}")
            return False

    # Utility Methods
    def clone_effect(self, from_slot: int, to_slot: int):
        """Clone effect from one slot to another"""
        if not (1 <= from_slot <= 20 and 1 <= to_slot <= 20):
            raise ValueError("Slots must be 1-20")

        from_key = f'User_patch%fxItem({from_slot})'
        to_key = f'User_patch%fxItem({to_slot})'
        self.paramset[to_key] = self.paramset[from_key].copy()
        logger.info(f"Cloned effect: {from_slot} -> {to_slot}")

    def clear_all_effects(self):
        """Clear all effects"""
        for i in range(1, 21):
            self.disable_effect(i)
        logger.info("Cleared all effects")

    def summary(self) -> str:
        """Get patch summary"""
        lines = []
        lines.append(f"Patch: {self.get_patch_name()}")
        lines.append(f"Active Effects: {len(self.get_active_effects())}")
        lines.append(f"Effect Chain: {self.get_effect_chain()}")

        active_assigns = sum(1 for i in range(1, 21) if self.is_assign_active(i))
        lines.append(f"Active Assigns: {active_assigns}")

        return '\n'.join(lines)


# Example Usage
if __name__ == '__main__':
    # Create new patch
    tsl = TSLFile()
    tsl.set_patch_name("AI Rock")
    tsl.enable_effect(1, 0x02)  # Compressor
    tsl.enable_effect(2, 0x04)  # Overdrive
    tsl.enable_effect(3, 0x0C)  # Delay
    tsl.set_effect_chain([1, 2, 3])

    # Validate and save
    if tsl.validate():
        tsl.save('ai_rock_patch.tsl.json', pretty=True)
        print(tsl.summary())
```

---

## Document Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-16 | Initial version based on web research |
| 2.0 | 2026-01-17 | Complete rewrite based on actual GX-10.tsl.json analysis |

---

## References

**Primary Source:**
- Actual GX-10.tsl.json file analysis (2026-01-17)

**Community Research:**
- [VGuitarForums - BOSS TONE STUDIO .tsl file format](https://www.vguitarforums.com/smf/index.php?topic=10862.0)
- [VGuitarForums - TSL schema reference](https://www.vguitarforums.com/smf/index.php?topic=29800.0)
- Various Reddit discussions on BOSS patch formats

**Official Resources:**
- BOSS Tone Studio software (observation and testing)
- GX-10 User Manual (for parameter correlation)

---

**END OF SPECIFICATION**
