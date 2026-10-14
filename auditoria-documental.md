# Auditoría documental previa a la implementación

Fecha: 14 de octubre de 2026. Base revisada: commit `90860a3` del fork de documentación.

Contexto histórico: este archivo conserva los hallazgos y verificaciones de los primeros
PR documentales, anteriores a T-01. Las afirmaciones de repositorios vacíos, entorno local
y tareas pendientes se refieren a esas revisiones. El seguimiento posterior está en el
[registro SDD](registro-sdd.md); las correcciones del proceso están en la [revisión SDD](revision-sdd.md).

Se revisaron el README y los doce documentos de cada versión (`spec-python/` y `spec-.net/`),
incluidas las cuatro decisiones de arquitectura. Los otros cinco repositorios solo contienen su README.
No existe todavía una aplicación ni una base implementada para contrastar el modelo.

## Restricciones de esta revisión

- El alcance de producto de `spec.md` §2 permanece intacto: no se añaden ni eliminan funcionalidades.
- Se conservan nombres de entidades, tablas, columnas, rutas y carpetas. Singular/plural se reporta.
- Las correcciones inequívocas se aplican por igual a las dos versiones.
- Una decisión pendiente no se presenta como requisito aprobado ni como implementación terminada.
- Los commits y el PR conservan el análisis y las correcciones antes de escribir código de producto.
- No hay evidencia para identificar una inconsistencia concreta como la trampa intencional de la prueba.

## Hallazgos y disposición

Las referencias de la tabla existen en ambas versiones, salvo donde se indica lo contrario.
Esta tabla conserva las disposiciones iniciales del PR #1; el estado vigente de cierre
se registra al final para distinguir la detección original de las decisiones posteriores.

