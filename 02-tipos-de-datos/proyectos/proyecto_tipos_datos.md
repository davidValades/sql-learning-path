# 🏆 Proyecto — Tema 02: Esquema Tipado para E-commerce

---

## 🎯 Objetivo

Crear una estructura Oracle con tipos de datos correctos para soportar productos, clientes y pedidos sin inconsistencias.

---

## Entregable 1 — DDL con tipos Oracle

Crea tablas:
- `clientes`
- `productos`
- `pedidos`

Condiciones mínimas:
- uso correcto de `NUMBER(p,s)`,
- `VARCHAR2` para texto variable,
- `CHAR(1)` para estados,
- `DATE` o `TIMESTAMP` según precisión requerida.

---

## Entregable 2 — Validaciones de dominio

Incluye restricciones:
- precio y total `>= 0`,
- stock `>= 0`,
- estado en valores permitidos,
- email único.

---

## Entregable 3 — Prueba de inserciones

Inserta:
1. 3 filas válidas.
2. 2 filas inválidas para confirmar que los tipos/restricciones bloquean errores.

---

<div align="center">

⬅️ [**🏋️ Ejercicios del Tema 02**](../ejercicios/ejercicios_tipos_datos.md) · 🏠 [**Tema 02**](../README.md) · [**Tema 03: Normalización y Modelado ER →**](../../03-normalizacion)

</div>
