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

Resultado consulta 1:
| dia | pedidos | facturacion |
|-----|---------|-------------|
| 2025-02-15 | 1 | 179.00 |
| 2025-02-10 | 1 | 75.00 |
| 2025-02-01 | 1 | 350.00 |
| 2025-01-20 | 1 | 99.95 |
| 2025-01-12 | 1 | 136.50 |
| 2025-01-05 | 2 | 1301.00 |

Resultado consulta 2 (depende de la fecha de ejecución, con datos históricos fijos):
> 💡 Si se ejecuta mucho después de febrero 2025, no habrá pedidos en los últimos 2 días → no rows selected. Esto ya sería un hallazgo diagnóstico válido.

Resultado consulta 3:
| pedidos_sin_total |
|-------------------|
| 0 |

> 💡 No hay pedidos con total NULL o negativo. Esto descarta problemas de calidad de datos como causa de la discrepancia.

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

Resultado: no rows selected.

> 💡 No hay emails duplicados en la tabla `clientes` porque la columna tiene restricción `UNIQUE`. La consulta es igualmente útil como validación rutinaria: confirma que la constraint está funcionando correctamente.

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

Resultado:
| pct_completadas |
|-----------------|
| 57.14 |

> 💡 De las 7 citas: 4 están Completadas ('C'), 2 Pendientes ('P') y 1 Cancelada ('X'). El SLA de completadas es 4/7 × 100 = 57.14%.

</details>

---

<div align="center">

⬅️ [**Tema 19: Casos Reales y Proyectos Finales**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 19 →**](../proyectos/proyecto_casos_reales.md)

</div>
