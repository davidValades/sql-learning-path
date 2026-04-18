# 🏋️ Ejercicios — Tema 15: SQL Avanzado

> 💡 Aplica CTEs, funciones de ventana y operadores de conjuntos sobre el estado acumulado de la base de datos.

---

## Ejercicio 1 — CTE de ventas por cliente

Calcula total de pedidos y total gastado por cliente usando CTE.

<details>
<summary>👉 Ver solución</summary>

```sql
WITH ventas_cliente AS (
  SELECT id_cliente, COUNT(*) AS total_pedidos, SUM(total) AS total_gastado
  FROM pedidos
  GROUP BY id_cliente
)
SELECT *
FROM ventas_cliente
ORDER BY total_gastado DESC;
```

</details>

---

## Ejercicio 2 — ROW_NUMBER por categoría

Saca el top 2 de productos más caros por categoría.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT *
FROM (
  SELECT p.id_categoria, p.nombre, p.precio,
         ROW_NUMBER() OVER (PARTITION BY p.id_categoria ORDER BY p.precio DESC) AS rn
  FROM productos p
) t
WHERE rn <= 2
ORDER BY id_categoria, rn;
```

</details>

---

## Ejercicio 3 — LAG para detectar cambios

Muestra por cliente cada pedido y diferencia respecto al pedido anterior (`LAG`).

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT id_cliente, id_pedido, fecha_pedido, total,
       total - LAG(total) OVER (PARTITION BY id_cliente ORDER BY fecha_pedido) AS dif_vs_anterior
FROM pedidos
ORDER BY id_cliente, fecha_pedido;
```

</details>

---

## Ejercicio 4 — SUM OVER acumulado

Calcula acumulado de ventas por cliente en orden temporal.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT id_cliente, id_pedido, fecha_pedido, total,
       SUM(total) OVER (PARTITION BY id_cliente ORDER BY fecha_pedido
                        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS acumulado_cliente
FROM pedidos
ORDER BY id_cliente, fecha_pedido;
```

</details>

---

## Ejercicio 5 — UNION/INTERSECT

- Lista IDs de clientes con pedidos (`pedidos`)
- Intersecta con clientes con email corporativo `@empresa.com` (tabla `clientes`).

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT id_cliente FROM pedidos
INTERSECT
SELECT id_cliente FROM clientes WHERE email LIKE '%@empresa.com';
```

</details>

---

<div align="center">

⬅️ [**Volver al Tema 15**](../README.md) · �� [**Índice del Curso**](../../README.md) · [**Proyecto SQL Avanzado →**](../proyectos/proyecto_sql_avanzado.md)

</div>
