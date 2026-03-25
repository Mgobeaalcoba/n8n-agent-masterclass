<div align="center">

# 🤖 Nathan — Autonomous Executive Agent

### El agente de IA que gestiona tu agenda y correos en lenguaje natural

[![n8n](https://img.shields.io/badge/built%20with-n8n-orange?style=for-the-badge&logo=n8n)](https://n8n.io)
[![Gemini](https://img.shields.io/badge/AI-Google%20Gemini-4285F4?style=for-the-badge&logo=google)](https://ai.google.dev)
[![OpenRouter](https://img.shields.io/badge/fallback-OpenRouter-6366F1?style=for-the-badge)](https://openrouter.ai)
[![LangChain](https://img.shields.io/badge/orchestrator-LangChain%20v3.1-1C3C3C?style=for-the-badge)](https://docs.langchain.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](./LICENSE)

**Proyecto final de la Masterclass de AI Automation — Henry**

[Ver Demo](#-demo-en-vivo) · [Arquitectura](#-arquitectura-técnica) · [Instalación](#-cómo-importar-el-workflow) · [Inscribite al curso](#-querés-construir-esto-vos-mismo)

</div>

---

## ✨ ¿Qué es Nathan?

**Nathan** es un agente ejecutivo autónomo construido íntegramente en **n8n** durante la masterclass de Henry sobre AI Automation. No es un chatbot que responde preguntas genéricas: es un sistema que toma decisiones, ejecuta acciones reales sobre tus herramientas de Google, y recuerda el contexto de la conversación.

Podés decirle cosas como:

> *"Nathan, ¿tengo algún mail de mi jefe sin responder?"*
> *"Agendame una reunión con el equipo para el martes a las 10."*
> *"Mejor a las 15:00, y agregá a Lucas como invitado."*

Y Nathan lo hace. Sin formularios. Sin clicks. Solo lenguaje natural.

---

## 🎯 ¿Qué hace exactamente?

| Capacidad | Descripción |
|---|---|
| 📬 **Leer Gmail** | Busca y resume hilos de correos (hasta 20 threads) |
| ✉️ **Enviar emails** | Redacta y envía respuestas con destinatario, asunto y cuerpo generados por IA |
| 📅 **Leer el Calendario** | Consulta tu agenda para verificar disponibilidad antes de agendar |
| 🗓️ **Crear eventos** | Crea eventos con título, descripción, ubicación, horario e invitados |
| 🧠 **Memoria contextual** | Recuerda los últimos 10 intercambios — podés corregir sobre la marcha |
| 🕐 **Razonamiento temporal** | Entiende "mañana", "el lunes que viene" y los convierte a timestamps precisos |

---

## 🏗️ Arquitectura Técnica

Este workflow no es un flujo lineal. Es un **ecosistema de agentes y herramientas**, diseñado con los mismos principios que usan los equipos de ingeniería de IA en producción.

```
                    ┌─────────────────────────────────────────────────┐
                    │            EXECUTIVE ASSISTANT AGENT             │
                    │              (LangChain Agent v3.1)              │
                    │                                                   │
                    │  ┌──────────────┐    ┌─────────────────────┐    │
  Chat Trigger ────▶│  │ Gemini (1°)  │    │  Window Buffer      │    │
  (UI Pública)      │  │ OpenRouter   │    │  Memory (10 msgs)   │    │
                    │  │ GLM-4.5 (2°) │    └─────────────────────┘    │
                    │  └──────────────┘                                │
                    │                                                   │
                    │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐ │
                    │  │  Gmail   │ │ Calendar │ │ Calendar Create  │ │
                    │  │  Read    │ │  Read    │ │    + $fromAI      │ │
                    │  └──────────┘ └──────────┘ └──────────────────┘ │
                    │                    ┌─────────────────────┐       │
                    │                    │     Gmail Send      │       │
                    │                    │      + $fromAI      │       │
                    │                    └─────────────────────┘       │
                    └─────────────────────────────────────────────────┘
```

### Decisiones de diseño que marcan la diferencia

**1. Cerebro híbrido (Multi-Model Support)**
El agente no depende de un solo proveedor de IA. Usa **Google Gemini** como modelo primario —por su integración nativa con el ecosistema Google— y **OpenRouter (GLM-4.5-Air:Free)** como fallback automático. Si Gemini cae, Nathan sigue funcionando.

**2. Herramientas separadas por responsabilidad**
Existe un nodo `read` y un nodo `write` separados para cada servicio. Esto evita alucinaciones de parámetros y sigue el principio de **mínimo privilegio**: el agente solo actúa con el nivel de permisos que la tarea requiere en ese momento.

**3. AI Overrides (`$fromAI`)**
Los nodos de escritura (crear eventos, enviar emails) usan variables dinámicas capturadas directamente desde la IA. Esto garantiza que campos críticos como `Start`, `End`, `Subject` o `Attendees` sean completados de forma estructurada, no improvisada.

**4. Window Buffer Memory**
La memoria está limitada a 10 intercambios. Esto no es una limitación — es una decisión de ingeniería que optimiza el consumo de tokens y evita que el agente se confunda con instrucciones antiguas del chat.

---

## 📦 Stack de Nodos

| Nodo | Función | Versión |
|---|---|---|
| `Chat Trigger` | Interfaz pública "Nathan" con persistencia de sesión | 1.4 |
| `Executive Assistant Agent` | Orquestador central con LangChain | 3.1 |
| `Google Gemini Model` | Modelo primario de razonamiento | 1.0 |
| `OpenRouter Chat Model` | Modelo de fallback (GLM-4.5-Air:Free) | 1.0 |
| `Window Buffer Memory` | Contexto conversacional (10 mensajes) | 1.3 |
| `Simple Memory` | Memoria conectada a la UI de chat | 1.3 |
| `Google Calendar Tool` | Lectura de agenda (verificación de slots) | 1.3 |
| `Create an Event Tool` | Escritura con variables dinámicas de IA | 1.3 |
| `Gmail Tool` | Búsqueda de contexto en bandeja de entrada | 2.2 |
| `Send a Message Tool` | Ejecución de comunicaciones salientes | 2.2 |

---

## 🚀 Cómo importar el workflow

### Prerrequisitos

- Una instancia de **n8n** (self-hosted o cloud)
- Una cuenta de **Google** con acceso a Gmail y Calendar
- Una API key de **Google Gemini** (gratuita)
- Una cuenta de **OpenRouter** (plan gratuito suficiente para el fallback)

### Pasos

**1. Clonar el repositorio**
```bash
git clone https://github.com/Mgobeaalcoba/n8n-agent-masterclass.git
cd n8n-agent-masterclass
```

**2. Importar el workflow en n8n**

Abrí tu instancia de n8n y seguí estos pasos:
```
Settings → Import workflow → Subir archivo → workflows/n8n-agent-masterclass.json
```

**3. Configurar las credenciales**

En n8n, creá y conectá las siguientes credenciales:

| Servicio | Tipo de credencial |
|---|---|
| Google Gemini | `Google PaLM API` |
| Google Calendar | `Google Calendar OAuth2 API` |
| Gmail | `Gmail OAuth2` |
| OpenRouter | `OpenRouter API` |

**4. Activar el workflow**

Hacé click en el toggle de activación. La URL pública del Chat Trigger está lista para usar.

---

## 🎬 Demo en Vivo

Durante la masterclass se demostraron tres escenarios que muestran el potencial del agente:

### Escenario 1 — Lectura inteligente de Gmail
```
Usuario:  "Nathan, ¿tengo algún mail de Henry sobre la carrera?"
Nathan:   Busca en los últimos 20 hilos, resume los relevantes
          y ofrece responder directamente desde el chat.
```

### Escenario 2 — Agendado con verificación de disponibilidad
```
Usuario:  "Agendame una reunión para mañana a las 10 AM."
Nathan:   Consulta el calendario, verifica si el slot está libre
          y crea el evento con todos los detalles.
```

### Escenario 3 — Memoria contextual en acción
```
Usuario:  "Cámbiala para las 15:00."
Nathan:   Entiende a qué evento te referís sin que lo repitas,
          gracias al Window Buffer Memory.
```

---

## 📁 Estructura del repositorio

```
n8n-agent-masterclass/
├── workflows/
│   └── n8n-agent-masterclass.json   # El workflow listo para importar
├── SPECS/
│   └── SPEC Técnica y Funcional...  # Diseño técnico y funcional completo
├── presentation/
│   └── Nathan_Autonomous_Executive_Agent.pdf  # Slides de la masterclass
├── pyproject.toml                   # Metadata del proyecto
├── CHANGELOG.md                     # Historial de versiones
├── LICENSE                          # MIT
└── README.md                        # Este archivo
```

---

## 🎓 ¿Querés construir esto vos mismo?

Este proyecto es el trabajo final de la **Masterclass de AI Automation** de **[Henry](https://www.soyhenry.com)**, donde aprendés a construir agentes autónomos desde cero usando n8n, LLMs y APIs reales.

### Lo que vas a aprender en el curso

- ⚡ **n8n de cero a producción**: instalación, nodos, triggers, expressions y credenciales
- 🤖 **Arquitectura de agentes**: cómo diseñar un agente que razona, planifica y ejecuta acciones
- 🔗 **Integración de LLMs**: Gemini, OpenRouter, OpenAI — cuándo usar cada uno y por qué
- 🛠️ **Toolbox real**: conectar Gmail, Calendar, Slack, Notion, WhatsApp y más
- 🧠 **Gestión de memoria**: tipos de memoria, ventanas de contexto, optimización de tokens
- 🔒 **Fallbacks y resiliencia**: cómo evitar que tu agente muera si un proveedor falla
- 🚀 **Deploy y producción**: llevar workflows de la demo a un entorno real

### ¿Por qué vale la pena?

> **"No usamos un solo nodo; usamos un ecosistema. Esto es lo que hace un AI Automation Engineer senior."**

El mercado de **AI Automation Engineers** está explotando. Empresas de todos los rubros están buscando profesionales que sepan construir agentes y automatizaciones con IA. Este curso te da exactamente esas habilidades, con proyectos reales, no simulaciones.

**¿Querés que tu próximo proyecto sea como Nathan, pero tuyo?**

[![Inscribite en Henry](https://img.shields.io/badge/Inscribite%20en%20Henry-%F0%9F%9A%80%20Empezá%20ahora-FF6B35?style=for-the-badge)](https://www.soyhenry.com)

---

## 👤 Autor

**Mariano Gobea Alcoba**
Instructor · AI Automation Engineer · Henry Masterclass

[![GitHub](https://img.shields.io/badge/GitHub-Mgobeaalcoba-181717?style=flat-square&logo=github)](https://github.com/Mgobeaalcoba)

---

## 📄 Licencia

Este proyecto está bajo la licencia **MIT**. Podés usarlo, modificarlo y distribuirlo libremente, con atribución al autor original.

---

<div align="center">

*Construido con ❤️ para la comunidad de Henry*

**[⬆ Volver arriba](#-nathan--autonomous-executive-agent)**

</div>
