# 🏆 Proyecto — Tema 03: Rescatando una BD Legacy

> 📋 **Objetivo:** transformar una tabla legacy sin normalizar en el esquema base del curso. El resultado de este proyecto será la referencia estructural para el Tema 04 (DDL) y los temas prácticos siguientes.

---

## 🎯 Misión

Tienes una tabla heredada:

- `pedidos_legacy(id_pedido, cliente_nombre, cliente_email, producto_nombre, categoria_nombre, precio, cantidad, fecha_pedido)`

Debes entregar una **versión normalizada en 3NF** y un plan de migración mínimo.

---

## ✅ Entregable 1 — Modelo final

Tablas requeridas:
- `clientes`
- `categorias`
- `productos`
- `pedidos`
- `direcciones_cliente`

Con PK/FK y restricciones básicas de calidad (`NOT NULL`, `UNIQUE`, `CHECK`).

---

## ✅ Entregable 2 — Script de migración (plantilla)

```sql
-- 1) Catálogos
INSERT INTO categorias (id_categoria, nombre)
SELECT DISTINCT
       ROW_NUMBER() OVER (ORDER BY categoria_nombre) AS id_categoria,
       categoria_nombre
FROM pedidos_legacy;

-- 2) Clientes
INSERT INTO clientes (id_cliente, nombre, email)
SELECT DISTINCT
       ROW_NUMBER() OVER (ORDER BY cliente_email) AS id_cliente,
       cliente_nombre,
       cliente_email
FROM pedidos_legacy;

-- 3) Productos
INSERT INTO productos (id_producto, nombre, precio, id_categoria)
SELECT DISTINCT
       ROW_NUMBER() OVER (ORDER BY producto_nombre) AS id_producto,
       l.producto_nombre,
       l.precio,
       c.id_categoria
FROM pedidos_legacy l
JOIN categorias c ON c.nombre = l.categoria_nombre;

-- 4) Pedidos
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
SELECT l.id_pedido,
       cl.id_cliente,
       pr.id_producto,
       l.cantidad,
       l.fecha_pedido,
       l.precio * l.cantidad
FROM pedidos_legacy l
JOIN clientes cl ON cl.email = l.cliente_email
JOIN productos pr ON pr.nombre = l.producto_nombre;
```

---

## ✅ Entregable 3 — Reglas de continuidad

A partir de este punto:
1. Cualquier `ALTER TABLE` sobre estas tablas se considera parte del estado oficial.
2. Los ejercicios/proyectos siguientes deben usar este estado acumulado.
3. Las nuevas tablas deben documentar su relación con estas entidades base.

---

<div align="center">

⬅️ [**📝 Ejercicios del Tema 03**](../ejercicios/ejercicios_normalizacion.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 04: DDL (Data Definition Language) →**](../../04-ddl/README.md)

</div>
