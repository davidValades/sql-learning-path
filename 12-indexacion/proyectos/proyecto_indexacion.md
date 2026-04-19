# 🏆 Proyecto — Tema 12: Plan de Indexación Productiva

---

## 🎯 Objetivo

Diseñar y validar una estrategia de índices para mejorar consultas críticas sin degradar excesivamente escrituras.

---

## Entregable 1 — Diagnóstico

Selecciona 5 consultas frecuentes del dominio (e-commerce, hospital o aerolínea) y para cada una:
- muestra su plan (`EXPLAIN PLAN`),
- identifica si hay `TABLE ACCESS FULL`,
- estima impacto esperado.

---

## Entregable 2 — Implementación de índices

Crea al menos:
- 3 índices simples,
- 1 índice compuesto,
- 1 índice function-based (si aplica).

Usa convención `idx_tabla_columna`.

---

## Entregable 3 — Comparativa antes/después

Para cada consulta:
1. Plan antes.
2. Plan después.
3. Recomendación final (mantener, hacer invisible o eliminar índice).

---

<div align="center">

⬅️ [**📝 Ejercicios del Tema 12**](../ejercicios/ejercicios_indexacion.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 13: Transacciones y Propiedades ACID →**](../../13-transacciones-acid/README.md)

</div>
