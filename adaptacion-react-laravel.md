# Adaptación del reto a React y Laravel

Esta guía traduce responsabilidades técnicas del spec. No modifica el alcance de `spec.md` §2,
los actores, las historias, las reglas de negocio, las seis entidades ni el contrato HTTP.
Las decisiones no resueltas se registran en [la auditoría](auditoria-documental.md).

## Referencias y responsabilidades

Se usa `spec-python/` como referencia de lectura para evitar duplicar decisiones de implementación.
`spec-.net/` se conserva como referencia equivalente; las correcciones comunes se mantienen en ambas.
Los modelos y el contrato siguen en sus archivos originales, sin una tercera copia.

| Referencia técnica | Adaptación al reto | Verificación requerida |
|---|---|---|
| FastAPI / ASP.NET Core | Adaptador HTTP en Laravel dentro de `api` | Rutas, roles, cuerpos y códigos de `api-contract.md` |
| Dominio puro | Objetos PHP independientes de Laravel y del ORM | Pruebas de invariantes sin base ni servidor |
| Casos de uso y puertos | Aplicación en PHP, contratos definidos por quien los consume | Pruebas con implementaciones falsas, sin infraestructura |
| SQLAlchemy / EF Core | Adaptador de persistencia sobre PostgreSQL 16 | Integración real y restricciones de `data-model.md` |
| Alembic / migraciones EF | Migraciones de Laravel en `api` | Seis tablas, índices, restricciones y semilla de categorías; `infra` sin DDL |
| Composición de puertos | Un punto explícito de composición y registro de dependencias en Laravel | Se documenta su ruta antes de crear la estructura; pruebas de dependencias |
| Argon2id / BCrypt | Hash detrás de `PasswordHasher` | Decisión del algoritmo registrada antes de implementar; hashes fuera de respuestas/logs |
| Emisión de JWT | Adaptador detrás de `TokenIssuer` | HS256, 8 horas, `sub` y `role`, secreto de entorno |
| Reloj del sistema | Adaptador de `Clock` | UTC; vencimientos probados sin esperar 14 días |
| Portadas | Adaptador de `CoverStorage` | Validación de contenido, límite de 2 MB, clave opaca y orden de borrado |
| pytest / xUnit / pruebas de arquitectura | Herramientas de prueba compatibles con el entorno PHP elegido | Evidencias equivalentes de `tasks.md`; elección pendiente |
| Frontend operativo | React en `app` | Recorrido de T-22 y autorizaciones comprobadas también en servidor |

## Distribución entre repositorios

- `docs`: requisitos, decisiones, trazabilidad y evidencia documental.
- `api`: dominio, casos de uso, adaptadores, migraciones y aprovisionamiento inicial del admin.
- `app`: interfaz React que consume el contrato; no sustituye las validaciones del servidor.
- `infra`: contenedores, red, volúmenes y motor vacío; arranque coordinado con las migraciones de `api`.
- `tool`: datos de demostración y preparación de carga por API; no acceso SQL directo.
- `page`: presentación estática independiente de la API.

## Nombres y opciones automáticas del framework

Se preservan tablas singulares, esquema `library`, columnas y rutas originales. La auditoría informa
las diferencias de pluralidad sin renombrarlas. Los modelos de persistencia deberán mapear el esquema
explícitamente para evitar inferencias de nombres de tabla.

No se incorporan por defecto tablas adicionales de usuarios/tokens/sesiones, IDs incrementales,
columnas `created_at`/`updated_at`, `DEFAULT`, estado de préstamo persistido ni cambios de códigos HTTP.
El diseño existente usa seis tablas, UUID v4, JWT sin estado y fechas de negocio explícitas.

## Decisiones que deben quedar cerradas antes del código

- Versiones de PHP, Laravel, React, runtime del frontend y herramientas de prueba, contrastadas con el entorno disponible.
- Mapa concreto de carpetas y punto de composición, respetando dependencias hacia el dominio.
- Testigo de concurrencia y protocolo para préstamos por socio y devoluciones simultáneas (D-04 a D-06).
- Escenario de demostración de vencidos sin cambiar API ni plazo (D-07).
- Alineación del reporte, transporte seguro y detalles incompletos del contrato (D-08, D-09, D-12).
- Medición reproducible de RNF-1 y responsabilidad de la unicidad de socios (D-11, D-16).

No se declara la documentación cerrada ni se inicia implementación como consecuencia de esta guía.
Cada cierre debe enlazar la decisión, el requisito original y su futura evidencia de prueba.