| ID | Evidencia | Consecuencia | Disposición |
|---|---|---|---|
| D-01 | `api-contract.md` §3.6: petición con dos `items`; respuesta con uno | El ejemplo contradice CA-06.1 y puede inducir una implementación que omita líneas | Corregir el ejemplo a un préstamo completo de una línea; conservar cardinalidad 1 a 3 |
| D-02 | `api-contract.md` §3.4 describe `GET /books/{id}/cover`, pero la tabla no lo lista | Una implementación basada solo en la tabla puede omitir una operación ya requerida | Incorporar la operación existente a la tabla, con autenticación según §1 |
| D-03 | `tasks.md`: HU-05 y HU-08 carecen de evidencia explícita para sus casos de uso completos | Alcanzar T-17 no asegura cubrir todas las historias | Completar tareas existentes y dependencias; mantener T-01 a T-23 y el producto |
| D-04 | ADR-002, plan D-04 y T-13 exigen testigo de versión; `data-model.md` §3 no lo describe | Añadir una columna arbitraria contradice el modelo; omitir el testigo contradice el ADR | Pendiente: documentar si el testigo es metadato nativo de PostgreSQL o columna explícita y cómo se compara; no modificar el esquema por suposición |
| D-05 | CA-06.4 limita libros por socio; ADR-002 solo protege por libro | Dos préstamos de libros distintos para un socio con dos líneas abiertas pueden superar tres aun protegiendo ambos libros | Pendiente: definir protocolo transaccional por socio y prueba concurrente; no cambiar el límite ni el control optimista ya aceptado |
| D-06 | CA-07.2 y ADR-002 explican el rechazo por `returned_at` lleno, sin describir escritura condicional simultánea | Dos solicitudes pueden leer `null`; con varias copias prestadas, sumar dos veces puede incluso respetar el CHECK del libro | Pendiente: documentar transición atómica de la línea y actualización del libro en la misma transacción; prueba con dos devoluciones simultáneas |
| D-07 | T-21 exige tres vencidos creados por API; POST `/loans` no acepta fechas y CA-06.6 fija 14 días | El sembrador no puede producir vencidos inmediatamente con el reloj real y el contrato actual | Pendiente: decidir un escenario de demostración con reloj controlado usando el puerto Clock ya previsto; no añadir endpoint, fecha de entrada, SQL directo ni reducir el plazo |
| D-08 | CA-10.1 menciona libro y categoría; contrato §3.7 y ADR-004 incluyen también título congelado en la agrupación | Si cambia el título dentro del rango sin cambiar categoría, la redacción puede interpretarse como una o dos filas | Pendiente: confirmar alineación del criterio con la agrupación explícita del contrato y ADR; no reescribir el criterio de negocio unilateralmente |
| D-09 | RNF-3 dice que la contraseña nunca viaja en claro; login y creación de usuario la reciben en JSON | El hash en almacenamiento no protege el transporte | Pendiente: documentar HTTPS para el transporte y el entorno de verificación; no sustituir el contrato por un hash enviado por el cliente |
| D-10 | `data-model.md` §4 marca restricciones como motor; §10 exige verificarlas, pero no hay migraciones | Puede confundirse el diseño objetivo con una garantía ya implementada | Aclarar que las marcas describen el objetivo y todavía no existe evidencia del motor; conservar las pendientes de T-18 |
| D-11 | RNF-1 exige menos de 300 ms con 5.000 libros; T-21 solo exige al menos 40 | La demostración no acredita el requisito de rendimiento | Añadir trazabilidad documental de la prueba con 5.000 libros por API; pendiente definir consultas, entorno y criterio estadístico de medición |
| D-12 | `api-contract.md`: no define todos los cuerpos de creación/edición, listas de categorías y vencidos ni códigos para token ausente/vencido; `days_late` no define redondeo | Implementaciones distintas pueden ser incompatibles aun cumpliendo las historias | Pendiente: completar ejemplos y decisiones del contrato sin inventar ahora campos, códigos o reglas de redondeo |
| D-13 | `spec-.net/architecture.md` §2 muestra `src/Library/domain`, pero constitución y composición citan proyectos `src/Library.Domain` y `src/Library.Bootstrap` | La estructura de referencia .NET no coincide consigo misma | Reportar y conservar rutas; la adaptación Laravel deberá tener un mapa explícito coherente |
| D-14 | README exige React y Laravel; ambos planes usan frameworks, herramientas de prueba y rutas de otro stack | Copiar la tecnología del spec incumpliría el reto | Añadir guía de adaptación y mantener los specs de referencia; pendiente fijar versiones y herramientas antes del código |
| D-15 | T-01 habla de arranque en contenedor antes de T-20; T-20 exige frontend, pero solo depende de T-17 | La secuencia no distingue API mínima, sistema integrado y frontend terminado | Aclarar evidencia de T-01 y añadir T-22 como dependencia de T-20; no reducir RNF-5 |
| D-16 | `architecture.md` atribuye documento único a `Member`, pero la unicidad depende del repositorio y restricción del motor | Un objeto aislado no puede garantizar unicidad global | Pendiente: aclarar responsabilidad dominio/aplicación/motor sin retirar la unicidad de CA-05.1 |
| D-17 | Constitución Art. VI exige imponer las invariantes expresables en la base; modelo §4 omite ISBN de 13 dígitos, título/documento obligatorios, usuario normalizado y portada no vacía | T-18 con solo total y rol no completa las invariantes ya requeridas | Hallazgo de segunda revisión: añadir las restricciones como pendientes de T-18, sin columnas nuevas ni cambio de reglas |

## Singular y plural: reportar sin renombrar

| Elemento | Forma observada | Evaluación |
|---|---|---|
| Entidades y tablas | `Book`/`book`, `Member`/`member`, `User`/`user`, `Loan`/`loan`, `LoanItem`/`loan_item`, `category` | Las seis tablas son singulares y consistentes con la convención |
| Colecciones HTTP | `/books`, `/members`, `/users`, `/loans`, `/categories`, `items`, `rows` | Plurales para colecciones; diferencia de capa, no prueba de defecto |
| Columnas | `total_copies`, `available_copies` | Forma plural aunque §0 prescribe singular para columnas; se reporta y se conserva |
| Campo de socio | `active_loans` | Nombre plural y referido a préstamos, pero §3.5 lo define como conteo de libros sin devolver; se conserva y se registra su significado |
| Carpetas/casos de uso | `services`, `adapters`, `LendBooksService`, `ReturnItemService` | Nombres conservados; pluralidad no implica cambiar entidades ni rutas |

