# 🏋️ Ejercicios — Tema 17: Optimización de Consultas

---

## Ejercicio 1 — Leer plan de ejecución

**Enunciado:** Obtén el plan de ejecución de una consulta por `id_cliente` en `pedidos`.

<details>
<summary>👉 Ver solución</summary>

```sql
EXPLAIN PLAN FOR
SELECT *
FROM pedidos
WHERE id_cliente = 1;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

Salida esperada del plan:
```
Plan hash value: XXXXXXXXX

---------------------------------------------------------------------------
| Id  | Operation         | Name    | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |         |     3 |   xxx |     3   (0)| 00:00:01 |
|*  1 |  TABLE ACCESS FULL| PEDIDOS |     3 |   xxx |     3   (0)| 00:00:01 |
---------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   1 - filter("ID_CLIENTE"=1)
```

> 💡 Sin índice en `id_cliente`, Oracle realiza un **TABLE ACCESS FULL** (recorre toda la tabla). Con un índice en esa columna, cambiaría a **INDEX RANGE SCAN** + **TABLE ACCESS BY INDEX ROWID**.

</details>

---

## Ejercicio 2 — Probar índice útil

**Enunciado:** Crea un índice para acelerar consultas por `fecha_pedido` y vuelve a comparar el plan.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE INDEX idx_pedidos_fecha_opt ON pedidos(fecha_pedido);

EXPLAIN PLAN FOR
SELECT *
FROM pedidos
WHERE fecha_pedido >= DATE '2024-01-01'
  AND fecha_pedido < DATE '2024-02-01';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

Salida esperada del plan (con índice):
```
Plan hash value: XXXXXXXXX

-----------------------------------------------------------------------------------------------
| Id  | Operation                           | Name                  | Rows  | Cost (%CPU)| Time     |
-----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                    |                       |     x |     2   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID BATCHED| PEDIDOS               |     x |     2   (0)| 00:00:01 |
|*  2 |   INDEX RANGE SCAN                  | IDX_PEDIDOS_FECHA_OPT |     x |     1   (0)| 00:00:01 |
-----------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   2 - access("FECHA_PEDIDO">=... AND "FECHA_PEDIDO"<...)
```

> 💡 Ahora Oracle usa **INDEX RANGE SCAN** en lugar de FULL TABLE SCAN. El coste se reduce porque solo lee las filas dentro del rango de fechas.

</details>

---

## Ejercicio 3 — Detectar antipatrón

**Enunciado:** Reescribe esta condición para no romper el uso de índice:
`WHERE TRUNC(fecha_pedido) = DATE '2024-01-20'`

<details>
<summary>👉 Ver solución</summary>

```sql
WHERE fecha_pedido >= DATE '2024-01-20'
  AND fecha_pedido < DATE '2024-01-21'
```

Salida esperada: La consulta devuelve los mismos resultados que `WHERE TRUNC(fecha_pedido) = DATE '2024-01-20'`, pero ahora Oracle **SÍ puede usar un índice** en `fecha_pedido`.

Comparación:
| Versión | Usa índice | Motivo |
|---------|------------|--------|
| `TRUNC(fecha_pedido) = DATE '...'` | ❌ No | La función `TRUNC()` sobre la columna impide el uso del índice B-Tree |
| `fecha_pedido >= ... AND < ...` | ✅ Sí | Comparación directa sobre la columna permite INDEX RANGE SCAN |

> 💡 **Regla de oro:** nunca apliques funciones sobre la columna en el `WHERE` si quieres aprovechar un índice. Transforma la constante, no la columna.

</details>

---

<div align="center">

⬅️ [**Tema 17: Optimización de Consultas**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 17 →**](../proyectos/proyecto_optimizacion.md)

</div>
