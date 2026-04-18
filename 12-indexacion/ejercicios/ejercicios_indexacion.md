# 🏋️ Ejercicios — Tema 11: Indexación

> 💡 **Instrucciones:** Resuelve cada ejercicio pensando en el rendimiento y la arquitectura de la base de datos. No todos los ejercicios requieren escribir código — algunos son de análisis y decisión.

---

## Ejercicio 1 — E-commerce · Crear Índices para JOINs

**Enunciado:** La consulta más ejecutada en la tienda online es:

```sql
SELECT cl.nombre, pr.nombre, pe.total, pe.fecha_pedido
FROM pedidos pe
JOIN clientes cl ON pe.id_cliente = cl.id_cliente
JOIN productos pr ON pe.id_producto = pr.id_producto
WHERE pe.fecha_pedido >= DATE '2024-01-01';
```

Crea los índices necesarios para optimizar esta consulta. Explica por qué cada uno.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- 1. Índice en la columna del WHERE
CREATE INDEX idx_pedidos_fecha ON pedidos(fecha_pedido);
-- Razón: filtra filas por fecha, evita Full Table Scan en pedidos

-- 2. Índice en la FK de pedidos → clientes
CREATE INDEX idx_pedidos_cliente ON pedidos(id_cliente);
-- Razón: acelera el JOIN con clientes

-- 3. Índice en la FK de pedidos → productos
CREATE INDEX idx_pedidos_producto ON pedidos(id_producto);
-- Razón: acelera el JOIN con productos
```

> 💡 Las tablas `clientes` y `productos` ya tienen índices automáticos en sus PKs (`id_cliente`, `id_producto`), que Oracle usa para resolver el otro lado del JOIN. Los índices en `pedidos` aceleran la tabla que se recorre.

</details>

---

## Ejercicio 2 — Hospital · Índice Bitmap vs B-Tree

**Enunciado:** La tabla `citas` tiene una columna `estado` con solo 3 valores posibles: `'P'`, `'C'`, `'X'`. Se ejecuta frecuentemente:

```sql
SELECT * FROM citas WHERE estado = 'P';
```

1. ¿Qué tipo de índice sería más eficiente: B-Tree o Bitmap? ¿Por qué?
2. Escribe la sentencia para crearlo.
3. ¿Cambiaría tu respuesta si la tabla `citas` recibiera 10.000 inserciones por hora?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

1. **Bitmap** — La columna `estado` tiene baja cardinalidad (solo 3 valores distintos). Un índice Bitmap es mucho más compacto y eficiente que un B-Tree para este caso.

2. ```sql
   CREATE BITMAP INDEX idx_citas_estado ON citas(estado);
   ```

3. **Sí, cambiaría.** Con 10.000 inserciones/hora, un índice Bitmap sería una mala idea. Los Bitmap sufren graves problemas de bloqueo en entornos con alta concurrencia de escritura porque un solo bloque de bitmap referencia muchas filas — un `INSERT` puede bloquear cientos de filas. En ese caso, sería mejor **no indexar** la columna o usar un B-Tree estándar.

> 💡 Regla: **Bitmap = tablas de lectura intensiva (data warehouses)**. **B-Tree = tablas transaccionales (OLTP)**.

</details>

---

## Ejercicio 3 — Aerolínea · Índice Compuesto y Orden de Columnas

**Enunciado:** Se ejecutan frecuentemente estas dos consultas:

```sql
-- Consulta A
SELECT * FROM vuelos WHERE id_ruta = 'MADLHR' AND fecha_salida > SYSDATE;

-- Consulta B
SELECT * FROM vuelos WHERE fecha_salida > SYSDATE;
```

1. Crea UN solo índice compuesto que optimice **ambas** consultas.
2. ¿El orden de las columnas importa? ¿Cuál pondrías primero?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE INDEX idx_vuelos_ruta_fecha ON vuelos(id_ruta, fecha_salida);
```

**¿Funciona para ambas consultas?**
- ✅ **Consulta A** (`WHERE id_ruta = ... AND fecha_salida > ...`): Sí — usa ambas columnas del índice.
- ❌ **Consulta B** (`WHERE fecha_salida > ...`): No — solo filtra por `fecha_salida`, que es la **segunda** columna del índice compuesto. Oracle no puede usar el índice sin el prefijo.

**Para optimizar ambas consultas, necesitas dos índices:**

```sql
CREATE INDEX idx_vuelos_ruta_fecha ON vuelos(id_ruta, fecha_salida);  -- Consulta A
CREATE INDEX idx_vuelos_fecha ON vuelos(fecha_salida);                -- Consulta B
```

