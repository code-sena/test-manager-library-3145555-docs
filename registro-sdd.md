# Registro vigente de trazabilidad SDD

Foto de revisión: 14 de octubre de 2026. Método y correcciones en [revisión SDD](revision-sdd.md).
Los enlaces a API están fijados por commit para que una fusión posterior no cambie la evidencia revisada.
Los documentos de negocio continúan en ambas carpetas originales.

## Bases revisadas y situación de repositorios

| Repositorio | Base observada | Situación |
|---|---|---|
| docs | main `eab4cf7` | Auditoría y decisiones documentales fusionadas; este registro se propone en feature/docs-sdd-review |
| api | origin/main `a7843b0` | PR #1, #2 y #3 fusionados: bootstrap, esquema y dominio |
| api, avance T-07 | feature/api-auth `8b3a44b` | Rama publicada; la consulta pública de PR no mostró un PR de esta rama. Su código no está en la base main observada |
| app | main `07d7b54` | Checkout revisado: únicamente README, sin implementación React |
| infra | main `0a62569` | Checkout revisado: únicamente README, sin despliegue integrado |
| page | main `88e3657` | Checkout revisado: únicamente README, sin sitio implementado |
| tool | main `0e2ee8a` | Checkout revisado: únicamente README, sin sembrador implementado |

La consulta remota de esta revisión corresponde a docs y API. Los otros cuatro estados
son locales, sin certificar que sus remotos no hayan cambiado. La revisión de los
checkouts empezó sin cambios locales. No se creó la rama api-books.

## Evidencia ejecutada consultada

| Avance | Evidencia por commit | Ejecución y alcance |
|---|---|---|
| T-01 | [t-01.md][e01] | [Bootstrap][ci01]: construcción, tests y salud pública sin DB; no despliegue integrado |
| T-02 | [t-02.md][e02] | [Schema][ci02]: PostgreSQL 16 real, contrato físico, restricciones de esa etapa y semilla |
| T-03 a T-06 | [t-03-t-06.md][e03] | [Dominio/aplicación][ci03]: objetos puros, socios con fakes e aislamiento limitado; [regresión de esquema][ci03db] |
| T-07 | [t-07.md][e07] | [Aplicación y regresión][ci07]: fakes de usuario/hash/token; [regresión de esquema][ci07db]. Ver atribución incorrecta de CA-01.3 en SDD-02 |

Son resultados previamente ejecutados en CI y consultados durante los avances.
Esta revisión documental inspecciona fuentes y evidencias; no vuelve a ejecutar esas suites
ni acredita ejecución local. Las versiones de runtime y dependencias de API están fijadas
en sus archivos Dockerfile/composer.lock, no por este registro.

## Estado de tareas

«Evidencia de tarea» significa que existe la verificación específica de ese nivel.
«Parcial» indica un componente probado con capas de aceptación pendientes.
El artículo IX y la revisión de arquitectura siguen abiertos según SDD-03 y SDD-04.
Una prueba verde o un merge no acredita todas las historias ni los principios transversales.

