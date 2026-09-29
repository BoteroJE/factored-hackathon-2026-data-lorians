# 03 — Contratos compartidos

Estado: versión 0 de diseño. Cualquier cambio requiere revisión de los tres responsables antes de escribir código consumidor.

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

La interfaz del adaptador de modelo recibe instrucciones de sistema versionadas, idioma, mensaje del usuario y hechos mínimos autorizados; devuelve `ModelDraft`, uso y estado. El proveedor/modelo se selecciona por configuración, sin cambiar contratos del resto del servicio.

## Errores y estados

- `401 SESSION_REQUIRED` o `SESSION_EXPIRED`: no consultar datos y pedir nuevo ingreso.
- `403 FORBIDDEN`: recurso de otro cliente o rol insuficiente; no revelar si existe.
- `422 INVALID_REQUEST`: entrada inválida o demasiado extensa.
- `429 RATE_LIMITED`: límite de acceso o de chat.
- `503 DEPENDENCY_UNAVAILABLE`: base o modelo indisponible; jamás declarar éxito de una acción fallida.
- Una búsqueda sin coincidencias produce `clarify` o `abstain`, no inventa transacciones.
- Una transacción `Pending` se describe solo como pendiente en el corte; no se predice fecha de aprobación.
- En repetición de un mensaje por fallo de red, la derivación usa clave idempotente por conversación y solicitud para no crear duplicados.
