# Clawfy API — Style Systems

> Style Systems let you define brand identity to guide AI image generation in workflows.
> Base URL: `$CLAWFY_BASE_URL` (default: `http://localhost:3001`)
> All requests require: `x-api-key: $CLAWFY_API_KEY`

---

## Overview

A **Style System** is a brand identity template that captures:

| Field | Purpose |
|-------|---------|
| `name` | Display name for the style system |
| `brandName` | Brand name |
| `industry` | Industry (beauty, fashion, food, tech, wellness, etc.) |
| `mood` | Mood driver: minimal, bold, playful, luxurious, natural, etc. |
| `stylePreset` | Preset category: clean_beauty, bold_fashion, minimal_wellness, etc. |
| `colors` | 5-color palette (primary, secondary, accent, background, text) |
| `typography` | Font choices (headline + body) |
| `logo` | Logo URL + placement |
| `layoutPreference` | How content is laid out (product_hero, flatlay, etc.) |
| `customInstructions` | Text guidance for the AI generator |
| `examples` | 5 reference image URLs that exemplify the style |

---

## Quick Reference

### GET /api/style-systems

List all style systems for the authenticated user.

**Response:**
```json
{
  "styleSystems": [
    {
      "id": "ss_abc123",
      "name": "Minimal Wellness Brand",
      "brandName": "Wellness Co",
      "industry": "wellness",
      "mood": "minimal",
      "stylePreset": "minimal_wellness",
      "colors": {
        "primary": "#F5F0E8",
        "secondary": "#E8E0D5",
        "accent": "#C4B896",
        "background": "#FDFBF8",
        "text": "#3D3D3D"
      },
      "examples": ["https://..."]
    }
  ]
}
```

### POST /api/style-systems

Create a new style system.

**Request Body:**
```json
{
  "name": "Minimal Wellness Brand",
  "brandName": "Wellness Co",
  "industry": "wellness",
  "mood": "minimal",
  "stylePreset": "minimal_wellness",
  "colors": {
    "primary": "#F5F0E8",
    "secondary": "#E8E0D5",
    "accent": "#C4B896",
    "background": "#FDFBF8",
    "text": "#3D3D3D"
  },
  "typography": {
    "headlineFont": "Playfair Display",
    "headlineWeight": "Bold",
    "bodyFont": "Lato",
    "bodyWeight": "Regular"
  },
  "logo": {
    "url": "https://brand.com/logo.png",
    "position": "bottom_right",
    "maxWidth": "100px"
  },
  "layoutPreference": "product_hero",
  "customInstructions": "Soft natural light, natural textures, minimal composition",
  "examples": [
    "https://images.pexels.com/photos/4889036/pexels-photo-4889036.jpeg",
    "https://images.pexels.com/photos/7795661/pexels-photo-7795661.jpeg",
    "https://images.pexels.com/photos/27357181/pexels-photo-27357181.jpeg",
    "https://images.pexels.com/photos/33362027/pexels-photo-33362027.jpeg",
    "https://images.pexels.com/photos/6167444/pexels-photo-6167444.jpeg"
  ]
}
```

### PUT /api/style-systems/:id

Update a style system (partial update supported).

### DELETE /api/style-systems/:id

Delete a style system.

---

## Reasoning Example — Understanding the Decision Chain

> ⚠️ **This is an EXAMPLE to illustrate reasoning — DO NOT copy values directly.**
> Values are context-dependent. Change `industry` OR `mood` and the other fields should change accordingly.