| Tarea | Estado observado | Límite o siguiente evidencia |
|---|---|---|
| T-01 | Evidencia de tarea disponible, fusionada | Solo API mínima y salud; T-20 conserva el arranque integrado |
| T-02 | Evidencia de tarea disponible, fusionada | CHECK pendientes siguen en T-18; no administrador inicial |
| T-03 | Evidencia de tarea disponible, fusionada | Objetos de valor; no valida todavía sus entradas a través de HTTP |
| T-04 | Evidencia de tarea disponible, fusionada | Inventario y baja en objetos; repositorio y casos de uso de catálogo pendientes |
| T-05 | Evidencia de tarea disponible, fusionada | Invariantes y estados en dominio; transacción/concurrencia y consulta HTTP pendientes |
| T-06 | Evidencia de tarea disponible, fusionada | Entidades y socios con fakes; acceso por rol, persistencia y respuestas HTTP pendientes |
| T-07 | Evidencia de tarea disponible en rama | Login/alta con fakes; corregir SDD-02, verificar fusión; JWT/Argon2id/HTTP pendientes |
| T-08 | Pendiente | Casos de uso de libros, búsqueda y portadas con fakes |
| T-09 | Pendiente | Caso de uso de préstamo coordinado por puertos |
| T-10 | Pendiente | Devolución y consultas de préstamos/vencidos por puertos |
| T-11 | Pendiente | Reporte agregado con snapshots y privacidad |
| T-12 | Pendiente | Repositorios reales; unicidad y consultas contra PostgreSQL |
| T-13 | Pendiente | xmin, transacciones, último ejemplar, límite concurrente y devolución atómica |
| T-14 | Pendiente | Argon2id y JWT HS256 real con expiración y Clock |
| T-15 | Pendiente | Almacenamiento externo; tipo por contenido y límite de 2 MB |
| T-16 | Pendiente | Contrato completo de rutas, autenticación, roles y errores |
| T-17 | Pendiente | Composición y admin inicial desde entorno |
| T-18 | Pendiente | CHECK de invariantes ya descritas en data-model.md §4 |
| T-19 | Pendiente | Artículos I–III completos y detector que rechace infracciones introducidas |
| T-20 | Pendiente | Compose, motor vacío, sistema integrado y HTTPS confiable |
| T-21 | Pendiente | Sembrador exclusivamente por API, vencidos y medición RNF-1 |
| T-22 | Pendiente | Interfaz React y recorrido completo |
| T-23 | Pendiente | Página estática independiente y adaptable a móvil |

## Criterios de aceptación: garantías parciales y faltantes

Fuente: [spec.md §3](spec-python/spec.md). Los códigos HTTP se aceptan mediante T-16.
Las columnas siguientes distinguen lo probado en una capa de lo que exige el criterio completo.
Los nombres de método citados existen en los archivos enlazados.

