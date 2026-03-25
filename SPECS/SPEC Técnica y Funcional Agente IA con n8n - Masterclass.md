# **📄 SPEC Técnica: "Nathan", el Agente Ejecutivo Multi-LLM**

**Proyecto:** Agente Autónomo de Productividad con n8n y Gemini.

**Objetivo Henry:** Demostrar la orquestación de múltiples modelos de IA y herramientas de terceros en un entorno de producción.

## ---

**1\. Definición Funcional (User Experience)**

El agente, bautizado como **"Nathan"**, actúa como un asistente de organización personal accesible vía chat web.

* **Capacidad de Memoria:** Nathan recuerda los últimos 10 intercambios, permitiendo correcciones sobre la marcha (ej. "No, en ese horario no puedo, busca otro").  
* **Gestión de Gmail:** Puede buscar hilos de correos (limitado a los últimos 20 para eficiencia) y redactar/enviar respuestas.  
* **Gestión de Calendario:** Lee la agenda actual del usuario y tiene permiso para crear eventos con detalles completos (invitados, ubicación, descripción).  
* **Razonamiento Temporal:** Interpreta lenguaje humano relativo ("el lunes que viene", "mañana a la tarde") y lo traduce a timestamps precisos mediante el contexto de fecha actual.

## ---

**2\. Arquitectura Técnica (System Design)**

## **A. El "Cerebro" Híbrido (Multi-Model Support)**

A diferencia de flujos simples, este workflow utiliza una arquitectura de **redundancia/especialización**:

1. **Modelo Primario:** Google Gemini (vía nodo nativo). Elegido por su integración nativa con el ecosistema Google.  
2. **Modelo Secundario (Fallback/Complemento):** OpenRouter (GLM-4.5-Air:Free). Esto demuestra a los alumnos cómo diversificar proveedores de cómputo para asegurar disponibilidad y aprovechar créditos gratuitos.

## **B. El Orquestador (Executive Assistant Agent)**

El nodo central es un **Agent de LangChain (v3.1)** configurado con una lógica de "razonamiento antes de la acción":

* **System Message:** Instrucciones estrictas para verificar disponibilidad antes de agendar y resumir antes de ofrecer acciones de mail.  
* **Manejo de Fallbacks:** Configurado para recuperarse si una de las herramientas de Google devuelve un error.

## **C. El Toolbox (Granularidad de Herramientas)**

Se han separado las responsabilidades para evitar alucinaciones en los parámetros:

* **Read-Only Tools:** Nodos específicos para getAll en Gmail y Calendar.  
* **Write Tools:** Nodos configurados con **AI Overrides** ($fromAI) para capturar campos específicos (Subject, Message, Start, End) de forma estructurada.

## **D. Gestión de Estado (Memory)**

* **Window Buffer Memory:** Limita el contexto a 10 mensajes. Esto optimiza el consumo de tokens y evita que el agente se confunda con instrucciones muy antiguas del chat.

## ---

**3\. Desglose de Nodos (The Build Stack)**

| Nodo | Función Específica | Versión |
| :---- | :---- | :---- |
| **Chat Trigger** | Interfaz "Nathan" con persistencia de sesión. | 1.4 |
| **Gemini & OpenRouter** | Proveedores de inteligencia (Dual-brain). | 1.0 |
| **Simple Memory** | Buffer de memoria conectado al trigger para UI. | 1.3 |
| **Google Calendar Tool** | Operación getAll (Verificación de slots). | 1.3 |
| **Create Event Tool** | Escritura estructurada con variables dinámicas de IA. | 1.3 |
| **Gmail Tool** | Búsqueda de contexto en bandeja de entrada. | 2.2 |
| **Send Email Tool** | Ejecución de comunicaciones salientes. | 2.2 |

## ---

**4\. Estrategia de Masterclass (Live Demo)**

Para los 50 minutos de Henry, este workflow te permite lucirte con tres escenarios:

