# 🚀 Tema 15: SQL Avanzado (Window Functions y CTEs)

> **"Las funciones de ventana son el superpoder que separa a un usuario SQL intermedio de un analista experto."** Con ellas puedes crear rankings, calcular totales acumulados, comparar filas consecutivas y generar reportes analíticos complejos — todo en una sola consulta, sin subconsultas ni tablas temporales.

## 📋 Índice

- [15.1 CTEs con WITH (Common Table Expressions)](#151-ctes-con-with-common-table-expressions)
- [15.2 Funciones de Ventana: OVER() y PARTITION BY](#152-funciones-de-ventana-over-y-partition-by)
- [15.3 ROW_NUMBER() — Numerar Filas](#153-row_number--numerar-filas)
- [15.4 RANK() y DENSE_RANK() — Rankings con y sin Huecos](#154-rank-y-dense_rank--rankings-con-y-sin-huecos)
- [15.5 LAG() y LEAD() — Acceder a Filas Anteriores/Siguientes](#155-lag-y-lead--acceder-a-filas-anterioressiguientes)
- [15.6 SUM/AVG/COUNT con OVER() — Agregaciones con Ventana](#156-sumavgcount-con-over--agregaciones-con-ventana)
- [15.7 Operadores de Conjuntos: UNION, INTERSECT, MINUS](#157-operadores-de-conjuntos-union-intersect-minus)

---

---

## 15.1 CTEs con WITH (Common Table Expressions)

### 📘 El Concepto

Una **CTE (Common Table Expression)** es una consulta temporal con nombre que existe solo durante la ejecución de la sentencia. Se define con la cláusula `WITH` y permite:

1. **Simplificar consultas complejas** — dividir una consulta grande en partes nombradas y legibles.
2. **Evitar subconsultas anidadas** — en lugar de 3 niveles de subconsultas, usas 3 CTEs secuenciales.
3. **Reutilizar resultados** — referencia la misma CTE múltiples veces en la consulta principal.

```sql
WITH nombre_cte AS (
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT * FROM nombre_cte;
```

**Múltiples CTEs:**
```sql
WITH 
    cte1 AS (SELECT ...),
    cte2 AS (SELECT ... FROM cte1),  -- puede referenciar CTEs anteriores
    cte3 AS (SELECT ...)
SELECT ...
FROM cte1
JOIN cte2 ON ...
JOIN cte3 ON ...;
```

### 🏠 La Analogía

Piensa en una **receta de cocina** con pasos preparatorios. En lugar de una instrucción gigante como "mezcla la harina que tamizaste con el azúcar que pesaste y los huevos que batiste", la receta dice:

1. **Paso A (Ingredientes secos):** Tamiza 200g de harina con la levadura.
2. **Paso B (Ingredientes húmedos):** Bate 3 huevos con el azúcar.
3. **Paso final:** Mezcla el resultado del Paso A con el Paso B.

Cada paso tiene un nombre y un resultado claro. Eso es exactamente una CTE.

### 💻 El Código

```sql
-- Sin CTE: consulta compleja anidada
SELECT c.nombre AS categoria, sub.total_productos, sub.precio_promedio
FROM categorias c
JOIN (
    SELECT id_categoria, COUNT(*) AS total_productos, ROUND(AVG(precio), 2) AS precio_promedio
    FROM productos
    GROUP BY id_categoria
) sub ON c.id_categoria = sub.id_categoria
WHERE sub.total_productos >= 2;

-- Con CTE: mucho más legible
WITH stats_categoria AS (
    SELECT id_categoria,
           COUNT(*) AS total_productos,
           ROUND(AVG(precio), 2) AS precio_promedio
    FROM productos
    GROUP BY id_categoria
)
SELECT c.nombre AS categoria, sc.total_productos, sc.precio_promedio
FROM categorias c
JOIN stats_categoria sc ON c.id_categoria = sc.id_categoria
WHERE sc.total_productos >= 2;

-- Múltiples CTEs: análisis completo del hospital
WITH 
    citas_por_medico AS (
        SELECT id_medico, COUNT(*) AS total_citas
        FROM citas
        GROUP BY id_medico
    ),
    medicos_con_esp AS (
        SELECT m.id_medico, m.nombre_completo, e.nombre AS especialidad, m.salario
        FROM medicos m
        JOIN especialidades e ON m.id_especialidad = e.id_especialidad
    )
SELECT mce.nombre_completo, mce.especialidad, mce.salario,
       NVL(cpm.total_citas, 0) AS total_citas
FROM medicos_con_esp mce
LEFT JOIN citas_por_medico cpm ON mce.id_medico = cpm.id_medico
ORDER BY total_citas DESC;

-- CTE con la aerolínea: rutas más populares
WITH vuelos_por_ruta AS (
    SELECT id_ruta, COUNT(*) AS total_vuelos, ROUND(AVG(precio), 2) AS precio_medio
    FROM vuelos
    GROUP BY id_ruta
)
SELECT r.origen, r.destino, r.distancia_km,
       vpr.total_vuelos, vpr.precio_medio
FROM rutas r
JOIN vuelos_por_ruta vpr ON r.id_ruta = vpr.id_ruta
ORDER BY vpr.total_vuelos DESC;
```

### 🧠 El Reto

Escribe una consulta con CTEs que muestre: para cada cliente, su nombre, la cantidad total de pedidos y el gasto total. Muestra solo clientes que hayan gastado más que el promedio general.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
WITH 
    gasto_por_cliente AS (
        SELECT id_cliente,
               COUNT(*) AS total_pedidos,
               SUM(total) AS gasto_total
        FROM pedidos
        GROUP BY id_cliente
    ),
    promedio_general AS (
        SELECT AVG(gasto_total) AS avg_gasto
        FROM gasto_por_cliente
    )
SELECT c.nombre_completo, gc.total_pedidos, gc.gasto_total
FROM clientes c
JOIN gasto_por_cliente gc ON c.id_cliente = gc.id_cliente
CROSS JOIN promedio_general pg
WHERE gc.gasto_total > pg.avg_gasto
ORDER BY gc.gasto_total DESC;
```

> 💡 Sin CTEs, esta consulta requeriría subconsultas anidadas difíciles de leer. Con CTEs, cada paso es claro y nombrado.

</details>

---

---

## 15.2 Funciones de Ventana: OVER() y PARTITION BY

### 📘 El Concepto

Las **funciones de ventana** (window functions) realizan cálculos sobre un conjunto de filas relacionadas con la fila actual, **sin colapsar** el resultado como hace `GROUP BY`.

```
GROUP BY:  10 filas → 3 grupos → 3 filas de resultado
OVER():    10 filas → cálculo por ventana → 10 filas de resultado (con dato extra)
```

**Sintaxis base:**
```sql
funcion() OVER (
    [PARTITION BY columna]     -- divide en grupos (como GROUP BY, pero sin colapsar)
    [ORDER BY columna]         -- ordena dentro de cada grupo
    [ROWS/RANGE frame]         -- define el rango de filas para el cálculo
)
```

| Componente | Significado | Ejemplo |
|-----------|-------------|---------|
| `OVER()` | "Aplica sobre todas las filas" | `COUNT(*) OVER()` → total general en cada fila |
| `PARTITION BY` | "Divide en grupos" | `PARTITION BY id_categoria` → calcula por categoría |
| `ORDER BY` | "Ordena dentro del grupo" | `ORDER BY precio DESC` → ranking por precio |

### 🏠 La Analogía

Imagina un **maratón**. Con `GROUP BY`, solo sabrías el tiempo promedio por categoría (Sub-20, Sub-25, etc.). Con funciones de ventana, cada corredor mantiene su fila individual, pero **además** ves su posición dentro de su categoría, el tiempo del corredor anterior, y el promedio de su grupo. Todo en la misma línea.

### 💻 El Código

```sql
-- Diferencia visual: GROUP BY vs OVER()

-- GROUP BY: colapsa filas
SELECT id_categoria, COUNT(*) AS total, ROUND(AVG(precio), 2) AS avg_precio
FROM productos
GROUP BY id_categoria;
-- Resultado: 4 filas (una por categoría)

-- OVER(): mantiene todas las filas + agrega el cálculo
SELECT nombre, precio, id_categoria,
       COUNT(*) OVER (PARTITION BY id_categoria) AS productos_en_categoria,
       ROUND(AVG(precio) OVER (PARTITION BY id_categoria), 2) AS avg_precio_categoria
FROM productos;
-- Resultado: 8 filas (una por producto) con datos extra por categoría

-- Ejemplo hospital: cada médico con el promedio salarial de su especialidad
SELECT m.nombre_completo, e.nombre AS especialidad, m.salario,
       ROUND(AVG(m.salario) OVER (PARTITION BY m.id_especialidad), 2) AS avg_salario_esp
FROM medicos m
JOIN especialidades e ON m.id_especialidad = e.id_especialidad;

-- Ejemplo aerolínea: cada vuelo con el precio promedio de su ruta
SELECT v.id_vuelo, r.origen, r.destino, v.precio,
       ROUND(AVG(v.precio) OVER (PARTITION BY v.id_ruta), 2) AS avg_precio_ruta
FROM vuelos v
JOIN rutas r ON v.id_ruta = r.id_ruta;
```

### 🧠 El Reto

Escribe una consulta que muestre cada producto con su precio, el precio máximo de su categoría y la diferencia entre ambos (cuánto le falta para ser el más caro de su categoría).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio, id_categoria,
       MAX(precio) OVER (PARTITION BY id_categoria) AS max_precio_cat,
       MAX(precio) OVER (PARTITION BY id_categoria) - precio AS diferencia
FROM productos
ORDER BY id_categoria, precio DESC;
```

</details>

---

---

## 15.3 ROW_NUMBER() — Numerar Filas

### 📘 El Concepto

`ROW_NUMBER()` asigna un **número secuencial único** a cada fila dentro de su partición, según el orden especificado. Siempre produce números consecutivos sin huecos: 1, 2, 3, 4...

```sql
ROW_NUMBER() OVER (
    [PARTITION BY grupo]
    ORDER BY criterio
)
```

**Usos principales:**
- Numerar filas dentro de un grupo.
- Seleccionar el "Top N" por grupo (el producto más caro por categoría).
- Eliminar duplicados (quedarse con la primera fila de cada grupo).
- Paginación de resultados.

### 🏠 La Analogía

Es como **numerar los asientos de un cine** por fila. En la Fila A: asiento 1, 2, 3... En la Fila B: se reinicia a 1, 2, 3... Cada fila (PARTITION) tiene su propia numeración independiente y secuencial.

### 💻 El Código

```sql
-- Numerar productos por precio dentro de cada categoría (más caro = 1)
SELECT nombre, precio,
       ROW_NUMBER() OVER (PARTITION BY id_categoria ORDER BY precio DESC) AS ranking
FROM productos;
-- Laptop Pro: ranking 1 en Electrónica (1200)
-- Teclado Mecánico: ranking 2 en Electrónica (75)
-- etc.

-- Top 1 producto más caro por categoría
WITH ranking AS (
    SELECT p.nombre, p.precio, c.nombre AS categoria,
           ROW_NUMBER() OVER (PARTITION BY p.id_categoria ORDER BY p.precio DESC) AS rn
    FROM productos p
    JOIN categorias c ON p.id_categoria = c.id_categoria
)
SELECT nombre, precio, categoria
FROM ranking
WHERE rn = 1;

-- Hospital: el médico mejor pagado de cada especialidad
WITH ranking_medicos AS (
    SELECT m.nombre_completo, e.nombre AS especialidad, m.salario,
           ROW_NUMBER() OVER (PARTITION BY m.id_especialidad ORDER BY m.salario DESC) AS rn
    FROM medicos m
    JOIN especialidades e ON m.id_especialidad = e.id_especialidad
)
SELECT nombre_completo, especialidad, salario
FROM ranking_medicos
WHERE rn = 1;

-- Aerolínea: el vuelo más barato de cada ruta
WITH vuelos_rank AS (
    SELECT v.id_vuelo, r.origen, r.destino, v.precio, v.fecha_salida,
           ROW_NUMBER() OVER (PARTITION BY v.id_ruta ORDER BY v.precio ASC) AS rn
    FROM vuelos v
    JOIN rutas r ON v.id_ruta = r.id_ruta
)
SELECT id_vuelo, origen, destino, precio, fecha_salida
FROM vuelos_rank
WHERE rn = 1;

-- Paginación: mostrar productos de la página 2 (filas 4-6)
WITH numerados AS (
    SELECT nombre, precio,
           ROW_NUMBER() OVER (ORDER BY nombre) AS fila
    FROM productos
)
SELECT nombre, precio
FROM numerados
WHERE fila BETWEEN 4 AND 6;
```

### 🧠 El Reto

Escribe una consulta que muestre los 2 pedidos más recientes de cada cliente (nombre del cliente, producto, fecha y total).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
WITH pedidos_rank AS (
    SELECT p.id_cliente, p.id_producto, p.fecha_pedido, p.total,
           ROW_NUMBER() OVER (PARTITION BY p.id_cliente ORDER BY p.fecha_pedido DESC) AS rn
    FROM pedidos p
)
SELECT c.nombre_completo, pr.nombre AS producto, 
       pr2.fecha_pedido, pr2.total
FROM pedidos_rank pr2
JOIN clientes c ON pr2.id_cliente = c.id_cliente
JOIN productos pr ON pr2.id_producto = pr.id_producto
WHERE pr2.rn <= 2
ORDER BY c.nombre_completo, pr2.rn;
```

</details>

---

---

## 15.4 RANK() y DENSE_RANK() — Rankings con y sin Huecos

### 📘 El Concepto

| Función | Empates | Secuencia | Ejemplo con empate |
|---------|---------|-----------|-------------------|
| `ROW_NUMBER()` | Ignora empates | Siempre única | 1, 2, 3, 4 |
| `RANK()` | Mismo número para empates | **Salta** posiciones | 1, 2, 2, **4** |
| `DENSE_RANK()` | Mismo número para empates | **No salta** posiciones | 1, 2, 2, **3** |

```
Ejemplo visual con precios: 1200, 75, 75, 45.50

ROW_NUMBER():  1, 2, 3, 4      ← siempre únicos
RANK():        1, 2, 2, 4      ← empate en 2, salta al 4
DENSE_RANK():  1, 2, 2, 3      ← empate en 2, sigue con 3
```

### 🏠 La Analogía

Piensa en una **carrera de atletismo**:
- **ROW_NUMBER:** "Llegaron en posiciones 1, 2, 3, 4" (si dos cruzan juntos, el juez elige uno).
- **RANK:** "Dos corredores empataron en el 2º puesto. No hay 3er puesto. El siguiente es 4º" (como en los Juegos Olímpicos — dos medallas de plata, sin bronce).
- **DENSE_RANK:** "Dos corredores empataron en el 2º puesto. El siguiente es 3º" (el podio no salta).

### 💻 El Código

```sql
-- Comparar las 3 funciones con productos por precio
SELECT nombre, precio,
       ROW_NUMBER() OVER (ORDER BY precio DESC) AS row_num,
       RANK()       OVER (ORDER BY precio DESC) AS ranking,
       DENSE_RANK() OVER (ORDER BY precio DESC) AS dense_ranking
FROM productos;

-- Hospital: ranking de médicos por salario dentro de cada especialidad
SELECT m.nombre_completo, e.nombre AS especialidad, m.salario,
       RANK() OVER (PARTITION BY m.id_especialidad ORDER BY m.salario DESC) AS rank_salario,
       DENSE_RANK() OVER (PARTITION BY m.id_especialidad ORDER BY m.salario DESC) AS dense_rank_salario
FROM medicos m
JOIN especialidades e ON m.id_especialidad = e.id_especialidad;

-- Aerolínea: ranking de vuelos por precio (los más baratos primero)
SELECT v.id_vuelo, r.origen, r.destino, v.precio,
       DENSE_RANK() OVER (ORDER BY v.precio ASC) AS ranking_precio
FROM vuelos v
JOIN rutas r ON v.id_ruta = r.id_ruta;

-- Caso práctico: top 3 productos más caros con DENSE_RANK
-- (si hay empates, queremos incluir todos los empatados)
WITH ranking AS (
    SELECT nombre, precio, id_categoria,
           DENSE_RANK() OVER (ORDER BY precio DESC) AS dr
    FROM productos
)
SELECT nombre, precio, dr AS posicion
FROM ranking
WHERE dr <= 3;
```

### 🧠 El Reto

Si 3 médicos tienen el mismo salario más alto (empate triple), ¿cuál es el RANK y DENSE_RANK del siguiente médico? ¿Y cuál es mejor para un reporte de "Top 3 salarios" si quieres incluir a TODOS los empatados?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

Con un empate triple en la posición 1:
- **RANK** del siguiente médico: **4** (salta las posiciones 2 y 3).
- **DENSE_RANK** del siguiente médico: **2** (no salta).

Para un "Top 3 salarios" incluyendo todos los empatados, usa **DENSE_RANK**:

```sql
WITH ranking AS (
    SELECT nombre_completo, salario,
           DENSE_RANK() OVER (ORDER BY salario DESC) AS dr
    FROM medicos
)
SELECT nombre_completo, salario, dr
FROM ranking
WHERE dr <= 3;
-- Si 3 médicos empatan en el 1er puesto, los 3 aparecen + los del 2do y 3er puesto
-- RANK con WHERE rank <= 3 solo mostraría los 3 empatados (el 4o tiene rank 4)
```

</details>

---

---

## 15.5 LAG() y LEAD() — Acceder a Filas Anteriores/Siguientes

### 📘 El Concepto

| Función | Accede a | Sintaxis |
|---------|----------|----------|
| `LAG(col, n, default)` | La fila **anterior** (n posiciones atrás) | `LAG(precio, 1, 0) OVER (ORDER BY fecha)` |
| `LEAD(col, n, default)` | La fila **siguiente** (n posiciones adelante) | `LEAD(precio, 1, 0) OVER (ORDER BY fecha)` |

**Parámetros:**
- `col` — columna a consultar.
- `n` — número de filas hacia atrás/adelante (por defecto 1).
- `default` — valor si no existe la fila (por defecto NULL).

```
Filas ordenadas por fecha:
  Fila 1: 2024-01-15  ← LAG de Fila 2
  Fila 2: 2024-02-20  ← actual
  Fila 3: 2024-03-10  ← LEAD de Fila 2
```

**Usos:**
- Comparar con el valor anterior (cambio porcentual mes a mes).
- Calcular diferencias entre filas consecutivas.
- Detectar tendencias (subida/bajada).

### 🏠 La Analogía

Es como mirar tu **extracto bancario**. Estás en el movimiento de hoy y puedes mirar "arriba" (LAG) para ver el movimiento anterior, o mirar "abajo" (LEAD) para ver el siguiente. Con eso calculas: "¿Gasté más o menos que ayer?"

### 💻 El Código

```sql
-- Pedidos: comparar cada pedido con el anterior por fecha
SELECT id_pedido, fecha_pedido, total,
       LAG(total, 1) OVER (ORDER BY fecha_pedido) AS total_anterior,
       total - LAG(total, 1, 0) OVER (ORDER BY fecha_pedido) AS diferencia
FROM pedidos
ORDER BY fecha_pedido;

-- Aerolínea: comparar vuelos consecutivos en la misma ruta
SELECT v.id_vuelo, r.origen, r.destino, v.fecha_salida, v.precio,
       LAG(v.precio) OVER (PARTITION BY v.id_ruta ORDER BY v.fecha_salida) AS precio_vuelo_anterior,
       LEAD(v.precio) OVER (PARTITION BY v.id_ruta ORDER BY v.fecha_salida) AS precio_vuelo_siguiente,
       v.precio - LAG(v.precio, 1, v.precio) OVER (PARTITION BY v.id_ruta ORDER BY v.fecha_salida) AS cambio_precio
FROM vuelos v
JOIN rutas r ON v.id_ruta = r.id_ruta
ORDER BY r.origen, r.destino, v.fecha_salida;

-- Hospital: diferencia de días entre citas consecutivas de un paciente
SELECT c.id_cita, p.nombre_completo, c.fecha_cita,
       LAG(c.fecha_cita) OVER (PARTITION BY c.id_paciente ORDER BY c.fecha_cita) AS cita_anterior,
       ROUND(c.fecha_cita - LAG(c.fecha_cita) OVER (PARTITION BY c.id_paciente ORDER BY c.fecha_cita)) AS dias_entre_citas
FROM citas c
JOIN pacientes p ON c.id_paciente = p.id_paciente
ORDER BY p.nombre_completo, c.fecha_cita;

-- E-commerce: tendencia de precios (si tuviéramos historial)
-- Simular con productos ordenados por id
SELECT nombre, precio,
       LAG(precio) OVER (ORDER BY id_producto) AS precio_prod_anterior,
       LEAD(precio) OVER (ORDER BY id_producto) AS precio_prod_siguiente,
       CASE 
           WHEN precio > LAG(precio) OVER (ORDER BY id_producto) THEN '📈 Subida'
           WHEN precio < LAG(precio) OVER (ORDER BY id_producto) THEN '📉 Bajada'
           ELSE '➡️ Igual'
       END AS tendencia
FROM productos;
```

### 🧠 El Reto

Escribe una consulta que para cada vuelo muestre el precio y la diferencia porcentual respecto al vuelo anterior de la misma ruta. Formato: "+15.3%" o "-8.7%".

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT v.id_vuelo, r.origen, r.destino, v.precio,
       LAG(v.precio) OVER (PARTITION BY v.id_ruta ORDER BY v.fecha_salida) AS precio_anterior,
       CASE 
           WHEN LAG(v.precio) OVER (PARTITION BY v.id_ruta ORDER BY v.fecha_salida) IS NULL THEN 'N/A'
           ELSE TO_CHAR(
               ROUND((v.precio - LAG(v.precio) OVER (PARTITION BY v.id_ruta ORDER BY v.fecha_salida)) 
               / LAG(v.precio) OVER (PARTITION BY v.id_ruta ORDER BY v.fecha_salida) * 100, 1),
               'S990.0'
           ) || '%'
       END AS cambio_pct
FROM vuelos v
JOIN rutas r ON v.id_ruta = r.id_ruta
ORDER BY v.id_ruta, v.fecha_salida;
```

</details>

---

---

## 15.6 SUM/AVG/COUNT con OVER() — Agregaciones con Ventana

### 📘 El Concepto

Las funciones de agregación clásicas (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) se pueden usar como **funciones de ventana** añadiendo `OVER()`. Esto permite calcular:

| Tipo | Sintaxis | Resultado |
|------|----------|-----------|
| **Total general** | `SUM(col) OVER ()` | La misma suma en todas las filas |
| **Total por grupo** | `SUM(col) OVER (PARTITION BY grupo)` | Suma dentro de cada grupo |
| **Acumulado (running total)** | `SUM(col) OVER (ORDER BY fecha)` | Suma que crece fila a fila |
| **Media móvil** | `AVG(col) OVER (ORDER BY fecha ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` | Promedio de las últimas 3 filas |

**Running Total (total acumulado):**
```
Fecha       Total    Running SUM
2024-01     100      100
2024-02     200      300     (100+200)
2024-03     150      450     (100+200+150)
2024-04     300      750     (100+200+150+300)
```

### 🏠 La Analogía

El **running total** es como tu **saldo bancario**. Cada movimiento suma o resta del saldo anterior. No ves solo "hoy gastaste 50€", sino "tu saldo acumulado es 1.250€". Cada fila conoce todo lo que pasó antes.

### 💻 El Código

```sql
-- E-commerce: acumulado de ventas por fecha (running total)
SELECT id_pedido, fecha_pedido, total,
       SUM(total) OVER (ORDER BY fecha_pedido) AS acumulado
FROM pedidos
ORDER BY fecha_pedido;

-- Cada producto con su precio, el total de su categoría y su % del total
SELECT nombre, precio, id_categoria,
       SUM(precio) OVER (PARTITION BY id_categoria) AS total_categoria,
       ROUND(precio / SUM(precio) OVER (PARTITION BY id_categoria) * 100, 1) AS pct_categoria
FROM productos;

-- Hospital: conteo acumulado de citas por fecha
SELECT id_cita, fecha_cita, estado,
       COUNT(*) OVER (ORDER BY fecha_cita) AS citas_acumuladas,
       COUNT(*) OVER () AS total_citas
FROM citas
ORDER BY fecha_cita;

-- Aerolínea: precio de cada vuelo vs promedio de su ruta
SELECT v.id_vuelo, r.origen, r.destino, v.precio,
       ROUND(AVG(v.precio) OVER (PARTITION BY v.id_ruta), 2) AS avg_ruta,
       v.precio - ROUND(AVG(v.precio) OVER (PARTITION BY v.id_ruta), 2) AS vs_promedio
FROM vuelos v
JOIN rutas r ON v.id_ruta = r.id_ruta
ORDER BY v.id_ruta, v.precio DESC;

-- Media móvil de las últimas 3 transacciones (pedidos)
SELECT id_pedido, fecha_pedido, total,
       ROUND(AVG(total) OVER (ORDER BY fecha_pedido ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS media_movil_3
FROM pedidos
ORDER BY fecha_pedido;

-- Porcentaje de cada pedido sobre el total general
SELECT id_pedido, total,
       SUM(total) OVER () AS total_general,
       ROUND(total / SUM(total) OVER () * 100, 2) AS porcentaje
FROM pedidos;
```

### 🧠 El Reto

Escribe una consulta que muestre para cada médico: su salario, el salario acumulado dentro de su especialidad (ordenado por salario DESC), y qué porcentaje del gasto salarial total de la especialidad acumula.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT m.nombre_completo, e.nombre AS especialidad, m.salario,
       SUM(m.salario) OVER (PARTITION BY m.id_especialidad ORDER BY m.salario DESC) AS salario_acumulado,
       SUM(m.salario) OVER (PARTITION BY m.id_especialidad) AS total_especialidad,
       ROUND(SUM(m.salario) OVER (PARTITION BY m.id_especialidad ORDER BY m.salario DESC) 
             / SUM(m.salario) OVER (PARTITION BY m.id_especialidad) * 100, 1) AS pct_acumulado
FROM medicos m
JOIN especialidades e ON m.id_especialidad = e.id_especialidad
ORDER BY e.nombre, m.salario DESC;
```

</details>

---

---

## 15.7 Operadores de Conjuntos: UNION, UNION ALL, INTERSECT, MINUS

### 📘 El Concepto

Los **operadores de conjuntos** combinan los resultados de dos o más consultas `SELECT`:

| Operador | Resultado | Duplicados | Ejemplo |
|----------|-----------|:----------:|---------|
| `UNION` | Filas de A **o** B | Eliminados | Todas las ciudades únicas de clientes y rutas |
| `UNION ALL` | Filas de A **y** B | Conservados | Todos los registros combinados (más rápido) |
| `INTERSECT` | Filas en A **y** en B | Eliminados | Ciudades que son origen Y destino de vuelos |
| `MINUS` | Filas en A **pero no** en B | Eliminados | Clientes que nunca han hecho pedidos |

**Reglas:**
1. Ambos SELECT deben tener el **mismo número de columnas**.
2. Los tipos de datos deben ser **compatibles**.
3. El nombre de las columnas viene del **primer SELECT**.

```
A = {1, 2, 3, 4}    B = {3, 4, 5, 6}

A UNION B     = {1, 2, 3, 4, 5, 6}
A INTERSECT B = {3, 4}
A MINUS B     = {1, 2}
B MINUS A     = {5, 6}
```

### 🏠 La Analogía

Piensa en dos **listas de invitados** para una fiesta:
- **UNION:** "Invita a todos los de la lista A o la lista B" (sin repetir nombres).
- **UNION ALL:** "Pon ambas listas juntas" (si alguien está en ambas, aparece dos veces — útil para contar).
- **INTERSECT:** "Solo invita a los que están en AMBAS listas" (los amigos en común).
- **MINUS:** "Invita a los de la lista A que NO están en la lista B" (los exclusivos de A).

### 💻 El Código

```sql
-- UNION: todas las ciudades mencionadas (clientes + rutas)
SELECT ciudad AS lugar FROM clientes
UNION
SELECT origen FROM rutas
UNION
SELECT destino FROM rutas
ORDER BY lugar;

-- UNION ALL: combinar registros de todas las bases (para un log unificado)
SELECT 'E-commerce' AS sistema, nombre_completo AS persona, ciudad AS info
FROM clientes
UNION ALL
SELECT 'Hospital', nombre_completo, telefono
FROM pacientes
UNION ALL
SELECT 'Aerolínea', modelo, TO_CHAR(capacidad)
FROM aviones;

-- INTERSECT: ciudades que son origen Y destino de algún vuelo
SELECT origen AS ciudad FROM rutas
INTERSECT
SELECT destino FROM rutas;

-- MINUS: clientes que NO tienen pedidos
SELECT id_cliente, nombre_completo FROM clientes
MINUS
SELECT c.id_cliente, c.nombre_completo 
FROM clientes c
JOIN pedidos p ON c.id_cliente = p.id_cliente;

-- MINUS: pacientes sin citas
SELECT id_paciente, nombre_completo FROM pacientes
MINUS
SELECT p.id_paciente, p.nombre_completo
FROM pacientes p
JOIN citas ci ON p.id_paciente = ci.id_paciente;

-- Aplicación práctica: rutas sin vuelos programados
SELECT id_ruta, origen, destino FROM rutas
MINUS
SELECT r.id_ruta, r.origen, r.destino
FROM rutas r
JOIN vuelos v ON r.id_ruta = v.id_ruta;
```

### 🧠 El Reto

Usando operadores de conjuntos, escribe una consulta que muestre las especialidades médicas que tienen médicos asignados pero NO tienen citas registradas.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Especialidades con médicos
SELECT DISTINCT e.nombre
FROM especialidades e
JOIN medicos m ON e.id_especialidad = m.id_especialidad
MINUS
-- Especialidades con citas
SELECT DISTINCT e.nombre
FROM especialidades e
JOIN medicos m ON e.id_especialidad = m.id_especialidad
JOIN citas c ON m.id_medico = c.id_medico;
```

> 💡 Esto devuelve especialidades que sí tienen doctores pero ninguno de ellos tiene citas. `MINUS` elimina del primer conjunto aquellas que aparecen en el segundo.

</details>

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 15

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de SQL Avanzado](ejercicios/ejercicios_sql_avanzado.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Proyecto: El Reporte Analítico Avanzado](proyectos/proyecto_sql_avanzado.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 16: PL/SQL](../16-plsql) |

---

<div align="center">

⬅️ [**Tema 14: DCL y Seguridad**](../14-dcl-seguridad) · 🏠 [**Índice del Curso**](../README.md) · [**Tema 16: PL/SQL →**](../16-plsql)

</div>
