# 🏋️ Ejercicios — Tema 08: Relaciones y JOINs

> 💡 **Instrucciones:** Resuelve cada ejercicio escribiendo la consulta SQL completa. Todos los ejercicios usan el estado acumulado de la base de datos, incluyendo la expansión de datos del Tema 08 (tabla `pedidos`, `citas` y `vuelos`).

---

## Ejercicio 1 — E-commerce · INNER JOIN Básico

**Enunciado:** Muestra un listado de **productos con el nombre de su categoría**. Usa `INNER JOIN` entre `productos` y `categorias`.

Muestra: `nombre` (producto), `precio`, `nombre_categoria`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT p.nombre,
       p.precio,
       c.nombre_categoria
FROM productos p
INNER JOIN categorias c ON p.id_categoria = c.id_categoria
ORDER BY c.nombre_categoria, p.nombre;
```

Resultado (8 filas):
| nombre | precio | nombre_categoria |
|--------|--------|-----------------|
| Laptop Pro | 1220.00 | Electrónica |
| Monitor 4K | 350.00 | Electrónica |
| Ratón Inalámbrico | 45.50 | Electrónica |
| Teclado Mecánico | 75.00 | Electrónica |
| Lámpara LED | 40.50 | Hogar |
| Sofá de Cuero | 405.00 | Hogar |
| Camiseta Básica | 19.99 | Ropa |
| Zapatillas Running | 89.50 | Ropa |

</details>

---

## Ejercicio 2 — Hospital · INNER JOIN Básico

**Enunciado:** Muestra todos los **médicos con el nombre de su especialidad**. Ordena por especialidad y luego por salario descendente.

Muestra: `nombre_completo`, `especialidad`, `salario_base`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT m.nombre_completo,
       e.nombre AS especialidad,
       m.salario_base
FROM medicos m
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
ORDER BY e.nombre, m.salario_base DESC;
```

Resultado:
| nombre_completo | especialidad | salario_base |
|----------------|--------------|-------------|
| Dr. Luis Moreno | Medicina General | 2600 |
| Dra. Marta López | Neurología | 4500 |
| Dra. Sarah Adams | Neurología | 4000 |
| Dra. Elena Ruiz | Traumatología | 3200 |
| Dr. Carlos Vega | Traumatología | 2800 |

</details>

---

## Ejercicio 3 — Aerolínea · LEFT JOIN (aviones sin vuelos)

**Enunciado:** Muestra **TODOS los aviones** y, para los que tengan vuelos, el número de vuelos asignados. El Boeing 737 debería aparecer con `0` vuelos.

Muestra: `modelo`, `capacidad`, `num_vuelos`.

Pista: Usa `LEFT JOIN` y `COUNT(v.id_vuelo)` que ignora NULLs.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT a.modelo,
       a.capacidad,
       COUNT(v.id_vuelo) AS num_vuelos
FROM aviones a
LEFT JOIN vuelos v ON a.id_avion = v.id_avion
GROUP BY a.modelo, a.capacidad
ORDER BY num_vuelos DESC;
```

Resultado:
| modelo | capacidad | num_vuelos |
|--------|-----------|-----------|
| Airbus A350 | 300 | 2 |
| Boeing 777 | 350 | 2 |
| Embraer E195 | 120 | 2 |
| Boeing 737 | 180 | 0 |

</details>

---

## Ejercicio 4 — E-commerce · LEFT JOIN (productos sin pedidos)

**Enunciado:** Lista **TODOS los productos** indicando cuántas unidades se han vendido en total. Los productos sin pedidos deben mostrar `0`.

Muestra: `nombre` (producto), `precio`, `unidades_vendidas`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pr.nombre,
       pr.precio,
       NVL(SUM(pe.cantidad), 0) AS unidades_vendidas
FROM productos pr
LEFT JOIN pedidos pe ON pr.id_producto = pe.id_producto
GROUP BY pr.nombre, pr.precio
ORDER BY unidades_vendidas DESC;
```