1. **Escenario de Lectura:** *"Nathan, ¿tengo algún mail de Henry sobre la carrera?"* (Muestra la Tool de Gmail).  
2. **Escenario de Conflicto:** *"Agendame una reunión para mañana a las 10 AM"*. (El agente leerá el calendario, verá que no hay nada o hay algo, y responderá en consecuencia).  
3. **Escenario de Memoria:** *"Cámbiala para las 15:00"*. (Aquí demuestras el **Window Buffer Memory**, ya que no hace falta repetir qué es lo que quieres cambiar).

## ---

**5\. El "Impacto Henry" (Marketing & Ventas)**

Durante la explicación de este JSON, enfatiza estos puntos:

* **Nivel Profesional:** "No usamos un solo nodo; usamos un ecosistema. Esto es lo que hace un **AI Automation Engineer** senior".  
* **Integración Real:** "Estamos tocando datos reales de Google, no es una simulación".  
* **Versatilidad:** "Si mañana Gemini cae, Nathan sigue vivo gracias a nuestra conexión con OpenRouter".

---

```json
{
  "nodes": [
    {
      "parameters": {
        "public": true,
        "initialMessages": "¡Hola! 👋\nMe llamo Nathan. ¿En qué puedo ayudarte hoy?",
        "availableInChat": true,
        "agentIcon": {
          "type": "icon",
          "value": "messages-square"
        },
        "agentName": "Nathan",
        "agentDescription": "Un asistente de organización personal",
        "options": {
          "loadPreviousSession": "memory"
        }
      },
      "id": "8f108339-7fe9-44a0-925b-12513b2319e0",
      "name": "Chat Trigger",
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.4,
      "position": [
        336,
        304
      ],
      "webhookId": "c93b4eca-fddf-4602-bb40-63a431c9a6ef"
    },
    {
      "parameters": {
        "needsFallback": true,
        "options": {
          "systemMessage": "Actúa como un Asistente Ejecutivo de alto nivel. Tu objetivo es gestionar la agenda y correos del usuario.\n\nSi el usuario pide agendar algo, primero verifica la disponibilidad usando el calendario.\n\nSi te piden buscar un mail, resume el contenido antes de ofrecer acciones.\n\nUsa siempre la fecha y hora actual como referencia para términos relativos como \"mañana\" o \"la semana que viene\"."
        }
      },
      "id": "45b92a89-a602-498b-a899-393d025dc4f1",
      "name": "Executive Assistant Agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        1008,
        304
      ]
    },
    {
      "parameters": {
        "options": {}
      },
      "id": "8f7bf678-b9b9-46e3-bce9-8256d0538172",
      "name": "Google Gemini Model",
      "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
      "typeVersion": 1,
      "position": [
        688,
        528
      ],
      "credentials": {
        "googlePalmApi": {
          "id": "OD88yay032b6k7Hu",
          "name": "Google Gemini(PaLM) Api account"
        }
      }
    },
    {
      "parameters": {
        "contextWindowLength": 10
      },
      "id": "8a57e246-e9a8-472b-a481-06a6be7c2e98",
      "name": "Window Buffer Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [
        944,
        528
      ]
    },
    {
      "parameters": {
        "operation": "getAll",
        "calendar": {
          "__rl": true,
          "mode": "list",
          "value": "primary"
        },
        "options": {}
      },
      "id": "090f2b2c-7152-488c-acfe-6e2f3f150883",
      "name": "Google Calendar Tool",
      "type": "n8n-nodes-base.googleCalendarTool",
      "typeVersion": 1.3,
      "position": [
        1072,
        528
      ],
      "credentials": {
        "googleCalendarOAuth2Api": {
          "id": "1tGi4mK9cE2A8FfN",
          "name": "Google Calendar OAuth2 API"
        }
      }
    },
    {
      "parameters": {
        "operation": "getAll",
        "limit": 20,
        "filters": {
          "readStatus": "both"
        }
      },
      "id": "16095bb7-a567-4158-823e-9ac1350bde91",
      "name": "Gmail Tool",
      "type": "n8n-nodes-base.gmailTool",
      "typeVersion": 2.2,
      "position": [
        1200,
        528
      ],
      "webhookId": "e8084037-495c-4465-b046-de21f3e54cfa",
      "credentials": {
        "gmailOAuth2": {
          "id": "qBqHR4Ctwvw5icyM",
          "name": "gobeamariano@gmail.com Gmail"
        }
      }
    },
    {
      "parameters": {},
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [
        416,
        528
      ],
      "id": "aa69e489-698f-4aef-b92a-605b7d369027",
      "name": "Simple Memory"
    },
    {
      "parameters": {
        "model": "z-ai/glm-4.5-air:free",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenRouter",
      "typeVersion": 1,
      "position": [
        816,
        528
      ],
      "id": "641953fe-caa2-4f8a-a63c-1a559ae3e00d",
      "name": "OpenRouter Chat Model",
      "credentials": {
        "openRouterApi": {
          "id": "xZpAyjDIWgQms8rD",
          "name": "OpenRouter account"
        }
      }
    },
    {
      "parameters": {
        "calendar": {
          "__rl": true,
          "value": "gobeamariano@gmail.com",
          "mode": "list",
          "cachedResultName": "gobeamariano@gmail.com"
        },
        "start": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Start', ``, 'string') }}",
        "end": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('End', ``, 'string') }}",
        "additionalFields": {
          "attendees": [
            "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('attendees0_Attendees', ``, 'string') }}"
          ],
          "description": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Description', ``, 'string') }}",
          "location": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Location', ``, 'string') }}",
          "summary": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Summary', ``, 'string') }}"
        }
      },
      "type": "n8n-nodes-base.googleCalendarTool",
      "typeVersion": 1.3,
      "position": [
        1328,
        528
      ],
      "id": "5ce64fd8-ae90-406b-96bc-a961bcd83b90",
      "name": "Create an event in Google Calendar",
      "credentials": {
        "googleCalendarOAuth2Api": {
          "id": "1tGi4mK9cE2A8FfN",
          "name": "Google Calendar OAuth2 API"
        }
      }
    },
    {
      "parameters": {
        "sendTo": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('To', ``, 'string') }}",
        "subject": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Subject', ``, 'string') }}",
        "message": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Message', ``, 'string') }}",
        "options": {}
      },
      "type": "n8n-nodes-base.gmailTool",
      "typeVersion": 2.2,
      "position": [
        1456,
        528
      ],
      "id": "f7440484-ae28-40aa-b06d-3eda7fe35164",
      "name": "Send a message in Gmail",
      "webhookId": "e4bec25b-f195-4ef3-958e-c6223983d6cb",
      "credentials": {
        "gmailOAuth2": {
          "id": "qBqHR4Ctwvw5icyM",
          "name": "gobeamariano@gmail.com Gmail"
        }
      }
    }
  ],
  "connections": {
    "Chat Trigger": {
      "main": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Google Gemini Model": {
      "ai_languageModel": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Window Buffer Memory": {
      "ai_memory": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "Google Calendar Tool": {
      "ai_tool": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "Gmail Tool": {
      "ai_tool": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "Simple Memory": {
      "ai_memory": [
        [
          {
            "node": "Chat Trigger",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "OpenRouter Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "ai_languageModel",
            "index": 1
          }
        ]
      ]
    },
    "Create an event in Google Calendar": {
      "ai_tool": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "Send a message in Gmail": {
      "ai_tool": [
        [
          {
            "node": "Executive Assistant Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    }
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "db8d09bbe9f4d9927fc4c8a17afea05396ceeae3fa505bb8a1751699adf525b5"
  }
}

```

---

