# 🏋️ Ejercicios — Tema 09: Subconsultas

> 💡 **Instrucciones:** Resuelve cada ejercicio escribiendo la consulta SQL completa. Todos los ejercicios usan el estado acumulado de la base de datos, incluyendo la tabla `pedidos`, `citas` y `vuelos` del Tema 08.

---

## Ejercicio 1 — E-commerce · Subconsulta Escalar en WHERE

**Enunciado:** Encuentra el **producto más barato** de toda la tienda. Usa una subconsulta escalar con `MIN()` en el `WHERE`.

Muestra: `nombre`, `precio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
WHERE precio = (SELECT MIN(precio) FROM productos);
```

Resultado:
| nombre | precio |
|--------|--------|
| Camiseta Básica | 19.99 |

</details>

---

## Ejercicio 2 — Hospital · Subconsulta Escalar en SELECT

**Enunciado:** Para cada médico, muestra su salario y la **diferencia respecto al salario medio** de todos los médicos. Usa una subconsulta escalar en el `SELECT`.

Muestra: `nombre_completo`, `salario_base`, `diferencia_vs_media`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre_completo,
       salario_base,
       salario_base - (SELECT ROUND(AVG(salario_base), 2) FROM medicos) AS diferencia_vs_media
FROM medicos
ORDER BY diferencia_vs_media DESC;
```

AVG = (4000 + 3200 + 2800 + 4500 + 2600) / 5 = 3420.00

Resultado:
| nombre_completo | salario_base | diferencia_vs_media |
|----------------|-------------|---------------------|
| Dra. Marta López | 4500 | 1080.00 |
| Dra. Sarah Adams | 4000 | 580.00 |
| Dra. Elena Ruiz | 3200 | -220.00 |
| Dr. Carlos Vega | 2800 | -620.00 |
| Dr. Luis Moreno | 2600 | -820.00 |

</details>

---

## Ejercicio 3 — E-commerce · IN con subconsulta

**Enunciado:** Lista los **clientes que han comprado productos de la categoría Electrónica** (id_categoria = 1). Usa `IN` con una subconsulta que primero obtenga los id_producto de Electrónica y luego los id_cliente de pedidos.

Muestra: `nombre`, `email`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, email
FROM clientes
WHERE id_cliente IN (
    SELECT pe.id_cliente
    FROM pedidos pe
    WHERE pe.id_producto IN (
        SELECT id_producto
        FROM productos
        WHERE id_categoria = 1
    )
);
```

Resultado:
| nombre | email |
|--------|-------|
| Ana López | ana@email.com |
| Pedro Ruiz | pedro@email.com |

> 💡 Ana compró Laptop Pro y Teclado Mecánico (Electrónica). Pedro compró Ratón Inalámbrico y Monitor 4K (Electrónica). María solo compró Ropa.

</details>

---

## Ejercicio 4 — E-commerce · NOT IN

**Enunciado:** Encuentra los **productos que nadie ha comprado**. Usa `NOT IN` con una subconsulta.

Muestra: `nombre`, `precio`, `stock`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio, stock
FROM productos
WHERE id_producto NOT IN (
    SELECT id_producto FROM pedidos
);
```

Resultado:
| nombre | precio | stock |
|--------|--------|-------|
| Sofá de Cuero | 405.00 | 0 |

> 💡 Solo el Sofá de Cuero no tiene pedidos (y además tiene stock = 0, así que no se puede comprar).

</details>

---

## Ejercicio 5 — E-commerce · Subconsulta Correlacionada

**Enunciado:** Para cada producto, muestra si su precio está **por encima o por debajo del promedio de su categoría**. Usa una subconsulta correlacionada.

Muestra: `nombre`, `precio`, `precio_medio_cat`, `posicion` (`'Por encima'` / `'Por debajo'` / `'En la media'`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT p.nombre,
       p.precio,
       (SELECT ROUND(AVG(p2.precio), 2)
        FROM productos p2
        WHERE p2.id_categoria = p.id_categoria) AS precio_medio_cat,
       CASE
           WHEN p.precio > (SELECT AVG(p2.precio) FROM productos p2 WHERE p2.id_categoria = p.id_categoria)
               THEN 'Por encima'
           WHEN p.precio < (SELECT AVG(p2.precio) FROM productos p2 WHERE p2.id_categoria = p.id_categoria)
               THEN 'Por debajo'
           ELSE 'En la media'
       END AS posicion
FROM productos p
ORDER BY p.id_categoria, p.precio DESC;
```

Resultado:
| nombre | precio | precio_medio_cat | posicion |
|--------|--------|-----------------|----------|
| Laptop Pro | 1220.00 | 422.63 | Por encima |
| Monitor 4K | 350.00 | 422.63 | Por debajo |
| Teclado Mecánico | 75.00 | 422.63 | Por debajo |
| Ratón Inalámbrico | 45.50 | 422.63 | Por debajo |
| Sofá de Cuero | 405.00 | 222.75 | Por encima |
| Lámpara LED | 40.50 | 222.75 | Por debajo |
| Zapatillas Running | 89.50 | 54.75 | Por encima |
| Camiseta Básica | 19.99 | 54.75 | Por debajo |

</details>

---

## Ejercicio 6 — Hospital · EXISTS

