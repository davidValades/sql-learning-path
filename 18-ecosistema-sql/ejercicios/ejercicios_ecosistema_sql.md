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

Comparación:
| Motor | Sintaxis de paginación |
|-------|----------------------|
| Oracle 12c+ | `FETCH FIRST 10 ROWS ONLY` |
| PostgreSQL | `LIMIT 10` |
| MySQL | `LIMIT 10` |
| SQL Server | `TOP 10` (en el SELECT) |

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

Tabla resumen de equivalencias:
| Oracle | PostgreSQL | MySQL | SQL Server |
|--------|-----------|-------|------------|
| `NUMBER(10,2)` | `NUMERIC(10,2)` | `DECIMAL(10,2)` | `DECIMAL(10,2)` |
| `VARCHAR2(200)` | `VARCHAR(200)` | `VARCHAR(200)` | `NVARCHAR(200)` |
| `DATE` | `TIMESTAMP` | `DATETIME` | `DATETIME2` |

</details>

---

## Ejercicio 3 — Compatibilidad de funciones

**Enunciado:** Da alternativas multiplataforma para `NVL(campo, 0)`.

<details>
<summary>👉 Ver solución</summary>

```sql
COALESCE(campo, 0)
```

Comparación multiplataforma:
| Motor | Función nativa | Alternativa estándar |
|-------|---------------|---------------------|
| Oracle | `NVL(campo, 0)` | `COALESCE(campo, 0)` |
| PostgreSQL | `COALESCE(campo, 0)` | — (ya es estándar) |
| MySQL | `IFNULL(campo, 0)` | `COALESCE(campo, 0)` |
| SQL Server | `ISNULL(campo, 0)` | `COALESCE(campo, 0)` |

> 💡 `COALESCE` es la opción más portable porque forma parte del estándar SQL (ANSI). Funciona en todos los motores.

</details>

---

<div align="center">

⬅️ [**Tema 18**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 18 →**](../proyectos/proyecto_ecosistema_sql.md)

</div>