Resultado:
| nombre | precio | unidades_vendidas |
|--------|--------|-------------------|
| Camiseta Básica | 19.99 | 5 |
| Ratón Inalámbrico | 45.50 | 3 |
| Lámpara LED | 40.50 | 2 |
| Zapatillas Running | 89.50 | 2 |
| Laptop Pro | 1220.00 | 1 |
| Monitor 4K | 350.00 | 1 |
| Teclado Mecánico | 75.00 | 1 |
| Sofá de Cuero | 405.00 | 0 |

> 💡 El Sofá de Cuero no tiene pedidos (además tiene stock = 0). Los 3 clientes sí tienen pedidos todos.

</details>

---

## Ejercicio 5 — E-commerce · RIGHT JOIN

**Enunciado:** Usando `RIGHT JOIN`, muestra **todos los productos** (tabla derecha) con la información de sus pedidos (tabla izquierda). Muestra los productos que nadie ha comprado.

Muestra: `id_pedido`, `cantidad`, `nombre` (producto), `precio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pe.id_pedido,
       pe.cantidad,
       pr.nombre,
       pr.precio
FROM pedidos pe
RIGHT JOIN productos pr ON pe.id_producto = pr.id_producto
ORDER BY pr.nombre;
```

Resultado (8 filas, Sofá de Cuero con NULLs en pedido):
| id_pedido | cantidad | nombre | precio |
|-----------|----------|--------|--------|
| NULL | NULL | Sofá de Cuero | 405.00 |
| 4 | 5 | Camiseta Básica | 19.99 |
| 1 | 1 | Laptop Pro | 1220.00 |
| 2 | 2 | Lámpara LED | 40.50 |
| 5 | 1 | Monitor 4K | 350.00 |
| 3 | 3 | Ratón Inalámbrico | 45.50 |
| 6 | 1 | Teclado Mecánico | 75.00 |
| 7 | 2 | Zapatillas Running | 89.50 |

</details>

---

## Ejercicio 6 — Aerolínea · FULL OUTER JOIN

**Enunciado:** Genera un informe que muestre **todos los aviones y todos los vuelos**, incluso si no coinciden. Añade una columna `situacion` que indique:
- `'Avión sin vuelo'` si el avión no tiene vuelos.
- `'En servicio'` si hay coincidencia.

Muestra: `modelo`, `id_vuelo`, `puerta_embarque`, `situacion`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT a.modelo,
       v.id_vuelo,
       v.puerta_embarque,
       CASE
           WHEN v.id_vuelo IS NULL THEN 'Avión sin vuelo'
           ELSE 'En servicio'
       END AS situacion
FROM aviones a
FULL OUTER JOIN vuelos v ON a.id_avion = v.id_avion
ORDER BY a.modelo, v.id_vuelo;
```

Resultado (7 filas):
| modelo | id_vuelo | puerta_embarque | situacion |
|--------|----------|-----------------|-----------|
| Airbus A350 | 1001 | T4-A1 | En servicio |
| Airbus A350 | 1004 | T4-B5 | En servicio |
| Boeing 737 | NULL | NULL | Avión sin vuelo |
| Boeing 777 | 1002 | B-22 | En servicio |
| Boeing 777 | 1005 | T4-A1 | En servicio |
| Embraer E195 | 1003 | T1-C3 | En servicio |
| Embraer E195 | 1006 | T2-D1 | En servicio |

</details>

---

## Ejercicio 7 — E-commerce · Multi-table JOIN (4 tablas)

**Enunciado:** Genera el **informe completo de pedidos** cruzando 4 tablas: `pedidos`, `clientes`, `productos` y `categorias`. Muestra todo lo necesario para una factura.

Muestra: `id_pedido`, `fecha_pedido`, `cliente`, `producto`, `categoria`, `cantidad`, `total`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pe.id_pedido,
       pe.fecha_pedido,
       cl.nombre AS cliente,
       pr.nombre AS producto,
       ca.nombre_categoria AS categoria,
       pe.cantidad,
       pe.total
FROM pedidos pe
INNER JOIN clientes cl ON pe.id_cliente = cl.id_cliente
INNER JOIN productos pr ON pe.id_producto = pr.id_producto
INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
ORDER BY pe.fecha_pedido, pe.id_pedido;
```