**Enunciado:** Lista los **médicos que tienen al menos una cita completada** (estado = `'C'`). Usa `EXISTS`.

Muestra: `nombre_completo`, `salario_base`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT m.nombre_completo, m.salario_base
FROM medicos m
WHERE EXISTS (
    SELECT 1
    FROM citas ci
    WHERE ci.id_medico = m.id_medico
      AND ci.estado = 'C'
);
```

Resultado:
| nombre_completo | salario_base |
|----------------|-------------|
| Dra. Sarah Adams | 4000 |
| Dra. Elena Ruiz | 3200 |
| Dr. Carlos Vega | 2800 |
| Dra. Marta López | 4500 |

> 💡 Dr. Luis Moreno no aparece porque su única cita (cita 6) está en estado 'P' (Pendiente).

</details>

---

## Ejercicio 7 — Aerolínea · NOT EXISTS

**Enunciado:** Encuentra las **rutas que NO tienen vuelos operados por el Boeing 777** (id_avion = 50). Usa `NOT EXISTS`.

Muestra: `id_ruta`, `origen`, `destino`, `distancia_km`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT r.id_ruta, r.origen, r.destino, r.distancia_km
FROM rutas r
WHERE NOT EXISTS (
    SELECT 1
    FROM vuelos v
    WHERE v.id_ruta = r.id_ruta
      AND v.id_avion = 50
);
```

El Boeing 777 (id=50) opera: JFKLAX (vuelo 1002) y MADLHR (vuelo 1005).

Resultado:
| id_ruta | origen | destino | distancia_km |
|---------|--------|---------|-------------|
| BCNCDG | BCN | CDG | 830 |
| BCNFCO | BCN | FCO | 850 |
| MADFRA | MAD | FRA | 1420 |

</details>

---

## Ejercicio 8 — Hospital · ANY / ALL

**Enunciado:** Encuentra los **médicos cuyo salario es mayor que TODOS los traumatólogos** (id_especialidad = 100). Usa `> ALL`.

Muestra: `nombre_completo`, `salario_base`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base > ALL (
    SELECT salario_base
    FROM medicos
    WHERE id_especialidad = 100
);
```

Traumatólogos: Dra. Elena Ruiz (3200), Dr. Carlos Vega (2800).
`> ALL (3200, 2800)` equivale a `> 3200`.

Resultado:
| nombre_completo | salario_base |
|----------------|-------------|
| Dra. Sarah Adams | 4000 |
| Dra. Marta López | 4500 |

</details>

---

## Ejercicio 9 — E-commerce · Inline View (subconsulta en FROM)

**Enunciado:** Usa una inline view para generar un **ranking de clientes por gasto total**. Muestra solo los clientes cuyo gasto supera los 400€.

Muestra: `cliente`, `num_pedidos`, `gasto_total`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT ranking.cliente,
       ranking.num_pedidos,
       ranking.gasto_total
FROM (
    SELECT cl.nombre AS cliente,
           COUNT(pe.id_pedido) AS num_pedidos,
           SUM(pe.total) AS gasto_total
    FROM pedidos pe
    INNER JOIN clientes cl ON pe.id_cliente = cl.id_cliente
    GROUP BY cl.nombre
) ranking
WHERE ranking.gasto_total > 400
ORDER BY ranking.gasto_total DESC;
```

Resultado:
| cliente | num_pedidos | gasto_total |
|---------|-------------|-------------|
| Ana López | 3 | 1376.00 |
| Pedro Ruiz | 2 | 486.50 |

> 💡 María García queda excluida con 278.95€ de gasto total.

</details>

---

## Ejercicio 10 — Aerolínea · Inline View + JOIN

**Enunciado:** Genera un informe que muestre cada avión con su **número de vuelos y distancia total**, pero solo para aviones que cubren una **distancia total mayor a 2000 km**. Usa una inline view para el cálculo y luego cruza con la tabla `aviones` para obtener detalles.

Muestra: `modelo`, `capacidad`, `anio_fabricacion`, `num_vuelos`, `distancia_total_km`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT a.modelo,
       a.capacidad,
       a.anio_fabricacion,
       stats.num_vuelos,
       stats.distancia_total_km
FROM aviones a
INNER JOIN (
    SELECT v.id_avion,
           COUNT(v.id_vuelo) AS num_vuelos,
           SUM(r.distancia_km) AS distancia_total_km
    FROM vuelos v
    INNER JOIN rutas r ON v.id_ruta = r.id_ruta
    GROUP BY v.id_avion
) stats ON a.id_avion = stats.id_avion
WHERE stats.distancia_total_km > 2000
ORDER BY stats.distancia_total_km DESC;
```

Resultado:
| modelo | capacidad | anio_fabricacion | num_vuelos | distancia_total_km |
|--------|-----------|-----------------|-----------|-------------------|
| Boeing 777 | 350 | 2016 | 2 | 5350 |
| Airbus A350 | 300 | 2022 | 2 | 2770 |

> 💡 El Embraer E195 tiene 1680 km (830+850) y queda excluido. El Boeing 737 no tiene vuelos.

</details>

---

<div align="center">

⬅️ [**Volver al Tema 09**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto Subconsultas →**](../proyectos/proyecto_subconsultas.md)

</div>
