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

</details>

---

<div align="center">

⬅️ [**Tema 17**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 17 →**](../proyectos/proyecto_optimizacion.md)

</div>
