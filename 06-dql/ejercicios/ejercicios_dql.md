# 🏋️ Ejercicios — Tema 06: DQL Básico

---

## Ejercicio 1 — SELECT de columnas específicas

**Enunciado:** Lista `nombre` y `precio` de la tabla `productos`.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT nombre, precio
FROM productos;
```

</details>

---

## Ejercicio 2 — Filtro con WHERE

**Enunciado:** Muestra pedidos de enero de 2024 con total mayor a 100.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT id_pedido, id_cliente, total, fecha_pedido
FROM pedidos
WHERE fecha_pedido >= DATE '2024-01-01'
  AND fecha_pedido < DATE '2024-02-01'
  AND total > 100;
```

</details>

---

## Ejercicio 3 — ORDER BY

**Enunciado:** Muestra los 5 productos más caros.

<details>
<summary>👉 Ver solución</summary>

```sql
SELECT id_producto, nombre, precio
FROM productos
ORDER BY precio DESC
FETCH FIRST 5 ROWS ONLY;
```

</details>

---

<div align="center">

⬅️ [**Tema 06**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyectos del Tema 06 →**](../proyectos/proyecto_dql_basico.md)

</div>
