# Tareas — Manager Library (Python)

Cada tarea termina cuando su **evidencia** existe y se puede ejecutar (artículo XII).
La tabla define el trabajo y su evidencia objetivo; el estado inicial era **pendiente**.
El [registro vigente SDD](../registro-sdd.md) fecha los avances, las ejecuciones y sus límites.

Las pruebas del dominio y de casos de uso con fakes acreditan ese nivel. Una historia que
incluye respuestas HTTP, roles, unicidad real o seguridad conserva esas comprobaciones en
T-12 a T-16. Las evidencias deben distinguir ambos alcances, sin cerrar la historia por una
prueba del núcleo. El [procedimiento SDD](../revision-sdd.md) guía la preparación de cada avance.

| # | Tarea | Depende de | Evidencia |
|---|---|---|---|
| T-01 | Esqueleto del proyecto y `GET /health` | — | `GET /health` responde `{"status":"ok"}`; el contenedor de la API mínima arranca con un comando (el sistema integrado se verifica en T-20) |
| T-02 | Migración inicial: seis tablas, FK, índices únicos y las cinco categorías sembradas | T-01 | Las consultas de `data-model.md` §10 devuelven lo documentado; ninguna columna con `DEFAULT` |
| T-03 | Dominio: `Isbn`, `Quantity`, `DateRange` (objetos de valor) | T-01 | Pruebas unitarias: ISBN de 13 dígitos, cantidad > 0, fin ≥ inicio |
| T-04 | Dominio: `Book` (`Withdraw`, `Restock`, baja lógica) | T-03 | Pruebas: nunca < 0, nunca > total |
| T-05 | Dominio: `Loan` (`AddItem`, `ReturnItem`, estado calculado, 14 días, máx. 3) | T-04 | Pruebas con un `Clock` falso: `open`, `overdue`, `closed` |
| T-06 | Dominio: `Member` y `User`; puertos y casos de uso de socios (crear, buscar, consultar) | T-03 | `"Ana "` queda `ana`; rol válido; HU-05 y detalle de socio con conteo sin devolver pasan contra fakes (el rechazo por API se verifica en T-16) |
| T-07 | Puertos y casos de uso de **autenticación** (`LoginService`, `CreateLibrarianService`) con fakes | T-06 | Lógica de HU-01/HU-02 con fakes: vigencia de 8 h solicitada al emisor, error genérico, normalización y duplicados, actor admin y rol solicitado librarian; JWT/hash reales en T-14 y aceptación HTTP, incluido CA-01.3, en T-16 |
| T-08 | Puertos y casos de uso de **libros** (crear, buscar, editar, baja, portada) con fakes | T-04 | HU-03 y HU-04 pasan contra fakes |
| T-09 | Caso de uso **prestar** (`LendBooksService`) con fakes | T-05, T-08 | HU-06 completa: límites, vencidos, sin ejemplares |
| T-10 | Casos de uso **devolver**, **consultar/listar préstamos** y **vencidos** | T-09 | HU-07, HU-08 y HU-09; devolver dos veces → `ITEM_ALREADY_RETURNED` |
| T-11 | Caso de uso **reporte de más prestados** y su puerto de lectura | T-09 | HU-10; el título viejo se conserva tras renombrar (ADR-004) |
| T-12 | Adaptador de persistencia: repositorios de libro, socio, usuario | T-02, T-06, T-07, T-08 | Pruebas de integración contra PostgreSQL real |
| T-13 | Adaptador de persistencia: repositorio de préstamos + testigo de versión | T-12, T-09, T-10 | Último ejemplar: uno 201 y otro 409; límite concurrente por socio y devolución simultánea según C-01 a C-03 |
| T-14 | Adaptador de seguridad: hash de claves y token JWT | T-07 | La clave nunca aparece en logs ni respuestas (RNF-3) |
| T-15 | Adaptador de portadas (disco) | T-08 | Subir PNG de 2 MB ok; 3 MB → 422; sin portada → `cover_key` nulo |
| T-16 | Adaptador HTTP: rutas de [`api-contract.md`](api-contract.md), cuerpo de error único, autorización por rol | T-07, T-08, T-10, T-11, T-13, T-14, T-15 | Prueba de contrato recorre todas las rutas y todos los códigos de §2 |
| T-17 | Composición y arranque (admin inicial desde entorno) | T-16 | El sistema arranca con la base vacía y se puede iniciar sesión como admin |
| T-18 | Bajar a la base las invariantes pendientes de `data-model.md` §4 (total, rol, ISBN, título, portada, usuario y documento) | T-12 | Un `INSERT` por `psql` que las viole falla; `data-model.md` §4 se actualiza |
| T-19 | Pruebas de arquitectura (artículos I, II y III) | T-17 | un contrato de `import-linter` y un test en `tests/architecture/` pasa y falla si se introduce una importación prohibida |
| T-20 | `infra`: `docker compose up` levanta motor vacío, API y frontend | T-17, T-22 | Un comando; la base arranca sin tablas hasta correr las migraciones |
| T-21 | `tool`: sembrador de demostración **por la API** (≥ 40 libros, 10 socios, 15 préstamos, 3 vencidos) | T-17 | Tras correrlo, `/loans/overdue` devuelve 3 líneas y el reporte tiene filas |
| T-22 | `app`: login, catálogo, socios, prestar, devolver, vencidos, reporte | T-16 | Recorrido manual de punta a punta en la feria |
| T-23 | `page`: sitio público estático de presentación | — | Se abre sin la API; correcto en móvil y escritorio |

## Definición de terminado de la prueba

Quien llegue a T-17 con evidencia tiene el **núcleo** hecho. T-18 a T-23 son lo que separa un trabajo bueno de uno sobresaliente. Se evalúa
el desempeño con SDD, no la cantidad de tareas: una tarea con evidencia vale más que tres a medias.

## Trazabilidad adicional de requisitos existentes

T-21 prepara también el escenario de RNF-1 con 5.000 libros creados por la API; 40 libros solo acreditan
la demostración. La medición debe documentar entorno, consultas y resultados antes de declarar RNF-1 cumplido.
La medición y la creación de tres vencidos están definidas en C-07 y C-04 de
[decisiones previas](../decisiones-previas.md). No se amplía el contrato para satisfacer el sembrador.


Cada tarea conserva estado pendiente hasta disponer de su evidencia ejecutable específica.
El registro identifica los avances con evidencia y las garantías parciales; no certifica
el sistema completo ni modifica las dependencias de esta tabla.
Las precisiones de documentación no acreditan T-13, T-18, T-20 ni T-21.