```json
{
  // ══════════════════════════════════════════════════════════════
  // REASONING EXAMPLE — understand WHY, don't copy values
  // ══════════════════════════════════════════════════════════════
  
  // INPUT: industry + brand personality
  // Chain: industry → mood → colors → typography → layout → examples
  
  "industry": "wellness",        // ← Industry sets the DOMAIN
  "mood": "minimal",             // ← Mood is the FILTER — all choices derive from this

  // ────────────────────────────────────────────────────────────
  // COLORS: derived from mood
  // WHY warm neutrals?
  //   - wellness = approachable, nurturing → warm tones
  //   - minimal = not loud, not clinical → neutrals, not saturated
  // WHY NOT blue? too clinical/healthcare associations
  // WHY NOT red? too energetic/stimulating, conflicts with calm mood
  // ══════════════════════════════════════════════════════════════
  "colors": {
    "primary": "#F5F0E8",      // warm cream — nurturing, soft
    "secondary": "#E8E0D5",    // light taupe — approachable
    "accent": "#C4B896",       // sage green — natural, wellness
    "background": "#FDFBF8",   // off-white — clean but warm
    "text": "#3D3D3D"          // soft black — readable, not harsh
  },

  // ────────────────────────────────────────────────────────────
  // TYPOGRAPHY: derived from mood + industry
  // WHY serif headline?
  //   - wellness = trust, established, premium feel
  //   - serif communicates established credibility
  // WHY sans body?
  //   - modern readability, approachable
  // If mood=bold → display font + condensed might fit instead
  // ══════════════════════════════════════════════════════════════
  "typography": {
    "headlineFont": "Playfair Display",  // serif — trust, elegance
    "headlineWeight": "Bold",
    "bodyFont": "Lato",                  // sans — modern, readable
    "bodyWeight": "Regular"
  },

  // ────────────────────────────────────────────────────────────
  // LAYOUT: derived from mood
  // minimal → airy, product as hero, lots of whitespace
  // ══════════════════════════════════════════════════════════════
  "layoutPreference": "product_hero",

  // ────────────────────────────────────────────────────────────
  // CUSTOM INSTRUCTIONS: derived from mood — guides AI generation
  // "Soft natural light" → because minimal = soft, not harsh flash
  // "natural textures" → wellness = authentic, not synthetic
  // "minimal composition" → mood-driven, not busy
  // WHY specific? "beautiful" = too vague, no guidance for AI
  // ══════════════════════════════════════════════════════════════
  "customInstructions": "Soft natural light, natural textures, minimal composition, warm earth tones, avoid harsh filters",

  // ────────────────────────────────────────────────────────────
  // EXAMPLES: 5 images that all share the mood
  // WHY variety?
  //   - each image adds a different dimension:
  //     1. texture close-up → real product texture, not generic
  //     2. natural context → eco/natural connection
  //     3. soft lighting → shadows and mood direction
  //     4. elegant display → premium positioning
  //     5. warm palette → color temperature check
  // WHY NOT generic stock? Examples should TRAIN the AI on the style
  // ══════════════════════════════════════════════════════════════
  "examples": [
    "https://images.pexels.com/photos/4889036/...",  // texture, eco context
    "https://images.pexels.com/photos/7795661/...",  // natural, organic
    "https://images.pexels.com/photos/27357181/...",  // soft shadows, minimal
    "https://images.pexels.com/photos/33362027/...",  // lighting reference
    "https://images.pexels.com/photos/6167444/..."   // warm palette in context
  ]
}
```

### Decision Chain Summary

```
┌─────────────────────────────────────────────────────────────────┐
│  INPUT                                                          │
│    industry: "wellness"                                         │
│    brand personality: "natural, approachable, premium"           │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1: Choose mood                                             │
│    mood = "minimal"                                              │
│    WHY? minimal + wellness = calm, authentic, not clinical        │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2: Mood → Color palette direction                          │
│    warm neutrals (cream, taupe, sage)                            │
│    WHY? wellness needs warmth, minimal needs restraint            │
│    AVOID: blue (clinical), red (energetic), neon                 │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3: Mood + Industry → Typography pairing                    │
│    serif headline (trust/elegance) + sans body (readable)       │
│    IF mood=bold → display + condensed                            │
│    IF mood=playful → rounded sans                                │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4: Mood → Layout preference                               │
│    minimal → product_hero with whitespace                       │
│    bold → dynamic composition                                   │
│    playful → colorful flatlay                                   │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 5: Mood → Custom instructions                             │
│    specific guidance: "soft light, natural textures"             │
│    NOT generic: "beautiful" or "high quality"                    │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 6: Mood → Example selection criteria                      │
│    5 images that ALL share the same mood                        │
│    Variety: texture + lighting + composition + palette + context  │
│    Each adds a different dimension of the same style             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Field Reference

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name (1-255 chars) |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `brandName` | string | Brand name |
| `industry` | string | Industry category |
| `mood` | string | Mood: minimal, bold, playful, luxurious, natural, etc. |
| `stylePreset` | string | Preset category |
| `colors` | object | 5-color palette |
| `typography` | object | Font choices |
| `logo` | object | Logo URL + placement |
| `layoutPreference` | string | Layout type |
| `customInstructions` | string | AI generation guidance |
| `examples` | string[] | 5 reference image URLs |
| `styleParams` | object | Additional parameters |

### Color Object

| Key | Type | Description |
|-----|------|-------------|
| `primary` | string | Primary brand color (hex) |
| `secondary` | string | Secondary color (hex) |
| `accent` | string | Accent/highlight color (hex) |
| `background` | string | Background color (hex) |
| `text` | string | Text color (hex) |

### Typography Object

| Key | Type | Description |
|-----|------|-------------|
| `headlineFont` | string | Headline font family |
| `headlineWeight` | string | Headline weight |
| `bodyFont` | string | Body font family |
| `bodyWeight` | string | Body weight |

### Logo Object

| Key | Type | Description |
|-----|------|-------------|
| `url` | string | Logo image URL |
| `position` | string | Placement: top_left, top_right, bottom_left, bottom_right, center |
| `maxWidth` | string | Max width (e.g., "100px") |

---

## Validation Rules

| Field | Rule |
|-------|------|
| `name` | Required, 1-255 chars |
| `examples` | Max 5 URLs |
| `colors` | All 5 color keys required if provided |
| `typography` | All 4 keys required if provided |
| `logo` | All 3 keys required if provided |

---

## Error Codes

| HTTP Status | Error | Meaning |
|------------|-------|---------|
| 400 | `INVALID_REQUEST` | Malformed request body |
| 401 | `UNAUTHORIZED` | Invalid or missing API key |
| 404 | `STYLE_SYSTEM_NOT_FOUND` | Style system ID does not exist or belongs to another user |

