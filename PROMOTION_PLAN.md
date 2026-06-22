# План популяризации gx-10-tsl-spec

## 1. GitHub SEO (сделано ✅)

### Описание репозитория (Settings → About)
```
Reverse-engineered byte-level specification of the BOSS GX-10 TSL (Tone Studio Liveset) file format. Includes Python code examples and AI generation guidelines.
```

### Topics (Settings → Topics)
Добавить эти теги в настройках репозитория:
```
boss gx-10 tsl tone-studio guitar-pedal multi-effect reverse-engineering
file-format specification guitar-effects boss-tone-studio preset
tone-patch python hex-encoding ai-generation hardware-hacking
```

### Пины (About → pin)
Закрепить specification.md в About секции.

---

## 2. Reddit — Посты

### r/ReverseEngineering
**Title:** `Reverse-engineered the BOSS GX-10 TSL file format — full byte-level spec with Python examples`

**Body:**
```
I reverse-engineered the proprietary TSL (Tone Studio Liveset) file format 
used by the BOSS GX-10 guitar multi-effect processor.

BOSS doesn't publish any documentation for this format, so I analyzed an 
actual .tsl.json file and documented everything — from the top-level JSON 
structure down to individual byte positions for each parameter.

Key findings:
- All values are hex-encoded strings
- Fixed-size parameter arrays (4,699 bytes per patch)
- Patch names are ASCII in the first 16 bytes of common params
- Effect chain routing is separate from effect parameters
- 20 effect slots with 179-byte parameter blocks each

The spec includes:
- Complete byte-level mapping of every parameter category
- 6 practical Python examples
- A full TSLFile Python class (~250 lines) in Appendix C
- AI generation guidelines for programmatic patch creation
- Research notes on remaining unknowns

Looking for contributors to help validate byte mappings against real 
hardware — especially the 3-byte encoding hypothesis in fxItem blocks.

Spec: https://github.com/ihorman/gx-10-tsl-spec/blob/main/specification.md
Repo: https://github.com/ihorman/gx-10-tsl-spec
```

---

### r/GuitarPedals
**Title:** `I reverse-engineered the BOSS GX-10 patch file format — here's the full spec`

**Body:**
```
Hey everyone!

If you've ever wanted to programmatically create, modify, or batch-process 
patches for the BOSS GX-10 outside of BOSS Tone Studio, I've got something 
for you.

I spent time reverse-engineering the TSL (Tone Studio Liveset) file format 
that the GX-10 uses and documented the entire thing — every byte position, 
every parameter, every encoding convention.

What you can do with this:
- Create patches programmatically (Python examples included)
- Build your own patch editor
- Batch-rename or modify hundreds of patches at once
- Back up and restore patches without Tone Studio
- Use AI to generate new patches (guidelines included)

The spec is 1,500+ lines covering:
- Complete JSON → byte mapping
- Effect chain routing logic
- Control assignment (pedal → parameter) mappings
- Working Python code you can copy-paste

No official documentation exists for this format — this is the only public 
reference.

Spec: https://github.com/ihorman/gx-10-tsl-spec/blob/main/specification.md

Would love to hear from other GX-10 owners! Especially interested in 
testing the byte mappings against real hardware.
```

---

### r/guitar
**Title:** `BOSS GX-10: I reverse-engineered the patch file format — full specification with Python examples`

**Body:**
```
For any GX-10 owners out there who want more control over their patches:

I reverse-engineered the TSL file format that BOSS Tone Studio uses and 
documented it completely. The spec covers every byte position, encoding 
convention, and parameter mapping.

Practical things you can do:
- Create/edit patches without Tone Studio
- Batch-process hundreds of patches
- Build custom preset managers
- Generate patches with AI/ML

The repo includes working Python examples and a complete TSLFile class 
you can use right away.

Would be great to get feedback from other GX-10 users — especially if 
anyone wants to help validate the spec against real hardware.

https://github.com/ihorman/gx-10-tsl-spec
```

---

### r/Python
**Title:** `Built a Python library for reading/writing BOSS GX-10 guitar effect patches (reverse-engineered format)`

**Body:**
```
I reverse-engineered the proprietary TSL file format used by the BOSS GX-10 
guitar multi-effect processor and documented the complete specification.

The repo includes:
- A full TSLFile Python class (~250 lines) in Appendix C
- 6 practical examples: load, modify, validate, save
- Hex encoding/decoding utilities
- Effect chain routing logic
- AI generation guidelines with scikit-learn pipeline sketch

All parameter values are hex-encoded strings in JSON. The spec maps every 
byte position for 20 effect slots, control assignments, and patch metadata.

Looking for contributors to help:
- Extract the embedded code into proper .py modules
- Add type hints and tests
- Validate against real hardware

Spec: https://github.com/ihorman/gx-10-tsl-spec/blob/main/specification.md
```

---

## 3. Hacker News — Show HN

