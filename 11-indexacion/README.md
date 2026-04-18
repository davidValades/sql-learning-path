# ⚡ Tema 11: Indexación

> **"Sin índices, buscar un dato en una tabla de un millón de filas es como buscar una palabra en un libro leyendo página por página."** Los índices son la diferencia entre una consulta que tarda 30 segundos y una que responde al instante. Dominarlos es dominar el rendimiento.

## 📋 Índice

- [11.1 ¿Qué es un Índice?](#111-qué-es-un-índice)
- [11.2 CREATE INDEX](#112-create-index)
- [11.3 Tipos de Índices](#113-tipos-de-índices)
- [11.4 Cuándo Crear y Cuándo NO Crear Índices](#114-cuándo-crear-y-cuándo-no-crear-índices)
- [11.5 Impacto en INSERT/UPDATE/DELETE](#115-impacto-en-insertupdatedelete)
- [11.6 DROP INDEX y REBUILD INDEX](#116-drop-index-y-rebuild-index)

---

---

## 11.1 ¿Qué es un Índice?

### 📘 El Concepto

Un **índice** es una estructura auxiliar que Oracle mantiene separada de la tabla y que permite localizar filas **sin leer toda la tabla** (lo que se llama *full table scan*).

Internamente, la mayoría de los índices en Oracle usan una estructura de **árbol B (B-Tree)**:

```
                    [M]
                   /   \
              [D, H]    [R, V]
             / |  \    / |  \
           [A-C][E-G][I-L][N-Q][S-U][W-Z]
            ↓    ↓    ↓    ↓    ↓    ↓
          ROWID ROWID ROWID ROWID ROWID ROWID
```

- **Nodo raíz:** punto de entrada.
- **Nodos intermedios:** guían la búsqueda (como un árbol de decisiones).
- **Nodos hoja:** contienen el valor indexado + un puntero (`ROWID`) a la fila real en la tabla.

**Sin índice:** Oracle lee **todas** las filas de la tabla (Full Table Scan).  
**Con índice:** Oracle navega el árbol en 3-4 pasos y salta directamente a la fila (Index Scan).

> 📌 Oracle crea **automáticamente** un índice para cada `PRIMARY KEY` y cada restricción `UNIQUE`. No necesitas crearlos manualmente.

### 🏠 La Analogía

Piensa en el **índice de un libro** (las páginas del final que dicen "Subconsultas → pág. 145"). Si buscas "Subconsultas" sin índice, tienes que leer el libro completo página por página. Con el índice, vas directamente a la página 145. El índice ocupa unas páginas extra en el libro, pero te ahorra horas de búsqueda.

### 💻 El Código

```sql
-- Sin índice: Oracle hace un Full Table Scan
-- EXPLAIN PLAN nos muestra cómo Oracle ejecuta la consulta
EXPLAIN PLAN FOR
SELECT * FROM productos WHERE precio > 100;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
-- Operation: TABLE ACCESS FULL (lee las 8 filas)

-- Con índice: Oracle usa un Index Range Scan
CREATE INDEX idx_productos_precio ON productos(precio);

EXPLAIN PLAN FOR
SELECT * FROM productos WHERE precio > 100;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
-- Operation: INDEX RANGE SCAN + TABLE ACCESS BY INDEX ROWID
-- Oracle salta directamente a los productos con precio > 100

-- Ver índices existentes de una tabla
SELECT index_name, column_name, uniqueness
FROM user_ind_columns uic
JOIN user_indexes ui ON uic.index_name = ui.index_name
WHERE uic.table_name = 'PRODUCTOS';
-- Resultado incluye el índice de la PK (automático) + idx_productos_precio
```

### 🧠 El Reto

Si la tabla `pedidos` tiene 10 millones de filas y ejecutas `SELECT * FROM pedidos WHERE fecha_pedido = DATE '2024-01-15'`, ¿qué tipo de operación hace Oracle sin índice? ¿Y con un índice en `fecha_pedido`?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

- **Sin índice:** `TABLE ACCESS FULL` — Oracle lee las 10 millones de filas una por una buscando las que coincidan con esa fecha. Puede tardar minutos.
- **Con índice en `fecha_pedido`:** `INDEX RANGE SCAN` — Oracle navega el B-Tree en 3-4 pasos hasta la fecha `2024-01-15` y lee solo las filas que coinciden. Respuesta en milisegundos.

> 💡 La diferencia es abismal: de leer 10 millones de filas a leer quizás 50. Por eso los índices son críticos en tablas grandes.

</details>

---

---

## 11.2 CREATE INDEX

### 📘 El Concepto

La sintaxis básica para crear un índice:

```
CREATE INDEX nombre_indice ON tabla(columna);

-- Índice sobre múltiples columnas (compuesto)
CREATE INDEX nombre_indice ON tabla(columna1, columna2);

-- Índice único
CREATE UNIQUE INDEX nombre_indice ON tabla(columna);
```

**Convención de nombres:** `idx_tabla_columna` — facilita identificar a qué tabla y columna pertenece.

### 🏠 La Analogía

Crear un índice es como contratar a un bibliotecario que elabora un catálogo ordenado de tu biblioteca. Tarda un rato en crearlo (tiene que revisar todos los libros), pero una vez listo, encontrar cualquier libro es instantáneo. Si añades libros nuevos, el bibliotecario actualiza el catálogo automáticamente.

### 💻 El Código

```sql
-- E-commerce: índice en la columna de búsqueda más frecuente
CREATE INDEX idx_productos_categoria ON productos(id_categoria);
CREATE INDEX idx_pedidos_cliente ON pedidos(id_cliente);
CREATE INDEX idx_pedidos_fecha ON pedidos(fecha_pedido);

-- Hospital: índice para búsquedas por especialidad
CREATE INDEX idx_medicos_especialidad ON medicos(id_especialidad);

-- Aerolínea: índices para las JOINs más frecuentes
CREATE INDEX idx_vuelos_ruta ON vuelos(id_ruta);
CREATE INDEX idx_vuelos_avion ON vuelos(id_avion);

-- Verificar que se crearon
SELECT index_name, table_name, uniqueness
FROM user_indexes
WHERE table_name IN ('PRODUCTOS', 'PEDIDOS', 'MEDICOS', 'VUELOS')
ORDER BY table_name;
```

> 📌 **¿En qué columnas crear índices?** Principalmente en:
> - Columnas usadas en `WHERE` frecuentemente.
> - Columnas usadas en `JOIN` (claves foráneas).
> - Columnas usadas en `ORDER BY`.
> - Columnas usadas en `GROUP BY`.

### 🧠 El Reto

Mira estas tres consultas y decide qué índice crearías para cada una:

1. `SELECT * FROM citas WHERE estado = 'P';`
2. `SELECT * FROM productos WHERE id_categoria = 1 AND precio < 100;`
3. `SELECT * FROM vuelos ORDER BY fecha_salida;`

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- 1. Índice en estado (columna en WHERE)
CREATE INDEX idx_citas_estado ON citas(estado);

-- 2. Índice compuesto (ambas columnas en WHERE)
CREATE INDEX idx_productos_cat_precio ON productos(id_categoria, precio);

-- 3. Índice en fecha_salida (columna en ORDER BY)
CREATE INDEX idx_vuelos_fecha ON vuelos(fecha_salida);
```

> 💡 La consulta 2 se beneficia de un **índice compuesto** porque filtra por dos columnas simultáneamente. Oracle puede satisfacer ambas condiciones navegando un solo árbol.

</details>

---

---

## 11.3 Tipos de Índices

### 📘 El Concepto

Oracle soporta varios tipos de índices, cada uno optimizado para un caso de uso distinto:

| Tipo | Estructura | Mejor para | Ejemplo |
|------|-----------|------------|---------|
| **B-Tree** | Árbol balanceado | Alta cardinalidad (muchos valores distintos) | `precio`, `fecha`, `id_cliente` |
| **Bitmap** | Mapa de bits | Baja cardinalidad (pocos valores distintos) | `estado` (P/C/X), `sexo` (M/F) |
| **Unique** | B-Tree sin duplicados | Garantizar unicidad | `email`, `nif` |
| **Compuesto** | B-Tree multi-columna | Consultas que filtran por varias columnas | `(id_categoria, precio)` |
| **Function-Based** | B-Tree sobre expresión | Consultas con funciones | `UPPER(nombre)` |

```
-- B-Tree (por defecto)
CREATE INDEX idx_normal ON tabla(columna);

-- Bitmap
CREATE BITMAP INDEX idx_bitmap ON tabla(columna);

-- Unique
CREATE UNIQUE INDEX idx_unico ON tabla(columna);

-- Compuesto
CREATE INDEX idx_compuesto ON tabla(col1, col2);

-- Function-Based
CREATE INDEX idx_funcion ON tabla(UPPER(columna));
```

### 🏠 La Analogía

Piensa en diferentes tipos de catálogos de biblioteca:

- **B-Tree:** Un catálogo ordenado alfabéticamente — perfecto cuando cada libro tiene un título único o casi único.
- **Bitmap:** Un sistema de etiquetas de colores — si solo hay 3 géneros (Novela, Ciencia, Historia), pegas una etiqueta de color a cada libro. Encontrar "todos los de Ciencia" es instantáneo.
- **Compuesto:** Un catálogo con dos niveles — primero por género, luego por autor dentro de cada género. Perfecto si siempre buscas "Novela de García Márquez".
- **Function-Based:** Un catálogo donde los títulos están en mayúsculas — útil si la gente busca sin respetar mayúsculas/minúsculas.

### 💻 El Código

```sql
-- BITMAP: perfecto para la columna estado de citas (solo 3 valores: P, C, X)
CREATE BITMAP INDEX idx_citas_estado_bmp ON citas(estado);

-- Con 7 citas y 3 estados, el bitmap internamente se ve así:
-- estado 'C': 1 0 0 1 0 0 1   (citas 1,4,7 = Completadas)
-- estado 'P': 0 0 1 0 0 1 0   (citas 3,6 = Pendientes)
-- estado 'X': 0 0 0 0 1 0 0   (cita 5 = Cancelada)
-- Buscar todas las 'P' = leer el bitmap de 'P' → filas 3 y 6

-- COMPUESTO: búsquedas frecuentes por categoría + precio
CREATE INDEX idx_prod_cat_precio ON productos(id_categoria, precio);
-- Optimiza: WHERE id_categoria = 1 AND precio < 100
-- También optimiza: WHERE id_categoria = 1 (usa el prefijo)
-- NO optimiza: WHERE precio < 100 (no usa el prefijo)

-- FUNCTION-BASED: búsquedas case-insensitive de médicos
CREATE INDEX idx_medicos_nombre_upper ON medicos(UPPER(nombre_completo));

-- Ahora esta consulta usa el índice:
SELECT * FROM medicos WHERE UPPER(nombre_completo) = 'DRA. SARAH ADAMS';
-- Sin el índice function-based, Oracle haría Full Table Scan

-- UNIQUE: garantizar emails únicos (si no hay constraint UNIQUE)
-- CREATE UNIQUE INDEX idx_clientes_email ON clientes(email);
-- Si intentas insertar un email duplicado:
-- ORA-00001: unique constraint violated
```

> ⚠️ **Regla del prefijo en índices compuestos:** Un índice `(col1, col2)` se usa para filtrar por `col1`, o por `col1 + col2`, pero **NO** para filtrar solo por `col2`. El orden importa.

### 🧠 El Reto

¿Qué tipo de índice elegirías para cada columna? Justifica.

1. `aviones.modelo` (4 valores distintos en 4 filas)
2. `pedidos.total` (valores únicos, rango amplio)
3. `pacientes.nombre_completo` (búsquedas frecuentes con `LIKE UPPER(...)`)

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

1. **No crear índice** — Con solo 4 filas, un Full Table Scan es más rápido que usar cualquier índice. Los índices solo valen la pena en tablas con muchas filas.

2. **B-Tree** — Valores numéricos con alta cardinalidad y consultas de rango (`WHERE total > 500`). Es el caso ideal para B-Tree.

3. **Function-Based** — `CREATE INDEX idx_pac_nombre ON pacientes(UPPER(nombre_completo));` para que consultas como `WHERE UPPER(nombre_completo) LIKE 'LAURA%'` usen el índice.

> 💡 El tipo de índice depende de la cardinalidad de la columna, el tamaño de la tabla y el tipo de consultas que se ejecutan.

</details>

---

---

## 11.4 Cuándo Crear y Cuándo NO Crear Índices

### 📘 El Concepto

**✅ SÍ crear índice cuando:**

| Situación | Razón |
|-----------|-------|
| Columna frecuente en `WHERE` | Acelera la búsqueda de filas |
| Clave foránea (FK) | Acelera los `JOIN` y los `DELETE` en cascada |
| Columna en `ORDER BY` / `GROUP BY` | Evita el ordenamiento en tiempo de ejecución |
| Tabla grande (>10.000 filas) | El beneficio es proporcional al tamaño |
| Consultas de alta frecuencia | El coste de mantenimiento se amortiza |

**❌ NO crear índice cuando:**

| Situación | Razón |
|-----------|-------|
| Tabla pequeña (<1.000 filas) | Full Table Scan es igual de rápido o más |
| Columna con muy pocos valores distintos | Un B-Tree no aporta (excepto Bitmap) |
| Tabla con muchos INSERT/UPDATE/DELETE | El coste de mantener el índice supera el beneficio |
| Columna que rara vez se consulta | Ocupa espacio sin usarse |
| Casi todas las filas se devuelven | Si el 80% de la tabla cumple el filtro, el índice no ayuda |

### 🏠 La Analogía

Imagina que tienes un cajón con **5 calcetines**. ¿Necesitas un sistema de organización por color? No — los encuentras a simple vista. Ahora imagina un almacén con **100.000 cajas**. Ahí sí necesitas un sistema de ubicación (índice). Pero si estás constantemente moviendo cajas (muchos INSERT/UPDATE), mantener el sistema de ubicación actualizado te ralentiza las operaciones.

### 💻 El Código

```sql
-- ✅ BUENA IDEA: índice en FK de tabla grande
-- pedidos.id_cliente se usa en casi todos los JOINs con clientes
CREATE INDEX idx_pedidos_cliente ON pedidos(id_cliente);

-- ✅ BUENA IDEA: índice en columna de búsqueda frecuente
-- "Buscar pedidos del último mes" es una consulta diaria
CREATE INDEX idx_pedidos_fecha ON pedidos(fecha_pedido);

-- ❌ MALA IDEA: índice en tabla con 3 filas
-- CREATE INDEX idx_categorias_nombre ON categorias(nombre_categoria);
-- Con solo 3 categorías, Oracle lee las 3 filas más rápido que consultar el índice

-- ❌ MALA IDEA: índice en columna con poca variedad (a menos que sea Bitmap)
-- CREATE INDEX idx_citas_estado ON citas(estado);
-- Solo 3 valores (P, C, X) → un B-Tree normal no ayuda
-- Mejor: CREATE BITMAP INDEX idx_citas_estado ON citas(estado);

-- Consultar índices existentes para auditar
SELECT ui.table_name, ui.index_name, ui.uniqueness, uic.column_name
FROM user_indexes ui
JOIN user_ind_columns uic ON ui.index_name = uic.index_name
ORDER BY ui.table_name, ui.index_name;
```

### 🧠 El Reto

Un desarrollador junior ha creado estos 4 índices. ¿Cuáles mantendrías y cuáles eliminarías? Justifica.

1. `CREATE INDEX idx_esp_nombre ON especialidades(nombre_especialidad);` — (tabla con 3 filas)
2. `CREATE INDEX idx_vuelos_ruta ON vuelos(id_ruta);` — (FK usada en JOINs)
3. `CREATE INDEX idx_prod_nombre ON productos(nombre);` — (búsquedas frecuentes por nombre)
4. `CREATE INDEX idx_cli_telefono ON clientes(telefono);` — (columna que nunca se filtra)

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

1. ❌ **Eliminar** — Solo 3 filas en `especialidades`. Un Full Table Scan es más rápido.
2. ✅ **Mantener** — `id_ruta` es una FK usada en todos los JOINs de vuelos con rutas. Excelente índice.
3. ✅ **Mantener** — Si se buscan productos por nombre frecuentemente (`WHERE nombre = 'Laptop Pro'`), tiene sentido.
4. ❌ **Eliminar** — Si nadie filtra por teléfono, el índice ocupa espacio sin beneficio.

> 💡 Regla de oro: **cada índice que creas debe justificar su existencia** con consultas reales que lo utilicen.

</details>

---

---

## 11.5 Impacto en INSERT/UPDATE/DELETE

### 📘 El Concepto

Los índices **aceleran las lecturas** (`SELECT`) pero **ralentizan las escrituras** (`INSERT`, `UPDATE`, `DELETE`). Cada vez que modificas datos, Oracle debe actualizar **todos** los índices asociados a esa tabla.

| Operación | Sin índice | Con 1 índice | Con 5 índices |
|-----------|-----------|-------------|---------------|
| `SELECT` con filtro | 🐌 Lento | ⚡ Rápido | ⚡ Rápido |
| `INSERT` | ⚡ Rápido | 🔄 Un poco más lento | 🐌 Notablemente más lento |
| `UPDATE` de columna indexada | ⚡ Rápido | 🔄 Más lento | 🐌 Mucho más lento |
| `DELETE` | ⚡ Rápido | 🔄 Más lento | 🐌 Más lento |

**El trade-off fundamental:**
- **Tablas de lectura frecuente** (informes, dashboards): más índices = mejor.
- **Tablas de escritura frecuente** (logs, transacciones en tiempo real): menos índices = mejor.

### 🏠 La Analogía

Imagina una biblioteca con 10 catálogos diferentes (por título, autor, género, año, editorial...). Buscar un libro es facilísimo: abres el catálogo adecuado y lo encuentras al instante. Pero si llega un libro nuevo, el bibliotecario debe actualizar **los 10 catálogos** antes de ponerlo en la estantería. Si llegan 1.000 libros por hora, el bibliotecario no da abasto manteniendo los catálogos.

### 💻 El Código

```sql
-- Ejemplo: tabla productos con 2 índices
-- idx_productos_categoria, idx_productos_precio

-- Un INSERT debe actualizar AMBOS índices:
INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria)
VALUES (18, 'Altavoz Bluetooth', 35.00, 80, 1);
-- 1. Inserta la fila en la tabla
-- 2. Actualiza idx_productos_categoria (nuevo valor id_categoria = 1)
-- 3. Actualiza idx_productos_precio (nuevo valor precio = 35.00)
-- → 3 operaciones de escritura en vez de 1

-- Un UPDATE de columna indexada es aún peor:
UPDATE productos SET precio = 38.00 WHERE id_producto = 18;
-- 1. Localiza la fila (puede usar índice PK)
-- 2. Actualiza la fila
-- 3. Elimina entrada antigua en idx_productos_precio (35.00)
-- 4. Inserta entrada nueva en idx_productos_precio (38.00)
-- → Doble operación en el índice

-- Limpiar el ejemplo
DELETE FROM productos WHERE id_producto = 18;

-- Consejo: en cargas masivas, desactivar índices temporalmente
-- ALTER INDEX idx_productos_precio UNUSABLE;
-- ... cargar datos masivamente ...
-- ALTER INDEX idx_productos_precio REBUILD;
```

### 🧠 El Reto

Una tabla `log_accesos` recibe 50.000 inserciones por hora y rara vez se consulta. Un compañero ha creado 4 índices sobre ella. ¿Qué le recomendarías?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Recomendación: eliminar todos los índices innecesarios** (probablemente los 4).

Razón:
- 50.000 inserciones/hora × 4 índices = 200.000 actualizaciones de índice por hora adicionales.
- Si la tabla rara vez se consulta, esas 200.000 operaciones extra **no se compensan** con lecturas más rápidas.
- Solución: mantener solo el índice de la PK (obligatorio) y crear índices temporales solo cuando se necesite hacer un análisis puntual.

> 💡 En tablas de tipo "log" o "audit", la regla es **mínimos índices**. Si necesitas analizar datos históricos, crea una vista materializada o exporta a un data warehouse con sus propios índices.

</details>

---

---

## 11.6 DROP INDEX y REBUILD INDEX

### 📘 El Concepto

```
-- Eliminar un índice permanentemente
DROP INDEX nombre_indice;

-- Reconstruir un índice (reorganizar su estructura interna)
ALTER INDEX nombre_indice REBUILD;

-- Hacer un índice invisible (Oracle lo ignora pero no lo elimina)
ALTER INDEX nombre_indice INVISIBLE;

-- Hacerlo visible de nuevo
ALTER INDEX nombre_indice VISIBLE;

-- Desactivar temporalmente (para cargas masivas)
ALTER INDEX nombre_indice UNUSABLE;
-- Reactivar
ALTER INDEX nombre_indice REBUILD;
```

**¿Cuándo reconstruir un índice?**
- Después de cargas masivas de datos (`INSERT` de miles de filas).
- Cuando el rendimiento de las consultas se degrada con el tiempo.
- Después de eliminar un gran porcentaje de filas (el índice queda "fragmentado").

**¿Cuándo usar INVISIBLE?**
- Para probar si un índice realmente se está usando. Lo haces invisible y observas si alguna consulta se ralentiza. Si nada cambia, puedes eliminarlo con seguridad.

### 🏠 La Analogía

- **DROP INDEX:** Destruir el catálogo de la biblioteca. Los libros siguen ahí, pero ya no tienes forma rápida de encontrarlos.
- **REBUILD INDEX:** Reorganizar un catálogo desordenado. Con el tiempo, las fichas se desordenan y hay huecos. Reconstruirlo es como rehacer las fichas de forma limpia y compacta.
- **INVISIBLE:** Tapar el catálogo con una sábana. Sigue ahí, pero nadie puede usarlo. Perfecto para probar si alguien lo necesita.

### 💻 El Código

```sql
-- Crear un índice de prueba
CREATE INDEX idx_test_stock ON productos(stock);

-- Verificar que existe
SELECT index_name, status, visibility
FROM user_indexes
WHERE index_name = 'IDX_TEST_STOCK';
-- STATUS: VALID, VISIBILITY: VISIBLE

-- Hacerlo invisible para probar impacto
ALTER INDEX idx_test_stock INVISIBLE;
-- Ahora Oracle ignora el índice en sus planes de ejecución
-- Si las consultas siguen igual de rápidas → no se usaba

-- Decidir eliminarlo
DROP INDEX idx_test_stock;

-- Reconstruir un índice existente tras una carga masiva
ALTER INDEX idx_productos_precio REBUILD;
-- Reorganiza el B-Tree de forma compacta y eficiente

-- Ver el estado de todos los índices
SELECT index_name, table_name, status, visibility, blevel, leaf_blocks
FROM user_indexes
WHERE table_name = 'PRODUCTOS';
-- blevel = profundidad del árbol, leaf_blocks = bloques hoja
```

### 🧠 El Reto

El DBA del hospital nota que las consultas de citas por fecha se han ralentizado después de un año de operación (miles de INSERT y DELETE). ¿Qué debería hacer primero: `DROP INDEX` y recrear, o `REBUILD`? ¿Por qué?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Respuesta: `ALTER INDEX idx_citas_fecha REBUILD;`**

Razones:
- **REBUILD** reorganiza el índice sin eliminarlo — es más seguro y no deja ventana de tiempo sin índice.
- **DROP + CREATE** elimina el índice temporalmente, dejando todas las consultas sin optimización durante la recreación.
- `REBUILD` es la operación estándar de mantenimiento. Solo usa `DROP + CREATE` si necesitas cambiar la definición del índice (columnas, tipo, etc.).

```sql
-- Operación recomendada:
ALTER INDEX idx_citas_fecha REBUILD;
```

> 💡 En Oracle Enterprise Edition, existe `REBUILD ONLINE` que permite reconstruir el índice sin bloquear las consultas concurrentes.

</details>

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 11

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de Indexación](ejercicios/ejercicios_indexacion.md) |
| 3️⃣ | Avanzar al siguiente tema | ➡️ [Tema 12: Transacciones](../12-transacciones) |

---

<div align="center">

⬅️ [**Tema 10: Vistas y Objetos de BD**](../10-vistas) · 🏠 [**Índice del Curso**](../README.md) · [**Tema 12: Transacciones →**](../12-transacciones)

</div>