| Criterio | Evidencia existente y adaptación | Aceptación pendiente |
|---|---|---|
| CA-01.1 | [AuthenticationTest][auth-tests], test_valid_login_issues_eight_hours_with_only_identity_role_and_utc_times; Clock/TokenIssuer, 28.800 segundos | JWT firmado real, validación y respuesta del endpoint: T-14/T-16 |
| CA-01.2 | [AuthenticationTest][auth-tests], test_missing_user_and_wrong_password_have_the_same_error_and_neither_issues_a_token | 401 genérico por HTTP, hash real y recorrido de usuario ausente: T-14/T-16 |
| CA-01.3 | Pendiente; la autorización de CreateLibrarianService solo demuestra el límite de ese caso de uso | Todas las rutas protegidas, Bearer ausente/inválido/vencido y 403 por rol antes de existencia: T-16 |
| CA-02.1 | [AuthenticationTest][auth-tests], test_librarian_cannot_create_users_and_authorization_precedes_lookup_and_validation y alta con actor admin | Actor autenticado real y POST /users: T-14/T-16 |
| CA-02.2 | [IdentityTest][identity-tests], normalización; [AuthenticationTest][auth-tests], duplicado normalizado y conflicto tras precheck; [SchemaMigrationTest][schema-tests], índice UNIQUE | Repositorio real y 409 específico bajo altas concurrentes: T-12/T-16 |
| CA-02.3 | [AuthenticationTest][auth-tests], test_admin_cannot_create_an_admin_or_any_other_requested_role y respuesta sin clave/hash | 422 por API y revisión de logs de los adaptadores reales: T-14/T-16; artículo IX abierto |
| CA-03.1 | [BookTest][book-tests], test_create_withdraw_and_restock_keep_inventory_in_range_without_mutating_originals | Alta por caso de uso y HTTP: T-08/T-12/T-16 |
| CA-03.2 | [ValuesTest][value-tests], ISBN de 13 dígitos sin checksum; [SchemaMigrationTest][schema-tests], test_partial_isbn_index_allows_reuse_only_after_soft_deletion | Traducción de ISBN duplicado a 409 y flujo de alta: T-08/T-12/T-16 |
| CA-03.3 | [BookTest][book-tests], test_total_changes_preserve_the_number_of_borrowed_copies_and_the_isbn; Quantity positivo | Edición persistida y 422: T-08/T-12/T-16; CHECK total: T-18 |
| CA-03.4 | [BookTest][book-tests], test_cover_can_be_set_or_removed_but_never_empty: solo clave nullable | Carga JPG/PNG por contenido, 2 MB, almacenamiento y HTTP: T-08/T-15/T-16 |
| CA-03.5 | [BookTest][book-tests], rechazo por préstamos abiertos; [SchemaMigrationTest][schema-tests], preservación de histórico | Baja coordinada con persistencia, filtro activo y 409: T-08/T-12/T-13/T-16 |
| CA-04.1 | Pendiente | Texto título/autor, categoría, activos, orden título/ID y paginación: T-08/T-12/T-16 |
| CA-04.2 | Pendiente para libros; la prueba de paginación de socios no lo acredita | Total real de búsqueda de libros: T-08/T-12/T-16 |
| CA-04.3 | Pendiente para libros | Consulta de libro ausente o dado de baja con 404: T-08/T-12/T-16 |
| CA-05.1 | [MembersTest][member-tests], alta, duplicado y conflicto de inserción; [SchemaMigrationTest][schema-tests], UNIQUE documento | Roles, unicidad concurrente real y 409: T-12/T-16 |
| CA-05.2 | [MembersTest][member-tests], test_search_by_name_or_document_is_paginated_and_reports_the_full_total | Consulta real y contrato HTTP: T-12/T-16 |
| CA-05.3 | Member carece de operación de borrado; el contrato tampoco define ruta de borrado | Verificar superficie HTTP y composición finales: T-16/T-17 |
| CA-06.1 | [LoanTest][loan-tests], cardinalidad; raíz guarda socio y operador recibidos | Operador desde autenticación verificada, aplicación y persistencia: T-09/T-13/T-16 |
| CA-06.2 | [LoanTest][loan-tests], test_duplicate_books_are_rejected | Validación del caso de uso y 422: T-09/T-16 |
| CA-06.3 | [LoanTest][loan-tests], decremento y fallo sin mutar objetos de entrada | Transacción real sin escritura parcial y 409: T-09/T-13/T-16 |
| CA-06.4 | [LoanTest][loan-tests], test_three_unreturned_books_are_counted_across_separate_loans | Historia leída por puertos, límite concurrente y 422 con conteo: T-09/T-13/T-16 |
| CA-06.5 | [LoanTest][loan-tests], test_overdue_member_cannot_borrow_even_when_the_new_request_fits_the_limit | Lectura real, coordinación por socio y 409: T-09/T-13/T-16 |
| CA-06.6 | [LoanTest][loan-tests], test_lending_freezes_catalog_values_and_decrements_each_book_once | Reloj en aplicación, snapshots persistidos y serialización: T-09/T-13/T-16 |
| CA-06.7 | Loan es inmutable salvo devolución; no expone editar/borrar | Repositorio y superficie HTTP finales: T-13/T-16 |
| CA-07.1 | [LoanTest][loan-tests], test_return_is_per_line_and_cannot_restock_twice | Escritura condicional y stock en una transacción: T-10/T-13/T-16 |
| CA-07.2 | Mismo test de devolución, sobre el estado actualizado | Dos solicitudes simultáneas, 409 y ninguna reposición doble: T-13/T-16 |
| CA-07.3 | [BookTest][book-tests], límite superior; [SchemaMigrationTest][schema-tests], CHECK disponibilidad | Devolución mediante adaptador real y contrato: T-13/T-16 |
| CA-07.4 | [LoanTest][loan-tests], test_status_is_computed_with_fake_time_and_the_due_boundary_is_strict | Consultas y respuesta final de préstamo cerrado: T-10/T-13/T-16 |
| CA-08.1 | Loan contiene líneas y calcula estado; [LoanTest][loan-tests], reloj falso | Caso de uso de detalle y HTTP: T-10/T-13/T-16 |
| CA-08.2 | [ValuesTest][value-tests], rango de fechas como objeto aislado | Filtros, orden fecha/ID y paginación reales: T-10/T-13/T-16 |
| CA-08.3 | Dominio calcula estado; [SchemaMigrationTest][schema-tests], contrato físico sin columna status | Mantenerlo al incorporar consultas y serialización: T-10/T-13/T-16 |
| CA-09.1 | [LoanTest][loan-tests], instante estricto y periodos de 24 h; sin listado implementado | Listado de líneas con libro/socio y días: T-10/T-13/T-16 |
| CA-09.2 | Pendiente | Orden documentado y autorización admin/librarian: T-10/T-13/T-16 |
| CA-10.1 | Decisión C-05 interpreta agrupación existente por book_id + título + categoría congelados | Consulta agregada, conteos y orden: T-11/T-16 |
| CA-10.2 | [ValuesTest][value-tests], test_inverted_range_has_its_own_business_error | Reporte sin préstamos vacío y 422 por HTTP: T-11/T-16 |
| CA-10.3 | Contrato/ADR-004 excluyen desglose personal; sin reporte implementado | Verificar consulta y salida sin socio/bibliotecario: T-11/T-16 |
| CA-10.4 | Snapshots de préstamo probados; no existe reporte | Estabilidad del reporte tras renombrar libro/categoría: T-11/T-16 |
| CA-10.5 | Pendiente | Reporte exclusivo de admin y 403 por rol: T-16 |

