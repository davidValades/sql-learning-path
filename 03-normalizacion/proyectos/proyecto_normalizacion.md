# 🏆 Proyecto — Rescatando una BD Legacy

## Contexto

Recibes una base heredada con una tabla única `operaciones_legacy` con datos mezclados de cliente, pedido y producto.

## Objetivo

Rediseñar el modelo en 3FN y preparar migración limpia sin pérdida de información.

---

## Entregables

1. Diagrama ER objetivo (texto o imagen).
2. Script DDL final (tablas, PK, FK, UNIQUE, CHECK).
3. Script de transformación desde tabla legacy al modelo nuevo.
4. Consultas de validación (conteos y calidad de datos).

---

## Criterios de aceptación

- No hay columnas multivalor.
- No hay dependencias parciales ni transitivas.
- Integridad referencial completa entre tablas.

---

<div align="center">

⬅️ [**Ejercicios de Normalización**](../ejercicios/ejercicios_normalizacion.md) · 🏠 [**Volver al Tema 03**](../README.md) · [**Tema 04 →**](../../04-ddl)

</div>
