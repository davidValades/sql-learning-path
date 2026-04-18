# 🏋️ Ejercicios — Tema 14: DCL y Seguridad

> 💡 Usa el estado acumulado del modelo y aplica permisos como si fueras DBA de una empresa en producción.

---

## Ejercicio 1 — Crear roles por responsabilidad

Crea dos roles:

- `rol_analitica`: solo `SELECT` en `clientes`, `pedidos`, `productos`.
- `rol_soporte_hospital`: `SELECT` en `citas` y `UPDATE` solo de la columna `estado`.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE ROLE rol_analitica;
GRANT SELECT ON clientes TO rol_analitica;
GRANT SELECT ON pedidos TO rol_analitica;
GRANT SELECT ON productos TO rol_analitica;

CREATE ROLE rol_soporte_hospital;
GRANT SELECT ON citas TO rol_soporte_hospital;
GRANT UPDATE (estado) ON citas TO rol_soporte_hospital;
```

</details>

---

## Ejercicio 2 — Asignar usuarios a roles

Crea `usr_reportes` y `usr_guardia`, dales sesión y asigna los roles del ejercicio 1.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE USER usr_reportes IDENTIFIED BY "Reportes#2026";
CREATE USER usr_guardia IDENTIFIED BY "Guardia#2026";

GRANT CREATE SESSION TO usr_reportes, usr_guardia;
GRANT rol_analitica TO usr_reportes;
GRANT rol_soporte_hospital TO usr_guardia;
```

</details>

---

## Ejercicio 3 — Revocar acceso crítico

Revoca a `usr_guardia` cualquier capacidad de borrar en `citas`.

<details>
<summary>👉 Ver solución</summary>

```sql
REVOKE DELETE ON citas FROM usr_guardia;
```

</details>

---

## Ejercicio 4 — Vista segura para reporting

Crea `v_resumen_ventas` con cliente, producto, total y fecha. Concede solo `SELECT` a `rol_analitica`.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE OR REPLACE VIEW v_resumen_ventas AS
SELECT p.id_pedido, c.nombre AS cliente, pr.nombre AS producto, p.total, p.fecha_pedido
FROM pedidos p
JOIN clientes c ON c.id_cliente = p.id_cliente
JOIN productos pr ON pr.id_producto = p.id_producto;

GRANT SELECT ON v_resumen_ventas TO rol_analitica;
```

</details>

---

## Ejercicio 5 — Auditoría de privilegios

Saca un reporte de privilegios de objeto para `USR_REPORTES` y `USR_GUARDIA`.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT grantee, table_name, privilege
FROM dba_tab_privs
WHERE grantee IN ('USR_REPORTES', 'USR_GUARDIA')
ORDER BY grantee, table_name, privilege;
```

</details>

---

<div align="center">

⬅️ [**Volver al Tema 14**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto DCL →**](../proyectos/proyecto_dcl_seguridad.md)

</div>
