# 🏆 Proyecto — Tema 18: Migración SQL Multi-Motor

---

## 🎯 Objetivo

Diseñar una guía práctica para portar un módulo Oracle a PostgreSQL manteniendo funcionalidad y rendimiento base.

---

## Entregable 1 — Inventario de compatibilidad

Cataloga objetos usados en Oracle:
- tipos de datos,
- funciones,
- secuencias,
- sintaxis específica.

Clasifica cada ítem en: **compatible**, **requiere ajuste**, **requiere rediseño**.

---

## Entregable 2 — Script de adaptación

Prepara versión PostgreSQL para:
1. esquema principal,
2. 10 consultas clave,
3. estrategias equivalentes para funciones Oracle (`NVL`, `SYSDATE`, `MERGE`, etc.).

---

## Entregable 3 — Checklist operativo

Incluye validaciones:
- resultados equivalentes,
- tiempos aceptables,
- rollback plan,
- riesgos por diferencias de transacción y locking.

---

<div align="center">

⬅️ [**📝 Ejercicios del Tema 18**](../ejercicios/ejercicios_ecosistema_sql.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 19: Casos Reales y Proyectos Finales →**](../../19-casos-reales/README.md)

</div>
