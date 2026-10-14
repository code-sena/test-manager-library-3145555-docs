# Contrato de la API — Manager Library

El contrato HTTP es **independiente del lenguaje**: quien lo implemente en React + Laravel debe respetarlo igual que quien lo
implemente en Python o .NET. Qué hace cada operación está en [`spec.md`](spec.md); qué se guarda, en [`data-model.md`](data-model.md).

## 1. Convenciones

| Tema | Regla |
|---|---|
| Base | `/api/v1` |
| Formato | JSON, UTF-8. Nombres de campo en `snake_case` |
| Identificadores | UUID v4 como texto |
| Fechas | ISO 8601 en UTC, con `Z` (`2026-10-14T14:15:00Z`) |
| Dinero | No hay. El sistema no maneja importes |
| Autenticación | `Authorization: Bearer <token>` en todo salvo `POST /auth/login` y `GET /health` |
| Paginación | `?page=1&size=20`. `size` máximo 50. La respuesta es `{ "items": [...], "page": 1, "size": 20, "total": 134 }` |
| Idioma | Mensajes de error en español; campos y códigos en inglés |

### Cuerpo de error (único para todos los errores)

```json
{
  "code": "BOOK_NOT_AVAILABLE",
  "message": "No quedan ejemplares disponibles de «Cien años de soledad».",
  "details": [ { "field": "items[0].book_id", "issue": "sin ejemplares" } ]
}
```

`details` puede ir vacío. **Nunca** incluye trazas, SQL, rutas del servidor ni la contraseña.

### Códigos de estado

| Estado | Cuándo |
|---|---|
| 200 / 201 / 204 | Éxito / creado / sin contenido |
| 401 | Sin token, token inválido o vencido, credenciales inválidas |
| 403 | Rol insuficiente |
| 404 | No existe (o está dado de baja) |
| 409 | Conflicto de estado: duplicado, sin ejemplares, ya devuelto, socio con vencidos, libro con préstamos abiertos |
| 422 | Validación: campo inválido, regla de entrada rota, rango de fechas invertido |

## 2. Códigos de error de negocio

| `code` | Estado | Origen |
|---|---|---|
| `INVALID_CREDENTIALS` | 401 | CA-01.2 |
| `FORBIDDEN_ROLE` | 403 | CA-01.3 |
| `NOT_FOUND` | 404 | Cualquier recurso |
| `USERNAME_TAKEN` | 409 | CA-02.2 |
| `ISBN_TAKEN` | 409 | CA-03.2 |
| `DOCUMENT_TAKEN` | 409 | CA-05.1 |
| `BOOK_HAS_OPEN_LOANS` | 409 | CA-03.5 |
| `BOOK_NOT_AVAILABLE` | 409 | CA-06.3 |
| `MEMBER_HAS_OVERDUE` | 409 | CA-06.5 |
| `ITEM_ALREADY_RETURNED` | 409 | CA-07.2 |
| `MEMBER_LIMIT_EXCEEDED` | 422 | CA-06.4 |
| `DUPLICATE_BOOK_IN_LOAN` | 422 | CA-06.2 |
| `VALIDATION_ERROR` | 422 | Cualquier campo inválido |
| `INVALID_DATE_RANGE` | 422 | CA-10.2 |

## 3. Endpoints

### 3.1 Autenticación y salud

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/auth/login` | público | Devuelve token |
| GET | `/health` | público | `{ "status": "ok" }` |

`POST /auth/login`

```json
// petición
{ "username": "ana", "password": "********" }
// 200
{ "access_token": "eyJ...", "token_type": "Bearer", "expires_in": 28800, "role": "librarian" }
```

### 3.2 Usuarios

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/users` | admin | Crea un bibliotecario |

```json
// petición  (role solo admite "librarian")
{ "username": "Ana ", "password": "********", "role": "librarian" }
// 201  (sin password ni password_hash)
{ "id": "…", "username": "ana", "role": "librarian" }
```

