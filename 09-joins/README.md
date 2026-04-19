# 🔗 Tema 09: Relaciones y JOINs

> **"Una tabla sola cuenta una historia parcial. La magia ocurre cuando conectas varias tablas y descubres la historia completa."** Los JOINs son el pegamento del modelo relacional: permiten cruzar información de distintas tablas en una única consulta.

## 📋 Índice

- [Expansión de Datos — Día 9](#-expansión-de-datos--día-9)
- [9.1 INNER JOIN](#91-inner-join)
- [9.2 LEFT JOIN (LEFT OUTER JOIN)](#92-left-join-left-outer-join)
- [9.3 RIGHT JOIN (RIGHT OUTER JOIN)](#93-right-join-right-outer-join)
- [9.4 FULL OUTER JOIN](#94-full-outer-join)
- [9.5 CROSS JOIN (Producto Cartesiano)](#95-cross-join-producto-cartesiano)
- [9.6 SELF JOIN](#96-self-join)
- [9.7 Sintaxis Antigua Oracle (+) vs ANSI](#97-sintaxis-antigua-oracle--vs-ansi)
- [9.8 Multi-table JOINs (3+ tablas)](#98-multi-table-joins-3-tablas)

---

## 🗃️ Expansión de Datos — Día 9

Antes de comenzar necesitamos datos en las tablas que aún están vacías y una nueva tabla de pedidos para practicar JOINs.

```sql
-- ====================================================
-- EXPANSIÓN DE DATOS — TEMA 09
-- Ejecutar ANTES de los ejercicios de este tema
-- ====================================================

-- Hospital: Citas
INSERT INTO citas VALUES (1, 1, 2, TIMESTAMP '2025-01-10 09:00:00', 'C');
INSERT INTO citas VALUES (2, 2, 4, TIMESTAMP '2025-01-10 10:30:00', 'C');
INSERT INTO citas VALUES (3, 3, 6, TIMESTAMP '2025-01-15 11:00:00', 'P');
INSERT INTO citas VALUES (4, 1, 5, TIMESTAMP '2025-01-20 09:30:00', 'C');
INSERT INTO citas VALUES (5, 4, 2, TIMESTAMP '2025-02-01 14:00:00', 'X');
INSERT INTO citas VALUES (6, 5, 7, TIMESTAMP '2025-02-05 08:00:00', 'P');
INSERT INTO citas VALUES (7, 2, 6, TIMESTAMP '2025-02-10 16:00:00', 'C');

-- Aerolínea: Vuelos
INSERT INTO vuelos VALUES (1001, 'MADLHR', 30, TIMESTAMP '2025-03-01 07:30:00', 'T4-A1');
INSERT INTO vuelos VALUES (1002, 'JFKLAX', 50, TIMESTAMP '2025-03-01 14:00:00', 'B-22');
INSERT INTO vuelos VALUES (1003, 'BCNCDG', 40, TIMESTAMP '2025-03-02 06:00:00', 'T1-C3');
INSERT INTO vuelos VALUES (1004, 'MADFRA', 30, TIMESTAMP '2025-03-02 11:15:00', 'T4-B5');
INSERT INTO vuelos VALUES (1005, 'MADLHR', 50, TIMESTAMP '2025-03-03 07:30:00', 'T4-A1');
INSERT INTO vuelos VALUES (1006, 'BCNFCO', 40, TIMESTAMP '2025-03-04 09:00:00', 'T2-D1');

-- E-commerce: Tabla pedidos (nueva)
CREATE TABLE pedidos (
    id_pedido   NUMBER(8) PRIMARY KEY,
    id_cliente  NUMBER(10) REFERENCES clientes(id_cliente),
    id_producto NUMBER(10) REFERENCES productos(id_producto),
    cantidad    NUMBER(3) NOT NULL,
    fecha_pedido DATE DEFAULT SYSDATE,
    total       NUMBER(10,2)
);

INSERT INTO pedidos VALUES (1, 1, 10, 1, TO_DATE('2025-01-05','YYYY-MM-DD'), 1220.00);
INSERT INTO pedidos VALUES (2, 1, 13, 2, TO_DATE('2025-01-05','YYYY-MM-DD'), 81.00);
INSERT INTO pedidos VALUES (3, 2, 11, 3, TO_DATE('2025-01-12','YYYY-MM-DD'), 136.50);
INSERT INTO pedidos VALUES (4, 3, 14, 5, TO_DATE('2025-01-20','YYYY-MM-DD'), 99.95);
INSERT INTO pedidos VALUES (5, 2, 16, 1, TO_DATE('2025-02-01','YYYY-MM-DD'), 350.00);
INSERT INTO pedidos VALUES (6, 1, 17, 1, TO_DATE('2025-02-10','YYYY-MM-DD'), 75.00);
INSERT INTO pedidos VALUES (7, 3, 15, 2, TO_DATE('2025-02-15','YYYY-MM-DD'), 179.00);

COMMIT;
```

> 📌 **Nota importante:** El avión Boeing 737 (`id_avion = 20`) **no tiene ningún vuelo asignado** intencionalmente. Esto es clave para practicar `LEFT JOIN`.

---

---

## 9.1 INNER JOIN

### 📘 El Concepto

`INNER JOIN` devuelve **solo las filas que tienen coincidencia** en ambas tablas. Si una fila de la tabla A no tiene correspondencia en la tabla B, se descarta del resultado.

```
SELECT columnas
FROM tabla_a A
INNER JOIN tabla_b B ON A.clave = B.clave;
```

Visualmente, es la **intersección** de dos conjuntos: solo lo que existe a ambos lados.

```
  Tabla A       Tabla B
 ┌───────┐    ┌───────┐
 │       │    │       │
 │   ┌───┼────┼───┐   │
 │   │ ██│████│██ │   │
 │   └───┼────┼───┘   │
 │       │    │       │
 └───────┘    └───────┘
      Resultado: ████
```

### 🏠 La Analogía

Imagina una **fiesta con lista VIP**. Hay dos listas: la lista de invitados (tabla A) y la lista de personas que realmente llegaron (tabla B). El `INNER JOIN` te da **solo las personas que estaban en la lista Y además llegaron a la fiesta**. Si alguien estaba invitado pero no vino, no aparece. Si alguien llegó sin invitación, tampoco.

### 💻 El Código

```sql
-- E-commerce: productos con el nombre de su categoría
SELECT p.nombre AS producto,
       p.precio,
       c.nombre_categoria AS categoria
FROM productos p
INNER JOIN categorias c ON p.id_categoria = c.id_categoria;

-- Hospital: médicos con el nombre de su especialidad
SELECT m.nombre_completo AS medico,
       e.nombre AS especialidad,
       m.salario_base
FROM medicos m
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad;

-- Aerolínea: vuelos con detalle de ruta y avión
SELECT v.id_vuelo,
       r.origen || ' → ' || r.destino AS ruta,
       a.modelo AS avion,
       v.fecha_hora_salida
FROM vuelos v
INNER JOIN rutas r ON v.id_ruta = r.id_ruta
INNER JOIN aviones a ON v.id_avion = a.id_avion;
```

### 🧠 El Reto

El departamento de ventas necesita un informe que muestre **cada pedido** junto con el **nombre del cliente** y el **nombre del producto**. Usa `INNER JOIN` para cruzar `pedidos`, `clientes` y `productos`.

Muestra: `id_pedido`, `nombre_cliente`, `nombre_producto`, `cantidad`, `total`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pe.id_pedido,
       cl.nombre AS nombre_cliente,
       pr.nombre AS nombre_producto,
       pe.cantidad,
       pe.total
FROM pedidos pe
INNER JOIN clientes cl ON pe.id_cliente = cl.id_cliente
INNER JOIN productos pr ON pe.id_producto = pr.id_producto;
```

Resultado:
| id_pedido | nombre_cliente | nombre_producto | cantidad | total |
|-----------|---------------|-----------------|----------|-------|
| 1 | Ana López | Laptop Pro | 1 | 1220.00 |
| 2 | Ana López | Lámpara LED | 2 | 81.00 |
| 3 | Pedro Ruiz | Ratón Inalámbrico | 3 | 136.50 |
| 4 | María García | Camiseta Básica | 5 | 99.95 |
| 5 | Pedro Ruiz | Monitor 4K | 1 | 350.00 |
| 6 | Ana López | Teclado Mecánico | 1 | 75.00 |
| 7 | María García | Zapatillas Running | 2 | 179.00 |

</details>

---

---

## 9.2 LEFT JOIN (LEFT OUTER JOIN)

### 📘 El Concepto

`LEFT JOIN` (o `LEFT OUTER JOIN`) devuelve **todas las filas de la tabla izquierda**, aunque no tengan coincidencia en la tabla derecha. Cuando no hay coincidencia, las columnas de la tabla derecha aparecen como `NULL`.

```
SELECT columnas
FROM tabla_a A
LEFT JOIN tabla_b B ON A.clave = B.clave;
```

```
  Tabla A       Tabla B
 ┌───────┐    ┌───────┐
 │       │    │       │
 │ ██┌───┼────┼───┐   │
 │ ██│ ██│████│██ │   │
 │ ██└───┼────┼───┘   │
 │       │    │       │
 └───────┘    └───────┘
 Resultado: ████████
 (toda tabla A + coincidencias de B)
```

### 🏠 La Analogía

Eres el profesor que pasa lista. Tienes la lista completa de alumnos (tabla izquierda) y la lista de los que entregaron la tarea (tabla derecha). El `LEFT JOIN` te muestra **todos los alumnos**: los que entregaron tarea aparecen con su nota, y los que no entregaron aparecen con `NULL` en la nota. ¡Ningún alumno se queda fuera de la lista!

### 💻 El Código

```sql
-- Aerolínea: TODOS los aviones, tengan o no vuelos asignados
SELECT a.modelo,
       a.capacidad,
       v.id_vuelo,
       v.fecha_hora_salida
FROM aviones a
LEFT JOIN vuelos v ON a.id_avion = v.id_avion
ORDER BY a.modelo;

-- El Boeing 737 (id=20) aparecerá con NULL en id_vuelo y fecha
-- porque NO tiene vuelos asignados

-- E-commerce: todos los productos, tengan o no pedidos
SELECT pr.nombre AS producto,
       pr.precio,
       pe.id_pedido,
       pe.cantidad
FROM productos pr
LEFT JOIN pedidos pe ON pr.id_producto = pe.id_producto
ORDER BY pr.nombre;

-- Productos como "Sofá de Cuero" aparecen con NULL en pedido
-- porque nadie los ha comprado aún

-- Truco: encontrar registros SIN coincidencia (anti-join)
-- Aviones que NO tienen vuelos asignados
SELECT a.modelo, a.capacidad
FROM aviones a
LEFT JOIN vuelos v ON a.id_avion = v.id_avion
WHERE v.id_vuelo IS NULL;
-- Resultado: Boeing 737 (único avión sin vuelos)
```

### 🧠 El Reto

El gerente del hospital quiere saber qué **pacientes NO tienen citas programadas**. Usa `LEFT JOIN` entre `pacientes` y `citas`, filtrando los que tienen `NULL` en el lado de citas.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pa.nombre_completo AS paciente,
       pa.telefono
FROM pacientes pa
LEFT JOIN citas ci ON pa.id_paciente = ci.id_paciente
WHERE ci.id_cita IS NULL;
```

Resultado: **0 filas** — en nuestros datos, todos los pacientes (1-5) tienen al menos una cita. Esto también es un resultado válido: confirma que no hay pacientes sin atención.

> 💡 Si quisiéramos un caso con resultado, la consulta de aviones sin vuelos (`WHERE v.id_vuelo IS NULL`) devuelve el Boeing 737.

</details>

---

---

## 9.3 RIGHT JOIN (RIGHT OUTER JOIN)

### 📘 El Concepto

`RIGHT JOIN` es el espejo del `LEFT JOIN`: devuelve **todas las filas de la tabla derecha**, aunque no tengan coincidencia en la tabla izquierda. Las columnas de la tabla izquierda aparecen como `NULL` cuando no hay match.

```
SELECT columnas
FROM tabla_a A
RIGHT JOIN tabla_b B ON A.clave = B.clave;
```

```
  Tabla A       Tabla B
 ┌───────┐    ┌───────┐
 │       │    │       │
 │   ┌───┼────┼───┐██ │
 │   │ ██│████│██ │██ │
 │   └───┼────┼───┘██ │
 │       │    │       │
 └───────┘    └───────┘
        Resultado: ████████
        (coincidencias de A + toda tabla B)
```

> 📌 **Consejo práctico:** La mayoría de desarrolladores prefieren usar `LEFT JOIN` y simplemente invertir el orden de las tablas, en lugar de usar `RIGHT JOIN`. Ambas formas producen el mismo resultado.

### 🏠 La Analogía

Imagina que eres el gerente de un restaurante. Tienes la lista de pedidos (tabla izquierda) y el menú completo (tabla derecha). Un `RIGHT JOIN` te muestra **todos los platos del menú**, incluso los que nadie ha pedido hoy. Los platos sin pedidos aparecen con `NULL` en la columna del pedido.

### 💻 El Código

```sql
-- E-commerce: TODOS los productos (derecha), con sus pedidos si existen
SELECT pe.id_pedido,
       pe.cantidad,
       pr.nombre AS producto,
       pr.precio
FROM pedidos pe
RIGHT JOIN productos pr ON pe.id_producto = pr.id_producto
ORDER BY pr.nombre;

-- Esto equivale a: productos LEFT JOIN pedidos
-- Los productos sin pedidos (Sofá de Cuero, etc.) aparecen con NULL

-- Hospital: TODAS las especialidades (derecha), con sus médicos
SELECT m.nombre_completo AS medico,
       e.nombre AS especialidad
FROM medicos m
RIGHT JOIN especialidades e ON m.id_especialidad = e.id_especialidad;
-- Todas las especialidades aparecen; si alguna no tuviera médicos,
-- la columna 'medico' sería NULL
```

### 🧠 El Reto

Usando `RIGHT JOIN`, muestra **todas las rutas de la aerolínea** con la información de sus vuelos (si tienen alguno). ¿Hay alguna ruta sin vuelos programados?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT r.id_ruta,
       r.origen || ' → ' || r.destino AS ruta,
       v.id_vuelo,
       v.fecha_hora_salida
FROM vuelos v
RIGHT JOIN rutas r ON v.id_ruta = r.id_ruta
ORDER BY r.id_ruta;
```

Resultado:
| id_ruta | ruta | id_vuelo | fecha_hora_salida |
|---------|------|----------|-------------------|
| BCNCDG | BCN → CDG | 1003 | 2025-03-02 06:00 |
| BCNFCO | BCN → FCO | 1006 | 2025-03-04 09:00 |
| MADFRA | MAD → FRA | 1004 | 2025-03-02 11:15 |
| MADLHR | MAD → LHR | 1001 | 2025-03-01 07:30 |
| MADLHR | MAD → LHR | 1005 | 2025-03-03 07:30 |
| JFKLAX | JFK → LAX | 1002 | 2025-03-01 14:00 |

Todas las rutas tienen al menos un vuelo asignado, así que no aparecen NULLs en este caso.

</details>

---

---

## 9.4 FULL OUTER JOIN

### 📘 El Concepto

`FULL OUTER JOIN` devuelve **todas las filas de ambas tablas**. Cuando hay coincidencia, se combinan. Cuando no la hay en un lado, se rellena con `NULL`.

```
SELECT columnas
FROM tabla_a A
FULL OUTER JOIN tabla_b B ON A.clave = B.clave;
```

```
  Tabla A       Tabla B
 ┌───────┐    ┌───────┐
 │       │    │       │
 │ ██┌───┼────┼───┐██ │
 │ ██│ ██│████│██ │██ │
 │ ██└───┼────┼───┘██ │
 │       │    │       │
 └───────┘    └───────┘
 Resultado: ████████████
 (todo de A + todo de B)
```

Es la **unión completa**: ves absolutamente todo, coincida o no.

### 🏠 La Analogía

Imagina dos equipos de fútbol que se fusionan. Tienes la lista de jugadores del equipo A y la del equipo B. Algunos jugadores estaban en ambos equipos. El `FULL OUTER JOIN` te da la **lista completa de TODOS los jugadores**: los que estaban solo en A, los que estaban solo en B, y los que estaban en ambos.

### 💻 El Código

```sql
-- Aerolínea: ver TODOS los aviones y TODAS las rutas de vuelos
-- Aviones sin vuelo aparecen con NULL en ruta
-- Rutas sin vuelo aparecerían con NULL en avión (si existieran)
SELECT a.modelo AS avion,
       v.id_vuelo,
       r.origen || ' → ' || r.destino AS ruta
FROM aviones a
FULL OUTER JOIN vuelos v ON a.id_avion = v.id_avion
FULL OUTER JOIN rutas r ON v.id_ruta = r.id_ruta
ORDER BY a.modelo;

-- E-commerce: cruce completo productos ↔ pedidos
SELECT pr.nombre AS producto,
       pe.id_pedido,
       pe.cantidad
FROM productos pr
FULL OUTER JOIN pedidos pe ON pr.id_producto = pe.id_producto
ORDER BY pr.nombre;
```

### 🧠 El Reto

El director de TI necesita un informe que muestre la relación completa entre **aviones** y **vuelos**. Usa `FULL OUTER JOIN` y añade una columna calculada `estado` que diga:
- `'Sin vuelos'` si el avión no tiene vuelos.
- `'Asignado'` si el avión tiene vuelos.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT a.modelo,
       a.capacidad,
       v.id_vuelo,
       v.fecha_hora_salida,
       CASE
           WHEN v.id_vuelo IS NULL THEN 'Sin vuelos'
           ELSE 'Asignado'
       END AS estado
FROM aviones a
FULL OUTER JOIN vuelos v ON a.id_avion = v.id_avion
ORDER BY a.modelo, v.id_vuelo;
```

Resultado:
| modelo | capacidad | id_vuelo | fecha_hora_salida | estado |
|--------|-----------|----------|-------------------|--------|
| Airbus A350 | 300 | 1001 | 2025-03-01 07:30 | Asignado |
| Airbus A350 | 300 | 1004 | 2025-03-02 11:15 | Asignado |
| Boeing 737 | 180 | NULL | NULL | Sin vuelos |
| Boeing 777 | 350 | 1002 | 2025-03-01 14:00 | Asignado |
| Boeing 777 | 350 | 1005 | 2025-03-03 07:30 | Asignado |
| Embraer E195 | 120 | 1003 | 2025-03-02 06:00 | Asignado |
| Embraer E195 | 120 | 1006 | 2025-03-04 09:00 | Asignado |

</details>

---

---

## 9.5 CROSS JOIN (Producto Cartesiano)

### 📘 El Concepto

`CROSS JOIN` genera el **producto cartesiano**: combina **cada fila** de la tabla A con **cada fila** de la tabla B. Si A tiene 5 filas y B tiene 3, el resultado tiene 5 × 3 = 15 filas.

```
SELECT columnas
FROM tabla_a A
CROSS JOIN tabla_b B;
```

> ⚠️ **Cuidado:** El CROSS JOIN puede generar resultados **enormes**. Con tablas de 1.000 filas cada una, el resultado tiene 1.000.000 de filas. Úsalo solo cuando realmente necesites todas las combinaciones.

### 🏠 La Analogía

Piensa en una tienda de camisetas. Tienes 4 colores y 3 tallas. El `CROSS JOIN` genera todas las combinaciones posibles: rojo-S, rojo-M, rojo-L, azul-S, azul-M, azul-L... ¡12 combinaciones en total! Es como generar un catálogo completo.

### 💻 El Código

```sql
-- E-commerce: todas las combinaciones de categorías × clientes
-- (útil para análisis de cobertura: ¿qué clientes no han comprado en qué categorías?)
SELECT cl.nombre AS cliente,
       ca.nombre_categoria AS categoria
FROM clientes cl
CROSS JOIN categorias ca
ORDER BY cl.nombre, ca.nombre_categoria;
-- Resultado: 3 clientes × 3 categorías = 9 filas

-- Aerolínea: todas las combinaciones posibles de avión × ruta
SELECT a.modelo,
       r.id_ruta,
       r.origen || ' → ' || r.destino AS ruta
FROM aviones a
CROSS JOIN rutas r
ORDER BY a.modelo, r.id_ruta;
-- Resultado: 4 aviones × 5 rutas = 20 combinaciones
```

### 🧠 El Reto

El departamento de marketing quiere generar una **matriz de todas las combinaciones posibles** entre clientes y categorías de productos, para analizar qué combinaciones faltan en los pedidos. Genera el `CROSS JOIN` entre `clientes` y `categorias`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT cl.nombre AS cliente,
       ca.nombre_categoria AS categoria
FROM clientes cl
CROSS JOIN categorias ca
ORDER BY cl.nombre, ca.nombre_categoria;
```

Resultado (9 filas):
| cliente | categoria |
|---------|-----------|
| Ana López | Electrónica |
| Ana López | Hogar |
| Ana López | Ropa |
| María García | Electrónica |
| María García | Hogar |
| María García | Ropa |
| Pedro Ruiz | Electrónica |
| Pedro Ruiz | Hogar |
| Pedro Ruiz | Ropa |

</details>

---

---

## 9.6 SELF JOIN

### 📘 El Concepto

Un `SELF JOIN` es cuando una tabla se une **consigo misma**. Se usa cuando hay una relación jerárquica o de referencia dentro de la misma tabla (por ejemplo: un empleado tiene un jefe, y el jefe también es un empleado de la misma tabla).

```
SELECT a.columna, b.columna
FROM tabla a
INNER JOIN tabla b ON a.referencia = b.clave_primaria;
```

> 📌 Es **obligatorio** usar alias diferentes para distinguir las dos "copias" de la misma tabla.

### 🏠 La Analogía

Piensa en un **árbol genealógico** donde todos los nombres están en una única lista. Si quieres saber quién es el padre de cada persona, necesitas "mirar la misma lista dos veces": una vez como hijo y otra como padre. El SELF JOIN es exactamente eso: consultar la misma tabla con dos roles diferentes.

### 💻 El Código

```sql
-- Hospital: comparar médicos de la misma especialidad
-- "Para cada médico, muestra otros médicos de su misma especialidad"
SELECT m1.nombre_completo AS medico,
       m2.nombre_completo AS colega,
       m1.id_especialidad
FROM medicos m1
INNER JOIN medicos m2
    ON m1.id_especialidad = m2.id_especialidad
   AND m1.id_medico <> m2.id_medico
ORDER BY m1.nombre_completo;

-- Resultado incluye pares como:
-- Dr. Carlos Vega ↔ Dra. Elena Ruiz (ambos en Traumatología)
-- Dra. Sarah Adams ↔ Dra. Marta López (ambos en Neurología)

-- E-commerce: productos más caros que otros de la misma categoría
SELECT p1.nombre AS producto,
       p1.precio,
       p2.nombre AS producto_mas_barato,
       p2.precio AS precio_mas_barato
FROM productos p1
INNER JOIN productos p2
    ON p1.id_categoria = p2.id_categoria
   AND p1.precio > p2.precio
ORDER BY p1.id_categoria, p1.precio DESC;
```

### 🧠 El Reto

Encuentra pares de **médicos que comparten especialidad** donde uno gana más que el otro. Muestra: `medico_senior` (el que gana más), `salario_senior`, `medico_junior` (el que gana menos), `salario_junior`, `diferencia_salarial`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT m1.nombre_completo AS medico_senior,
       m1.salario_base AS salario_senior,
       m2.nombre_completo AS medico_junior,
       m2.salario_base AS salario_junior,
       m1.salario_base - m2.salario_base AS diferencia_salarial
FROM medicos m1
INNER JOIN medicos m2
    ON m1.id_especialidad = m2.id_especialidad
   AND m1.salario_base > m2.salario_base
ORDER BY diferencia_salarial DESC;
```

Resultado:
| medico_senior | salario_senior | medico_junior | salario_junior | diferencia |
|---------------|----------------|---------------|----------------|------------|
| Dra. Marta López | 4500 | Dra. Sarah Adams | 4000 | 500 |
| Dra. Elena Ruiz | 3200 | Dr. Carlos Vega | 2800 | 400 |

</details>

---

---

## 9.7 Sintaxis Antigua Oracle (+) vs ANSI

### 📘 El Concepto

Antes del estándar SQL:1999, Oracle usaba una **sintaxis propietaria** para los JOINs basada en el operador `(+)` en la cláusula `WHERE`. El `(+)` se coloca en el lado de la tabla que puede tener `NULL`s (el lado "deficiente").

| Tipo JOIN | Sintaxis ANSI (moderna ✅) | Sintaxis Oracle antigua ⚠️ |
|-----------|---------------------------|----------------------------|
| INNER JOIN | `FROM a JOIN b ON a.id = b.id` | `FROM a, b WHERE a.id = b.id` |
| LEFT JOIN | `FROM a LEFT JOIN b ON a.id = b.id` | `FROM a, b WHERE a.id = b.id(+)` |
| RIGHT JOIN | `FROM a RIGHT JOIN b ON a.id = b.id` | `FROM a, b WHERE a.id(+) = b.id` |
| FULL OUTER | `FROM a FULL OUTER JOIN b ON ...` | ❌ No se puede con `(+)` |
| CROSS JOIN | `FROM a CROSS JOIN b` | `FROM a, b` (sin WHERE) |

> ⚠️ **Recomendación:** Siempre usa la **sintaxis ANSI**. Es más legible, portable y soporta `FULL OUTER JOIN`. La sintaxis `(+)` solo se mantiene por compatibilidad con código antiguo.

### 🏠 La Analogía

Es como las instrucciones de un mueble de IKEA. La versión antigua (`(+)`) era como instrucciones escritas solo en sueco: funcionaban, pero pocos las entendían a la primera. La versión ANSI es como las instrucciones con dibujos universales: cualquier persona de cualquier país las comprende.

### 💻 El Código

```sql
-- ❌ SINTAXIS ANTIGUA (no recomendada, solo para leer código heredado)
-- LEFT JOIN: todos los aviones, con sus vuelos si existen
SELECT a.modelo, v.id_vuelo
FROM aviones a, vuelos v
WHERE a.id_avion = v.id_avion(+);
-- El (+) va en el lado de vuelos porque es el que puede faltar

-- ✅ EQUIVALENTE ANSI (recomendada)
SELECT a.modelo, v.id_vuelo
FROM aviones a
LEFT JOIN vuelos v ON a.id_avion = v.id_avion;

-- ❌ SINTAXIS ANTIGUA: RIGHT JOIN
SELECT m.nombre_completo, e.nombre
FROM medicos m, especialidades e
WHERE m.id_especialidad(+) = e.id_especialidad;

-- ✅ EQUIVALENTE ANSI
SELECT m.nombre_completo, e.nombre
FROM medicos m
RIGHT JOIN especialidades e ON m.id_especialidad = e.id_especialidad;
```

### 🧠 El Reto

Traduce esta consulta con sintaxis antigua a **sintaxis ANSI moderna**:

```sql
SELECT p.nombre, pe.id_pedido, pe.total
FROM productos p, pedidos pe
WHERE p.id_producto = pe.id_producto(+);
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Traducción a ANSI:
SELECT p.nombre, pe.id_pedido, pe.total
FROM productos p
LEFT JOIN pedidos pe ON p.id_producto = pe.id_producto;
```

Explicación: El `(+)` estaba en el lado de `pedidos`, lo que significa que `pedidos` es la tabla que puede faltar → es un `LEFT JOIN` con `productos` como tabla principal (izquierda).

</details>

---

---

## 9.8 Multi-table JOINs (3+ tablas)

### 📘 El Concepto

En el mundo real, las consultas rara vez involucran solo 2 tablas. Es muy común encadenar **3, 4 o más JOINs** en una sola consulta para obtener toda la información necesaria.

```
SELECT columnas
FROM tabla_a A
INNER JOIN tabla_b B ON A.clave = B.clave
INNER JOIN tabla_c C ON B.clave2 = C.clave2
LEFT JOIN  tabla_d D ON C.clave3 = D.clave3;
```

La clave es pensar en **cadenas de relaciones**: A se conecta con B, B con C, y C con D. Cada `JOIN` añade una nueva fuente de datos.

### 🏠 La Analogía

Piensa en una **cadena de montaje** en una fábrica:
1. Estación 1: tomas el pedido (tabla `pedidos`).
2. Estación 2: buscas la ficha del cliente (JOIN con `clientes`).
3. Estación 3: buscas los detalles del producto (JOIN con `productos`).
4. Estación 4: clasificas por categoría (JOIN con `categorias`).

Cada estación añade más información al producto final (tu consulta).

### 💻 El Código

```sql
-- E-commerce: informe completo de pedidos (4 tablas)
SELECT pe.id_pedido,
       cl.nombre AS cliente,
       pr.nombre AS producto,
       ca.nombre_categoria AS categoria,
       pe.cantidad,
       pe.total,
       pe.fecha_pedido
FROM pedidos pe
INNER JOIN clientes cl ON pe.id_cliente = cl.id_cliente
INNER JOIN productos pr ON pe.id_producto = pr.id_producto
INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
ORDER BY pe.fecha_pedido;

-- Hospital: historial completo de citas (4 tablas)
SELECT ci.id_cita,
       pa.nombre_completo AS paciente,
       m.nombre_completo AS medico,
       e.nombre AS especialidad,
       ci.fecha_hora,
       DECODE(ci.estado, 'C', 'Completada', 'P', 'Pendiente', 'X', 'Cancelada') AS estado
FROM citas ci
INNER JOIN pacientes pa ON ci.id_paciente = pa.id_paciente
INNER JOIN medicos m ON ci.id_medico = m.id_medico
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
ORDER BY ci.fecha_hora;

-- Aerolínea: panel de vuelos completo (3 tablas)
SELECT v.id_vuelo,
       r.origen || ' → ' || r.destino AS ruta,
       r.distancia_km,
       a.modelo AS avion,
       a.capacidad,
       v.fecha_hora_salida,
       v.puerta_embarque
FROM vuelos v
INNER JOIN rutas r ON v.id_ruta = r.id_ruta
INNER JOIN aviones a ON v.id_avion = a.id_avion
ORDER BY v.fecha_hora_salida;
```

### 🧠 El Reto

El CEO del E-commerce quiere un **informe de ventas por categoría** que muestre: `categoria`, `num_pedidos`, `total_vendido`, `ticket_medio`. Necesitas cruzar `pedidos` → `productos` → `categorias` y luego agrupar con `GROUP BY`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT ca.nombre_categoria AS categoria,
       COUNT(pe.id_pedido) AS num_pedidos,
       SUM(pe.total) AS total_vendido,
       ROUND(AVG(pe.total), 2) AS ticket_medio
FROM pedidos pe
INNER JOIN productos pr ON pe.id_producto = pr.id_producto
INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
GROUP BY ca.nombre_categoria
ORDER BY total_vendido DESC;
```

Resultado:
| categoria | num_pedidos | total_vendido | ticket_medio |
|-----------|-------------|---------------|--------------|
| Electrónica | 4 | 1781.50 | 445.38 |
| Ropa | 2 | 278.95 | 139.48 |
| Hogar | 1 | 81.00 | 81.00 |

</details>

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 09

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de JOINs](ejercicios/ejercicios_joins.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Proyecto: El Informe Trimestral](proyectos/proyecto_joins.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 10: Subconsultas](../10-subconsultas) |

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 08**](../08-funciones/proyectos/proyecto_funciones.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 09 →**](ejercicios/ejercicios_joins.md)

</div>
