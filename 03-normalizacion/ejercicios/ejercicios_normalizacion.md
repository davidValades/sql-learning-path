# 🏋️ Ejercicios — Tema 03: Normalización y Modelado ER

> 💡 Trabaja con ejemplos de negocio para pasar de una tabla desordenada a un modelo 3FN listo para producción.

---

## Ejercicio 1 — Detectar violaciones de 1FN

En una tabla `ventas_raw` existe una columna `productos` con valores como `"Laptop|Mouse|Teclado"`.

- Explica por qué viola 1FN.
- Propón estructura corregida.

<details>
<summary>👉 Ver solución</summary>

Viola 1FN porque la columna no es atómica (guarda múltiples valores).

Modelo corregido mínimo:
- `ventas(id_venta, id_cliente, fecha_venta)`
- `ventas_detalle(id_venta, id_producto, cantidad, precio_unitario)`

</details>

---

## Ejercicio 2 — De 1FN a 2FN

La tabla `inscripciones(id_alumno, id_curso, nombre_alumno, nombre_curso, fecha)` tiene PK compuesta (`id_alumno`, `id_curso`).

Identifica dependencias parciales y separa en 2FN.

<details>
<summary>👉 Ver solución</summary>

Dependencias parciales:
- `nombre_alumno` depende solo de `id_alumno`.
- `nombre_curso` depende solo de `id_curso`.

Separación:
- `alumnos(id_alumno, nombre_alumno)`
- `cursos(id_curso, nombre_curso)`
- `inscripciones(id_alumno, id_curso, fecha)`

</details>

---

## Ejercicio 3 — De 2FN a 3FN

`clientes(id_cliente, ciudad, id_region, nombre_region)` incumple 3FN. Detecta la dependencia transitiva y corrígela.

<details>
<summary>👉 Ver solución</summary>

Dependencia transitiva: `id_cliente -> id_region -> nombre_region`.

Separación:
- `regiones(id_region, nombre_region)`
- `clientes(id_cliente, ciudad, id_region)`

</details>

---

## Ejercicio 4 — Definir claves correctamente

Propón PK/FK para modelo de pedidos:

- `clientes`
- `pedidos`
- `productos`
- `pedidos_detalle`

<details>
<summary>👉 Ver solución</summary>

- `clientes(id_cliente PK)`
- `pedidos(id_pedido PK, id_cliente FK -> clientes)`
- `productos(id_producto PK)`
- `pedidos_detalle(id_pedido FK, id_producto FK, cantidad, precio_unitario, PK(id_pedido,id_producto))`

</details>

---

## Ejercicio 5 — Validación final

Explica si este modelo está en 3FN y por qué:

- `empleados(id_empleado, id_departamento, nombre_departamento, salario)`

<details>
<summary>👉 Ver solución</summary>

No está en 3FN porque `nombre_departamento` depende de `id_departamento`, no directamente de `id_empleado`.

Corrección:
- `departamentos(id_departamento, nombre_departamento)`
- `empleados(id_empleado, id_departamento, salario)`

</details>

---

<div align="center">

⬅️ [**Volver al Tema 03**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto Normalización →**](../proyectos/proyecto_normalizacion.md)

</div>