### 3.3 Categorías

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/categories` | autenticado | Las cinco categorías, orden por nombre |

### 3.4 Libros

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/books` | autenticado | Búsqueda: `q`, `category_id`, `page`, `size` |
| GET | `/books/{id}` | autenticado | Detalle |
| POST | `/books` | admin | Crea |
| PUT | `/books/{id}` | admin | Reemplaza título, autor, categoría y total |
| DELETE | `/books/{id}` | admin | Baja lógica |
| PUT | `/books/{id}/cover` | admin | Sube portada (`multipart/form-data`, campo `file`) |
| DELETE | `/books/{id}/cover` | admin | Quita la portada |
| GET | `/books/{id}/cover` | autenticado | Devuelve el binario de la portada o 404 |

Libro (respuesta):

```json
{
  "id": "…", "title": "Cien años de soledad", "author": "Gabriel García Márquez",
  "isbn": "9780307474728", "category": { "id": "…", "name": "Novela" },
  "total_copies": 4, "available_copies": 3, "cover_key": null
}
```

`POST /books` (petición): `title`, `author`, `isbn`, `category_id`, `total_copies`. Los disponibles los calcula el servidor.
`cover_key` es una clave opaca: para pintar la portada se usa `GET /books/{id}/cover`, que devuelve el binario o 404.

