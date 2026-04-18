# 🏆 Proyecto — Plan de Migración Oracle → PostgreSQL

## Contexto

La empresa mantendrá Oracle para core transaccional, pero moverá analítica operativa a PostgreSQL.

## Objetivo

Diseñar un plan realista de migración con riesgo controlado y continuidad operativa.

---

## Entregables

1. Matriz de equivalencias de tipos y funciones.
2. Scripts DDL de destino.
3. Estrategia de carga inicial + delta.
4. Validaciones funcionales y de rendimiento.
5. Plan de rollback y ventana de cutover.

---

## Criterio de aceptación

- Se mantienen reglas de negocio.
- Se mantiene seguridad por roles.
- Consultas críticas funcionan en ambos motores con resultados equivalentes.

---

<div align="center">

⬅️ [**Ejercicios Ecosistema SQL**](../ejercicios/ejercicios_ecosistema_sql.md) · 🏠 [**Volver al Tema 18**](../README.md) · [**Tema 19 →**](../../19-casos-reales)

</div>
