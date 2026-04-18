# 🏆 Proyecto — Reporte Analítico Avanzado

## Contexto

Dirección pide un reporte mensual con rankings, variaciones y alertas de negocio sin mover datos fuera de la base.

## Objetivo

Construir un reporte end-to-end solo con SQL avanzado (CTEs + ventanas + conjuntos).

---

## Entregables

1. CTE base de métricas de ventas por cliente y categoría.
2. Ranking por categoría (`ROW_NUMBER`/`RANK`).
3. Variación vs periodo anterior (`LAG`/`LEAD`).
4. Dataset final consolidado en una vista `v_reporte_analitico`.

---

## Criterios de aceptación

- Consulta legible y modular con CTEs.
- Uso correcto de ventanas sin colapsar filas innecesariamente.
- Resultado directamente utilizable por BI/reporting.

---

<div align="center">

⬅️ [**Ejercicios SQL Avanzado**](../ejercicios/ejercicios_sql_avanzado.md) · 🏠 [**Volver al Tema 15**](../README.md) · [**Tema 16 →**](../../16-plsql)

</div>