Resultado:
| id_pedido | fecha_pedido | cliente | producto | categoria | cantidad | total |
|-----------|-------------|---------|----------|-----------|----------|-------|
| 1 | 2025-01-05 | Ana López | Laptop Pro | Electrónica | 1 | 1220.00 |
| 2 | 2025-01-05 | Ana López | Lámpara LED | Hogar | 2 | 81.00 |
| 3 | 2025-01-12 | Pedro Ruiz | Ratón Inalámbrico | Electrónica | 3 | 136.50 |
| 4 | 2025-01-20 | María García | Camiseta Básica | Ropa | 5 | 99.95 |
| 5 | 2025-02-01 | Pedro Ruiz | Monitor 4K | Electrónica | 1 | 350.00 |
| 6 | 2025-02-10 | Ana López | Teclado Mecánico | Electrónica | 1 | 75.00 |
| 7 | 2025-02-15 | María García | Zapatillas Running | Ropa | 2 | 179.00 |

</details>

---

## Ejercicio 8 — Hospital · Multi-table JOIN (4 tablas)

**Enunciado:** Genera el **historial completo de citas** cruzando `citas`, `pacientes`, `medicos` y `especialidades`. Traduce el estado de la cita: `'C'` → Completada, `'P'` → Pendiente, `'X'` → Cancelada.

Muestra: `id_cita`, `paciente`, `medico`, `especialidad`, `fecha_hora`, `estado_texto`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT ci.id_cita,
       pa.nombre_completo AS paciente,
       m.nombre_completo AS medico,
       e.nombre AS especialidad,
       ci.fecha_hora,
       DECODE(ci.estado, 'C', 'Completada', 'P', 'Pendiente', 'X', 'Cancelada') AS estado_texto
FROM citas ci
INNER JOIN pacientes pa ON ci.id_paciente = pa.id_paciente
INNER JOIN medicos m ON ci.id_medico = m.id_medico
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
ORDER BY ci.fecha_hora;
```

Resultado:
| id_cita | paciente | medico | especialidad | fecha_hora | estado_texto |
|---------|----------|--------|-------------|------------|-------------|
| 1 | Laura Martinez | Dra. Sarah Adams | Neurología | 2025-01-10 09:00 | Completada |
| 2 | Carlos Gomez | Dra. Elena Ruiz | Traumatología | 2025-01-10 10:30 | Completada |
| 3 | Pedro Sánchez | Dra. Marta López | Neurología | 2025-01-15 11:00 | Pendiente |
| 4 | Laura Martinez | Dr. Carlos Vega | Traumatología | 2025-01-20 09:30 | Completada |
| 5 | Ana Ruiz | Dra. Sarah Adams | Neurología | 2025-02-01 14:00 | Cancelada |
| 6 | David Torres | Dr. Luis Moreno | Medicina General | 2025-02-05 08:00 | Pendiente |
| 7 | Carlos Gomez | Dra. Marta López | Neurología | 2025-02-10 16:00 | Completada |

</details>

---

## Ejercicio 9 — Aerolínea · Multi-table JOIN (3 tablas)

**Enunciado:** Genera el **panel de vuelos** completo cruzando `vuelos`, `rutas` y `aviones`. Muestra la ruta formateada como `"MAD → LHR"`.

Muestra: `id_vuelo`, `ruta`, `distancia_km`, `avion`, `capacidad`, `fecha_hora_salida`, `puerta_embarque`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
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

Resultado:
| id_vuelo | ruta | distancia_km | avion | capacidad | fecha_hora_salida | puerta_embarque |
|----------|------|-------------|-------|-----------|-------------------|----------------|
| 1001 | MAD → LHR | 1350 | Airbus A350 | 300 | 2025-03-01 07:30 | T4-A1 |
| 1002 | JFK → LAX | 4000 | Boeing 777 | 350 | 2025-03-01 14:00 | B-22 |
| 1003 | BCN → CDG | 830 | Embraer E195 | 120 | 2025-03-02 06:00 | T1-C3 |
| 1004 | MAD → FRA | 1420 | Airbus A350 | 300 | 2025-03-02 11:15 | T4-B5 |
| 1005 | MAD → LHR | 1350 | Boeing 777 | 350 | 2025-03-03 07:30 | T4-A1 |
| 1006 | BCN → FCO | 850 | Embraer E195 | 120 | 2025-03-04 09:00 | T2-D1 |

</details>

---

## Ejercicio 10 — Hospital · SELF JOIN

**Enunciado:** Encuentra **pares de médicos que comparten la misma especialidad**. Evita duplicados (muestra cada par una sola vez) y excluye el par de un médico consigo mismo.

Pista: Usa `m1.id_medico < m2.id_medico` para evitar duplicados.

Muestra: `medico_1`, `medico_2`, `especialidad`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT m1.nombre_completo AS medico_1,
       m2.nombre_completo AS medico_2,
       e.nombre AS especialidad
FROM medicos m1
INNER JOIN medicos m2
    ON m1.id_especialidad = m2.id_especialidad
   AND m1.id_medico < m2.id_medico
INNER JOIN especialidades e ON m1.id_especialidad = e.id_especialidad
ORDER BY e.nombre;
```

