# Segunda revisión del repositorio de documentación

Fecha: 14 de octubre de 2026. Base de esta revisión: docs `39ab58f` en
`feature/docs-sdd-review`; main remoto observado `eab4cf7`.
La revisión SDD todavía no está fusionada en la base remota observada.
API se contrastó en `8b3a44b` (rama auth) y origin/main `a7843b0`.

## Lectura y comparación

Se revisaron los 31 documentos existentes: README, auditoría, adaptación, decisiones,
revisión, registro, verificación y los doce documentos de cada versión del spec.
La comparación de ambas versiones encontró:

- `spec.md`, `api-contract.md`, `data-model.md` y los cuatro ADR son iguales.
- Las diferencias de plan, constitución, arquitectura y panorama corresponden a los
  stacks de referencia y sus herramientas. El mapa .NET inconsistente permanece reportado
  en D-13; no se mueven carpetas.
- tasks.md comparte requisitos y dependencias; conserva el encabezado y la herramienta
  de arquitectura propios de cada referencia.
- React/Laravel, PHP nativo para Domain/Application y PostgreSQL 16 son la adaptación
  documentada. Los actores, seis tablas y contrato conservan su alcance.

El registro cubre los 38 CA, 23 tareas, 6 RN y 5 RNF. Esa cobertura de identificadores
no demuestra por sí sola cumplimiento del producto; cada fila conserva su evidencia parcial.

## Nuevos hallazgos

| ID | Evidencia y consecuencia | Disposición |
|---|---|---|
| SDD-08 | ADR-002 explicaba que SELECT FOR UPDATE sobre una fila serializaba toda la biblioteca. PostgreSQL describe bloqueo de las operaciones incompatibles sobre las filas afectadas | Se corrige la explicación en ambas versiones y se añade fuente oficial. Se conserva control optimista, xmin, coordinación por socio, reintento y resultados del contrato |
| SDD-09 | DateRange de API invoca createFromFormat directamente y solo comprueba false/formato. PHP documenta ValueError ante un byte nulo. ValuesTest no incluye ese caso | Hallazgo por inspección y documentación oficial, pendiente de reproducción ejecutable. Añadir una prueba de regresión para from y to y traducir la entrada inválida a ValidationError; el dominio no debe filtrar una excepción nativa de análisis |
| SDD-10 | La tabla de portadas expresa «2 MB» sin precisar bytes; una prueba justo en el límite necesita esa cifra | Ambigüedad de unidad reportada para T-08/T-15. Documentar el umbral antes de implementar la frontera; no cambiarlo silenciosamente a 2 MiB ni declarar un valor nuevo aprobado en esta revisión |

Fuentes contrastadas:
[bloqueos de fila de PostgreSQL 16](https://www.postgresql.org/docs/16/explicit-locking.html#LOCKING-ROWS)
y [errores de DateTimeImmutable::createFromFormat](https://www.php.net/manual/en/datetimeimmutable.createfromformat.php#refsect1-datetimeimmutable.createfromformat-errors).
No se ejecutó un runtime PHP durante esta revisión documental.
La necesidad de tratar SDD-09 se deduce del código y del comportamiento documentado;
su prueba en CI deberá preceder al cierre del hallazgo.

## Límites de interpretación que se mantienen

- SDD implica lectura, decisiones justificadas, implementación y evidencia trazable.
  El plan requiere código; una matriz retrospectiva no acredita análisis previo por sí sola.
- Un fake de TokenIssuer no acredita JWT firmado ni validación temporal.
  Un fake de PasswordHasher no acredita Argon2id ni logs seguros.
- Una autorización en CreateLibrarianService no acredita CA-01.3 en todas las rutas.
- El resultado inmutable de Loan no acredita una transacción PostgreSQL ni concurrencia.
- La introspección describe lo que existe en el motor. Una discrepancia debe registrarse
  y explicarse; un cambio manual contrario a ADR-001 no se legitima relajando el negocio.
- S3 aparece como alternativa técnica en los specs; T-15 pide disco. No se incorpora un
  servicio cloud nuevo para este avance.
- El orden de portada sigue siendo confirmar cover_key nula y luego borrar el binario.
  La revisión no cambia esa regla ni propone IO externo dentro de la transacción.
- Las observaciones singular/plural permanecen registradas sin renombramientos.

## Preparación del próximo PR de API

Rama prevista: `feature/api-sdd-fixes`, sobre la base integrada que corresponda al abrirla.
Verificar primero el estado de auth; evitar presentar su código como fusionado si solo está publicado.
Los siguientes son escenarios previstos, todavía no resultados ejecutados:

| Corrección | Fuente | Prueba y resultado esperado |
|---|---|---|
| Corregir evidencia T-07 | SDD-02, CA-01.3, tasks T-07/T-16 | La matriz original separa autorización del alta y protección de todos los endpoints; CA-01.3 permanece pendiente |
| Control de secretos en CI | SDD-03, artículo IX | Ejemplo sintético generado temporalmente detectado; checkout final sin ese ejemplo pasa. Scan documenta alcance y exclusiones justificadas, sin imprimir secretos |
| Límites del núcleo | SDD-04, artículos I/II | Imports cortos/cualificados, aliases, tipos externos y helpers prohibidos se rechazan; PHP nativo y dependencias internas permitidas pasan |
| Composición única | SDD-04, artículo III | Instanciar o vincular infraestructura fuera del punto autorizado falla; una composición válida pasa. Lo aún inexistente de T-17 no se declara implementado |
| Fecha con byte nulo | SDD-09, contrato §5.3 y T-03 | Cada extremo con byte nulo produce ValidationError de from/to, no ValueError; fechas válidas, fin inclusivo y rango invertido conservan su comportamiento |

Las correcciones no añaden endpoints ni alteran producto. El PR deberá informar resultados
reales, límites y commits. Solo después de revisar estos defectos se retoma T-08, dejando
SDD-10 precisado antes de probar la frontera de portada.

## Estado de cierre

La explicación técnica SDD-08 queda corregida en docs. SDD-02 a SDD-04 y SDD-09 siguen
pendientes en API; SDD-10 permanece como precisión pendiente de la frontera de portadas.
La revisión reduce incertidumbre y prepara correcciones verificables. No certifica ausencia
de más defectos ni identifica una trampa intencional. El corte de entrega del README se conserva.