### 3.5 Socios

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/members` | admin, librarian | Lista y busca: `q`, `page`, `size` |
| GET | `/members/{id}` | admin, librarian | Detalle con `active_loans` (conteo sin devolver) |
| POST | `/members` | admin, librarian | Crea: `full_name`, `document_number` |

### 3.6 Préstamos

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | `/loans` | admin, librarian | Registra un préstamo |
| GET | `/loans` | admin, librarian | Lista: `from`, `to`, `status`, `member_id`, `page`, `size` |
| GET | `/loans/{id}` | admin, librarian | Detalle con líneas y estado |
| POST | `/loans/{loan_id}/items/{item_id}/return` | admin, librarian | Devuelve una línea |
| GET | `/loans/overdue` | admin, librarian | Líneas vencidas |

`POST /loans`

```json
// petición  (1 a 3 elementos, sin libros repetidos)
{ "member_id": "…", "items": [ { "book_id": "…" } ] }
// 201
{
  "id": "…", "member_id": "…", "loaned_at": "2026-10-14T14:15:00Z", "status": "open",
  "items": [
    { "id": "…", "book_id": "…", "book_title": "Cien años de soledad", "isbn": "9780307474728",
      "category_name": "Novela", "due_at": "2026-10-28T14:15:00Z", "returned_at": null }
  ]
}
```

`status` es calculado: `open` (hay líneas sin devolver y ninguna vencida), `overdue` (alguna sin devolver y vencida), `closed` (todas devueltas).

`POST /loans/{loan_id}/items/{item_id}/return` → 200 con la línea (`returned_at` lleno).

`GET /loans/overdue` → `items`: `{ "loan_id", "item_id", "book_title", "member_name", "due_at", "days_late" }`. Aquí sí se muestra el nombre del
socio, porque es un listado operativo; **el reporte del §3.7 no**.

### 3.7 Reporte

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| GET | `/reports/most-borrowed` | admin | `from` y `to` obligatorios (fecha, ambos inclusive) |

```json
{
  "from": "2026-10-01", "to": "2026-10-31",
  "rows": [
    { "book_id": "…", "book_title": "Cien años de soledad", "category_name": "Novela", "times_borrowed": 12 }
  ]
}
```

Una fila por `book_id + book_title + category_name` congelados. Orden: `times_borrowed` descendente, luego título. Sin socios ni bibliotecarios.

## 4. Seguridad

| Aspecto | Regla |
|---|---|
| Token | JWT firmado, 8 horas, con `sub` y `role`. Sin datos personales dentro |
| Contraseñas | Solo se guarda el hash; el hash nunca sale en ninguna respuesta |
| Autorización | Se evalúa en cada endpoint por rol; un 403 no revela si el recurso existe |
| CORS | Solo el origen del frontend configurado por entorno |
| Subida de portada | Solo `image/jpeg` y `image/png`, máx. 2 MB; el tipo se valida por contenido, no por extensión |

## 5. Precisiones de serialización y validación (D-12)

Estas decisiones completan formas y casos ya contemplados. No añaden endpoints ni operaciones
sobre el producto. Los ejemplos anteriores con comentarios son ilustrativos: los comentarios
no forman parte de las peticiones JSON que se envían.

### 5.1. Entradas y campos del servidor

| Operación | Campos de entrada requeridos | Respuesta de éxito |
|---|---|---|
| POST `/users` | `username`, `password`, `role` (solo `librarian`) | 201, objeto de usuario de §3.2 |
| POST `/books` | `title`, `author`, `isbn`, `category_id`, `total_copies` | 201, objeto libro de §3.4 |
| PUT `/books/{id}` | `title`, `author`, `category_id`, `total_copies` | 200, objeto libro de §3.4; ISBN no se edita |
| DELETE `/books/{id}` | sin cuerpo | 204, sin cuerpo; mantiene las restricciones de baja lógica |
| PUT `/books/{id}/cover` | multipart, campo `file` | 200, objeto libro con `cover_key` opaca |
| DELETE `/books/{id}/cover` | sin cuerpo | 204, sin cuerpo; `cover_key` pasa a null |
| POST `/members` | `full_name`, `document_number` | 201, objeto base de socio descrito abajo |
| POST `/loans` | `member_id`, `items` (de 1 a 3 objetos con `book_id`) | 201, objeto préstamo de §3.6 con una línea por cada libro |
| POST `/loans/{loan_id}/items/{item_id}/return` | sin cuerpo | 200, objeto de línea de §3.6 con `returned_at` |

No se aceptan como entrada campos asignados por el servidor (`id`, `available_copies`,
`cover_key`, fechas, `status`, valores congelados o usuario que registra el préstamo).
Las entradas ajenas al esquema y tipos inválidos responden 422 `VALIDATION_ERROR`.
Los textos obligatorios no pueden quedar vacíos después de normalizar; las longitudes máximas
son las del modelo de datos. `password` es una cadena obligatoria no vacía: no se recorta,
no se transforma y no se incorpora una política de complejidad nueva. `username` se recorta
y pasa a minúsculas. El ISBN se valida por 13 dígitos, sin introducir checksum adicional.
UUIDs mal formados son 422; UUIDs válidos sin recurso son 404. Los ejemplos con `…` son marcadores.

### 5.2. Objetos y colecciones

Objeto base de socio, usado al crear y en listas:

```json
{ "id": "…", "full_name": "María Pérez", "document_number": "123456" }
```

Detalle de socio, con el campo ya previsto en §3.5:

```json
{ "id": "…", "full_name": "María Pérez", "document_number": "123456", "active_loans": 2 }
```

`active_loans` cuenta **líneas sin devolver**, incluidas las vencidas; no cabeceras de préstamo.
Se conserva su nombre aunque use plural y aunque el conteo represente libros.

- GET `/books`, `/members` y `/loans` usan el sobre paginado de §1; cada elemento es el objeto
  libro, socio base o préstamo de sus secciones. Préstamos incluyen todas sus líneas y el estado calculado.
- GET `/categories` devuelve `{"items":[{"id":"…","name":"Ciencia"}]}` con las cinco categorías
  ordenadas por nombre. La forma muestra un elemento ilustrativo; la respuesta real tiene los cinco.
  Es una colección fija sin parámetros de paginación.
- GET `/loans/overdue` devuelve `{"items":[...]}` con los campos exactos de §3.6, sin paginación
  no especificada. Las listas vacías conservan `items: []`.
- `page` y `size` son enteros positivos, por defecto 1 y 20; `size` mayor que 50 es 422.
  Una página posterior al último resultado devuelve `items: []` y el total real, no 404.
- Empates de libros se ordenan por título e ID; préstamos por fecha descendente e ID;
  vencidos por días de retraso descendentes, luego `due_at` ascendente e `item_id`.

La ruta literal `/loans/overdue` se resuelve antes que `/loans/{id}`.
La pertenencia de `item_id` al `loan_id` se valida: una línea de otro préstamo es 404.
GET de portada devuelve bytes con Content-Type `image/jpeg` o `image/png`; los errores mantienen JSON.
React solicita el binario con Bearer y lo muestra mediante una URL de objeto temporal; no se coloca
el JWT en la URL ni se interpreta `cover_key` como dirección pública.

### 5.3. Fechas, vencimiento y conteos

`from` y `to` son fechas `YYYY-MM-DD`. En préstamos son filtros opcionales sobre `loaned_at`;
en reporte son obligatorios. Ambos extremos son inclusivos: se traduce el final a un límite
exclusivo del inicio del día siguiente en UTC. Fecha inválida es 422 `VALIDATION_ERROR`;
rango invertido es 422 `INVALID_DATE_RANGE`. El filtro `status` admite `open`, `overdue`, `closed`.

Una línea está vencida solo si `returned_at` es null y `due_at < Clock.now()`; la igualdad no vence.
`days_late` es el número entero de periodos completos de 24 horas desde `due_at`, calculado
como floor de segundos de retraso / 86400. Una línea vencida hace menos de 24 horas muestra 0;
sigue vencida por el instante, no por ese entero. No se almacena ninguno de esos valores.

`MEMBER_LIMIT_EXCEEDED` conserva el sobre único; `details` informa el conteo actual sin agregar
un campo nuevo al sobre, por ejemplo:

```json
{
  "code": "MEMBER_LIMIT_EXCEEDED",
  "message": "El préstamo superaría el máximo de 3 libros sin devolver.",
  "details": [{ "field": "items", "issue": "El socio tiene actualmente 2 libros sin devolver." }]
}
```

### 5.4. Errores de autenticación y técnicos

| `code` | Estado | Caso |
|---|---|---|
| `UNAUTHENTICATED` | 401 | Bearer ausente, inválido o vencido; mismo mensaje genérico |
| `CONCURRENT_MODIFICATION` | 409 | Conflicto de escritura que persiste tras el reintento completo; en préstamos se conserva ADR-002 y `BOOK_NOT_AVAILABLE` |
| `METHOD_NOT_ALLOWED` | 405 | Método no permitido en una ruta existente |
| `INTERNAL_ERROR` | 500 | Error inesperado o defecto de integridad; mensaje genérico sin trazas |

`INVALID_CREDENTIALS` permanece para login incorrecto, sin revelar qué dato falló.
Los códigos técnicos completan el sobre de error; no añaden una funcionalidad de producto.
La API evalúa autenticación y rol antes de revelar existencia de un recurso protegido.
JSON mal formado se traduce a 422 `VALIDATION_ERROR`; rutas inexistentes a 404 `NOT_FOUND`.
Los errores de framework nunca se devuelven con su formato por defecto. Un fallo CHECK no es
un conflicto de disponibilidad normal. No se transforma indiscriminadamente todo error de la base en 409.

### 5.5. Transporte

Todas las llamadas de credenciales y su evidencia de aceptación usan HTTPS con confianza TLS
validada por el cliente. La terminación TLS se configura en `infra`; los secretos no se incluyen
en ejemplos reales, commits ni logs. La interpretación y evidencia de RNF-3 están en
[C-06](../decisiones-previas.md). El cuerpo de login de §3.1 se conserva.
