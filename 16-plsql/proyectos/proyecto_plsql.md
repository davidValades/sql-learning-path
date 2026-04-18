# 🏆 Proyecto — Tema 16: Automatización Operativa con PL/SQL

---

## 🎯 Objetivo

Construir automatizaciones PL/SQL reutilizando tablas analíticas del Tema 15 (`kpi_ventas_mensuales`, `alertas_negocio`).

---

## Entregable 1 — Procedimiento de actualización de KPIs

Crea procedimiento `sp_refrescar_kpis_mensuales` que:
1. recalcula KPIs por mes,
2. hace `MERGE`,
3. registra fecha de actualización.

---

## Entregable 2 — Procedimiento de alertas

Crea procedimiento `sp_generar_alertas_negocio` que inserte alertas para:
- productos sin ventas 90 días,
- clientes de alto valor con caída de gasto.

---

## Entregable 3 — Manejo de errores y logging

Añade bloque `EXCEPTION` en ambos procedimientos y registra fallos en tabla de log (`log_procesos_plsql`).

---

<div align="center">

⬅️ [**🏋️ Ejercicios del Tema 16**](../ejercicios/ejercicios_plsql.md) · 🏠 [**Tema 16**](../README.md) · [**Tema 17: Optimización de Consultas →**](../../17-optimizacion)

</div>
