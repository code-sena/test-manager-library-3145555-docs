# Deuda técnica registrada — Manager Library (PHP/Laravel)

Este registro no amplía el producto ni modifica su alcance. Contiene decisiones de implementación que pueden resolverse después de
cerrar la documentación funcional y contractual.

| ID | Deuda | Impacto | Criterio para cerrarla | Estado |
|---|---|---|---|---|
| DT-001 | Elegir y documentar el paquete Laravel para emitir y validar JWT HS256. | Seguridad y mantenimiento. | El paquete cumple el contrato, expone `sub` y `role`, y tiene pruebas de expiración y firma. | Pendiente |
| DT-002 | Definir la implementación de producción para `CoverStorage` (disco local o S3). | Operación y respaldo de portadas. | Existe un adaptador probado; el contrato HTTP permanece igual. | Pendiente |
| DT-003 | Definir la herramienta concreta para validar las reglas de arquitectura PHP. | Automatización de calidad. | Las reglas de los artículos I, II y III fallan ante una dependencia prohibida. | Pendiente |
| DT-004 | Fijar el procedimiento final de disponibilidad de PostgreSQL en Docker Compose. | Arranque local y reproducibilidad. | `docker compose up` espera salud de la base, ejecuta migraciones y deja disponible la API. | Pendiente |
| DT-005 | Definir el entorno exacto de la medición de 300 ms con 5 000 libros. | Comparabilidad del requisito RNF-1. | Se documentan hardware, carga, consulta y umbral de aceptación. | Pendiente |

## Reglas del registro

- Ninguna deuda autoriza agregar funcionalidades fuera de `spec.md`.
- Ninguna deuda bloquea el cierre del contrato HTTP o del modelo de datos.
- Cada deuda debe conservar el alcance actual y tener un criterio verificable de cierre.
