# 🏋️ Ejercicios — Tema 02: Tipos de Datos (Oracle)

---

## Ejercicio 1 — Elegir tipos correctos

**Enunciado:** Define tipos Oracle para:
- `precio_producto` (hasta 99999.99)
- `codigo_pais` (2 letras fijas)
- `descripcion` (texto variable hasta 300)
- `fecha_registro` (fecha y hora)

<details>
<summary>👉 Ver solución</summary>

```sql
precio_producto NUMBER(7,2)
codigo_pais     CHAR(2)
descripcion     VARCHAR2(300)
fecha_registro  DATE
```

</details>

---

## Ejercicio 2 — DATE vs TIMESTAMP

**Enunciado:** ¿Cuándo usarías `TIMESTAMP(6)` en lugar de `DATE`?

<details>
<summary>👉 Ver solución</summary>

Cuando necesitas **fracciones de segundo** (auditoría, pagos, logs de alta frecuencia). `DATE` no guarda esa precisión fina.

</details>

---

## Ejercicio 3 — Crear tabla válida en Oracle

**Enunciado:** Crea tabla `transacciones` con:
- id numérico PK
- monto con 2 decimales
- estado de 1 carácter (`P`, `C`, `X`)
- fecha exacta con milisegundos

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE TABLE transacciones (
  id_transaccion NUMBER PRIMARY KEY,
  monto          NUMBER(12,2) NOT NULL CHECK (monto >= 0),
  estado         CHAR(1) NOT NULL CHECK (estado IN ('P','C','X')),
  fecha_evento   TIMESTAMP(3) NOT NULL
);
```

</details>

---

<div align="center">

⬅️ [**Tema 02**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 02 →**](../proyectos/proyecto_tipos_datos.md)

</div>
