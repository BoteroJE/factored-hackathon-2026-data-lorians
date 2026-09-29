# 07 — Decisiones, dependencias y registro

Este documento separa decisiones ya acordadas de entradas que aún deben confirmarse. Ninguna pendiente se suplirá inventando resultados o permisos.

## Acordadas

| Tema | Decisión |
| --- | --- |
| Método | Spec Driven Development: especificar y aprobar contratos antes de código. |
| Flujo | Consultas históricas de transacciones, aclaración y derivación de cargos desconocidos. |
| Demo | Chat web cuidado y funcional, accesible a jueces en Render. |
| Identidad | Cuentas de prueba con sesión y roles de cliente/agente. |
| Plataforma de datos | Databricks Free Edition para seis tablas completas, pipeline Delta bronze/silver/gold, linaje y MLflow. Una pipeline y un job de entrenamiento, fuera de la ruta online. |
| Datos online | Muestra gold versionada y clasificador aprobado en Postgres administrado; demo pública en Render independiente de las cuotas de Databricks. |
| ML | Un clasificador entrenado y profundamente evaluado frente a reglas de palabras clave. |
| Equipo | A ingeniería de datos, B ML/evaluación, C aplicación/plataforma, con PR pequeños. |
| Proveedor IA | Configurable; la elección concreta fue pospuesta expresamente. |

## Pendientes antes de implementar o desplegar

| ID | Decisión o comprobación | Responsable y momento | Regla mientras siga pendiente |
| --- | --- | --- | --- |
| D1 | Revisar las condiciones concretas de Databricks Free Edition, incluida su cláusula sobre entrenamiento con datos cargados, y las reglas del organizador. El usuario indicó que se pueden usar proveedores externos; no apareció una licencia específica en los documentos suministrados ni en la raíz de S3. | A y equipo, día 1, antes de transferir filas. | Mantener inventario y contratos sin carga externa hasta completar la revisión. |
| D2 | Elegir proveedor, modelo, clave, presupuesto y límites de uso de la API de lenguaje. | B y equipo, día 1; usuario indicó resolverlo después. | Adaptador documentado, sin llamadas reales ni métricas de costo atribuidas. |
| D3 | Confirmar plan y acceso de Render Postgres/Web, dominio de demo y quién administra secretos. | C, día 1, antes de despliegue. | Mantener solo configuración de diseño. |
| D4 | Revisar motivos de contacto y calidad de etiquetas en datos reales. | A y B, días 1–2. | La importancia del problema sigue siendo hipótesis. |
| D5 | Seleccionar cuentas y registros de demo tras validación de calidad, con usuario/agente de prueba. | A y C, día 2. | No publicar filas ni credenciales. |
| D6 | Fijar umbral del clasificador con validación; congelarlo antes del conjunto reservado. | B, día 4. | Baja confianza produce aclaración o derivación. |
| D7 | Confirmar cuota de cómputo y espacio de Databricks Free Edition con el primer lote mensual y la proyección de las seis tablas. | A, día 1–2. | Procesar secuencialmente y no depender de Databricks para atender a los jueces. |

## Registro de cambios de decisión

Cuando cambie una decisión, anotar fecha, responsables, motivo, documentos afectados y efecto en pruebas y demo. Las métricas medidas se registran en el informe de evaluación, no en esta tabla de decisiones.
