# Decisiones previas a la implementación

Fecha: 14 de octubre de 2026. Continúa la [auditoría documental](auditoria-documental.md)
fusionada en el PR #1. Estado: decisiones documentadas en esta rama, sujetas a revisión del PR.
La evidencia ejecutable de las tareas sigue pendiente. No se modifica `spec.md` §2 ni se renombra ningún elemento.

## C-01 · Testigo de versión del libro (D-04)

Se adopta `xmin`, metadato nativo de cada versión de fila de PostgreSQL 16. El adaptador lee
`xmin::text` junto al libro y conserva ese valor opaco dentro de la operación. Cada escritura
compara el testigo leído mediante una condición parametrizada sobre `id` y `xmin::text`.
Una fila afectada acredita éxito; cero obliga a revertir la transacción y releer/revalidar.
En préstamo, tras un único reintento completo, el conflicto responde 409 `BOOK_NOT_AVAILABLE`,
según ADR-002. No se reutilizan objetos mutados de la transacción revertida.

No se añade una columna `version`, un `DEFAULT` ni un campo HTTP. El testigo no entra al dominio,
no es un identificador permanente y no se conserva entre peticiones: los identificadores de
transacción pueden dar la vuelta. Las escrituras del catálogo también comparan versión y
reevalúan sus reglas tras releer; un conflicto persistente usa el error técnico del contrato.

