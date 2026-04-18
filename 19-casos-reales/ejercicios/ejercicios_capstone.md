# 🏋️ Ejercicios — Tema 19: Capstone

> 💡 Simula incidencias reales integrando modelado, seguridad, transacciones, PL/SQL y optimización.

---

## Ejercicio 1 — Incidencia de stock negativo

Detecta productos con stock negativo y corrígelos con transacción segura.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT id_producto, nombre, stock
FROM productos
WHERE stock < 0;

SAVEPOINT stock_fix;
UPDATE productos SET stock = 0 WHERE stock < 0;
COMMIT;
```

</details>

---

## Ejercicio 2 — Auditoría de permisos excesivos

Lista usuarios con privilegios DML directos sobre tablas de negocio.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT grantee, table_name, privilege
FROM dba_tab_privs
WHERE table_name IN ('PRODUCTOS', 'PEDIDOS', 'CITAS')
  AND privilege IN ('INSERT', 'UPDATE', 'DELETE')
ORDER BY grantee, table_name;
```

</details>

---

## Ejercicio 3 — Hot query

Encuentra una consulta de ventas mensual y propón índice para mejorarla.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT id_cliente, SUM(total)
FROM pedidos
WHERE fecha_pedido >= ADD_MONTHS(TRUNC(SYSDATE), -1)
GROUP BY id_cliente;

CREATE INDEX idx_pedidos_fecha_cliente ON pedidos(fecha_pedido, id_cliente);
```

</details>

---

## Ejercicio 4 — Lógica reusable

Crea función para devolver estado operativo de un producto (`OK`, `RIESGO`, `SIN_STOCK`).

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE OR REPLACE FUNCTION fn_estado_producto(p_id_producto IN NUMBER)
RETURN VARCHAR2
AS
  v_stock NUMBER;
BEGIN
  SELECT stock INTO v_stock FROM productos WHERE id_producto = p_id_producto;

  RETURN CASE
    WHEN v_stock = 0 THEN 'SIN_STOCK'
    WHEN v_stock <= 5 THEN 'RIESGO'
    ELSE 'OK'
  END;
END;
/
```

</details>

---

## Ejercicio 5 — Evidencia ejecutiva

Genera una consulta resumen con: total ventas, total pedidos, clientes activos (últimos 90 días).

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT NVL(SUM(total),0) AS total_ventas,
       COUNT(*) AS total_pedidos,
       COUNT(DISTINCT id_cliente) AS clientes_activos
FROM pedidos
WHERE fecha_pedido >= TRUNC(SYSDATE) - 90;
```

</details>

---

<div align="center">

⬅️ [**Volver al Tema 19**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto Final →**](../proyectos/proyecto_final_integrador.md)

</div>
