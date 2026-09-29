# 06 — Ejecución en seis días y colaboración

Estado: plan de implementación futura. No autoriza código durante la fase actual de especificación.

## Reparto para tres personas

| Persona | Entregable propio | Carpeta futura principal | Dependencia compartida |
| --- | --- | --- | --- |
| A — Datos y evaluación | Perfilado, preparación, muestra privada, clasificador, línea base e informe | `src/data/`, `src/evaluation/`, sus pruebas | Contratos de hechos y etiquetas. |
| B — Asistente y reglas | Orquestación, control de decisiones, herramientas de lectura, derivación y adaptador de modelo | `src/assistant/`, sus pruebas | Contratos de API y acceso a datos. |
| C — Web y plataforma | Chat, vista de agente, autenticación, Render, CI y observabilidad | `src/web/`, `src/platform/`, sus pruebas | Contratos HTTP y sesiones. |

La persona C es integradora y única responsable de los archivos transversales (configuración de proyecto, migraciones compartidas, contratos integrados y workflow de CI). Cambios a contratos requieren acuerdo de A, B y C antes de modificar consumidores. Cada responsable mantiene sus pruebas dentro de su módulo; C mantiene escenarios de extremo a extremo sin que A o B editen sus fixtures directamente.

## Secuencia

| Día | A — Datos | B — Asistente | C — Web y plataforma | Puerta común |
| --- | --- | --- | --- | --- |
| 1 | Perfilado de motivos y calidad; revisar términos de uso | Congelar estados y reglas | Congelar API, sesiones y boceto visual | Aprobar `03-contratos.md`; decidir proveedor de modelo y acceso a datos. |
| 2 | Pipeline y muestra de desarrollo | Herramientas de lectura y reglas | Login, sesión y chat base | Contratos de consumidor con datos ficticios. |
| 3 | Clasificador y línea base | Contexto, aclaración y derivación | Vista de agente y persistencia | Recorrido normal integrado. |
| 4 | Etiquetas y conjunto reservado | Adaptador de modelo y fallos | Diseño visual, trazas y CI | Ambigüedad y derivación integradas. |
| 5 | Medición y error analysis | Ajustes según validación | Despliegue de ensayo en Render | Congelar versiones; ejecutar conjunto reservado. |
| 6 | Informe final | Correcciones críticas | Despliegue final, cuentas y runbook | Ensayo con español/portugués y revisión de secretos. |

Si el tiempo se reduce, conservar el flujo completo, autenticación, evaluación y derivación. Reducir primero adornos visuales o tamaño de la muestra online; no presentar una ruta incompleta como terminada.

## Integración Git

El remoto del equipo es `https://github.com/BoteroJE/factored-hackathon-2026-data-lorians`. El directorio actual no tenía Git al iniciar esta documentación y el remoto se reportó vacío. La base documental se publicará primero; después, cada persona crea una rama corta desde `main`, abre PR por módulo y sincroniza diariamente. Requerir revisión cruzada y CI verde antes de merge. Evitar PR que renombren carpetas ajenas o modifiquen a la vez archivos transversales.

El CI futuro tendrá puertas separadas: contrato y formato, pruebas de módulo, escenarios integrados, escaneo de secretos y build. Los datos de evaluación individuales y los artefactos privados no entran en PR. Los cambios de pruebas comunes los coordina C para evitar conflictos entre ramas.

## Entrega a jueces

Publicar URL de Render, cuentas de cliente y agente de prueba por canal permitido, tres guiones de demo, explicación de corte de datos, informe de evaluación, diagrama, instrucciones reproducibles y límites. La URL se prueba desde una sesión de navegador sin credenciales de desarrollador. Registrar versiones exactas del despliegue y fecha de la muestra.
