# 03 — Contratos compartidos

Estado: versión 1 de diseño. Cualquier cambio requiere revisión de los tres responsables antes de escribir código consumidor.

## Convenciones

- JSON UTF-8; fechas ISO 8601 con zona horaria, importes como cadenas decimales y monedas ISO 4217.
- La identidad viene de la sesión, nunca del cuerpo de la solicitud. El navegador envía token CSRF en solicitudes mutables autenticadas.
- Los errores tienen `{ "error": { "code": "...", "message": "..." }, "trace_id": "..." }`; el mensaje público no contiene datos internos.
- Los identificadores del ejemplo que siga la implementación serán creados por el equipo y no representan registros reales del dataset.

## API web

| Método y ruta | Solicitud | Respuesta y permisos |
| --- | --- | --- |
| `POST /api/login` | `username`, `password` | Crea cookie de sesión; devuelve rol y token CSRF. Límite de intentos. |
| `POST /api/logout` | Token CSRF | Invalida sesión y cookie. |
| `GET /api/session` | Cookie | Rol, caducidad y perfil de demostración sin datos sensibles. |
| `POST /api/chat` | `message` y `conversation_id` opcional | Cliente autenticado; devuelve respuesta tipada descrita abajo. |
| `GET /api/handoffs` | Cookie | Agente autenticado; lista resumida de casos. |
| `GET /api/handoffs/{handoff_id}` | Cookie | Agente autenticado; paquete de derivación. |
| `GET /healthz` | Ninguna | Estado técnico sin datos de clientes. |

`POST /api/chat` devuelve `conversation_id`, `assistant_message`, `outcome`, `evidence`, `handoff_id` opcional, `data_as_of` opcional y `trace_id`. `outcome` es exactamente `resolved`, `clarify`, `handoff` o `abstain`. Cada elemento de `evidence` incluye `source_type`, `source_ref` y campos mostrados; una respuesta factual requiere al menos una evidencia de transacción autorizada. El cliente no envía instrucciones de herramienta ni cambia el resultado declarado por la API.

## Contratos internos

| Contrato | Campos mínimos | Invariante |
| --- | --- | --- |
| `SessionContext` | `session_id`, `role`, `customer_id?`, `expires_at` | `customer_id` se obtiene del servidor; un agente no hereda la identidad de un cliente. |
| `IntentDecision` | `intent`, `confidence`, `language`, `classifier_version` | Intenciones: `transaction_lookup`, `unknown_charge`, `unsupported`, `unclear`. |
| `TransactionQuery` | `customer_id`, filtros de fecha/monto/comercio/referencia y límite | La capa de datos impone pertenencia y máximo de resultados. |
| `TransactionFact` | ID, fecha, tipo, monto, moneda, comercio opcional, estado, producto enmascarado, corte | Solo campos aprobados para cliente; el valor ausente se representa como ausente, sin inferencia. |
| `Decision` | `outcome`, `reason_code`, evidencias permitidas, pregunta pendiente | Lo produce la orquestación de reglas, nunca el modelo. |
| `HandoffRecord` | ID, cliente, conversación, solicitud, hechos, evidencias, acciones confirmadas, preguntas pendientes, fecha | Persistir antes de confirmar al cliente. |
| `ModelDraft` | idioma, texto propuesto, referencias usadas, versión | Se valida contra la decisión y los hechos permitidos antes de mostrarse. |

La interfaz del adaptador de lenguaje recibe instrucciones de sistema versionadas, idioma, mensaje del usuario y hechos mínimos autorizados; devuelve `ModelDraft`, uso y estado. El proveedor/modelo se selecciona por configuración, sin cambiar contratos del resto del servicio.

## Contratos de datos y modelo

| Entrega de A o B | Campos mínimos | Consumidor y regla |
| --- | --- | --- |
| `source_manifest` | `source_key`, `source_size`, `source_sha256`, `process_date`, `ingested_at`, `batch_id` | A conserva el mapeo de cada fila a su objeto S3; una misma clave y hash no se carga dos veces. |
| `gold.contact_demand_monthly` | `month`, `country`, `contact_reason`, `interactions`, `fcr_known`, `fcr_count`, `escalated_count`, `batch_id` | B usa agregados para justificar alcance; ningún nulo de FCR se trata como falso. |
| `gold.intent_candidates` | `interaction_id`, `customer_id`, `interaction_date`, `customer_text_clean`, `contact_reason`, `detected_intents`, `batch_id` | A entrega candidatos sin imponer etiquetas de verdad. B usa solo texto del cliente como rasgo. |
| `ml.reviewed_intent_examples` | `interaction_id`, `reviewed_label`, `label_version`, `review_status`, `source_batch_id` | B posee etiquetas revisadas y su unión con gold; `contact_reason` y `detected_intents` no entran como rasgos. |
| `gold.transaction_snapshot` | `transaction_id`, `customer_id`, `product_id`, `transaction_date`, `amount`, `currency`, `merchant_name?`, `transaction_status`, `data_as_of`, `batch_id` | C importa solo filas aptas y comprueba propiedad de producto y transacción en cada consulta. |
| `ClassifierRelease` | `model_version`, `artifact_sha256`, `training_batch_id`, `label_version`, `threshold`, `metrics_ref`, `created_at` | B publica una versión aprobada; C verifica hash y carga esa versión de la base privada al iniciar. |

Todos los gold incluyen esquema versionado, recuento, fecha de corte y manifiesto de entrada. Los identificadores y textos individuales quedan fuera del repositorio público. Cambiar nombres, tipos o semántica exige actualizar este contrato antes de cambiar productores o consumidores.

## Errores y estados

- `401 SESSION_REQUIRED` o `SESSION_EXPIRED`: no consultar datos y pedir nuevo ingreso.
- `403 FORBIDDEN`: recurso de otro cliente o rol insuficiente; no revelar si existe.
- `422 INVALID_REQUEST`: entrada inválida o demasiado extensa.
- `429 RATE_LIMITED`: límite de acceso o de chat.
- `503 DEPENDENCY_UNAVAILABLE`: base o modelo indisponible; jamás declarar éxito de una acción fallida.
- Una búsqueda sin coincidencias produce `clarify` o `abstain`, no inventa transacciones.
- Una transacción `Pending` se describe solo como pendiente en el corte; no se predice fecha de aprobación.
- En repetición de un mensaje por fallo de red, la derivación usa clave idempotente por conversación y solicitud para no crear duplicados.
