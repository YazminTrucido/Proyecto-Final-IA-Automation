# Gestor de Leads con IA y aprobación humana

**Proyecto final · IA Automation · Diplomatura**
Alumna: Yazmin Trucido

Ecosistema de automatización que resuelve de punta a punta la atención de consultas comerciales de **Estudio Lumen**, una agencia ficticia: recibe el lead, lo valida, lo guarda en Airtable, lo hace analizar por la IA (clasificación VIP / Calificado / Frío / Spam, puntaje y borrador de respuesta basado en el catálogo real), pide aprobación humana por email y recién entonces le responde al cliente por Gmail, guardando el Thread ID.

## Enlaces de la entrega

| Qué | Link |
|---|---|
| Base de datos Airtable (modo lectura) | [LINK DE LA VISTA COMPARTIDA](https://airtable.com/appbZ1Pefps0x5rZy/shrhHYOXvNbDbkBOJ) |
| Dashboard de control (KPIs y tasa de error) | [LINK DEL DASHBOARD](https://airtable.com/appbZ1Pefps0x5rZy/shr2dm179kRLeuIdD) |
| Video demo (3 min) | `PEGAR AQUÍ EL LINK DEL VIDEO` |
| Documento de arquitectura (PDF) | [docs/Arquitectura-y-Documentacion.pdf](docs/Documentacion_y_Arquitectura.pdf) |
| Flujo principal n8n (JSON) | [workflow/gestor-leads-ia.json](workflow/gestor-leads-ia.json) |
| Flujo de alertas globales (JSON) | [workflow/alertas-errores-globales.json](workflow/alertas-errores-globales.json) |

## Stack (4 tecnologías)

| Rol | Herramienta | Uso en el proyecto |
|---|---|---|
| Orquestador | **n8n** | Flujo principal de 39 nodos (más 5 notas explicativas) + flujo de alertas de errores |
| Base de datos | **Airtable** | Memoria del sistema: Leads, Servicios, Interacciones y Errores (tablas vinculadas) |
| Procesamiento IA | **Google Gemini** (`gemini-3.5-flash-lite`, por su API compatible con OpenAI) | Clasificación + redacción con prompt dinámico, salida JSON y `max_tokens` limitado. Cambiar a OpenAI es editar `ia_url` y `modelo_ia` en Configuración |
| Canal de salida | **Gmail** | Email de aprobación (HITL), respuesta al cliente y confirmaciones dentro del mismo hilo |

## Cómo funciona

```
Formulario web (webhook)
  → Configuración + normalización
  → ¿Datos completos y válidos? ──no──→ Lead "Datos incompletos" + registro en Errores
  → ¿Lead nuevo? (anti-duplicados) ──no──→ Registro "Lead duplicado"
  → Airtable: crear Lead "Pendiente" + leer catálogo de Servicios
  → IA (Gemini): clasificar y redactar (3 reintentos) ──error──→ Errores + Lead "Error (revisión manual)" + aviso
  → ¿Respuesta válida? ──no──→ (misma ruta de error)
  → ¿Spam? ──sí──→ Lead "Descartado (spam)"
  → Lead "Procesado por IA" + Interacción "Borrador IA"
  → HITL: email con botones Aprobar / Rechazar → Wait (hasta 24 h)
      ├─ aprobar  → Gmail al cliente → Lead "Enviado" + Thread ID + confirmación en el hilo
      ├─ rechazar → Lead "Rechazado por humano" + aviso en el hilo
      └─ vence    → Lead "Sin respuesta (vencido)" + aviso en el hilo
```

El diagrama completo, el recorrido nodo por nodo, el manual de datos, la matriz de costos y la sección de seguridad están en el [PDF](docs/Documentacion_y_Arquitectura.pdf).

## Requisitos de la consigna

- **Disparo automático y sin intervención manual:** trigger por webhook (formulario de n8n); la única intervención humana es el HITL.
- **Rutas de error:** validación de datos faltantes, filtro anti-duplicados, reintentos y salidas de error en la IA y en Gmail, registro de cada error en la tabla `Errores` y workflow global con Error Trigger.
- **Human-in-the-loop:** el flujo se detiene antes de escribirle al cliente y espera el clic del aprobador.
- **Nodos nombrados y sin datos hardcodeados:** todos los valores editables están en el nodo `Configuración del sistema`; el prompt se arma con variables.
- **Campos de estado y relaciones:** `Estado` con 10 valores (Pendiente → Procesado por IA → Esperando aprobación → Aprobado por Humano → Enviado, más estados de error) y 4 tablas vinculadas.
- **Mapeo de la respuesta y Max Tokens:** se lee `choices[0].message.content` y se limita `max_tokens = 600`.
- **Thread ID:** se guarda el `threadId` del email al cliente y las confirmaciones se responden dentro del hilo de aprobación.

### Check de seguridad

1. **Filtro contra bucles infinitos:** sí. Hay un filtro de duplicados por email (24 h), se descartan los remitentes `noreply`/`mailer-daemon` y el trigger es un formulario, no la bandeja de entrada.
2. **Tipos de datos correctos:** sí. El presupuesto se convierte a número y los filtros comparan número con número (`duplicados = 0`, largo del mensaje ≥ 10) con validación estricta.
3. **Prompt dinámico:** sí. Empresa, tono, firma, agenda, umbral VIP y catálogo salen de la configuración y de Airtable.

## Pruebas (test de estrés)

| # | Prueba | Resultado esperado | Evidencia |
|---|---|---|---|
| 1 | Lead VIP completo, aprobado | Estado `Enviado` + Thread ID | `screenshots/01-...` |
| 2 | Consulta vaga sin presupuesto | Categoría Calificado o Frío, enviado | `screenshots/02-...` |
| 3 | Rechazo humano | `Rechazado por humano`, el cliente no recibe nada | `screenshots/03-...` |
| 4 | Datos incompletos (sin email) | `Datos incompletos` + registro en Errores | `screenshots/04-...` |
| 5 | Mismo email repetido | Registro "Lead duplicado" | `screenshots/05-...` |
| 6 | Falla de la API de IA (clave inválida) | "Fallo API IA" + revisión manual + aviso | `screenshots/07-...` |

## Cómo importarlo

1. En n8n: **Create workflow → ⋯ → Import from File** y elegí `workflow/gestor-leads-ia.json`. Repetí con `alertas-errores-globales.json`.
2. Creá las credenciales de **Airtable** (Personal Access Token), **Gemini** (API key de Google AI Studio, como credencial *Header Auth*) y **Gmail** (OAuth2) y asignalas a los nodos.
3. Completá el nodo **Configuración del sistema** (`airtable_base_id`, `email_aprobador`, `link_agenda`).
4. En la configuración del flujo principal, elegí el flujo de alertas como **Error Workflow**.
5. Activá el flujo (Publish) y abrí la URL de producción del formulario.

La base de Airtable se genera con la instrucción de [`database/omni-ai-prompt.md`](database/omni-ai-prompt.md) y el catálogo de ejemplo está en [`database/servicios.csv`](database/servicios.csv).

> El JSON no contiene ninguna API key ni credencial. Las claves quedan guardadas, cifradas, en el gestor de credenciales de n8n.

## Estructura del repositorio

```
├── README.md
├── docs/Arquitectura-y-Documentacion.pdf
├── workflow/
│   ├── gestor-leads-ia.json
│   └── alertas-errores-globales.json
├── database/
│   ├── omni-ai-prompt.md
│   └── servicios.csv
└── screenshots/   (evidencias de las pruebas)
```

Para este proyecto se trabajó con un asistente de IA (Claude) como apoyo.
