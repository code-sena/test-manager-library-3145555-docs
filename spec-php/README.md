# Manager Library — Spec PHP/Laravel

Esta carpeta es la referencia objetivo para implementar **Manager Library** con PHP/Laravel y React.

## Reglas de esta versión

- El nombre del producto es **Manager Library**.
- El alcance es exactamente el definido en [`spec.md`](spec.md); no se agregan funcionalidades ni se reabren exclusiones.
- Las historias, criterios de aceptación, reglas de negocio y contrato HTTP conservan el producto definido en las especificaciones de referencia.
- Las decisiones específicas de PHP/Laravel viven aquí, sin trasladar detalles de Python o .NET al código objetivo.
- No se inicia código ejecutable hasta cerrar esta documentación.
- Las decisiones que no bloquean el alcance se registran en [`deuda-tecnica.md`](deuda-tecnica.md).

## Orden de lectura

1. [`constitution.md`](constitution.md): principios obligatorios.
2. [`spec.md`](spec.md): alcance, actores, historias y reglas.
3. [`architecture.md`](architecture.md) y [`arquitectura-panoramica.md`](arquitectura-panoramica.md): estructura del sistema.
4. [`data-model.md`](data-model.md): tablas, relaciones e invariantes.
5. [`api-contract.md`](api-contract.md): contrato HTTP consumido por React.
6. [`plan.md`](plan.md): decisiones técnicas y ciclo de arranque.
7. [`tasks.md`](tasks.md): tareas y evidencias.
8. [`adr/`](adr/): decisiones arquitectónicas.
9. [`deuda-tecnica.md`](deuda-tecnica.md): asuntos pendientes que no cambian el alcance.

## Fuente de verdad

Cuando exista una discrepancia dentro de esta carpeta, se corrige primero la documentación. El modelo de datos y el contrato HTTP son
referencias obligatorias para los repositorios `api`, `app`, `infra`, `page` y `tool`.