## Reglas y requisitos no funcionales

| Requisito | Relación y evidencia parcial | Pendiente |
|---|---|---|
| RN-1 | CA-03.3, CA-07.3; dominio y CHECK de disponibilidad | Persistencia y concurrencia de operaciones reales |
| RN-2 | CA-06.4/6; dominio con historial recibido y reloj explícito | Caso de uso, lectura y coordinación concurrente |
| RN-3 | CA-06.5, política de dominio | Aplicación y transacción por socio |
| RN-4 | CA-06.6, snapshots en LoanItem | Persistencia y reporte estable |
| RN-5 | CA-03.5, CA-06.7 y CA-07; modelo y objetos | Superficie final y preservación del historial en los flujos reales |
| RN-6 | Fechas de dominio normalizadas UTC, conexión PG en UTC; mensajes españoles | Serialización Z y revisión de todos los adaptadores |
| RNF-1 | Decisión C-07 documentada | 5.000 libros por API y cada muestra menor de 300 ms; T-21 |
| RNF-2 | xmin/C-01/C-02/C-03 documentados | Dos operadores, último ejemplar y resultados 201/409; T-13/T-16 |
| RNF-3 | Hash privado en User, resultados de alta sin hash; C-06 | Argon2id real, HTTPS confiable, logs/respuestas y control de secretos en CI |
| RNF-4 | [BootstrapTest][bootstrap-tests] cubre salud, 404, 405 y 500 | Todos los errores/rutas y códigos del contrato; T-16 |
| RNF-5 | API mínima construida y salud con un comando | Compose con API/React y motor vacío sin DDL en infra; T-20 |

## Principios y seguimiento de correcciones

