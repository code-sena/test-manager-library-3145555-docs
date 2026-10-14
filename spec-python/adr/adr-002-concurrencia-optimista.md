# ADR-002 · Concurrencia optimista sobre los ejemplares

- **Estado:** aceptada
- **Contexto:** dos bibliotecarios pueden prestar el **último ejemplar** del mismo libro al mismo tiempo (RNF-2). Si ambos leen
  `available_copies = 1` y ambos escriben, el sistema presta un libro que no tiene.
- **Decisión:** control de concurrencia **optimista**. El libro lleva un testigo de versión; al guardar, si la versión cambió, la
  operación falla y el servidor responde **409 `BOOK_NOT_AVAILABLE`** (o reintenta una vez si el ejemplar sigue disponible).
  La **última barrera** es el `CHECK` `ck_book_available_range` (`0 <= available_copies <= total_copies`).
- **Alternativas descartadas:** bloqueo pesimista de la fila (`SELECT … FOR UPDATE`) en cada préstamo: obliga a esperar a los escritores
  incompatibles sobre los mismos libros; se conserva la elección optimista. Contador en memoria: no sobrevive a más de un proceso.
- **Consecuencias:** si el `CHECK` salta, **algo escribió fuera del adaptador**: es un defecto, no un caso de negocio. Devolver dos veces
  la misma línea no suma dos veces porque `returned_at` ya está lleno (CA-07.2).
- **Cómo se verifica:** prueba concurrente con dos peticiones simultáneas sobre un libro con un ejemplar: exactamente una recibe 201.

## Precisión de implementación para el reto

El testigo es `book.xmin::text`, metadato nativo de PostgreSQL, sin columna nueva ni exposición HTTP.
La comparación se aplica a la escritura del libro. El CHECK sigue siendo barrera de integridad;
no sustituye la comparación y no detecta por sí solo una devolución doble con otras copias prestadas.
El protocolo completo está en [C-01 a C-03](../../decisiones-previas.md): transacción, coordinación por
socio, lectura posterior al bloqueo, devolución condicional y reintento completo acotado.
La coordinación consultiva por socio no sustituye la decisión optimista para los libros.
T-13 debe probar último ejemplar, límite concurrente por socio y devolución simultánea.

La explicación del bloqueo pesimista se precisa sin cambiar la decisión: un bloqueo de fila
afecta a escritores y bloqueadores incompatibles de esa fila; no serializa por sí solo toda
la biblioteca. Fuente: [PostgreSQL 16, bloqueos de fila](https://www.postgresql.org/docs/16/explicit-locking.html#LOCKING-ROWS).
