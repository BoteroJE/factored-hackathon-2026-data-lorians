# 04 — Datos, preparación y linaje

Estado: plan de trabajo. Se consultó el inventario de objetos S3, pero todavía no se han descargado, perfilado ni cargado registros.

## Fuentes y uso previsto

| Tabla | Uso | Campos clave | Restricción |
| --- | --- | --- | --- |
| `call_center_interactions` | Demanda, motivos, FCR y derivación histórica | `interaction_id`, `customer_id`, `contact_reason`, `reason_category`, `was_resolved`, `was_escalated`, `interaction_date` | FCR histórico no prueba que este prototipo resuelva el caso. |
| `call_transcripts` | Texto del cliente y etiquetas revisadas para intención | `interaction_id`, `customer_text`, `detected_language`, `detected_intents` | No usar `agent_text` ni respuestas del agente como entrada del clasificador. |
| `transactions` | Evidencia factual de la demo | `transaction_id`, `customer_id`, `product_id`, fecha, monto, moneda, comercio, estado | No usar `is_fraud` ni `fraud_score` para decidir un reclamo. |
| `products` | Verificación de pertenencia y producto enmascarado | `product_id`, `customer_id`, `product_number`, `product_status` | Nunca mostrar el número completo de producto. |
| `customers` | Asignación privada de cuentas de prueba y cortes por país/segmento | `customer_id`, `country`, `segment` | No usar documentos nacionales como autenticación. |
| `complaints` | Contexto agregado sobre reclamos y derivación | `complaint_id`, `customer_id`, `category`, `status`, `resolution_days` | No inferir políticas ni prometer plazos a partir de casos históricos. |

El dataset se describe como sintético y con textos en español. El usuario autorizó considerar proveedores cloud; antes de cargar filas, el equipo revisará las condiciones concretas de Databricks Free Edition y del modelo de lenguaje. Los agregados y metadatos de preparación pueden publicarse; filas individuales quedan en almacenamiento privado.

El inventario de S3 confirmó 1.097 CSV diarios para cada una de `transactions`, `call_center_interactions`, `call_transcripts` y `complaints`, unos **1,03 GiB** en conjunto. `customers.csv` y `products.csv` suman unos **0,11 GiB**. Son tamaños de objetos, no conteos de filas observados ni tamaño final de Delta. La carga planificada procesa todas las filas de estas seis tablas; la aplicación recibe solo una muestra apta.

## Perfilado inicial y decisión de producto

Medir por país y mes: volumen de `contact_reason`, proporción transaccional, duración, FCR, seguimiento y derivación. Revisar una muestra estratificada de motivos y transcripciones para definir qué es «consulta de estado/detalles» y qué es «cargo desconocido». Documentar categorías no interpretables, nulos y sesgo por presencia de transcripción. Si la demanda elegida es pequeña, mantener el flujo como demostración acotada y explicar la elección; no inventar una prioridad basada en volumen.

## Preparación repetible por capas

1. **Inventario y staging:** enumerar las seis tablas, guardar clave S3, partición, tamaño, fecha de modificación y SHA-256 en `source_manifest`. Descargar con acceso de solo lectura a un directorio privado ignorado por Git. Agrupar CSV diarios por tabla y mes en Parquet, conservando `source_key`, `process_date` y `batch_id` por fila. Subir los Parquet a un volumen administrado de Unity Catalog; el límite de la interfaz web es 5 GB por archivo, por encima del tamaño de los lotes previstos. [Volúmenes de Databricks](https://docs.databricks.com/aws/en/volumes/volume-files).
2. **Bronze Delta:** ingerir todos los campos fuente como snapshot reproducible, con metadatos de archivo y tiempo de carga. Verificar que el conjunto de hashes coincide con el manifiesto. Columnas nuevas se capturan y reportan; la ausencia de una clave o columna obligatoria detiene el lote.
3. **Silver Delta:** convertir tipos y fechas, normalizar estados y monedas, deduplicar por clave de negocio y validar FK. Una repetición exacta de clave y contenido se colapsa; versiones con fechas distintas siguen una precedencia documentada por partición y actualización; empates contradictorios van a cuarentena. Medir nulos, duplicados, huérfanos y desfases de partición sin ocultarlos en los agregados.
4. **Gold Delta:** producir `contact_demand_monthly`, `intent_candidates` y `transaction_snapshot` según `03-contratos.md`. Para candidatos ML, limpiar `customer_text` y eliminar documentos, números de producto, nombres y otros identificadores antes de entrenar. B mantiene las etiquetas revisadas en su propio módulo y las une por `interaction_id`; el snapshot excluye uniones o estados dudosos e incluye `data_as_of`.
5. **Publicación:** elegir clientes con transacciones suficientes para los recorridos, distintos países/monedas y estados pertinentes. Crear usuarios y contraseñas **nuevos** ligados a esos clientes; los campos de identidad del dataset no autentican. Importar por conexión cifrada a Postgres una versión gold pequeña y verificada, manteniendo la anterior hasta superar las comprobaciones.

Cada capa produce un informe por `batch_id` con filas leídas, aceptadas, duplicadas, en cuarentena y publicadas. El linaje de Unity Catalog muestra transformaciones entre tablas; el manifiesto enlaza bronze con el objeto original. No se necesita conectar Databricks directamente al S3 del organizador: Free Edition restringe el acceso saliente y el staging local evita depender de esa conectividad. [Límites de Free Edition](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations), [linaje de Unity Catalog](https://docs.databricks.com/aws/en/data-governance/unity-catalog/data-lineage).

La aplicación solo puede describir los registros presentes en su muestra. Si el corte o una relación entre producto y cliente es incierto, el registro queda fuera de la muestra o se marca no apto para resolución automática.

## Actualización y retención

Para la demo, la muestra es un snapshot versionado. Una nueva carga se ejecuta por manifiesto, omite objetos con hash ya procesado y permite volver a una versión previa si falla calidad o publicación. Un fixture con una partición antigua llegada tarde demuestra corrección incremental; repetirlo no puede cambiar conteos. No hay requisito de streaming. Conservar agregados publicables y manifiestos sin filas individuales en Git; eliminar la muestra privada y las cuentas de prueba al cierre según términos del organizador. La política definitiva de retención para producción queda fuera del prototipo.