Fuente técnica: [columnas de sistema de PostgreSQL 16](https://www.postgresql.org/docs/16/ddl-system-columns.html).
La elección de `xmin` como testigo es una decisión de este proyecto, no una recomendación atribuida al manual.
Evidencia futura: T-13, dos préstamos del último ejemplar; exactamente uno 201 y otro 409,
una sola línea persistida y ninguna disponibilidad negativa.

## C-02 · Operaciones del mismo socio (D-05)

La unidad de trabajo de prestar y devolver abre una transacción `READ COMMITTED` y obtiene
un bloqueo consultivo **de transacción por socio** antes de consultar líneas abiertas.
La clave es un entero de 64 bits con signo, obtenido de los primeros ocho bytes de SHA-256
del UUID canónico en minúsculas del socio, interpretados en complemento a dos y orden de bytes big-endian.
El adaptador invoca `pg_advisory_xact_lock` con esa clave. Una colisión solo serializa dos socios
adicionales: no omite validaciones. Todos los escritores de préstamos y devoluciones usan el protocolo.

Después de adquirir el bloqueo se captura una vez `Clock.now()` para la operación y se vuelve
a contar **líneas sin devolver**, no préstamos. Se comprueba vencimiento estricto (`due_at < now`)
y límite actual más líneas solicitadas. El segundo préstamo concurrente ve el resultado del primero.
El dominio decide las reglas; aplicación coordina; el adaptador asegura la transacción.

Se adquiere primero el bloqueo de socio y se actualizan libros en orden ascendente de UUID.
No se bloquea toda la biblioteca ni se sustituye la comparación optimista de libros por
`SELECT FOR UPDATE`. Las actualizaciones de PostgreSQL sí adquieren sus bloqueos internos normales.
No hay esperas por entrada del usuario ni almacenamiento externo dentro de la transacción.
Un error de deadlock revierte todo y permite un único reintento completo.

Fuentes: [bloqueos consultivos](https://www.postgresql.org/docs/16/explicit-locking.html#ADVISORY-LOCKS)
y [aislamiento READ COMMITTED](https://www.postgresql.org/docs/16/transaction-iso.html#XACT-READ-COMMITTED).
Evidencia futura: T-13, socio con dos líneas abiertas y dos peticiones simultáneas de libros distintos:
una 201, otra 422 `MEMBER_LIMIT_EXCEEDED`, conteo final tres. Socios distintos no comparten bloqueo salvo colisión.

## C-03 · Devolución atómica (D-06)

La aplicación obtiene el socio propietario del préstamo y aplica C-02. Dentro de la transacción
valida que la línea pertenece al `loan_id` de la ruta. Identificadores no relacionados responden 404.
La transición de la línea se escribe condicionada por `id`, `loan_id` y `returned_at IS NULL`.
Solo cuando afecta una fila se incrementa una copia del libro, comparando su versión de C-01.
La línea y el libro se confirman juntos. Si la escritura del libro falla, se revierte también la línea.
Una línea ya devuelta responde 409 `ITEM_ALREADY_RETURNED` y no modifica existencias.

Releer y comprobar la línea forma parte de cualquier reintento. Una condición CHECK incumplida
es defecto de integridad y no se disfraza de devolución repetida. Se conserva el cuerpo de error único.
El estado del préstamo se sigue calculando; no se agrega columna ni tabla de devoluciones.

Evidencia futura: T-13 y T-16, dos devoluciones simultáneas de una línea cuando el libro tiene
otras copias prestadas: una 200, otra 409, un incremento y un único `returned_at` confirmado.

## C-04 · Demostración de vencidos por API (D-07)

Se usa un adaptador de reloj fijo detrás del puerto `Clock` existente, solo en perfiles `demo`
y `testing`, con base aislada. `CLOCK_MODE=system` es el valor normal. `CLOCK_MODE=fixed` requiere
un `CLOCK_FIXED_AT` ISO 8601 UTC válido y se rechaza al arrancar fuera de esos perfiles.
No se cambia el reloj del sistema operativo ni se admite cambiarlo mediante HTTP.
La emisión/validación temporal del JWT y las reglas de negocio usan el mismo reloj inyectado.

Secuencia reproducible, coordinada por `infra` y `tool`:

1. Arrancar el entorno aislado en `2026-09-29T14:15:00Z`, ejecutar las migraciones de `api`
   y aprovisionar el admin desde entorno. `tool` inicia sesión y crea por API 40 libros y 10 socios.
2. Registrar tres préstamos de una línea, para tres socios diferentes, sin devolverlos.
3. Reiniciar únicamente la API del entorno demo con el reloj en `2026-10-14T14:15:00Z` y la misma base.
   Los tres vencimientos son `2026-10-13T14:15:00Z`. `tool` inicia sesión de nuevo; el JWT anterior expiró.
4. Crear otros doce préstamos de una línea entre los siete socios restantes, sin superar tres por socio.
5. Verificar por API 15 préstamos, exactamente tres líneas vencidas y reporte con filas para ese rango.

Esto es una fixture de demostración, no una función del producto. Todos los datos de negocio entran
por API; el reloj se configura durante el despliegue. No se añade `loaned_at` a la petición,
no se cambia el plazo de 14 días y no se usa SQL directo desde `tool`.
Evidencia futura: T-21 guarda comandos de reproducción, respuestas y conteos, sin tokens ni contraseñas.

## C-05 · Agrupación histórica (D-08)

Se conserva la definición explícita de `api-contract.md` §3.7 y ADR-004:
`book_id + book_title + category_name` congelados. La expresión abreviada de CA-10.1
se interpreta con esas claves existentes; no se reescribe el criterio ni se añade dimensión.
Un mismo libro con dos títulos congelados dentro del rango produce dos filas, aun si la categoría
no cambió. Renombrar el catálogo después del periodo no altera ninguna fila del periodo.
Solo se unen `loan` y `loan_item`, nunca el catálogo vivo para obtener el texto del reporte.
Evidencia futura: T-11 cubre cambio de título dentro y después del rango, categoría y privacidad.

## C-06 · Transporte y secretos (D-09)

RNF-3 se verifica en el transporte mediante HTTPS con certificado confiable para el cliente;
la petición JSON de login y alta conserva `password`. No se reemplaza por un hash enviado por React.
TLS termina en el punto de entrada de `infra`; API y base se comunican dentro de la red privada.
La prueba de credenciales no se acredita con HTTP simple ni con validación TLS desactivada.
La búsqueda de rendimiento también utiliza HTTPS.

Argon2id se elige para el adaptador `PasswordHasher`, con soporte verificado al construir el runtime.
La API no arranca si ese algoritmo no está disponible. Los hashes de usuarios se generan en el
adaptador: el secreto de despliegue es la credencial del admin y `JWT_SECRET`; no todos los hashes
de los bibliotecarios proceden del entorno. Se conservan Artículo IX y CA-02.3.
No se registran cuerpos de login/alta, Authorization ni secretos en logs o evidencias.

Fuentes: [hash en PHP](https://www.php.net/manual/en/function.password-hash.php)
y [hash en Laravel](https://laravel.com/docs/13.x/hashing).
Evidencia futura: T-14/T-16 prueban hash, login, expiración, errores genéricos y ausencia de secretos;
T-20 documenta terminación TLS y confianza del certificado sin desactivar protecciones del cliente.

## C-07 · Medición de búsqueda (D-11)

Preparar por API una base de 5.000 libros activos distribuidos entre las cinco categorías.
Usar el mismo catálogo determinista en cada ejecución, títulos y autores con coincidencias conocidas.
Con API y PostgreSQL en modo de ejecución normal y reloj real, medir desde un cliente externo a la API
cuatro búsquedas: sin filtros, texto en título, texto en autor y categoría más texto; todas con `size=20`.

Por búsqueda: 20 solicitudes de calentamiento excluidas, luego 100 medidas secuenciales sin caché
de respuestas, sin reutilizar resultados en memoria. Se permite conexión HTTPS persistente.
Medir desde envío hasta recepción completa con reloj monotónico. Guardar los 400 tiempos, máximo,
mediana y p95, códigos HTTP, versión/commit, consultas exactas, CPU/RAM y ubicación de cliente/servidor.
No descartar muestras lentas ni fallidas. **Cada muestra debe ser exitosa y menor de 300 ms**;
el p95 no sustituye el umbral original. Registrar también un recorrido con diez clientes simultáneos
como diagnóstico, sin convertirlo en un nuevo umbral de aceptación de RNF-1.
Evidencia futura: T-21 y RNF-1; una demostración de 40 libros no acredita rendimiento.

## C-08 · Unicidad de socios (D-16)

`Member` valida documento obligatorio y sus límites de representación. La aplicación consulta
duplicados para dar un error claro; la restricción UNIQUE de `member.document_number` impone
la unicidad global, incluidas altas concurrentes. El adaptador traduce una violación de esa
restricción a `DOCUMENT_TAKEN`, no cualquier error SQL. Se conserva CA-05.1 y no se permite borrar socios.
Evidencia futura: T-06/T-12/T-16; dos altas concurrentes del mismo documento dejan un socio, una 201 y otra 409.

## C-09 · Contrato y adaptación (D-12, D-14)

Los detalles del contrato se completan en los dos archivos `api-contract.md`, que siguen siendo
la fuente de verdad HTTP. Las versiones y el mapa concreto se registran en
[adaptacion-react-laravel.md](adaptacion-react-laravel.md), sin copiar el esquema ni las rutas.
No se atribuye una funcionalidad nueva a un detalle de serialización o manejo de errores.

## C-10 · Restricciones de reglas existentes (D-17)

La segunda revisión detecta invariantes ya descritas que no figuraban en la lista de restricciones.
Se registran como pendientes de T-18: ISBN de 13 dígitos en libro/línea, título no vacío,
portada nula o no vacía, usuario normalizado y documento obligatorio. T-18 conserva también
total positivo y rol válido. No se cambia la estructura de tablas ni se añade validación de
checksum ISBN. Las expresiones SQL exactas y su evidencia vivirán en las migraciones de `api`;
el inventario de reglas permanece en `data-model.md`, sin prometer aplicación actual en el motor.

## Estado al cierre de la revisión previa

Los hallazgos D-04 a D-09, D-11, D-12, D-14, D-16 y D-17 tienen decisiones documentales.
D-13 y las observaciones singular/plural permanecen reportadas sin movimientos.
La aceptación del PR fija esta base documental; no acredita ninguna tarea de implementación.
En aquella revisión, antes de ejecutar pruebas debía estar disponible el runtime previsto
y el motor de Docker: PHP/Composer no se encontraron en PATH y Docker no respondió como servidor.
La evidencia posterior de implementación y ejecución se consulta en el [registro SDD](registro-sdd.md).
Las decisiones C-01 a C-10 se mantienen; esta actualización distingue su adopción documental
de las garantías que aún deben verificar los adaptadores.
