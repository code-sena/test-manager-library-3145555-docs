# Revisión del proceso SDD y correcciones

Revisión del 14 de octubre de 2026. Rama documental creada desde `main`, commit
`eab4cf729f749c67923c1615cb5f981b46c5ae39`. La lectura de los requisitos y las evidencias
existentes precede a estas correcciones. Esta revisión conserva el producto, las seis
entidades, las rutas y las observaciones singular/plural.

## Interpretación del propósito de la prueba

El [README](README.md) evalúa cómo se lee, interpreta y aplica la especificación al stack
React/Laravel. El [plan](spec-python/plan.md) §4 también exige construir una implementación
en un orden definido. El trabajo de código es parte de la prueba: cada decisión debe poder
explicarse desde un requisito y cada resultado debe tener evidencia con alcance explícito.

La auditoría y las decisiones previas aportan una base reutilizable. Las pruebas existentes
aportan evidencia del esqueleto, esquema y núcleo. Esta revisión corrige el seguimiento y
las afirmaciones de aceptación; no concluye que todos los cambios anteriores sean inválidos.
Una revisión posterior de código deberá registrar las desviaciones concretas que encuentre.
No se identifica una supuesta trampa intencional sin evidencia.

## Fuentes y vigencia

- Negocio: `spec.md`; principios: `constitution.md`; esquema: `data-model.md`;
  HTTP: `api-contract.md`. Las dos versiones conservan su equivalencia de negocio.
- Adaptación técnica: [adaptación React/Laravel](adaptacion-react-laravel.md) y
  [decisiones previas](decisiones-previas.md).
- Seguimiento vigente: [registro SDD](registro-sdd.md), con referencias a pruebas y commits
  del repositorio que ejecuta la tarea. Los archivos de auditoría inicial conservan su
  contexto histórico.

El registro es una foto de la revisión, no una certificación global de aceptación.
Cada actualización deberá declarar los commits revisados y distinguir una rama publicada,
un PR creado, un PR fusionado y una prueba ejecutada. Los estados se apoyan en evidencia.

## Hallazgos y correcciones

| ID | Evidencia observada | Corrección y cierre requerido |
|---|---|---|
| SDD-01 | `docs` mantiene afirmaciones de estado previo a T-01; la evidencia de T-01 a T-07 está distribuida en `api/evidence` | Este PR añade el registro central y enlaza las evidencias por commit. Las afirmaciones históricas quedan fechadas y diferenciadas del seguimiento vigente |
| SDD-02 | `api/evidence/t-07.md`, fila «CA-01.3 / CA-02.1», usa la prueba de autorización de CreateLibrarianService | Este PR separa los criterios en el registro: CA-02.1 tiene evidencia de aplicación; CA-01.3 sigue pendiente de T-16. El próximo PR de API debe corregir esa fila en su archivo original |
| SDD-03 | Artículo IX exige gitleaks o una búsqueda de patrones en CI; los dos workflows de API revisados no incluyen ese paso | Pendiente en API: añadir el control de secretos, probar que un ejemplo sintético prohibido lo hace fallar y retirar el ejemplo del estado final. Registrar herramienta/patrones, alcance y resultado. Esto no acredita por sí solo logs ni transporte de RNF-3 |
| SDD-04 | DomainBoundaryTest recorre tokens de nombres cualificados; no verifica todos los imports cortos, llamadas a helpers ni instanciación de infraestructura fuera de composición | Evidencia inicial limitada. Pendiente ampliar el detector con casos prohibidos y comprobar los artículos I–III; el aislamiento ejecutado cubre un recorrido del dominio. T-19 continúa pendiente |
| SDD-05 | Los encabezados T-06/T-07 hablan de historias que también incluyen códigos HTTP, unicidad real y seguridad | Este PR precisa el alcance de pruebas con fakes en ambos tasks.md. La aceptación de historia exige además las capas y pruebas correspondientes; el resultado de una tarea del núcleo no cierra automáticamente la historia |
| SDD-06 | Adaptación y decisiones previas contienen un diagnóstico local anterior a T-01 | Este PR identifica ese diagnóstico como histórico y remite al registro. Las ejecuciones verificadas fueron en GitHub Actions; no acreditan ejecución local ni T-20 |
| SDD-07 | El README fija el corte en 2026-10-14 11:00, America/Bogota (16:00 UTC) | El reloj consultado en esta revisión marcó 16:00:03 UTC. Los commits nuevos de esta corrección son posteriores al corte. Se conserva la regla y se informa la limitación; no se altera la fecha ni se afirma que este PR cuente para la entrega |

## Secuencia para corregir la implementación

1. Fusionar este PR de documentación para fijar el registro y el procedimiento acordados.
2. En una rama de API, corregir la atribución de CA-01.3 en la evidencia T-07 e incorporar
   la verificación del artículo IX. Revisar los límites de arquitectura identificados en SDD-04.
   Cada corrección debe aportar una prueba que distinga el defecto del comportamiento permitido.
3. Contrastar T-01 a T-07 con sus criterios y pruebas concretas, registrando defectos
   reproducibles y cambios necesarios. Conservar el código que cumpla la especificación.
4. Actualizar el registro con los resultados, commits y situación de los PR.
5. Retomar T-08 desde la base integrada de API, con análisis y escenarios previos documentados.

Esta secuencia no cambia las dependencias T-01 a T-23 ni incorpora tareas de producto.
Las correcciones de gobierno y evidencia atienden principios ya exigidos por la constitución.
Los siguientes PR se evalúan sobre su resultado final, sin declarar que todo el proyecto está terminado.

## Procedimiento SDD para cada avance

1. **Leer y delimitar:** identificar HU, CA, RN, RNF y artículos afectados; registrar también
   lo que corresponde a capas posteriores. Consultar las dos versiones ante diferencias.
2. **Resolver antes del código:** documentar ambigüedades, alternativas y consecuencias.
   Una propuesta nueva debe distinguirse de una regla ya fijada. Conservar el alcance;
   singular/plural se reporta sin renombrar.
3. **Diseñar la verificación:** relacionar cada comportamiento esperado y caso de rechazo
   con una prueba prevista, su nivel y los puertos/adaptadores necesarios.
4. **Implementar por tarea:** cambios acotados, commits continuos y adaptaciones técnicas
   justificadas. El dominio permanece independiente de Laravel.
5. **Ejecutar y registrar:** conservar commit, entorno, prueba y resultado reproducible.
   El estado parcial sigue indicando las garantías que faltan.
6. **Preparar el PR y mantener trazabilidad:** título y descripción en inglés; documentación
   y mensajes en español. El PR explica problema, decisión, cambio, verificación y límites.
   Registrar publicación y fusión por separado.

La fecha de escritura de una evidencia retrospectiva debe reconocerse. Añadirla después
no demuestra que el análisis se hubiera documentado antes del código.
Este procedimiento hace visible el razonamiento requerido por SDD; no impone TDD como
una condición nueva del examen.

Las comprobaciones documentales de este PR y sus límites se registran en
[verificación SDD](verificacion-sdd.md).

La [segunda revisión completa](segunda-revision-sdd.md) registra la explicación corregida
de ADR-002, un caso de fechas pendiente de prueba y los escenarios previstos para corregir API.
