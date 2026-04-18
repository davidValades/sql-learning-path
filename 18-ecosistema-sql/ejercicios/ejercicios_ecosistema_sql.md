# 🏋️ Ejercicios — Tema 18: Ecosistema SQL

---

## Ejercicio 1 — Traducir sintaxis entre motores

**Enunciado:** Convierte esta paginación Oracle a PostgreSQL/MySQL:
`FETCH FIRST 10 ROWS ONLY`.

<details>
<summary>👉 Ver solución</summary>

```sql
-- PostgreSQL / MySQL
LIMIT 10
```

</details>

---

## Ejercicio 2 — Tipos de datos equivalentes

**Enunciado:** Propón equivalencias de:
- Oracle `NUMBER(10,2)`
- Oracle `VARCHAR2(200)`
- Oracle `DATE`

<details>
<summary>👉 Ver solución</summary>

- `NUMBER(10,2)` → `NUMERIC(10,2)` (PostgreSQL / SQL Server)
- `VARCHAR2(200)` → `VARCHAR(200)`
- `DATE` (Oracle incluye hora) → `TIMESTAMP` en PostgreSQL si quieres hora

</details>

---

## Ejercicio 3 — Compatibilidad de funciones

**Enunciado:** Da alternativas multiplataforma para `NVL(campo, 0)`.

<details>
<summary>👉 Ver solución</summary>

```sql
COALESCE(campo, 0)
```

</details>

---

<div align="center">

⬅️ [**Tema 18**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 18 →**](../proyectos/proyecto_ecosistema_sql.md)

</div>
