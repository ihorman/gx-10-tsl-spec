# 🎸 BOSS GX-10 TSL File Format Specification

[![Version](https://img.shields.io/badge/version-2.0-blue.svg)](https://github.com/ihorman/gx-10-tsl-spec)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Format](https://img.shields.io/badge/format-JSON-orange.svg)]()
[![Platform](https://img.shields.io/badge/platform-BOSS%20GX--10-black.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()

**Reverse-engineered, byte-level specification of the proprietary TSL (Tone Studio Liveset) file format used by the BOSS GX-10 guitar multi-effect processor.**

> No official documentation exists — this is the only publicly available reference for the GX-10 TSL format.

---

## 📖 What is this?

The BOSS GX-10 uses `.tsl.json` files to store guitar effect patches (presets) via the BOSS Tone Studio software. This project provides:

- **Complete JSON structure mapping** with exact byte-level layouts
- **Hex encoding conventions** for all parameter values
- **Effect chain routing logic** — how 20 effect slots are ordered in the signal path
- **Control assignment mappings** — expression pedal and footswitch → parameter routing
- **Working Python code examples** for reading, modifying, generating, and validating TSL files
- **AI Generation Guidelines** — specifically designed for LLMs and automated tools to programmatically generate valid patches

---

## 🗂 Repository Structure

| File | Description |
|------|-------------|
| [`specification.md`](specification.md) | The full 1,500+ line specification (v2.0) |
| [`README.md`](README.md) | This file |

---

## 🔑 Key Findings

| Finding | Detail |
|---------|--------|
| All values are hex-encoded strings | `"08"`, `"0F"`, `"1E"` |
| Fixed array sizes | Each param category has a strict byte count |
| Patch name = ASCII in bytes 0–15 | First 16 bytes of common parameters |
| Effect chain routing is separate | From effect parameters |
| 3-byte parameter encoding | In fxItem blocks (working hypothesis) |

---

## 🐍 Python Examples (from spec)

The specification includes **6 practical examples** and a complete **`TSLFile` Python class** (~250 lines) in Appendix C:

```python
# Load and inspect a TSL file
import json

with open("my_patch.tsl.json", "r") as f:
    tsl = json.load(f)

print(f"Name: {tsl['name']}")
print(f"Device: {tsl['device']}")
print(f"Patches: {len(tsl['data'])}")
```

**More examples in the spec:**
- Setting patch names
- Enabling/disabling effects
- Building effect chains
- Copying patches between slots
- Validating TSL file integrity

---

## 🤖 AI / LLM Integration

This spec is explicitly written with AI generation in mind:

- **Template-based patch generation** — start from a known-good template, modify parameters
- **Validation pipeline** — verify generated files before loading
- **ML feature extraction** — scikit-learn pipeline sketch for learning from existing patches
- **`GX10Patch` helper class** — programmatic access to all parameters

---

## 📊 Specification Coverage

```
├── Top-level JSON structure (name, formatRev, device, data)
├── ParamSet structure — 43 keys per patch (4,699 bytes)
│   ├── Common parameters (129 bytes) — patch name, global config
│   ├── LED parameters (28 bytes) — display/color settings
│   ├── Assign parameters (45 bytes × 20) — control routing
│   ├── Effect chain (62 bytes) — routing map
│   └── FxItem parameters (179 bytes × 20) — per-effect params
├── Complete Python library (TSLFile class)
└── Research notes & known unknowns (Appendix B)
```

---

## 🎯 Who is this for?

- **🎸 Guitarists** — programmatically create, modify, or batch-process GX-10 patches
- **👩‍💻 Developers** — build patch editors, sharing platforms, preset managers
- **🤖 AI/ML engineers** — generate valid TSL files with LLMs or ML models
- **🔧 Reverse engineers** — reference for hardware protocol documentation

---

## 🚀 Getting Started

1. **Read the spec:** [`specification.md`](specification.md)
2. **Export a TSL file** from BOSS Tone Studio
3. **Use the Python examples** from the spec to load and modify it
4. **Load it back** into your GX-10

---

## 🤝 Contributing

Contributions welcome! Especially:

- 🔍 **Validation** of byte mappings against real GX-10 hardware
- 📝 **Documentation** improvements
- 🐛 **Bug reports** for incorrect parameter mappings
- 💡 **New examples** — share your use cases
- 🧪 **Testing** the 3-byte encoding hypothesis

See [Appendix B](specification.md#appendix-b-research-notes-and-next-steps) for known unknowns and recommended testing methodology.

---

## 📚 Related Projects & Resources

- [BOSS Tone Studio](https://www.boss.info/global/promos/boss_tone_studio/) — Official editor
- [VGuitarForums](https://vguitarforums.com/) — Active GX-10 community
- [r/GuitarPedals](https://reddit.com/r/GuitarPedals) — Reddit pedal community

---

## 📄 License

MIT — see [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built with ❤️ by reverse engineering • Not affiliated with BOSS/Roland</sub>
</p>
