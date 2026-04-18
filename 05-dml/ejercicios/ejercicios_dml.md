# 🏋️ Ejercicios — Tema 05: DML

---

## Ejercicio 1 — Inserción segura

**Enunciado:** Inserta un producto con id 99, nombre `SSD 1TB`, precio 95.50, stock 40 y categoría 1.

<details>
<summary>👉 Ver solución</summary>

```sql
INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria)
VALUES (99, 'SSD 1TB', 95.50, 40, 1);
```

</details>

---

## Ejercicio 2 — UPDATE con filtro

**Enunciado:** Sube un 10% el precio de los productos de categoría 2.

<details>
<summary>👉 Ver solución</summary>

```sql
UPDATE productos
SET precio = ROUND(precio * 1.10, 2)
WHERE id_categoria = 2;
```

</details>

---

## Ejercicio 3 — DELETE + rollback

**Enunciado:** Elimina temporalmente los pedidos con total menor a 20 y luego revierte el cambio.

<details>
<summary>👉 Ver solución</summary>

```sql
DELETE FROM pedidos
WHERE total < 20;

ROLLBACK;
```

</details>

---

<div align="center">

⬅️ [**Tema 05**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyectos del Tema 05 →**](../proyectos/proyecto_triatlon_dml.md)

</div>