Resultado:
| medico_1 | medico_2 | especialidad |
|----------|----------|--------------|
| Dra. Sarah Adams | Dra. Marta López | Neurología |
| Dra. Elena Ruiz | Dr. Carlos Vega | Traumatología |

> 💡 Dr. Luis Moreno no aparece porque es el único médico en Medicina General: no hay par posible.

</details>

---

## Ejercicio 11 — E-commerce · JOIN + GROUP BY

**Enunciado:** Genera un **resumen de gasto por cliente** cruzando `pedidos` con `clientes`. Muestra cuántos pedidos ha hecho cada cliente y su gasto total.

Muestra: `cliente`, `num_pedidos`, `gasto_total`. Ordena por gasto descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT cl.nombre AS cliente,
       COUNT(pe.id_pedido) AS num_pedidos,
       SUM(pe.total) AS gasto_total
FROM clientes cl
INNER JOIN pedidos pe ON cl.id_cliente = pe.id_cliente
GROUP BY cl.nombre
ORDER BY gasto_total DESC;
```

Resultado:
| cliente | num_pedidos | gasto_total |
|---------|-------------|-------------|
| Ana López | 3 | 1376.00 |
| Pedro Ruiz | 2 | 486.50 |
| María García | 2 | 278.95 |

</details>

---

## Ejercicio 12 — E-commerce · JOIN + GROUP BY + HAVING

**Enunciado:** Muestra las **categorías con más de 1 pedido**, indicando el número de pedidos y el ingreso total. Necesitas cruzar `pedidos` → `productos` → `categorias`.

Muestra: `categoria`, `num_pedidos`, `ingreso_total`. Solo categorías con más de 1 pedido.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT ca.nombre_categoria AS categoria,
       COUNT(pe.id_pedido) AS num_pedidos,
       SUM(pe.total) AS ingreso_total
FROM pedidos pe
INNER JOIN productos pr ON pe.id_producto = pr.id_producto
INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
GROUP BY ca.nombre_categoria
HAVING COUNT(pe.id_pedido) > 1
ORDER BY ingreso_total DESC;
```

Resultado:
| categoria | num_pedidos | ingreso_total |
|-----------|-------------|---------------|
| Electrónica | 4 | 1781.50 |
| Ropa | 2 | 278.95 |

> 💡 Hogar queda excluida porque solo tiene 1 pedido (Lámpara LED × 2).

</details>

---

<div align="center">

⬅️ [**Volver al Tema 08**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto JOINs →**](../proyectos/proyecto_joins.md)

</div>
