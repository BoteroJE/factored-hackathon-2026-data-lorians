# 01 — Especificación de producto

Estado: propuesta para aprobación del equipo. No hay datos perfilados ni resultados observados.

## Problema e hipótesis

Las preguntas sobre transacciones pueden consumir tiempo de atención aun cuando la respuesta dependa de campos verificables. LATAM Bank aporta interacciones, transcripciones y transacciones para medir esa demanda. **Hipótesis:** un asistente que ubica y explica un registro autorizado puede resolver una parte de esas consultas y entregar mejores antecedentes cuando se necesita un agente. La prevalencia, el tiempo y la calidad de servicio se determinarán durante el perfilado; no se presuponen.

## Usuarios y resultado esperado

- **Cliente de prueba:** consulta una transacción histórica de su cuenta, entiende el estado registrado y ve la fecha de corte de la información.
- **Agente de prueba:** recibe un caso derivado con solicitud, transacción seleccionada, hechos verificados, acciones ejecutadas y preguntas pendientes.
- **Jueces:** prueban tres recorridos desde una URL de Render con cuentas suministradas por el equipo y revisan evidencia de evaluación reproducible.

## Alcance del flujo

1. **Resolución normal:** después de iniciar sesión, el cliente pregunta por fecha, comercio, monto o estado de una transacción. El servicio busca solo transacciones suyas, presenta coincidencias y responde con campos verificados, moneda y fecha de corte. `Approved`, `Declined`, `Pending` y `Reversed` se describen como valores registrados, sin inferir causas o plazos.
2. **Ambigüedad:** si hay varias coincidencias, faltan datos o el mensaje combina solicitudes, el asistente pide una aclaración concreta. Nunca elige arbitrariamente una transacción ni revela registros de otra cuenta.
3. **Intervención humana:** una afirmación de cargo desconocido, posible fraude, reverso solicitado o reclamo requiere derivación. El sistema persiste un resumen y un agente de prueba puede verlo. El bot no adjudica el reclamo.
4. **Fuera de alcance:** transferir dinero, bloquear tarjetas, revertir cargos, determinar fraude, prometer compensación, emitir decisiones crediticias o responder con políticas que no constan en fuentes aprobadas. El asistente explica su límite y deriva cuando corresponde.
5. **Idiomas:** chat en español y portugués. La fuente textual disponible está en español; los casos portugueses se crean y revisan por el equipo, y sus resultados se reportan aparte.

La palabra «estado» significa **estado registrado en la muestra histórica**, cuyo corte más reciente documentado es 2026-06-17; la aplicación no se presentará como banca en tiempo real.

## Criterios de aceptación funcional

| ID | Escenario | Resultado requerido |
| --- | --- | --- |
| P1 | Cliente válido pregunta por una transacción propia inequívoca | Respuesta con monto, moneda, fecha, estado, referencia y fecha de corte verificables. |
| P2 | Dos transacciones coinciden | Pregunta de aclaración que permite seleccionar una, sin exponer datos de terceros. |
| P3 | Cliente menciona un cargo desconocido | Derivación persistida y confirmada; resumen visible para agente. |
| P4 | Cliente pide acción fuera de alcance | Abstención explícita y ruta de agente cuando sea útil. |
| P5 | Sesión vencida o referencia a otro cliente | Sin divulgación ni acción; mensaje seguro y registro del intento. |
| P6 | Fallo de modelo o base de datos | Sin inventar hechos ni confirmar una acción no verificada; respuesta de fallo recuperable. |
| P7 | Mismo flujo en portugués | Respuesta o derivación comprensible; limitación de cobertura documentada. |

## Criterios de entrega

La demo debe mostrar P1, P2 y P3 de principio a fin. La evidencia principal de la entrega será: demanda medida en las tablas pertinentes, ejecución reproducible de bronze/silver/gold con calidad y linaje, un clasificador entrenado y comparado con línea base, y resultados reservados del sistema completo. Publicar también especificación, resultados de seguridad y runbook. Si una puerta de calidad no se cumple, se informa el fallo y no se afirma resolución segura para ese caso. Contención, resolución segura y derivación son resultados distintos.
