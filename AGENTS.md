# Instrucciones de trabajo — Data Lorians

## Fase actual

Este repositorio está **solo en planificación**. No crear aplicación, scripts de ingesta, modelos, pruebas ejecutables, infraestructura ni despliegues hasta que el equipo autorice explícitamente iniciar la implementación. Durante esta fase, editar únicamente documentación de diseño y coordinación.

## Fuentes y verdad

- Leer primero `README.md` y los documentos de `docs/plan/`.
- Las reglas del reto están en `docs/Factored AI & Data Hackathon 2026.md`; el esquema publicable está en `docs/LATAM_Bank_Data_Dictionary_Public.md`.
- El diccionario original `docs/LATAM_Bank_Complete_Data_Dictionary.md` contiene credenciales de acceso y está excluido de Git. Nunca copiar sus claves, valores de secretos o contenido de autenticación a commits, PR, logs, prompts o servicios externos.
- El dataset es sintético, pero sus condiciones de uso y las de recursos externos siguen siendo vinculantes. Verificar permisos antes de subir una muestra a Render o enviar datos a un modelo.
- Distinguir en toda documentación datos observados, hipótesis, datos sintéticos del organizador y casos creados por el equipo. No inventar métricas ni prometer resultados de producción.

## Desarrollo guiado por especificaciones

1. Cambiar primero la especificación de producto si cambia el alcance.
2. Revisar los contratos de `03-contratos.md` con los tres responsables antes de implementar interfaces compartidas.
3. Registrar decisiones que cambien seguridad, datos, evaluación o despliegue en `07-decisiones.md`.
4. Cada PR debe citar el criterio de aceptación que satisface y explicar evidencia, limitaciones y efecto en otros módulos.
5. Los cambios de contratos requieren aprobación de los tres responsables. Evitar que una rama modifique archivos de propiedad ajena sin coordinación previa.

## Propiedad para tres personas

| Responsable | Propiedad principal cuando empiece la implementación | Pruebas de su módulo |
| --- | --- | --- |
| Datos y evaluación | Ingesta, calidad, muestra privada, clasificación y evaluación | Datos y evaluación |
| Asistente y reglas | Orquestación, autorización de herramientas, estados, derivación y adaptador de modelo | Asistente y reglas |
| Interfaz y plataforma | Web, sesiones, despliegue y observabilidad | Interfaz y plataforma |

El responsable de interfaz y plataforma integra los archivos transversales y PR hacia `main`. La estructura exacta de código se creará solo al abrir la fase de implementación; los límites de módulo y contratos ya están definidos en la documentación.

## Git y CI al iniciar implementación

- Una rama corta por cambio y PR pequeño hacia `main`; nunca desarrollar tres módulos en un mismo archivo compartido.
- Pruebas de módulo junto a su propietario; escenarios integrados y datos de prueba comunes en una única ubicación cuya propiedad es de integración.
- El CI validará contratos, pruebas de módulo, escenarios integrados y ausencia de secretos antes de permitir merge. No regenerar snapshots globales desde ramas paralelas.
- Los datos crudos, credenciales, archivos `.env`, muestras privadas y artefactos de evaluación con registros individuales nunca se versionan.

## Reglas del asistente futuro

- Una sesión confiable, no un identificador de cliente proporcionado en el chat, determina qué registros se pueden consultar.
- Toda autorización y toda decisión de ejecutar una acción se aplica en el servicio; el modelo solo interpreta lenguaje y redacta usando hechos permitidos.
- Una respuesta de estado cita el registro verificado. La derivación se anuncia solo tras recibir confirmación de persistencia. Ante falta de datos, permisos, tiempo de sesión o fallos de herramientas, abstenerse o derivar según el contrato.
- Nunca afirmar que se resolvió fraude, se revirtió un cargo o se movió dinero.
