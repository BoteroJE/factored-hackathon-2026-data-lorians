# 07 — Decisiones, dependencias y registro

Este documento separa decisiones ya acordadas de entradas que aún deben confirmarse. Ninguna pendiente se suplirá inventando resultados o permisos.

## Acordadas

| Tema | Decisión |
| --- | --- |
| Método | Spec Driven Development: especificar y aprobar contratos antes de código. |
| Flujo | Consultas históricas de transacciones, aclaración y derivación de cargos desconocidos. |
| Demo | Chat web cuidado y funcional, accesible a jueces en Render. |
| Identidad | Cuentas de prueba con sesión y roles de cliente/agente. |
| Datos online | Muestra autorizada del dataset, privada en Postgres administrado; análisis offline más amplio. |
| Equipo | Tres módulos con propietarios separados y PR pequeños. |
| Proveedor IA | Configurable; la elección concreta fue pospuesta expresamente. |

## Pendientes antes de implementar o desplegar

| ID | Decisión o comprobación | Responsable y momento | Regla mientras siga pendiente |
| --- | --- | --- | --- |
| D1 | Confirmar condiciones del organizador para alojar una muestra en Render y enviar campos mínimos a un modelo externo. | A, día 1, antes de copiar datos. | No cargar filas ni enviarlas a APIs externas. |
| D2 | Elegir proveedor, modelo, clave, presupuesto y límites de uso de la API. | B y equipo, día 1; usuario indicó resolverlo después. | Adaptador documentado, sin llamadas reales ni métricas de costo atribuidas. |
| D3 | Confirmar plan y acceso de Render Postgres/Web, dominio de demo y quién administra secretos. | C, día 1, antes de despliegue. | Mantener solo configuración de diseño. |
| D4 | Revisar motivos de contacto y calidad de etiquetas en datos reales. | A, día 1. | La importancia del problema sigue siendo hipótesis. |
| D5 | Seleccionar cuentas y registros de demo tras validación de calidad, con usuario/agente de prueba. | A y C, día 2. | No publicar filas ni credenciales. |
| D6 | Fijar umbral del clasificador con validación; congelarlo antes del conjunto reservado. | A y B, día 4. | Baja confianza produce aclaración o derivación. |

## Registro de cambios de decisión

Cuando cambie una decisión, anotar fecha, responsables, motivo, documentos afectados y efecto en pruebas y demo. Las métricas medidas se registran en el informe de evaluación, no en esta tabla de decisiones.
