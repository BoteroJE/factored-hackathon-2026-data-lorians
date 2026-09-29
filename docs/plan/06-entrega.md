# 06 — Ejecución en seis días y colaboración

Estado: plan de implementación futura. No autoriza código durante la fase actual de especificación.

## Reparto para tres personas

| Persona | Entregable propio | Carpeta futura principal | Dependencia compartida |
| --- | --- | --- | --- |
| A — Ingeniería de datos | Inventario S3, staging mensual, pipeline Delta bronze/silver/gold, calidad, linaje y publicación de snapshot | `src/data/`, sus pruebas | Contratos gold y manifiesto. |
| B — ML y evaluación | Anotaciones, línea base, clasificador, calibración, MLflow e informe comparativo | `src/ml/`, `src/evaluation/`, sus pruebas | Gold de candidatos y contrato del modelo. |
| C — Aplicación y plataforma | Chat, reglas, autenticación, Postgres, Render, CI y observabilidad | `src/app/`, `src/platform/`, sus pruebas | Snapshot y artefacto versionados. |

La persona C es integradora y única responsable de los archivos transversales (configuración de proyecto, migraciones compartidas, contratos integrados y workflow de CI). A posee las pruebas de contrato y actualización de datos; B posee las de ML y los casos reservados; C posee las de API y seguridad. Cambios a contratos requieren acuerdo de A, B y C antes de modificar consumidores. Los datos de prueba comunes tienen un único propietario de integración.

## Secuencia

| Día | A — Ingeniería de datos | B — ML y evaluación | C — Aplicación y plataforma | Puerta común |
| --- | --- | --- | --- | --- |
| 1 | Confirmar cuenta/condiciones de Databricks; inventario S3 y manifiesto | Protocolo de etiquetas y línea base | Congelar API, seguridad y boceto visual | Aprobar gold y `ClassifierRelease`; medir cuota disponible. |
| 2 | Staging mensual y carga a volumen; bronze | Revisar etiquetas y crear split sin fuga | Login, sesión y chat base | Contratos con datos creados por el equipo. |
| 3 | Silver, controles y cuarentena | Entrenar primer modelo y registrar MLflow | Lectura autorizada y derivación | Reconciliación de datos y recorrido normal integrado. |
| 4 | Gold, prueba de llegada tardía y snapshot | Calibrar en validación y publicar modelo | Vista de agente, diseño y trazas | Congelar versiones de datos, modelo y API. |
| 5 | Perfil de demanda y linaje para el informe | Ejecutar conjunto reservado y análisis de errores | Despliegue de ensayo en Render | Comparación completa con línea base. |
| 6 | Evidencia de calidad y runbook de datos | Informe ML, métricas y límites | Despliegue final y cuentas | Ensayo en ambos idiomas, secretos, 4–6 láminas y video. |

Si el tiempo se reduce, conservar el flujo completo, controles de datos, clasificación evaluada, autenticación y derivación. Reducir primero adornos visuales o tamaño de la muestra online; las seis tablas relevantes siguen recorriendo el pipeline y se reporta cualquier límite de cuota alcanzado.

## Integración Git

El remoto del equipo es `https://github.com/BoteroJE/factored-hackathon-2026-data-lorians`. La base documental inicial ya está en `main`. Al iniciar implementación, cada persona crea una rama corta desde `main`, abre PR por módulo y sincroniza diariamente. Requerir revisión cruzada y CI verde antes de merge. Evitar PR que renombren carpetas ajenas o modifiquen a la vez archivos transversales.

El CI futuro tendrá puertas separadas: contrato y formato, pruebas de módulo, escenarios integrados, escaneo de secretos y build. Los datos de evaluación individuales y los artefactos privados no entran en PR. Los cambios de pruebas comunes los coordina C para evitar conflictos entre ramas.

## Entrega a jueces

Publicar URL de Render, cuentas de cliente y agente de prueba por canal permitido, tres guiones de demo, explicación de corte de datos, informe de evaluación, diagrama, controles y linaje del pipeline, instrucciones reproducibles y límites. Preparar además la presentación de 4–6 láminas y el video solicitados por la [página oficial del hackatón](https://www.factored.ai/careers/ai-data-hackathon). La URL se prueba desde una sesión de navegador sin credenciales de desarrollador. Registrar versiones exactas del despliegue, modelo y muestra.
