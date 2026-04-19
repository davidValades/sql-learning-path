# 🏋️ Ejercicios — Tema 15: SQL Avanzado

> 💡 **Instrucciones:** Todos los ejercicios usan el estado acumulado de la base de datos del curso. Si aquí aplicas `ALTER TABLE` o creas objetos analíticos, se asumen vigentes para los temas siguientes.

---

## Ejercicio 1 — CTE de rendimiento comercial

**Enunciado:** Construye una consulta con CTE que muestre por cliente:
- total de pedidos,
- gasto total,
- ticket medio.

Ordena por gasto total descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
WITH resumen_cliente AS (
  SELECT id_cliente,
         COUNT(*) AS total_pedidos,
         SUM(total) AS gasto_total,
         ROUND(AVG(total), 2) AS ticket_medio
  FROM pedidos
  GROUP BY id_cliente
)
SELECT c.nombre,
       rc.total_pedidos,
       rc.gasto_total,
       rc.ticket_medio
FROM resumen_cliente rc
JOIN clientes c ON c.id_cliente = rc.id_cliente
ORDER BY rc.gasto_total DESC;
```

Resultado:
| nombre | total_pedidos | gasto_total | ticket_medio |
|--------|---------------|-------------|--------------|
| Ana López | 3 | 1376.00 | 458.67 |
| Pedro Ruiz | 2 | 486.50 | 243.25 |
| María García | 2 | 278.95 | 139.48 |

</details>

---

## Ejercicio 2 — Evolución de modelo con ventana + ALTER

**Enunciado:** Segmenta clientes en 3 niveles de valor usando gasto histórico y guarda el resultado en la tabla `clientes`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
ALTER TABLE clientes ADD segmento_cliente VARCHAR2(20);

MERGE INTO clientes c
USING (
  SELECT id_cliente,
         CASE NTILE(3) OVER (ORDER BY SUM(total) DESC)
           WHEN 1 THEN 'ALTO_VALOR'
           WHEN 2 THEN 'MEDIO_VALOR'
           ELSE 'BASE'
         END AS segmento
  FROM pedidos
  GROUP BY id_cliente
) s
ON (c.id_cliente = s.id_cliente)
WHEN MATCHED THEN UPDATE SET c.segmento_cliente = s.segmento;
```

Resultado del ALTER: `Table altered.`

Resultado del MERGE: `3 rows merged.`

Verificación:
```sql
SELECT nombre, segmento_cliente FROM clientes ORDER BY nombre;
```
| nombre | segmento_cliente |
|--------|------------------|
| Ana López | ALTO_VALOR |
| María García | BASE |
| Pedro Ruiz | MEDIO_VALOR |

> 💡 Ana López (1376.00€) queda en ALTO_VALOR, Pedro Ruiz (486.50€) en MEDIO_VALOR y María García (278.95€) en BASE.

✅ **Esta columna queda incorporada al estado acumulado.**

</details>

---

## Ejercicio 3 — Ranking mensual por categoría

**Enunciado:** Crea una vista con el top-3 de productos por facturación en cada mes y categoría.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW vw_top_productos_mes AS
WITH ventas AS (
  SELECT TRUNC(p.fecha_pedido, 'MM') AS mes,
         pr.id_categoria,
         pr.id_producto,
         pr.nombre AS producto,
         SUM(p.total) AS facturacion
  FROM pedidos p
  JOIN productos pr ON pr.id_producto = p.id_producto
  GROUP BY TRUNC(p.fecha_pedido, 'MM'), pr.id_categoria, pr.id_producto, pr.nombre
), ranking AS (
  SELECT v.*,
         ROW_NUMBER() OVER (
           PARTITION BY v.mes, v.id_categoria
           ORDER BY v.facturacion DESC
         ) AS rn
  FROM ventas v
)
SELECT *
FROM ranking
WHERE rn <= 3;
```

Resultado: `View created.`

Consulta de verificación:
```sql
SELECT mes, producto, id_categoria, facturacion, rn FROM vw_top_productos_mes ORDER BY mes, id_categoria, rn;
```
| mes | producto | id_categoria | facturacion | rn |
|-----|----------|-------------|-------------|-----|
| 2025-01-01 | Laptop Pro | 1 | 1220.00 | 1 |
| 2025-01-01 | Ratón Inalámbrico | 1 | 136.50 | 2 |
| 2025-01-01 | Lámpara LED | 2 | 81.00 | 1 |
| 2025-01-01 | Camiseta Básica | 3 | 99.95 | 1 |
| 2025-02-01 | Monitor 4K | 1 | 350.00 | 1 |
| 2025-02-01 | Teclado Mecánico | 1 | 75.00 | 2 |
| 2025-02-01 | Zapatillas Running | 3 | 179.00 | 1 |

</details>

---

## Ejercicio 4 — Detección de caída de ventas

**Enunciado:** Detecta productos cuya venta mensual cayó más de 30% frente al mes anterior (usa `LAG`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
WITH ventas_mes AS (
  SELECT id_producto,
         TRUNC(fecha_pedido, 'MM') AS mes,
         SUM(total) AS ventas
  FROM pedidos
  GROUP BY id_producto, TRUNC(fecha_pedido, 'MM')
), comparativa AS (
  SELECT vm.*,
         LAG(vm.ventas) OVER (PARTITION BY vm.id_producto ORDER BY vm.mes) AS ventas_mes_anterior
  FROM ventas_mes vm
)
SELECT *
FROM comparativa
WHERE ventas_mes_anterior IS NOT NULL
  AND ventas < ventas_mes_anterior * 0.70;
```

Resultado: no rows selected.

> 💡 Con los datos actuales del curso (solo 2 meses de pedidos: enero y febrero 2025), ningún producto muestra una caída superior al 30%. Si un producto vendiera 1000€ en enero y 600€ en febrero (caída del 40%), sí aparecería.

</details>

---

<div align="center">

⬅️ [**Tema 15: SQL Avanzado**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 15 →**](../proyectos/proyecto_sql_avanzado.md)

</div>
