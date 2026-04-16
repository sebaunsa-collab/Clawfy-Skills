# Scenario: Manage Style Systems

This scenario shows how to list, create, update, and delete Style Systems — and how to guide users through defining their brand identity conversationally.

## When to use

- User asks "create a style system", "define my brand", "set up brand colors"
- User references "my brand", "brand style", "company colors"
- User wants to configure brand identity for image generation
- User says "quiero crear un estilo de marca", "definir marca", "configurar estilo"

---

## Flows Available

| Flow | When |
|------|------|
| [List Style Systems](#1-list-style-systems) | User wants to see what they already have |
| [Create Style System](#2-create-style-system) | User wants to define a new brand identity |
| [Update Style System](#3-update-style-system) | User wants to change existing brand parameters |
| [Delete Style System](#4-delete-style-system) | User wants to remove a brand identity |
| [Conversational Onboarding](#5-conversational-onboarding) | User has no Style System and needs guidance |

---

## 1. List Style Systems

### When to use

- User asks "what style systems do I have?"
- User asks to "see my brand styles"
- User wants to choose which style system to update or delete

### Steps

**1. Call List Style Systems API**

```bash
GET /api/v1/style-systems
```

**2. Present to user**

```
🎨 Your Style Systems

1. **Mercury Hub Brand Style**
   ID: LEF32mo3L3uCodboHq9o
   Brand: Mercury Hub
   Industry: tech
   Mood: premium
   Preset: clean_tech
   Last updated: 2 days ago

2. **Summer Vibes Campaign**
   ID: 7x9KpL2mN4oPqRsTuVwXyZaBcDeFgHiJk
   Brand: BeachCo
   Industry: lifestyle
   Mood: casual
   Preset: bold_color
   Last updated: 1 week ago

...

How many style systems total? [N]
```

**3. No Style Systems?**

```
You don't have any Style Systems yet.

A Style System defines your brand identity — colors, typography,
logo placement, mood, and visual style — so all your generated
content looks consistent.

Want to create one? I can guide you through it.
```

---

## 2. Create Style System

### When to use

- User wants to define a new brand identity
- User has no Style Systems and wants to set one up

### Steps

**1. Collect brand details** (see [Conversational Onboarding](#5-conversational-onboarding) for full guide)

**2. Create via API**

```bash
POST /api/v1/style-systems
```

**Request body — minimum required:**

```json
{
  "name": "Mercury Hub Brand Style",
  "brandName": "Mercury Hub",
  "industry": "tech",
  "mood": "premium"
}
```

**Request body — full example:**

```json
{
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
}
```

**3. Present result**

```
✅ Style System created!

**Mercury Hub Brand Style**
ID: ss_new456

Colors:
  Primary: #1A7F37
  Secondary: #2D3748
  Accent: #38A169
  Background: #FFFFFF
  Text: #1A202C

Mood: premium
Preset: clean_tech
Typography: Inter Bold (headlines), Inter Regular (body)

This style will now be applied automatically when you generate
images for Mercury Hub. Want to create a workflow that uses it?
```

---

## 3. Update Style System

### When to use

- User wants to change brand colors, typography, or other parameters
- User says "update my brand", "change the colors", "modify style"

### Steps

**1. List Style Systems** so user can pick which one to update

**2. Confirm what to change**

```
For "Mercury Hub Brand Style", what would you like to update?

1. Colors (primary, secondary, accent, background, text)
2. Typography (fonts and weights)
3. Logo (url, position, size)
4. Mood (premium, casual, bold, minimal, friendly)
5. Preset (clean_tech, premium_luxury, etc.)
6. Custom instructions (design notes)
7. All of the above

Enter the number(s) you want to change, or describe what you want to update.
```

**3. Collect new values**

For colors, ask individually:
```
Current primary color: #1A7F37
New primary color: [user input — must be hex, e.g. #FF0055]

Current secondary: #2D3748
New secondary: [user input]
...
```

**4. Update via API**

```bash
PUT /api/v1/style-systems/{id}
```

```json
{
  "colors": {
    "primary": "#FF0055",
    "secondary": "#1A202C",
    "accent": "#00E5FF",
    "background": "#0D1117",
    "text": "#FFFFFF"
  },
  "mood": "bold"
}
```

**5. Present result**

```
✅ Style System updated!

**Mercury Hub Brand Style** — changes applied

Colors updated:
  Primary: #FF0055 (was #1A7F37)
  Secondary: #1A202C (was #2D3748)
  Accent: #00E5FF (was #38A169)
  Background: #0D1117 (was #FFFFFF)
  Text: #FFFFFF (was #1A202C)

Mood updated: premium → bold

These changes will apply to all new image generations using
this Style System.
```

---

## 4. Delete Style System

### When to use

- User wants to remove a brand identity
- User says "delete style system", "remove brand", "delete my brand"

### Steps

**1. List Style Systems** so user can pick which one to delete

**2. Confirm before deleting**

```
⚠️ Delete "Summer Vibes Campaign"?

This will permanently remove this Style System and cannot be undone.
Any workflows using this style may produce unexpected output.

Are you sure? Type "yes" to confirm.
```

**3. Delete via API**

```bash
DELETE /api/v1/style-systems/{id}
```

**4. Present result**

```
✅ Deleted "Summer Vibes Campaign"

This Style System has been permanently removed.
```

---

## 5. Conversational Onboarding

Use this flow when the user has no Style System or wants to create one from scratch. Collect details progressively — start with minimum required and let them add more later.

### Minimum required fields

Only `name` is required to create a Style System. Everything else can be `null` and updated later.

### Full onboarding script

```
Agent: "Vamos a crear tu Style System. Esto va a definir cómo se ven
todas las imágenes que generemos para tu marca — colores, tipografía,
mood, y más.

Podemos empezar con lo básico e ir completando después."

---

STEP 1: Name
"1. **Nombre del Style System**
   ¿Cómo querés llamar a este estilo?
   (ej: 'Mi Marca', 'Mercury Hub Brand', 'Summer Collection')

   → User provides name

---

STEP 2: Brand name
"2. **Nombre de la marca**
   ¿Cuál es el nombre de tu negocio o marca?
   (ej: Mercury Hub, BeachCo, Natura)

   → User provides brandName (can skip)

---

STEP 3: Industry
"3. **Industria**
   ¿A qué industria pertenece tu marca?
   Opciones: tech, beauty, food, fashion, lifestyle, finance, health, other

   → User picks or describes

---

STEP 4: Colors
"4. **Colores**
   ¿Cuáles son tus colores de marca?

   Necesito los hex de cada uno:
   - Primary (tu color principal de marca): ___
   - Secondary (color secundario): ___
   - Accent (color de acento): ___
   - Background (color de fondo): ___
   - Text (color de texto): ___

   (Podés usar un selector de color o escribir los hex como #FF0055)

   → User provides colors (can skip or provide partial)

   If user doesn't know: "Podés usar '#000000' como placeholder
   y actualizarlos después."

---

STEP 5: Typography
"5. **Tipografía**
   ¿Qué fuentes usás en tu marca?

   - Headline (titulares): ___ (ej: Inter Bold)
   - Body (cuerpo de texto): ___ (ej: Inter Regular)

   → User provides fonts (can skip)

---

STEP 6: Mood
"6. **Mood**
   ¿Cómo querés que se sienta visualmente tu marca?

   Opciones:
   - premium     → sofisticado, limpio, profesional
   - casual     → relajado, amigable, accesible
   - bold       → impacto alto, colores fuertes, declaración
   - minimal    → lo menos posible, blanco, tipografía净额
   - friendly   → cálido, cercano, humano
   - dramatic   → oscuro, intenso, premium

   → User picks one

---

STEP 7: Preset (optional)
"7. **Preset** (opcional)
   ¿Querés basarte en alguna familia visual conocida?

   Opciones:
   - clean_tech       → Minimalista tech, luz suave
   - clean_beauty     → Cosméticos, fondo blanco, elegante
   - premium_luxury   → Oscuro, acentos dorados, dramático
   - streetwear       → Colores bold, ángulos dinámicos
   - wellness_natur   → Tonos tierra, luz natural, orgánico
   - bold_color       → Fondos sólidos vibrantes, alto impacto

   Si no sabés, podés dejarlo en blanco — el preset ayuda
   pero no es obligatorio.

   → User picks or skips

---

STEP 8: Logo (optional)
"8. **Logo** (opcional)
   ¿Tenés un logo URL?

   - URL del logo: ___
   - Posición: top_left / top_right / bottom_left / bottom_right / center
   - Tamaño máximo: ___ (ej: 80px, 10%)

   → User provides or skips

---

STEP 9: Custom instructions (optional)
"9. **Instrucciones extra** (opcional)
   ¿Alguna nota de diseño que deba saber?

   Ejemplos:
   - 'Usar fotos de producto con luz de estudio suave'
   - 'Evitar estilos cartoon o clipart'
   - 'siempre mostrar el producto en primer plano'

   → User provides or skips

---

STEP 10: Confirm and create
"Perfecto. Tenemos todo para crear tu Style System.

Resumen:
- Nombre: Mercury Hub Brand Style
- Marca: Mercury Hub
- Industria: tech
- Colores: #1A7F37 / #2D3748 / #38A169 / #FFFFFF / #1A202C
- Tipografía: Inter Bold / Inter Regular
- Mood: premium
- Preset: clean_tech

¿Algún cambio antes de crear? Decime 'sí' para confirmar
o indicame qué querés ajustar."
```

### After creation

```
✅ **Mercury Hub Brand Style** creado!

Tu Style System está listo. Ahora cuando generemos imágenes
para Mercury Hub, vamos a usar estos parámetros automáticamente.

¿Querés crear un workflow que use este estilo? O podemos
probar generando una imagen de prueba.
```

---

## Error Handling

| Error | Response |
|-------|----------|
| 401 Unauthorized | "Tu API key es inválida. Verificá CLAWFY_API_KEY." |
| 404 STYLE_SYSTEM_NOT_FOUND | "No encontré ese Style System. Puede que ya haya sido eliminado." |
| 422 VALIDATION_ERROR | "El valor de [campo] no es válido. [detalle]" |
| 500 Internal Error | "Falló la operación. Intentá de nuevo en un momento." |

---

## Tips

- **Partial creation is fine** — create with what user has, update later
- **Colors are the most important** — if user only provides one thing, get the primary color
- **Show current values** when updating — always display what the current value is before asking for new one
- **Confirm before delete** — never delete without explicit "yes" confirmation
- **Presets are shortcuts** — if user is unsure about custom instructions, a preset gets them 80% there
