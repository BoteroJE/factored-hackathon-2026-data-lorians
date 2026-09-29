# Data Lorians — Factored AI & Data Hackathon 2026

Este repositorio está en **fase de especificación**. Todavía no contiene un sistema de IA ni una demo funcional. La propuesta es un asistente de consultas sobre transacciones de LATAM Bank, con respuestas verificadas para clientes autenticados y derivación de posibles cargos desconocidos. La evidencia principal vendrá de un pipeline de datos reproducible y de un clasificador de intención evaluado frente a una línea base.

## Documentos de trabajo

1. [Producto y criterios de aceptación](docs/plan/01-producto.md)
2. [Arquitectura y seguridad](docs/plan/02-arquitectura.md)
3. [Contratos e interfaces](docs/plan/03-contratos.md)
4. [Datos y preparación](docs/plan/04-datos.md)
5. [Evaluación](docs/plan/05-evaluacion.md)
6. [Ejecución en seis días](docs/plan/06-entrega.md)
7. [Decisiones y dependencias pendientes](docs/plan/07-decisiones.md)

Las reglas de colaboración para personas y agentes están en [AGENTS.md](AGENTS.md). Las fuentes suministradas por la organización se conservan en `docs/`; el [diccionario publicable](docs/LATAM_Bank_Data_Dictionary_Public.md) omite la sección de credenciales.

## Estado y límites

- Arquitectura prevista: S3 del organizador → preparación y carga por lotes → Databricks Free Edition (bronze/silver/gold, calidad, linaje y MLflow) → muestra y clasificador versionados en Postgres → demo web en Render.
- Hipótesis de producto, arquitectura y contratos: definidos para revisión. Magnitud real del problema, calidad de etiquetas, cuota disponible de Databricks y proveedor de lenguaje: pendientes de validación.
- No hay métricas observadas, evaluación ejecutada, cuentas de prueba, despliegue ni código de aplicación.

Antes de implementar, el equipo debe cerrar las decisiones de `07-decisiones.md` y aprobar los contratos compartidos. Ningún número estimado en estos documentos debe presentarse como resultado medido.