| Artículo | Evidencia actual | Seguimiento |
|---|---|---|
| I | Dominio PHP nativo, recorrido aislado y referencias cualificadas comprobadas | SDD-04: detector limitado; mantener aislamiento y completar T-19 |
| II | Puertos en Application y tests de servicios con fakes | SDD-04: revisar helpers, imports y firmas de infraestructura |
| III | Punto de composición reservado en LibraryServiceProvider | T-17/T-19: bindings reales y prohibición de composición externa |
| IV | Servicios separados y puertos pequeños | Revisar sustitución fake/adaptador al añadir integración; no declarar SOLID por nombres |
| V | Migración en API; infra local solo README | Mantener el DDL fuera de infra al implementar T-20 |
| VI | Esquema verificado para T-02 | Aplicar/verificar las CHECK pendientes de T-18 |
| VII | Estado/días calculados; modelo sin columnas derivadas | Mantener en consultas y persistencia |
| VIII | Snapshots en LoanItem probados | T-11: consulta del reporte usa únicamente valores congelados |
| IX | Sin cuerpos de autenticación en el código revisado de los servicios | SDD-03: control CI ausente; T-14/T-16: logs y respuestas reales |
| X | Introspección y escrituras inválidas de T-02 | Corregir cualquier discrepancia reproducida sin acomodar el producto a un defecto de código |
| XI | Código en inglés, documentación/mensajes en español | Mantener; commits y PR en inglés por preferencia de la usuaria |
| XII | Evidencias reproducibles por etapa | SDD-01/02/05: registrar alcance preciso y separar evidencia de tarea de aceptación completa |

## Hallazgos de la segunda revisión

La [segunda revisión](segunda-revision-sdd.md) conserva la foto de bases y resultados de este
registro. Añade SDD-08 (explicación de bloqueo de fila corregida), SDD-09 (fecha con byte
nulo pendiente de prueba/corrección en API) y SDD-10 (umbral en bytes de portada pendiente
de precisión). La evidencia parcial de DateRange no cierra el nuevo caso de entrada inválida.
Las fuentes técnicas y los escenarios de corrección se enlazan en ese informe.

## Corte de entrega

El README fija 2026-10-14 a las 11:00, America/Bogota, equivalente a 16:00 UTC.
El reloj consultado al preparar esta revisión marcó 16:00:03 UTC. Este registro y sus nuevos
commits son posteriores al corte. La revisión no modifica esa regla ni certifica la elegibilidad
de una entrega. La publicación previa debe contrastarse con su fecha remota; una fecha local
de commit o la fecha de una fusión no sustituyen por sí solas la evidencia de publicación.

[e01]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/a7843b0/evidence/t-01.md
[e02]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/a7843b0/evidence/t-02.md
[e03]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/23404e9/evidence/t-03-t-06.md
[e07]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/8b3a44b/evidence/t-07.md
[ci01]: https://github.com/valerysinai/test-manager-library-3145555-api/actions/runs/38575336538
[ci02]: https://github.com/valerysinai/test-manager-library-3145555-api/actions/runs/38579510746
[ci03]: https://github.com/valerysinai/test-manager-library-3145555-api/actions/runs/38585289439
[ci03db]: https://github.com/valerysinai/test-manager-library-3145555-api/actions/runs/38585289601
[ci07]: https://github.com/valerysinai/test-manager-library-3145555-api/actions/runs/38587582285
[ci07db]: https://github.com/valerysinai/test-manager-library-3145555-api/actions/runs/38587582221
[auth-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/8b3a44b/tests/Application/AuthenticationTest.php
[identity-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/23404e9/tests/Domain/IdentityTest.php
[member-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/23404e9/tests/Application/MembersTest.php
[loan-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/23404e9/tests/Domain/LoanTest.php
[book-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/23404e9/tests/Domain/BookTest.php
[value-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/23404e9/tests/Domain/ValuesTest.php
[schema-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/a7843b0/tests/Integration/SchemaMigrationTest.php
[bootstrap-tests]: https://github.com/valerysinai/test-manager-library-3145555-api/blob/a7843b0/tests/Feature/BootstrapTest.php