**Title:** `Show HN: Reverse-engineered the BOSS GX-10 guitar effect file format`

**URL:** `https://github.com/ihorman/gx-10-tsl-spec`

**First comment (self-text):**
```
I reverse-engineered the proprietary TSL (Tone Studio Liveset) file format 
used by the BOSS GX-10 guitar multi-effect processor.

BOSS publishes zero documentation for this format. I analyzed an actual 
.tsl.json export and documented the complete byte-level structure — 1,500+ 
lines covering:

- JSON → hex-encoded parameter mapping (4,699 bytes per patch)
- 20 effect slots with 179-byte parameter blocks
- Effect chain routing (separate from parameters)
- Control assignment mappings (expression pedal → effect param)
- ASCII patch names in bytes 0–15

The spec includes a complete TSLFile Python class and 6 working examples. 
I also added "AI Generation Guidelines" — specifically written so LLMs and 
automated tools can generate valid patches programmatically.

Looking for:
1. GX-10 owners to validate byte mappings against hardware
2. Contributors to extract embedded Python into proper modules
3. Anyone who's reverse-engineered other BOSS/Roland formats

Key insight: the 3-byte parameter encoding in fxItem blocks is still a 
working hypothesis — documented in Appendix B with testing methodology.
```

---

## 4. X/Twitter (тред)

**Tweet 1:**
```
🎸 I reverse-engineered the BOSS GX-10 patch file format.

No docs exist. I analyzed the actual .tsl.json and documented every byte.

1,500+ lines: JSON structure → hex params → effect chains → control routing

Includes Python code + AI generation guidelines.

🧵👇

https://github.com/ihorman/gx-10-tsl-spec
```

**Tweet 2:**
```
Key findings:

• All params are hex-encoded strings
• 4,699 bytes per patch (fixed-size arrays)
• 20 effect slots × 179 bytes each
• Patch name = ASCII in bytes 0–15
• Effect chain routing is separate from effect params
• 3-byte encoding in fxItems (hypothesis — needs validation)
```

**Tweet 3:**
```
What you can build with this:

→ Custom patch editors (without Tone Studio)
→ Batch-rename/modify hundreds of presets
→ AI-powered patch generation
→ Backup/restore tools
→ Cross-device patch converters

Python class included (~250 lines, Appendix C).
```

**Tweet 4:**
```
Looking for:

1. GX-10 owners to validate byte mappings
2. Contributors for .py extraction + tests
3. Anyone who's RE'd other BOSS/Roland formats

PRs welcome. Issues welcome. DMs open.

https://github.com/ihorman/gx-10-tsl-spec
```

---

## 5. VGuitarForums

**Title:** `BOSS GX-10 TSL Format — Full Reverse-Engineered Specification`

**Body:**
```
Hey everyone,

I've been working on reverse-engineering the TSL (Tone Studio Liveset) 
file format used by the GX-10 and I've published the complete specification.

The spec documents every byte position, encoding convention, and parameter 
mapping — including things like:
- How 20 effect slots are encoded (179 bytes each)
- Effect chain routing logic (separate from effect parameters)
- Control assignment mappings (pedal → parameter)
- Patch name encoding (ASCII in first 16 bytes)

I've included Python code examples and a complete TSLFile class for 
programmatic patch manipulation.

The spec is here: https://github.com/ihorman/gx-10-tsl-spec/blob/main/specification.md

I'd especially appreciate feedback from anyone who can validate the byte 
mappings against their own GX-10. The 3-byte parameter encoding in fxItem 
blocks is still a working hypothesis (see Appendix B).

Would love to hear what other GX-10 owners think!
```

---

## 6. GitHub Awesome Lists

Отправить PR в эти репозитории:

| Awesome List | Ссылка | Что добавить |
|--------------|--------|---------------|
| awesome-guitar | github.com/search?q=awesome+guitar | Entry in "Tools" section |
| awesome-reverse-engineering | github.com/alphaSeclab/awesome-reverse-engineering | Entry in "Hardware" section |
| awesome-audio | github.com/search?q=awesome+audio | Entry in "Music Hardware" |

---

## 7. Дополнительные площадки

### DEV.to / Medium
Статья: "How I Reverse-Engineered a Guitar Pedal's File Format"
- Подробный туториал с скриншотами
- Процесс реверс-инжиниринга
- Код и примеры

### Stack Overflow
Ответить на существующие вопросы о BOSS Tone Studio / GX-10 со ссылкой на spec.

### Product Hunt
Если будет инструмент (web app / CLI), залить на Product Hunt.

---

## Статус

- [x] README переписан
- [x] LICENSE добавлен
- [ ] GitHub Topics настроены (нужен доступ к Settings)
- [ ] Reddit посты опубликованы
- [ ] Hacker News Show HN
- [ ] X/Twitter тред
- [ ] VGuitarForums пост
- [ ] Awesome Lists PR
- [ ] DEV.to статья
