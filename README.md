# Clawfy Agent Skills

<div align="center">

![Clawfy](https://img.shields.io/badge/Clawfy-Skills-00FFC6?style=for-the-badge&logo=github&logoColor=0D1117)
![Status](https://img.shields.io/badge/Status-Production-7FDBCA?style=for-the-badge)
![API](https://img.shields.io/badge/API-v1.0.0-00E5FF?style=for-the-badge&logo=cloudflareworkers&logoColor=0D1117)
![License](https://img.shields.io/badge/License-MIT-A8E6CF?style=for-the-badge)

**Skills para que agentes AI operen Clawfy via API.**

Workflows de generación de imagen, video y audio — controlados por agentes autonomous.

</div>

---

## ¿Qué es esto?

Un package de **skills markdown-based** que permite a cualquier agente AI compatible con el formato (OpenClaw, Claude Code, Codex, etc.) conectarse a la API de Clawfy y ejecutar workflows de manera autonomous.

Los archivos `SKILL.md` son consumidos por el runtime del agente para entender:
- Qué triggers activan la skill
- Cómo autenticar contra la API
- Cuáles endpoints usar y en qué orden
- Cómo manejar errores y presentar resultados

**No es código ejecutable** — es documentación estructurada que el agente interpreta y sigue.

---

## Estructura del repo

```
Clawfy-Skills/
├── SKILL.md                          ← Entry point (lo que lee el agente)
├── README.md                         ← Este archivo
└── rules/
    ├── api-workflows.md               ← Referencia completa de endpoints
    ├── best-practices.md             ← Seguridad, errores, rate limits
    └── scenarios/
        ├── scenario-list-workflows.md ← Guide: listar workflows
        └── scenario-execute-workflow.md ← Guide: ejecutar y monitorear
```

---

## Quick Start

### 1. Configurar credenciales

```bash
export CLAWFY_API_KEY="tu-api-key"
export CLAWFY_BASE_URL="http://localhost:3001"  # default
```

### 2. Identificar triggers

La skill se activa cuando el usuario dice algo como:

| Español | English |
|----------|---------|
| ejecutar workflow | execute workflow |
| generar imagen | generate image |
| automatizacion | automation |
| estudio de workflows | workflow studio |
| ejecutar DAG | run DAG |
| producir video | create video |

### 3. El agente sigue el protocolo

```
POST /api/workflows/{id}/execute → obtener executionId
↓
GET /api/executions/:id          → poll cada 5s
↓
status = completed?              → presentar resultados
status = failed?                 → reportar error
```

---

## Skills disponibles

### `clawfy-workflow-execution`

**Entry point:** `SKILL.md`

Activa cuando el usuario quiere listar, ejecutar, monitorear u obtener resultados de workflows, o definir identidades de marca (Style Systems).

**Capabilities:**
- Listar todos los workflows del usuario
- Ver detalles y estructura (graph DSL) de un workflow
- Crear y ejecutar workflows
- Monitorear ejecución en tiempo real (WebSocket o polling)
- Cancelar ejecuciones
- Estimar costos antes de ejecutar
- **Gestionar Style Systems** — definir marca, colores, tipografía, mood, presets
- Listar, crear, actualizar y eliminar Style Systems

**Node types soportados:**

```
TEXT_PROMPT    →  Input de texto
IMAGE_GEN      →  Generación de imagen (MiniMax)
VIDEO_GEN      →  Generación de video (Kling/Veo)
AUDIO_GEN      →  Generación de audio (ElevenLabs)
PASSTHROUGH    →  Paso de datos sin transformación
OUTPUT         →  Output final
```

**Style System features:**

```
Style Systems  →  Brand identity (colors, typography, logo, mood, presets)
```

---

## Seguridad

⚠️ **Repo público — nunca exponer credenciales.**

- API key via variable de entorno `$CLAWFY_API_KEY`, nunca hardcodeada
- WebSocket auth via query param `?apiKey=` (no en headers ni body)
- Errores 500: nunca mostrar stack traces al usuario
- Validar tamaño de inputs antes de enviar

Ver: [rules/best-practices.md](rules/best-practices.md) — sección Security Rules.

---

## API Reference

### Base URL
```
http://localhost:3001  # desarrollo
https://api.clawfy.com # producción (usar HTTPS)
```

### Auth
```
x-api-key: $CLAWFY_API_KEY
```

### Endpoints principales

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/api/health` | Health check |
| `GET` | `/api/workflows` | Listar workflows |
| `GET` | `/api/workflows/:id` | Detalle de workflow |
| `POST` | `/api/workflows` | Crear workflow |
| `POST` | `/api/executions` | Ejecutar workflow |
| `GET` | `/api/executions/:id` | Estado de ejecución |
| `POST` | `/api/executions/:id/cancel` | Cancelar |
| `GET` | `/api/templates` | Listar templates |
| `POST` | `/api/cost` | Estimar costo |

### Style Systems (Brand Identity)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/api/style-systems` | Listar todos los Style Systems |
| `GET` | `/api/style-systems/:id` | Detalle de un Style System |
| `POST` | `/api/style-systems` | Crear Style System |
| `PUT` | `/api/style-systems/:id` | Actualizar Style System |
| `DELETE` | `/api/style-systems/:id` | Eliminar Style System |

---

## Ejemplo de uso

```
User: "Quiero generar una imagen de un atardecer usando Clawfy"

Agent: Detecta trigger → activa skill → ejecuta:

1. GET /api/workflows
   → Lista workflows disponibles

2. POST /api/executions
   → { workflowId: "wf_xxx", input: { "1": "sunset" } }
   → { executionId: "exec_xyz789", websocketUrl: "..." }

3. Polling (cada 5s):
   GET /api/executions/exec_xyz789
   → status: "running" | "completed" | "failed"

4. Si completed:
   → Presenta imagen directamente al usuario
   → Incluya resumen + link de descarga
```

---

## Deploy a ClawHub (opcional)

Para distribución directa a agentes OpenClaw:

```bash
cd Clawfy-Skills
clawhub login --token clh_tu_token
clawhub package publish . --dry-run  # test
clawhub package publish .             # publicar
```

---

## Referencias

- [OpenCreator Skills](https://github.com/OpenCreator-ai/opencreator-skills)
- [Remotion Skills](https://github.com/remotion-dev/remotion)
- [ClawHub CLI](https://github.com/openclaw/clawhub)
- Docs de API: [rules/api-workflows.md](rules/api-workflows.md)

---

<div align="center">

`#FF0055` · `00E5FF` · `00FFC6`

</div>