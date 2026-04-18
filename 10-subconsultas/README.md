# 🪆 Tema 10: Subconsultas

> **"Una pregunta dentro de otra pregunta."** Las subconsultas son consultas anidadas que resuelven un problema paso a paso: primero averiguas algo y luego usas ese resultado para filtrar, comparar o calcular. Son el arma secreta del analista SQL avanzado.

## 📋 Índice

- [10.1 Subconsultas Escalares](#101-subconsultas-escalares)
- [10.2 Subconsultas en WHERE con IN / NOT IN](#102-subconsultas-en-where-con-in--not-in)
- [10.3 Subconsultas Correlacionadas](#103-subconsultas-correlacionadas)
- [10.4 EXISTS / NOT EXISTS](#104-exists--not-exists)
- [10.5 ANY / ALL](#105-any--all)
- [10.6 Subconsultas en FROM (Inline Views)](#106-subconsultas-en-from-inline-views)

---

---

## 10.1 Subconsultas Escalares

### 📘 El Concepto

Una **subconsulta escalar** devuelve **un solo valor** (una fila, una columna). Se puede usar en el `SELECT`, en el `WHERE` o en cualquier lugar donde Oracle espere un valor único.

```
-- En el WHERE:
SELECT columnas FROM tabla
WHERE columna = (SELECT valor FROM otra_tabla WHERE condicion);

-- En el SELECT:
SELECT columna,
       (SELECT valor FROM otra_tabla WHERE condicion) AS alias
FROM tabla;
```

> ⚠️ **Regla clave:** Si la subconsulta escalar devuelve **más de una fila**, Oracle lanzará el error `ORA-01427: single-row subquery returns more than one row`.

### 🏠 La Analogía

Imagina que estás en un examen y una pregunta dice: _"¿Quién sacó la nota más alta?"_. Primero necesitas **averiguar cuál fue la nota más alta** (subconsulta) y luego **buscar quién la tiene** (consulta principal). Es como resolver un acertijo en dos pasos: primero descubres el dato y luego lo usas.

### 💻 El Código

```sql
-- E-commerce: producto más caro (subconsulta en WHERE)
SELECT nombre, precio
FROM productos
WHERE precio = (SELECT MAX(precio) FROM productos);
-- Resultado: Laptop Pro, 1220.00

-- Hospital: médicos que ganan más que el salario promedio
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base > (SELECT AVG(salario_base) FROM medicos);
-- AVG = (4000+3200+2800+4500+2600)/5 = 3420
-- Resultado: Dra. Sarah Adams (4000), Dra. Marta López (4500)

-- Aerolínea: vuelos en la ruta más larga (subconsulta en WHERE)
SELECT v.id_vuelo, v.id_ruta, v.fecha_hora_salida
FROM vuelos v
INNER JOIN rutas r ON v.id_ruta = r.id_ruta
WHERE r.distancia_km = (SELECT MAX(distancia_km) FROM rutas);
-- Ruta más larga: JFKLAX (4000 km) → Vuelo 1002

-- E-commerce: cada producto con el precio medio de su categoría (subconsulta en SELECT)
SELECT p.nombre,
       p.precio,
       (SELECT ROUND(AVG(p2.precio), 2)
        FROM productos p2
        WHERE p2.id_categoria = p.id_categoria) AS precio_medio_cat
FROM productos p
ORDER BY p.id_categoria;
```

### 🧠 El Reto

El director del hospital quiere saber qué **médico tiene el salario más bajo**. Usa una subconsulta escalar en el `WHERE`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base = (SELECT MIN(salario_base) FROM medicos);
```

Resultado:
| nombre_completo | salario_base |
|----------------|-------------|
| Dr. Luis Moreno | 2600 |

</details>

---

---

## 10.2 Subconsultas en WHERE con IN / NOT IN

### 📘 El Concepto

Cuando la subconsulta devuelve **múltiples valores** (varias filas, una columna), no puedes usar `=`. Debes usar `IN` o `NOT IN`.

```
-- ¿Está el valor en la lista?
WHERE columna IN (SELECT columna FROM otra_tabla WHERE condicion)

-- ¿El valor NO está en la lista?
WHERE columna NOT IN (SELECT columna FROM otra_tabla WHERE condicion)
```

> ⚠️ **Cuidado con NULL y NOT IN:** Si la subconsulta con `NOT IN` devuelve algún `NULL`, el resultado será **vacío**. Oracle no puede comparar un valor con `NULL` y toda la condición se evalúa como `UNKNOWN`. Solución: añade `WHERE columna IS NOT NULL` en la subconsulta, o usa `NOT EXISTS` en su lugar.

### 🏠 La Analogía

Piensa en un portero de discoteca con una **lista negra** (NOT IN) y una **lista VIP** (IN):
- `IN`: _"¿Tu nombre está en la lista VIP? Pasa."_
- `NOT IN`: _"¿Tu nombre NO está en la lista negra? Pasa."_
- Pero si la lista negra tiene una entrada borrosa (NULL), el portero no deja pasar a nadie porque no puede verificar.

### 💻 El Código

```sql
-- E-commerce: productos que TIENEN al menos un pedido
SELECT nombre, precio
FROM productos
WHERE id_producto IN (SELECT id_producto FROM pedidos);
-- Resultado: 7 productos (todos menos Sofá de Cuero)

-- E-commerce: productos que NO tienen pedidos
SELECT nombre, precio
FROM productos
WHERE id_producto NOT IN (SELECT id_producto FROM pedidos);
-- Resultado: Sofá de Cuero (405.00)

-- Hospital: pacientes que han tenido citas con neurólogos
SELECT nombre_completo
FROM pacientes
WHERE id_paciente IN (
    SELECT ci.id_paciente
    FROM citas ci
    INNER JOIN medicos m ON ci.id_medico = m.id_medico
    WHERE m.id_especialidad = 200  -- Neurología
);
-- Resultado: Laura Martinez, Pedro Sánchez, Ana Ruiz, Carlos Gomez

-- Aerolínea: aviones que NO operan rutas desde Madrid
SELECT modelo
FROM aviones
WHERE id_avion NOT IN (
    SELECT v.id_avion
    FROM vuelos v
    INNER JOIN rutas r ON v.id_ruta = r.id_ruta
    WHERE r.origen = 'MAD'
);
-- Resultado: Boeing 737 (sin vuelos), Embraer E195 (solo BCN)
```

### 🧠 El Reto

Encuentra las **especialidades que tienen médicos con salario superior a 3500**. Usa `IN` con una subconsulta.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre AS especialidad
FROM especialidades
WHERE id_especialidad IN (
    SELECT id_especialidad
    FROM medicos
    WHERE salario_base > 3500
);
```

Resultado:
| especialidad |
|-------------|
| Neurología |

> 💡 Solo Neurología tiene médicos con salario > 3500: Dra. Sarah Adams (4000) y Dra. Marta López (4500).

</details>

---

---

## 10.3 Subconsultas Correlacionadas

### 📘 El Concepto

Una **subconsulta correlacionada** hace referencia a una columna de la consulta **externa**. Se ejecuta **una vez por cada fila** de la consulta principal, lo que la diferencia de las subconsultas normales (que se ejecutan una sola vez).

```
SELECT columnas
FROM tabla_externa ext
WHERE columna operador (
    SELECT funcion(columna)
    FROM tabla_interna int
    WHERE int.clave = ext.clave   -- ← referencia a la tabla externa
);
```

### 🏠 La Analogía

Imagina que eres profesor y quieres identificar a los **mejores alumnos de cada clase**. Para cada clase (consulta externa), necesitas calcular la nota media de ESA clase en particular (subconsulta correlacionada) y luego ver qué alumnos superan esa media. No puedes calcular una media global; necesitas una media específica **para cada grupo**.

### 💻 El Código

```sql
-- E-commerce: productos cuyo precio es mayor que el promedio de SU categoría
SELECT p1.nombre, p1.precio, p1.id_categoria
FROM productos p1
WHERE p1.precio > (
    SELECT AVG(p2.precio)
    FROM productos p2
    WHERE p2.id_categoria = p1.id_categoria  -- correlación
);
-- Para Electrónica: AVG = (1220+45.50+350+75)/4 = 422.63
--   → Laptop Pro (1220) está por encima
-- Para Hogar: AVG = (405+40.50)/2 = 222.75
--   → Sofá de Cuero (405) está por encima
-- Para Ropa: AVG = (19.99+89.50)/2 = 54.75
--   → Zapatillas Running (89.50) está por encima

-- Hospital: médicos que ganan más que el promedio de SU especialidad
SELECT m1.nombre_completo, m1.salario_base, m1.id_especialidad
FROM medicos m1
WHERE m1.salario_base > (
    SELECT AVG(m2.salario_base)
    FROM medicos m2
    WHERE m2.id_especialidad = m1.id_especialidad
);
-- Traumatología: AVG = (3200+2800)/2 = 3000 → Dra. Elena Ruiz (3200)
-- Neurología: AVG = (4000+4500)/2 = 4250 → Dra. Marta López (4500)
-- Med. General: solo 1 médico, no puede superar su propia media

-- E-commerce: pedidos cuyo total supera el gasto medio del cliente
SELECT pe.id_pedido, pe.id_cliente, pe.total
FROM pedidos pe
WHERE pe.total > (
    SELECT AVG(pe2.total)
    FROM pedidos pe2
    WHERE pe2.id_cliente = pe.id_cliente
);
```

### 🧠 El Reto

Encuentra los **vuelos cuya ruta tiene una distancia mayor al promedio de distancia de todas las rutas**. Usa una subconsulta (no necesita ser correlacionada en este caso — ¿por qué?).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT v.id_vuelo, r.origen || ' → ' || r.destino AS ruta, r.distancia_km
FROM vuelos v
INNER JOIN rutas r ON v.id_ruta = r.id_ruta
WHERE r.distancia_km > (SELECT AVG(distancia_km) FROM rutas);
```

AVG de rutas = (1350 + 4000 + 830 + 1420 + 850) / 5 = **1690 km**

Resultado:
| id_vuelo | ruta | distancia_km |
|----------|------|-------------|
| 1002 | JFK → LAX | 4000 |

> 💡 No necesita ser correlacionada porque el promedio es sobre TODAS las rutas, no sobre un subgrupo que cambia por fila. Si quisiéramos "vuelos en rutas más largas que el promedio de rutas que salen del mismo origen", ahí sí sería correlacionada.

</details>

---

---

## 10.4 EXISTS / NOT EXISTS

### 📘 El Concepto

`EXISTS` devuelve `TRUE` si la subconsulta retorna **al menos una fila**. `NOT EXISTS` devuelve `TRUE` si la subconsulta **no retorna ninguna fila**. Son especialmente útiles para comprobar relaciones de existencia.

```
-- ¿Existe al menos un registro relacionado?
WHERE EXISTS (SELECT 1 FROM otra_tabla WHERE condicion)

-- ¿NO existe ningún registro relacionado?
WHERE NOT EXISTS (SELECT 1 FROM otra_tabla WHERE condicion)
```

> 📌 `EXISTS` no se fija en QUÉ devuelve la subconsulta, solo en SI devuelve algo. Por eso es común escribir `SELECT 1` o `SELECT *` — da lo mismo.

> 💡 **`NOT EXISTS` vs `NOT IN`:** `NOT EXISTS` maneja correctamente los `NULL` y generalmente tiene mejor rendimiento en tablas grandes. Es la opción preferida.

### 🏠 La Analogía

Piensa en una fábrica que revisa sus productos:
- `EXISTS`: _"¿Este producto tiene al menos una queja registrada? Si sí, márcalo para revisión."_ No importa cuántas quejas ni cuáles son, solo si hay alguna.
- `NOT EXISTS`: _"¿Este producto NO tiene ninguna queja? Entonces está certificado."_

### 💻 El Código

```sql
-- E-commerce: clientes que HAN hecho al menos un pedido
SELECT cl.nombre
FROM clientes cl
WHERE EXISTS (
    SELECT 1 FROM pedidos pe
    WHERE pe.id_cliente = cl.id_cliente
);
-- Resultado: Ana López, Pedro Ruiz, María García (todos)

-- E-commerce: productos que NADIE ha comprado
SELECT pr.nombre, pr.precio
FROM productos pr
WHERE NOT EXISTS (
    SELECT 1 FROM pedidos pe
    WHERE pe.id_producto = pr.id_producto
);
-- Resultado: Sofá de Cuero (405.00)

-- Hospital: especialidades que tienen al menos una cita completada
SELECT e.nombre AS especialidad
FROM especialidades e
WHERE EXISTS (
    SELECT 1
    FROM citas ci
    INNER JOIN medicos m ON ci.id_medico = m.id_medico
    WHERE m.id_especialidad = e.id_especialidad
      AND ci.estado = 'C'
);
-- Resultado: Neurología, Traumatología

-- Aerolínea: aviones que NO tienen ningún vuelo asignado
SELECT a.modelo, a.capacidad
FROM aviones a
WHERE NOT EXISTS (
    SELECT 1 FROM vuelos v
    WHERE v.id_avion = a.id_avion
);
-- Resultado: Boeing 737
```

### 🧠 El Reto

Encuentra los **pacientes que NO tienen ninguna cita cancelada** (estado `'X'`). Usa `NOT EXISTS`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pa.nombre_completo
FROM pacientes pa
WHERE NOT EXISTS (
    SELECT 1 FROM citas ci
    WHERE ci.id_paciente = pa.id_paciente
      AND ci.estado = 'X'
);
```

Resultado:
| nombre_completo |
|----------------|
| Laura Martinez |
| Carlos Gomez |
| Pedro Sánchez |
| David Torres |

> 💡 Solo Ana Ruiz tiene una cita cancelada (cita 5), así que los otros 4 pacientes aparecen en el resultado.

</details>

---

---

## 10.5 ANY / ALL

### 📘 El Concepto

`ANY` (o su sinónimo `SOME`) y `ALL` comparan un valor contra **un conjunto de valores** devuelto por una subconsulta.

| Operador | Significado | Ejemplo |
|----------|-------------|---------|
| `> ANY (subconsulta)` | Mayor que **al menos uno** del conjunto | `precio > ANY (SELECT ...)` |
| `> ALL (subconsulta)` | Mayor que **todos** los del conjunto | `precio > ALL (SELECT ...)` |
| `< ANY (subconsulta)` | Menor que **al menos uno** | — |
| `< ALL (subconsulta)` | Menor que **todos** | — |
| `= ANY (subconsulta)` | Igual a **al menos uno** (equivale a `IN`) | — |

> 📌 **Truco para recordar:**
> - `> ANY` = mayor que el **mínimo** del conjunto.
> - `> ALL` = mayor que el **máximo** del conjunto.
> - `< ANY` = menor que el **máximo** del conjunto.
> - `< ALL` = menor que el **mínimo** del conjunto.

### 🏠 La Analogía

Imagina una competición de salto de altura:
- `> ANY (1.80, 1.90, 2.00)`: _"¿Saltaste más que ALGUNO de estos rivales?"_ → Si saltaste 1.85m, sí (superaste al de 1.80).
- `> ALL (1.80, 1.90, 2.00)`: _"¿Saltaste más que TODOS estos rivales?"_ → Necesitas superar los 2.00m.

### 💻 El Código

```sql
-- Hospital: médicos que ganan más que ALGÚN neurólogo
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base > ANY (
    SELECT salario_base FROM medicos WHERE id_especialidad = 200
);
-- Neurólogos ganan: 4000, 4500
-- > ANY (4000, 4500) = > 4000
-- Resultado: Dra. Marta López (4500)

-- Hospital: médicos que ganan más que TODOS los traumatólogos
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base > ALL (
    SELECT salario_base FROM medicos WHERE id_especialidad = 100
);
-- Traumatólogos ganan: 3200, 2800
-- > ALL (3200, 2800) = > 3200
-- Resultado: Dra. Sarah Adams (4000), Dra. Marta López (4500)

-- E-commerce: productos más caros que TODOS los productos de Ropa
SELECT nombre, precio
FROM productos
WHERE precio > ALL (
    SELECT precio FROM productos WHERE id_categoria = 3
);
-- Ropa: 19.99, 89.50. > ALL = > 89.50
-- Resultado: Laptop Pro(1220), Sofá(405), Monitor(350), Ratón no(45.50)

-- Aerolínea: rutas con distancia mayor que ALGUNA ruta desde Madrid
SELECT id_ruta, origen || ' → ' || destino AS ruta, distancia_km
FROM rutas
WHERE distancia_km > ANY (
    SELECT distancia_km FROM rutas WHERE origen = 'MAD'
);
-- Rutas desde MAD: 1350, 1420. > ANY = > 1350
-- Resultado: JFKLAX(4000), MADFRA(1420)
```

### 🧠 El Reto

Encuentra los **productos cuyo precio es menor que TODOS los productos de la categoría Electrónica** (id_categoria = 1). ¿Cuáles son?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
WHERE precio < ALL (
    SELECT precio FROM productos WHERE id_categoria = 1
);
```

Electrónica tiene: 1220, 45.50, 350, 75. El mínimo es 45.50.
`< ALL` = menor que 45.50.

Resultado:
| nombre | precio |
|--------|--------|
| Lámpara LED | 40.50 |
| Camiseta Básica | 19.99 |

</details>

---

---

## 10.6 Subconsultas en FROM (Inline Views)

### 📘 El Concepto

Una **inline view** (vista en línea) es una subconsulta colocada en la cláusula `FROM`. Actúa como una **tabla temporal** que puedes consultar, filtrar y unir con otras tablas.

```
SELECT alias.columna
FROM (
    SELECT columnas
    FROM tabla
    WHERE condicion
    GROUP BY columnas
) alias
WHERE alias.columna > valor;
```

> 📌 En Oracle, las inline views **deben tener un alias**. Es obligatorio.

### 🏠 La Analogía

Imagina que estás preparando una presentación con gráficos. Primero **resumes los datos en una tabla intermedia** (la inline view) y luego trabajas con ese resumen para generar el gráfico final. No modificas los datos originales; solo creas un "borrador intermedio" que te facilita la tarea.

### 💻 El Código

```sql
-- E-commerce: clientes cuyo gasto total supera los 300€
-- Paso 1: calcular el gasto por cliente (inline view)
-- Paso 2: filtrar los que superan 300
SELECT resumen.cliente, resumen.gasto_total
FROM (
    SELECT cl.nombre AS cliente,
           SUM(pe.total) AS gasto_total
    FROM pedidos pe
    INNER JOIN clientes cl ON pe.id_cliente = cl.id_cliente
    GROUP BY cl.nombre
) resumen
WHERE resumen.gasto_total > 300;
-- Resultado: Ana López (1376.00), Pedro Ruiz (486.50)

-- Hospital: especialidades con más de 1 cita completada
SELECT esp_resumen.especialidad, esp_resumen.citas_completadas
FROM (
    SELECT e.nombre AS especialidad,
           COUNT(*) AS citas_completadas
    FROM citas ci
    INNER JOIN medicos m ON ci.id_medico = m.id_medico
    INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
    WHERE ci.estado = 'C'
    GROUP BY e.nombre
) esp_resumen
WHERE esp_resumen.citas_completadas > 1;
-- Resultado: Neurología (2), Traumatología (2)

-- Aerolínea: aviones cuyo promedio de distancia de vuelo supera 1500 km
SELECT flota.avion, flota.vuelos, flota.distancia_promedio
FROM (
    SELECT a.modelo AS avion,
           COUNT(v.id_vuelo) AS vuelos,
           ROUND(AVG(r.distancia_km), 0) AS distancia_promedio
    FROM aviones a
    INNER JOIN vuelos v ON a.id_avion = v.id_avion
    INNER JOIN rutas r ON v.id_ruta = r.id_ruta
    GROUP BY a.modelo
) flota
WHERE flota.distancia_promedio > 1500;
-- Resultado: Boeing 777 (promedio 2675 km)
```

### 🧠 El Reto

Usa una inline view para encontrar las **categorías cuyo ingreso total por pedidos supere los 200€**. La inline view debe calcular el ingreso total por categoría (cruzando `pedidos` → `productos` → `categorias`), y la consulta externa debe filtrar las que superan 200.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT cat_ventas.categoria, cat_ventas.ingreso_total
FROM (
    SELECT ca.nombre_categoria AS categoria,
           SUM(pe.total) AS ingreso_total
    FROM pedidos pe
    INNER JOIN productos pr ON pe.id_producto = pr.id_producto
    INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
    GROUP BY ca.nombre_categoria
) cat_ventas
WHERE cat_ventas.ingreso_total > 200;
```

Resultado:
| categoria | ingreso_total |
|-----------|---------------|
| Electrónica | 1781.50 |
| Ropa | 278.95 |

> 💡 Hogar solo tiene 81.00€ en pedidos (2 × Lámpara LED), así que queda excluida.

</details>

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 10

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de Subconsultas](ejercicios/ejercicios_subconsultas.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Proyecto: La Investigación Interna](proyectos/proyecto_subconsultas.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 11: Vistas y Objetos de BD](../11-vistas) |

---

<div align="center">

⬅️ [**Tema 09: Relaciones y JOINs**](../09-joins) · 🏠 [**Índice del Curso**](../README.md) · [**Tema 11: Vistas y Objetos de BD →**](../11-vistas)

</div>
