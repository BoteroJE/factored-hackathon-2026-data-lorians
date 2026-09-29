# 05 — Evaluación y evidencia

Estado: protocolo previo a resultados. Todos los valores siguientes son tamaños de trabajo propuestos, no mediciones.

## Etiquetas, modelo y línea base

- Dos personas revisarán el mapeo de `contact_reason`, `detected_intents` y una muestra de `customer_text` hacia `transaction_lookup`, `unknown_charge`, `unsupported` y `unclear`. Los campos de intención suministrados son candidatos a etiqueta, no verdad de referencia sin revisión. Registrar acuerdo, desacuerdos y resolución antes de entrenar.
- La entrada del modelo usa solo `customer_text` limpiado; no contiene respuesta del agente, identificadores ni hechos posteriores a la interacción. La línea base usa reglas de palabras clave versionadas. La propuesta es TF-IDF de n-gramas de caracteres más regresión logística con probabilidades calibradas. La clasificación en portugués puede requerir abstención o el modelo conversacional; no se afirmará generalización del clasificador español sin evidencia.
- Dividir entrenamiento, validación y prueba por cliente y cronología, dejando el periodo más reciente para prueba. Deduplicar antes del split; fijar semillas, versiones de gold y de etiquetas. Ajustar umbral con validación para limitar cargos desconocidos atendidos como consultas normales; congelarlo antes de evaluar el conjunto reservado.
- Medir F1 macro, precisión y recall por intención, matriz de confusión, calibración y curva cobertura/error al variar el umbral. Examinar por separado falsos negativos de `unknown_charge`, ambigüedad, país y segmento. Registrar en MLflow parámetros, versión de datos, métricas, matriz y artefacto con hash. [MLflow en Databricks](https://docs.databricks.com/aws/en/mlflow).

## Casos de extremo a extremo

Preparar un conjunto reservado de **80 casos** con respuestas esperadas y evidencia de referencia: 50 casos españoles derivados de registros del dataset y revisados manualmente, 15 casos portugueses creados y revisados por el equipo, y 15 casos adversariales creados por el equipo. Incluir consultas normales, ambigüedad, cargo desconocido, solicitud fuera de alcance, datos incorrectos o ausentes, sesión vencida, acceso ajeno, inyección en mensaje o datos, caída de modelo/base y referencias multimoneda. Un segundo revisor comprobará al menos 20% de las etiquetas; los desacuerdos se resolverán antes de ejecutar la comparación.

Comparar línea base y propuesta con **los mismos 80 casos**, la misma versión gold y las mismas reglas de autorización. La línea base usa palabras clave y respuestas plantilladas; la propuesta usa el clasificador y el modelo de lenguaje para redactar. La evaluación del clasificador se reporta aparte para atribuirle sus errores y mejoras. Congelar versiones de prompts, modelo, clasificador y umbral. Repetir tres veces los casos que utilicen generación variable y reportar variación. No añadir casos al conjunto reservado después de ver resultados; los nuevos fallos irán a un conjunto de regresión separado.

## Métricas y denominadores

| Métrica | Definición |
| --- | --- |
| Resolución automática segura | Casos elegibles con resultado correcto, sustentado y conforme a reglas, sin agente / todos los casos de prueba dentro de alcance. |
| Intento de automatización | Casos en que el sistema intentó responder / todos los casos dentro de alcance. |
| Contención | Casos sin derivación / todos los casos; se reporta separada de resolución. |
| Calidad de derivación | Derivaciones correctas con paquete útil / casos que requerían agente; contar derivaciones omitidas e innecesarias. |
| Resultado inseguro | Divulgación no autorizada, acción no permitida o resultado materialmente incorrecto; reportar conteos y denominadores. |
| Eficiencia | Latencia de extremo a extremo p50/p95 y costo estimado por intento y por resolución segura, con supuestos de precios y carga. |

Desglosar resultados por idioma, país y segmento cuando haya muestra suficiente; mostrar tamaño de cada grupo y no interpretar diferencias pequeñas como conclusiones firmes. Un cero observado en 80 casos no demuestra riesgo cero. Si no hay resoluciones seguras, el costo por resolución se informa como «no definido».

## Puertas de entrega y reporte

La demo debe pasar los tres recorridos P1–P3 y no presentar divulgaciones o acciones no autorizadas en la batería ejecutada. El pipeline debe demostrar esquema, reconciliación de conteos, cuarentena, idempotencia y corrección de una llegada tardía. Cualquier fallo inseguro bloquea la afirmación de «resolución segura» hasta corregir y repetir los casos afectados. El informe incluye todos los fallos, tamaño y mezcla de muestras, ejemplos anonimizados, versiones, latencia, costos estimados y límites de datos. Un juez automático de respuestas solo puede complementar la rúbrica si se valida una muestra contra juicio humano o determinista.
