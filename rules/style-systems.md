# Style Systems API — Complete Reference

> Base URL: `$CLAWFY_BASE_URL` (default: `http://localhost:3001`)
> All requests require: `x-api-key: $CLAWFY_API_KEY`
> Content-Type: `application/json` for all requests

Style Systems let you define brand identities — colors, typography, logo, mood, layout preferences, and style presets — and reuse them across all image generations for consistent brand output.

---

## 1. List Style Systems

### GET /api/v1/style-systems

Returns all Style Systems for the authenticated user.

**Request:**
```bash
curl -s http://localhost:3001/api/v1/style-systems \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response:**
```json
{
  "styleSystems": [
    {
      "id": "LEF32mo3L3uCodboHq9o",
      "userId": "user_xxx",
      "name": "Mercury Hub Brand Style",
      "brandName": "Mercury Hub",
      "industry": "tech",
      "colors": {
        "primary": "#1A7F37",
        "secondary": "#2D3748",
        "accent": "#38A169",
        "background": "#FFFFFF",
        "text": "#1A202C"
      },
      "typography": {
        "headlineFont": "Inter",
        "headlineWeight": "Bold",
        "bodyFont": "Inter",
        "bodyWeight": "Regular"
      },
      "logo": {
        "url": "https://example.com/logo.png",
        "position": "bottom_right",
        "maxWidth": "80px"
      },
      "layoutPreference": "product_hero",
      "mood": "premium",
      "stylePreset": "clean_tech",
      "customInstructions": "Use product photography with soft studio lighting",
      "examples": ["https://example.com/ref1.jpg"],
      "createdAt": "2026-04-01T10:00:00.000Z",
      "updatedAt": "2026-04-05T15:30:00.000Z"
    }
  ]
}
```

> **Note:** IDs are generated via [nanoid](https://github.com/adioss/nanoid) — 21-character alphanumeric strings (e.g., `LEF32mo3L3uCodboHq9o`).

---

## 2. Get Style System

### GET /api/v1/style-systems/:id

Get a specific Style System by ID.

**Request:**
```bash
curl -s http://localhost:3001/api/v1/style-systems/LEF32mo3L3uCodboHq9o \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response (200 OK):**
```json
{
  "styleSystem": {
    "id": "LEF32mo3L3uCodboHq9o",
    "userId": "user_xxx",
    "name": "Mercury Hub Brand Style",
    "brandName": "Mercury Hub",
    "industry": "tech",
    "colors": { ... },
    "typography": { ... },
    "logo": { ... },
    "layoutPreference": "product_hero",
    "mood": "premium",
    "stylePreset": "clean_tech",
    "customInstructions": "Use product photography with soft studio lighting",
    "examples": ["https://example.com/ref1.jpg"],
    "createdAt": "2026-04-01T10:00:00.000Z",
    "updatedAt": "2026-04-05T15:30:00.000Z"
  }
}
```

**Error (404 Not Found):**
```json
{
  "error": "Style system not found"
}
```

---

## 3. Create Style System

### POST /api/v1/style-systems

Create a new Style System with brand identity parameters.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Human-readable name (e.g. "Mercury Hub Brand") |
| `brandName` | string | No | Brand/business name |
| `industry` | string | No | e.g. tech, beauty, food, fashion |
| `colors` | object | No | Brand color palette |
| `typography` | object | No | Font choices |
| `logo` | object | No | Logo placement rules |
| `layoutPreference` | string | No | e.g. product_hero, split_50_50, text_heavy |
| `mood` | string | No | e.g. premium, casual, bold, minimal, friendly |
| `stylePreset` | string | No | Visual family preset |
| `customInstructions` | string | No | Free-form design notes |
| `examples` | string[] | No | URLs to reference images |

### colors object

```json
{
  "primary": "#1A7F37",
  "secondary": "#2D3748",
  "accent": "#38A169",
  "background": "#FFFFFF",
  "text": "#1A202C"
}
```

All values are hex color codes (e.g. `#FF0055`).

### typography object

```json
{
  "headlineFont": "Inter",
  "headlineWeight": "Bold",
  "bodyFont": "Inter",
  "bodyWeight": "Regular"
}
```

### logo object

```json
{
  "url": "https://example.com/logo.png",
  "position": "bottom_right",
  "maxWidth": "80px"
}
```

Position options: `top_left`, `top_right`, `bottom_left`, `bottom_right`, `center`

### stylePreset

Presets are suggested visual families — you can use any string value. Common presets:

| Preset | Description |
|--------|-------------|
| `clean_tech` | Minimalist tech aesthetic, soft studio light, white/bold accents |
| `clean_beauty` | Skincare/cosmetic aesthetic, white backgrounds, elegant serif |
| `premium_luxury` | Dark backgrounds, gold accents, dramatic lighting |
| `streetwear` | Bold colors, dynamic angles, high contrast, urban feel |
| `wellness_natur` | Earth tones, natural lighting, organic layouts |
| `bold_color` | Vibrant solid backgrounds, strong color blocks, high impact |

> **Note:** `stylePreset` is a **free-form string** — any value is accepted. The presets above are suggestions only; validation is not enforced server-side.

### Request Example

