# 🏋️ Ejercicios — Tema 17: Optimización

> 💡 Mide antes y después de cada cambio. No optimices “a ciegas”.

---

## Ejercicio 1 — Detectar Full Table Scan

Saca el plan de una consulta por rango de fechas en `pedidos`.

<details>
<summary>👉 Ver solución</summary>

```sql
EXPLAIN PLAN FOR
SELECT *
FROM pedidos
WHERE fecha_pedido >= ADD_MONTHS(TRUNC(SYSDATE), -1);

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

</details>

---

## Ejercicio 2 — Crear índice compuesto útil

Crea un índice para consultas por cliente + fecha.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE INDEX idx_pedidos_cliente_fecha
ON pedidos(id_cliente, fecha_pedido);
```

</details>

---

## Ejercicio 3 — Reescribir filtro no sargable

Reescribe `TRUNC(fecha_pedido)=TRUNC(SYSDATE)` para permitir uso de índice.

<details>
<summary>👉 Ver solución</summary>

```sql
WHERE fecha_pedido >= TRUNC(SYSDATE)
  AND fecha_pedido < TRUNC(SYSDATE) + 1
```

</details>

---

## Ejercicio 4 — Indexar auditoría

Optimiza búsquedas frecuentes por producto en `auditoria_stock`.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE INDEX idx_aud_stock_producto_fecha
ON auditoria_stock(id_producto, fecha_cambio);
```

</details>

---

## Ejercicio 5 — Actualizar estadísticas

Ejecuta `DBMS_STATS` para tablas críticas de reporting.

<details>
<summary>👉 Ver solución</summary>

```sql
BEGIN
  DBMS_STATS.GATHER_TABLE_STATS(USER, 'PEDIDOS');
  DBMS_STATS.GATHER_TABLE_STATS(USER, 'PRODUCTOS');
  DBMS_STATS.GATHER_TABLE_STATS(USER, 'AUDITORIA_STOCK');
END;
/
```

</details>

---

<div align="center">

⬅️ [**Volver al Tema 17**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto Optimización →**](../proyectos/proyecto_optimizacion.md)

</div>
