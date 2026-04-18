# 🏋️ Ejercicios — Tema 18: Ecosistema SQL

> 💡 Traduce y adapta consultas del modelo acumulado entre Oracle, PostgreSQL y SQL Server.

---

## Ejercicio 1 — Portar paginación

Convierte una consulta Oracle con `FETCH FIRST` a PostgreSQL y SQL Server.

<details>
<summary>👉 Ver solución</summary>

```sql
-- Oracle
SELECT * FROM pedidos ORDER BY fecha_pedido DESC FETCH FIRST 20 ROWS ONLY;

-- PostgreSQL
SELECT * FROM pedidos ORDER BY fecha_pedido DESC LIMIT 20;

-- SQL Server
SELECT TOP 20 * FROM pedidos ORDER BY fecha_pedido DESC;
```

</details>

---

## Ejercicio 2 — Portar funciones de nulos

Reemplaza `NVL` por sintaxis portable.

<details>
<summary>👉 Ver solución</summary>

```sql
-- Oracle
SELECT NVL(total, 0) FROM pedidos;

-- Portable
SELECT COALESCE(total, 0) FROM pedidos;
```

</details>

---

## Ejercicio 3 — DDL equivalente

Define `productos` en PostgreSQL respetando tipos usados en Oracle.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE TABLE productos (
  id_producto NUMERIC(10,0) PRIMARY KEY,
  nombre VARCHAR(120) NOT NULL,
  precio NUMERIC(10,2) NOT NULL,
  stock NUMERIC(10,0) NOT NULL,
  id_categoria NUMERIC(10,0) NOT NULL
);
```

</details>

---

## Ejercicio 4 — Upsert multi-motor

Escribe estrategia de upsert para Oracle y PostgreSQL.

<details>
<summary>👉 Ver solución</summary>

```sql
-- Oracle
MERGE INTO productos t
USING (SELECT 10 id_producto, 99.99 precio FROM dual) s
ON (t.id_producto = s.id_producto)
WHEN MATCHED THEN UPDATE SET t.precio = s.precio
WHEN NOT MATCHED THEN INSERT (id_producto, precio) VALUES (s.id_producto, s.precio);

-- PostgreSQL
INSERT INTO productos(id_producto, precio)
VALUES (10, 99.99)
ON CONFLICT (id_producto)
DO UPDATE SET precio = EXCLUDED.precio;
```

</details>

---

## Ejercicio 5 — Checklist de migración

Escribe un checklist corto de validación post-migración.

<details>
<summary>👉 Ver solución</summary>

1. Conteo por tabla origen/destino.
2. Checksum por lotes.
3. Validación de claves foráneas.
4. Validación de permisos por rol.
5. Pruebas de consultas críticas.

</details>

---

<div align="center">

⬅️ [**Volver al Tema 18**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto Ecosistema SQL →**](../proyectos/proyecto_ecosistema_sql.md)

</div>
