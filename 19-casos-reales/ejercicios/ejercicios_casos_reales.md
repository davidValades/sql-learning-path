# 🏋️ Ejercicios — Tema 19: Casos Reales y Proyectos Finales

---

## Ejercicio 1 — Diagnóstico de incidente

**Enunciado:** Un reporte financiero muestra menos ventas que ayer. Enumera 3 consultas SQL rápidas para aislar el problema.

<details>
<summary>👉 Ver solución</summary>

```sql
-- 1) Volumen por día
SELECT TRUNC(fecha_pedido) dia, COUNT(*) pedidos, SUM(total) facturacion
FROM pedidos
GROUP BY TRUNC(fecha_pedido)
ORDER BY dia DESC;

-- 2) Caída por producto
SELECT id_producto, SUM(total) facturacion
FROM pedidos
WHERE fecha_pedido >= SYSDATE - 2
GROUP BY id_producto
ORDER BY facturacion DESC;

-- 3) Validar datos nulos o anómalos
SELECT COUNT(*) AS pedidos_sin_total
FROM pedidos
WHERE total IS NULL OR total < 0;
```

</details>

---

## Ejercicio 2 — Data quality mínima

**Enunciado:** Escribe una consulta que detecte clientes duplicados por email.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT email, COUNT(*) repeticiones
FROM clientes
GROUP BY email
HAVING COUNT(*) > 1;
```

</details>

---

## Ejercicio 3 — Seguimiento operativo

**Enunciado:** Crea una consulta de SLA que calcule porcentaje de citas completadas en hospital.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT ROUND(
  SUM(CASE WHEN estado = 'C' THEN 1 ELSE 0 END) * 100 / COUNT(*),
  2
) AS pct_completadas
FROM citas;
```

</details>

---

<div align="center">

⬅️ [**Tema 19**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto Final del Tema 19 →**](../proyectos/proyecto_casos_reales.md)

</div>
