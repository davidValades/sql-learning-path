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

Resultado: `1 row(s) inserted.`

Verificación:
```sql
SELECT nombre, precio, stock FROM productos WHERE id_producto = 99;
```
| nombre | precio | stock |
|--------|--------|-------|
| SSD 1TB | 95.50 | 40 |

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

Resultado: `2 row(s) updated.`

Verificación:
```sql
SELECT nombre, precio FROM productos WHERE id_categoria = 2;
```
| nombre | precio |
|--------|--------|
| Sofá de Cuero | 445.50 |
| Lámpara LED | 44.55 |

> 💡 Los precios originales eran 405.00 y 40.50. Tras subir un 10%: 405 × 1.10 = 445.50, 40.50 × 1.10 = 44.55.

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

Resultado del DELETE: `X row(s) deleted.` (depende de los datos en ese momento).

Resultado del ROLLBACK: Todos los pedidos eliminados se restauran. La tabla queda exactamente como estaba antes del `DELETE`.

Verificación:
```sql
SELECT COUNT(*) AS total_pedidos FROM pedidos;
```
El conteo es el mismo que antes del DELETE, confirmando que el ROLLBACK deshizo la eliminación.

</details>

---

<div align="center">

⬅️ [**Tema 05**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyectos del Tema 05 →**](../proyectos/proyecto_triatlon_dml.md)

</div>