> 💡 La **regla del prefijo** es la clave: en un índice `(col1, col2)`, Oracle puede usarlo para filtrar `col1`, o `col1 + col2`, pero NUNCA solo `col2`.

</details>

---

## Ejercicio 4 — Análisis · ¿Indexar o No Indexar?

**Enunciado:** Para cada escenario, decide si crearías un índice. Responde SÍ/NO con justificación.

| # | Escenario |
|---|-----------|
| A | Columna `categorias.nombre_categoria` (tabla con 3 filas) |
| B | Columna `pedidos.id_cliente` (FK usada en todos los informes) |
| C | Columna `pacientes.telefono` (a veces es NULL, rara vez se filtra) |
| D | Columna `vuelos.fecha_salida` (consultas de vuelos futuros cada 5 minutos) |
| E | Columna `productos.descripcion` (texto largo, nunca se usa en WHERE) |

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

| # | Decisión | Justificación |
|---|----------|---------------|
| A | ❌ NO | Solo 3 filas — Full Table Scan es más rápido que cualquier índice |
| B | ✅ SÍ | FK usada constantemente en JOINs. Índice esencial |
| C | ❌ NO | Se filtra raramente + tiene NULLs. No justifica el coste |
| D | ✅ SÍ | Consultas cada 5 minutos sobre esta columna. Alto retorno |
| E | ❌ NO | Nunca se filtra por descripción. Ocupa espacio sin beneficio |

> 💡 Antes de crear un índice, pregúntate: "¿Hay consultas frecuentes que filtren por esta columna?" Si la respuesta es no, el índice solo consume espacio y ralentiza las escrituras.

</details>

---

## Ejercicio 5 — E-commerce · Function-Based Index

**Enunciado:** Los usuarios de la tienda buscan productos por nombre sin respetar mayúsculas:

```sql
SELECT * FROM productos WHERE UPPER(nombre) = 'LAPTOP PRO';
```

1. ¿Por qué esta consulta NO usa un índice normal en `productos(nombre)`?
2. Crea el índice adecuado para optimizarla.
3. ¿Qué pasa si alguien busca con `LOWER(nombre)` en vez de `UPPER(nombre)`?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

1. Un índice normal en `nombre` almacena `'Laptop Pro'` (con mayúsculas/minúsculas originales). La consulta busca `UPPER(nombre) = 'LAPTOP PRO'`, que es una **transformación** del valor. Oracle no puede usar el índice normal porque los valores no coinciden.

2. ```sql
   CREATE INDEX idx_productos_nombre_upper ON productos(UPPER(nombre));
   ```

3. **No usaría el índice.** `LOWER(nombre)` es una función diferente a `UPPER(nombre)`. Necesitarías crear otro índice: `CREATE INDEX idx_prod_nombre_lower ON productos(LOWER(nombre));`. O mejor, estandariza todas las búsquedas con `UPPER()`.

> 💡 Cada function-based index es específico para la función exacta que se usa en las consultas. Por convención, usa siempre `UPPER()` para búsquedas case-insensitive.

</details>

---

## Ejercicio 6 — Mantenimiento · Auditoría de Índices

**Enunciado:** Escribe las consultas para:

1. Listar **todos los índices** de las tablas `productos`, `pedidos` y `citas`, mostrando nombre del índice, tabla, columna y si es UNIQUE.
2. Identificar los índices que están en estado **UNUSABLE** (desactivados).
3. Reconstruir todos los índices de la tabla `pedidos`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- 1. Listar todos los índices
SELECT ui.index_name, ui.table_name, uic.column_name, ui.uniqueness, ui.status
FROM user_indexes ui
JOIN user_ind_columns uic ON ui.index_name = uic.index_name
WHERE ui.table_name IN ('PRODUCTOS', 'PEDIDOS', 'CITAS')
ORDER BY ui.table_name, ui.index_name;

-- 2. Índices UNUSABLE
SELECT index_name, table_name, status
FROM user_indexes
WHERE status = 'UNUSABLE';

-- 3. Reconstruir índices de pedidos (hay que reconstruir uno por uno)
-- Primero obtener la lista:
SELECT index_name FROM user_indexes WHERE table_name = 'PEDIDOS';

-- Luego reconstruir cada uno:
ALTER INDEX idx_pedidos_cliente REBUILD;
ALTER INDEX idx_pedidos_fecha REBUILD;
ALTER INDEX idx_pedidos_producto REBUILD;
-- (y el índice de la PK si existe)
```

> 💡 En producción, es buena práctica hacer auditoría de índices mensualmente: verificar que están en estado VALID, reconstruir los fragmentados y eliminar los que no se usan.

</details>

---

<div align="center">

⬅️ [**Volver al Tema 11**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 12: Transacciones →**](../../12-transacciones)

</div>
