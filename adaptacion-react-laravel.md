# Adaptación del reto a React y Laravel

Esta guía traduce responsabilidades técnicas del spec. No modifica el alcance de `spec.md` §2,
los actores, las historias, las reglas de negocio, las seis entidades ni el contrato HTTP.
El cierre de los hallazgos se registra en [decisiones previas](decisiones-previas.md)
y su trazabilidad en [la auditoría](auditoria-documental.md).

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
| Composición de puertos | `app/Providers/LibraryServiceProvider.php` | Único punto de unión de puertos y adaptadores; pruebas de dependencias |
| Argon2id / BCrypt | Argon2id detrás de `PasswordHasher` | Soporte del runtime comprobado al construir; hashes fuera de respuestas/logs |
| Emisión de JWT | Adaptador detrás de `TokenIssuer` | HS256, 8 horas, `sub` y `role`, secreto de entorno |
| Reloj del sistema | Adaptador de `Clock` | UTC; vencimientos probados sin esperar 14 días |
| Portadas | Adaptador de `CoverStorage` | Validación de contenido, límite de 2 MB, clave opaca y orden de borrado |
| pytest / xUnit / pruebas de arquitectura | PHPUnit 12.5 y pruebas de dependencias | Evidencias equivalentes de `tasks.md`, incluida detección de una dependencia prohibida |
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

## Versiones elegidas y entorno

| Componente | Base elegida | Evidencia y reproducción |
|---|---|---|
| PHP | 8.4 | Runtime de contenedor; verificar extensiones de PostgreSQL y Argon2id |
| Laravel | 13.x | Compatible con PHP 8.4; resolución exacta en `composer.lock` al implementar |
| Composer | 2.x | Resolver y fijar dependencias sin ejecutar migraciones de paquetes ajenos |
| PostgreSQL | 16.x | Motor exigido por el spec; fijar imagen/digest en la implementación |
| PHPUnit | 12.5.x | Pruebas puras, aplicación, integración, HTTP y arquitectura |
| React / React DOM | 19.2.x | Aplicación cliente; versiones iguales y resolución en `package-lock.json` |
| Vite | 8.3.x | Compilación del frontend; fijar resolución de dependencias |
| Node.js / npm | 26.5.0 / 11.17.0 | Versiones locales observadas; fijar el runtime equivalente de construcción |

Fuentes contrastadas el 14 de octubre de 2026: [Laravel](https://laravel.com/docs/13.x/releases),
[PHPUnit](https://docs.phpunit.de/en/12.5/installation.html),
[React](https://react.dev/versions), [Vite](https://vite.dev/releases)
y [requisitos de Node para Vite](https://vite.dev/guide/).
Son versiones elegidas, no paquetes instalados ni una afirmación de ejecución satisfactoria.
Los lockfiles e imágenes exactas se crean con la implementación y se conservan en sus repositorios.

Diagnóstico histórico anterior a T-01: PHP y Composer no se encontraron en PATH; Docker
tenía cliente 29.6.2, pero su servidor no estaba disponible en esa revisión.
La ejecución posterior en GitHub Actions y sus límites se registran en el
[seguimiento vigente](registro-sdd.md). Ese diagnóstico inicial no describe por sí solo
el estado actual del runtime local ni acredita el despliegue integrado.

## Mapa de implementación de `api`

Este mapa no mueve ni renombra las carpetas existentes de las referencias Python/.NET.
Define las rutas de destino de la adaptación Laravel; el estado de lo implementado
se consulta en el registro SDD:

```text
app/
  Library/
    Domain/                   # entidades y valores PHP puros
    Application/
      Ports/                  # repositorios, Clock, hash, tokens, portadas y UnitOfWork
      Services/               # un servicio por caso de uso
    Adapters/
      Inbound/Http/           # controladores, serialización y traducción de errores
      Outbound/Persistence/   # repositorios, lectura agregada y transacciones PostgreSQL
      Outbound/Security/      # hash y JWT
      Outbound/Storage/       # portadas
      Outbound/Clock/         # reloj real y fijo para demo/testing
  Providers/
    LibraryServiceProvider.php # único punto de composición del núcleo
database/migrations/          # único dueño de DDL y categorías
routes/api.php                # declaración de rutas hacia el adaptador HTTP
tests/                        # dominio, aplicación, integración, contrato y arquitectura
```

`Domain` solo depende de PHP; `Application` depende del dominio y declara sus puertos;
los adaptadores implementan esos puertos. Los tipos de Laravel/PDO no aparecen en firmas del núcleo.
El punto de composición es el único que vincula puertos con adaptadores; los controladores reciben
casos de uso mediante inyección. `UnitOfWork` expresa atomicidad desde la aplicación;
los testigos y bloqueos quedan dentro del adaptador, sin filtrar tipos de PostgreSQL al dominio.

Las pruebas de arquitectura deben fallar ante una dependencia prohibida del dominio/aplicación,
un tipo de infraestructura en una firma del núcleo o composición fuera del punto establecido.
Los modelos persistentes se distinguen de las entidades de dominio; heredar de Eloquent en el dominio viola la constitución.

Las decisiones detalladas y sus pruebas requeridas están en [decisiones previas](decisiones-previas.md).
La revisión inicial del PR fijó esta base documental sin acreditar implementación.
El [registro SDD](registro-sdd.md) distingue ahora lo probado, publicado y fusionado
de las garantías que todavía requieren integración, HTTP o concurrencia.