No se encontró una tabla `books` o `members` que contradiga otra tabla singular en el modelo.
Los valores predeterminados de un futuro ORM no son evidencia de un defecto actual: no hay ORM instalado.

## Comprobaciones y límites

- Las versiones comparten `spec.md`, `api-contract.md`, `data-model.md` y los cuatro ADR sin diferencias de contenido en la base revisada.
- Las diferencias técnicas son runtime, framework, ORM/migraciones, hash, pruebas y rutas de composición.
- El modelo contiene seis tablas, categorías con UUID fijos, timestamps UTC, sin auditoría ni estado persistido.
- Todos los repositorios estaban limpios al iniciar la revisión; sus remotos apuntan a `valerysinai`.
- Una revisión documental no acredita pruebas de API, concurrencia, seguridad, rendimiento ni arranque.
- Esta lista registra hallazgos verificables; no garantiza que no existan más defectos ni identifica la intención del autor.

## Estado de cierre documental

**Correcciones documentales aplicadas:** D-01, D-02, D-03, D-10, la trazabilidad de D-11 y D-15,
en ambas versiones. La guía `adaptacion-react-laravel.md` registra ahora versiones elegidas,
mapa concreto de carpetas y punto de composición para D-14.

**Verificación del PR #1:** enlaces locales válidos; `spec.md`, `api-contract.md` y `data-model.md`
equivalentes entre versiones; `spec.md`, las constituciones y los cuatro ADR sin cambios entonces
frente a la base. Se conservaron las seis tablas, columnas, rutas y nombres de carpetas.

| Hallazgo | Cierre en esta rama | Evidencia de implementación futura |
|---|---|---|
| D-04 | C-01: `xmin` nativo, sin columna añadida; precisión en modelo y ADR-002 | T-13 |
| D-05 | C-02: transacción y coordinación por socio; conserva libros optimistas | T-13 |
| D-06 | C-03: transición condicional de línea y reposición atómica | T-13/T-16 |
| D-07 | C-04: reloj fijo de demo aislado y creación exclusivamente por API | T-21 |
| D-08 | C-05: interpretación con las claves congeladas de contrato y ADR-004 existentes | T-11 |
| D-09 | C-06: HTTPS, hash Argon2id y evidencia sin secretos | T-14/T-16/T-20 |
| D-11 | C-07: 5.000 libros, medición reproducible, cada muestra menor de 300 ms | RNF-1/T-21 |
| D-12 | Contrato §5: cuerpos, sobres, fechas, conteos, errores técnicos y binario de portada | T-16 |
| D-13 | Reportado sin mover ni renombrar las referencias; mapa Laravel explícito | T-19 |
| D-14 | Versiones, dependencias y carpetas de la guía de adaptación | T-01/T-19 |
| D-16 | C-08: validación local, consulta y UNIQUE como última barrera | T-06/T-12/T-16 |
| D-17 | CHECK adicionales de reglas existentes documentados pendientes en modelo y T-18 | T-18 |

Los detalles están en [decisiones previas](decisiones-previas.md). Las observaciones de
singular/plural quedan reportadas sin renombramientos. `spec.md` y las constituciones permanecen
intactos; ADR-002 incorpora precisiones técnicas y conserva la decisión optimista.

La base documental queda propuesta para revisión del siguiente PR. No se declara evidencia
ejecutable: todas las tareas de producto siguen pendientes. La ejecución requiere PHP/Composer
en el runtime previsto y servidor Docker disponible; el entorno local aún no los acredita.

Comprobaciones de esta segunda revisión: enlaces locales válidos en los 28 documentos;
contrato, modelo y ADR-002 iguales entre versiones; `spec.md` y constituciones intactos frente
al merge `6f75d35`; tablas y columnas del modelo físico y tabla de rutas originales conservadas;
23 tareas sin ciclos ni dependencias inexistentes. No se ejecutaron pruebas de producto.
