# 04 — Datos, preparación y linaje

Estado: plan de trabajo. Todavía no se han descargado, perfilado ni cargado registros.

## Fuentes y uso previsto

| Tabla | Uso | Campos clave | Restricción |
| --- | --- | --- | --- |
| `call_center_interactions` | Demanda, motivos, FCR y derivación histórica | `interaction_id`, `customer_id`, `contact_reason`, `reason_category`, `was_resolved`, `was_escalated`, `interaction_date` | FCR histórico no prueba que este prototipo resuelva el caso. |
| `call_transcripts` | Texto del cliente y etiquetas revisadas para intención | `interaction_id`, `customer_text`, `detected_language`, `detected_intents` | No usar `agent_text` ni respuestas del agente como entrada del clasificador. |
| `transactions` | Evidencia factual de la demo | `transaction_id`, `customer_id`, `product_id`, fecha, monto, moneda, comercio, estado | No usar `is_fraud` ni `fraud_score` para decidir un reclamo. |
| `products` | Verificación de pertenencia y producto enmascarado | `product_id`, `customer_id`, `product_number`, `product_status` | Nunca mostrar el número completo de producto. |
| `customers` | Asignación privada de cuentas de prueba y cortes por país/segmento | `customer_id`, `country`, `segment` | No usar documentos nacionales como autenticación. |
| `complaints` | Contexto agregado sobre reclamos y derivación | `complaint_id`, `customer_id`, `category`, `status`, `resolution_days` | No inferir políticas ni prometer plazos a partir de casos históricos. |

El dataset se describe como sintético y con textos en español. Los archivos reales y el permiso para alojar una muestra en Render se verificarán antes de cualquier carga. Los agregados y metadatos de preparación pueden publicarse; filas individuales quedan en almacenamiento privado.

## Perfilado inicial y decisión de producto

Medir por país y mes: volumen de `contact_reason`, proporción transaccional, duración, FCR, seguimiento y derivación. Revisar una muestra estratificada de motivos y transcripciones para definir qué es «consulta de estado/detalles» y qué es «cargo desconocido». Documentar categorías no interpretables, nulos y sesgo por presencia de transcripción. Si la demanda elegida es pequeña, mantener el flujo como demostración acotada y explicar la elección; no inventar una prioridad basada en volumen.

## Preparación repetible

1. Registrar versión del dataset, rutas/particiones leídas, fecha de extracción, conteos y hash del manifiesto; usar credenciales locales fuera de Git.
2. Validar esquema y tipos esperados contra el diccionario; aceptar columnas adicionales sin reinterpretarlas y detener la carga si faltan claves o campos obligatorios.
3. Medir duplicados por clave primaria, nulos, huérfanos de FK, moneda/estado inesperados y desfase entre `transaction_date` y `process_date`. Deduplicar de forma determinista según partición más reciente y documentar empates; no elegir un registro de estado ambiguo sin revisión.
4. Para el análisis longitudinal, partir por tiempo usando `interaction_date` y aislar clientes entre entrenamiento y prueba. Excluir registros duplicados y textos que incluyan la respuesta del agente.
5. Seleccionar una muestra pequeña de clientes con transacciones suficientes para los tres recorridos, países/monedas y estados relevantes. Crear usuarios y contraseñas **nuevos** que mapeen internamente a esos clientes; no usar campos de identidad del dataset como credenciales.
6. Cargar la muestra por conexión privada y cifrada a Postgres después de confirmar términos. Registrar versión, corte y resultados de validación; la carga repetida actualiza por clave sin duplicar. No incluir S3 en el runtime web.

La aplicación solo puede describir los registros presentes en su muestra. Si el corte o una relación entre producto y cliente es incierto, el registro queda fuera de la muestra o se marca no apto para resolución automática.

## Actualización y retención

Para la demo, la muestra es un snapshot versionado. Una nueva carga se ejecuta manualmente con el mismo contrato, compara conteos y claves antes de sustituir la versión activa y permite volver a la versión previa. No hay requisito de streaming. Conservar agregados publicables y manifiestos sin filas individuales en Git; eliminar la muestra privada y las cuentas de prueba al cierre según términos del organizador. La política definitiva de retención para producción queda fuera del prototipo.