```bash
curl -s -X POST http://localhost:3001/api/v1/style-systems \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Mercury Hub Brand Style",
    "brandName": "Mercury Hub",
    "industry": "tech",
    "colors": {
      "primary": "#1A7F37",
      "secondary": "#2D3748",
      "accent": "#38A169",
      "background": "#FFFFFF",
      "text": "#1A202C"
    },
    "typography": {
      "headlineFont": "Inter",
      "headlineWeight": "Bold",
      "bodyFont": "Inter",
      "bodyWeight": "Regular"
    },
    "logo": {
      "url": "https://mercuryhub.com/logo.png",
      "position": "bottom_right",
      "maxWidth": "80px"
    },
    "layoutPreference": "product_hero",
    "mood": "premium",
    "stylePreset": "clean_tech",
    "customInstructions": "Use product photography with soft studio lighting. Avoid cartoon or clipart styles."
  }'
```

**Response (201 Created):**
```json
{
  "styleSystem": {
    "id": "LEF32mo3L3uCodboHq9o",
    "userId": "user_xxx",
    "name": "Mercury Hub Brand Style",
    "brandName": "Mercury Hub",
    "industry": "tech",
    "colors": { "primary": "#1A7F37", "secondary": "#2D3748", "accent": "#38A169", "background": "#FFFFFF", "text": "#1A202C" },
    "typography": { "headlineFont": "Inter", "headlineWeight": "Bold", "bodyFont": "Inter", "bodyWeight": "Regular" },
    "logo": { "url": "https://mercuryhub.com/logo.png", "position": "bottom_right", "maxWidth": "80px" },
    "layoutPreference": "product_hero",
    "mood": "premium",
    "stylePreset": "clean_tech",
    "customInstructions": "Use product photography with soft studio lighting. Avoid cartoon or clipart styles.",
    "examples": null,
    "createdAt": "2026-04-14T19:00:00.000Z",
    "updatedAt": "2026-04-14T19:00:00.000Z"
  }
}
```

---

## 4. Update Style System

### PUT /api/v1/style-systems/:id

Update an existing Style System. Only include fields you want to change.

**Request Body:**

Same fields as POST. All fields optional — only provided fields are updated.

**Request Example: Update mood and colors**

```bash
curl -s -X PUT http://localhost:3001/api/v1/style-systems/LEF32mo3L3uCodboHq9o \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mood": "bold",
    "colors": {
      "primary": "#FF0055",
      "secondary": "#1A202C",
      "accent": "#00E5FF",
      "background": "#0D1117",
      "text": "#FFFFFF"
    }
  }'
```

**Response (200 OK):**
```json
{
  "styleSystem": {
    "id": "LEF32mo3L3uCodboHq9o",
    "userId": "user_xxx",
    "name": "Mercury Hub Brand Style",
    "brandName": "Mercury Hub",
    "industry": "tech",
    "colors": { "primary": "#FF0055", "secondary": "#1A202C", "accent": "#00E5FF", "background": "#0D1117", "text": "#FFFFFF" },
    "typography": { "headlineFont": "Inter", "headlineWeight": "Bold", "bodyFont": "Inter", "bodyWeight": "Regular" },
    "logo": { "url": "https://example.com/logo.png", "position": "bottom_right", "maxWidth": "80px" },
    "layoutPreference": "product_hero",
    "mood": "bold",
    "stylePreset": "clean_tech",
    "customInstructions": "Use product photography with soft studio lighting",
    "examples": null,
    "createdAt": "2026-04-01T10:00:00.000Z",
    "updatedAt": "2026-04-14T20:00:00.000Z"
  }
}
```

**Error (404 Not Found):**
```json
{
  "error": "Style system not found"
}
```

---

## 5. Delete Style System

### DELETE /api/v1/style-systems/:id

Delete a Style System permanently.

**Request:**
```bash
curl -s -X DELETE http://localhost:3001/api/v1/style-systems/LEF32mo3L3uCodboHq9o \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response (200 OK):**
```json
{
  "success": true
}
```

**Error (404 Not Found):**
```json
{
  "error": "Style system not found"
}
```

**Warning:** Deleting a Style System used by active workflows may cause unexpected output. Check workflow dependencies first.

---

## Error Responses

| HTTP Status | Meaning |
|-------------|---------|
| 400 | Malformed request body — JSON is invalid |
| 401 | Invalid or missing API key |
| 404 | Style System ID does not exist (or belongs to another user) |
| 422 | Validation failed — check `error` and `details` fields for specifics |

> **Note:** Error responses use plain `error` strings — there are **no structured error codes** like `STYLE_SYSTEM_NOT_FOUND`. The agent should read the `error` field directly.

---

## Conversational Onboarding Flow

When the user wants to create a Style System but doesn't have all details ready, collect them progressively:

```
Agent: "Vamos a crear tu Style System. Necesito unos datos:

1. **Nombre**: ¿Cómo querés llamar a este estilo?
2. **Nombre de marca**: ¿Cuál es el nombre de tu negocio o marca?
3. **Industria**: ¿A qué industria pertenece? (tech, beauty, food, fashion, etc.)
4. **Colores**: ¿Cuáles son tus colores principales? Dame los hex (ej: #1A7F37)
   - Primary (principal): ___
   - Secondary (secundario): ___
   - Accent (acento): ___
   - Background: ___
   - Text: ___
5. **Tipografía**: ¿Qué fuentes usás? (nombre y peso, ej: Inter Bold)
6. **Mood**: ¿Cómo querés que se sienta la marca? (premium, casual, bold, minimal, friendly)
7. **Preset** (opcional): ¿Querés basarlo en alguna familia visual? (clean_tech, premium_luxury, etc.)
8. **Instrucciones extra**: ¿Alguna nota de diseño que deba saber?

Podemos empezar con los que tengas y ir completando después."
```

**Partial creation:** If user only has some fields, create the Style System with what they provided and note the rest as `null`. They can update later with `PUT /api/v1/style-systems/:id`.
