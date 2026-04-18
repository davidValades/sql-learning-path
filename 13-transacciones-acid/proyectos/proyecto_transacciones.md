# 🏆 Proyecto: El Día del Desastre

> 📋 **Contexto:** Es viernes por la tarde. El DBA principal está de vacaciones y tú eres el DBA de guardia. De repente, un becario ejecuta una serie de operaciones catastróficas en las tres bases de datos. Por suerte, no hizo COMMIT. Tu misión: usar SAVEPOINT y ROLLBACK estratégicamente para recuperar lo que se pueda salvar y revertir el daño.

---

## 🎯 Misión 1 — E-commerce: El Becario Entusiasta

El becario intentó "actualizar" la tienda online con varios cambios. Algunos son correctos y otros son desastrosos. Analiza cada operación y decide qué salvar y qué revertir.

### Tarea 1.1 — Evaluar y Recuperar

El becario ejecutó (sin COMMIT) esta secuencia. Escribe los SAVEPOINT y ROLLBACK necesarios para **salvar las operaciones correctas y revertir las incorrectas**.

```
Operación A: Actualizar precio de Camiseta Básica de 19.99 a 22.99 (correcto: ajuste de inflación)
Operación B: Insertar nuevo producto "USB-C Cable" (id 25, precio 12.99, stock 200, cat 1) (correcto)
Operación C: DELETE FROM productos WHERE precio < 100 (¡DESASTRE! elimina 5 productos)
Operación D: Actualizar stock de Laptop Pro a 20 unidades (correcto: reposición)
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Operación A: ✅ correcta
UPDATE productos SET precio = 22.99 WHERE id_producto = 16;

SAVEPOINT despues_A;

-- Operación B: ✅ correcta
INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria)
VALUES (25, 'USB-C Cable', 12.99, 200, 1);

SAVEPOINT despues_B;

-- Operación C: ❌ DESASTRE
DELETE FROM productos WHERE precio < 100;
-- Esto eliminaría: Ratón (45.50), Teclado (75), Lámpara (40.50),
-- Camiseta (22.99), Zapatillas (89.50), USB-C Cable (12.99)

-- ¡REVERTIR solo la operación C!
ROLLBACK TO SAVEPOINT despues_B;

-- Operación D: ✅ correcta
UPDATE productos SET stock = 20 WHERE id_producto = 10;

-- Confirmar A, B y D
COMMIT;

-- Verificación
SELECT nombre, precio, stock FROM productos ORDER BY id_producto;
-- Camiseta Básica: 22.99 ✅
-- USB-C Cable: 12.99, stock 200 ✅
-- Todos los productos intactos ✅
-- Laptop Pro: stock 20 ✅
```

```sql
-- Limpiar a estado original
UPDATE productos SET precio = 19.99 WHERE id_producto = 16;
DELETE FROM productos WHERE id_producto = 25;
UPDATE productos SET stock = 15 WHERE id_producto = 10;
COMMIT;
```

</details>

### Tarea 1.2 — Pedidos Fantasma

El becario también insertó 3 pedidos, pero solo el primero es legítimo:

```
Pedido 90: Ana López compra Laptop Pro, cantidad 1, total 1220.00 (✅ legítimo)
Pedido 91: Cliente id 99 compra producto id 99 (❌ cliente y producto inexistentes)
Pedido 92: Pedro Ruiz compra Monitor 4K, cantidad -5 (❌ cantidad negativa)
```

Escribe la transacción con SAVEPOINT para salvar solo el pedido 90.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Pedido legítimo
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (90, 1, 10, 1, SYSDATE, 1220.00);

UPDATE productos SET stock = stock - 1 WHERE id_producto = 10;

SAVEPOINT pedido_90_ok;

-- Pedido inválido (cliente/producto inexistentes)
-- En la práctica, Oracle rechazaría esto por FK constraint
-- Pero si pasara: lo revertimos
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (91, 99, 99, 1, SYSDATE, 0);

ROLLBACK TO SAVEPOINT pedido_90_ok;

-- Otro pedido inválido (cantidad negativa)
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (92, 2, 12, -5, SYSDATE, -1750.00);

ROLLBACK TO SAVEPOINT pedido_90_ok;

-- Solo queda el pedido 90
COMMIT;

-- Verificación
SELECT * FROM pedidos WHERE id_pedido IN (90, 91, 92);
-- Solo aparece el pedido 90
```

```sql
-- Limpiar
DELETE FROM pedidos WHERE id_pedido = 90;
UPDATE productos SET stock = 15 WHERE id_producto = 10;
COMMIT;
```

</details>

---

## 🎯 Misión 2 — Hospital: La Noche Caótica

A las 3 AM, el sistema de citas del hospital se desincroniza. El becario intenta arreglarlo manualmente y empeora las cosas.

### Tarea 2.1 — Rescate de Citas

El becario ejecutó:

```
Operación A: Cambiar cita 1 de estado 'C' a 'P' (❌ error: ya estaba completada)
Operación B: Insertar cita 50 para paciente 3 con Dr. Luis Moreno (✅ legítima)
Operación C: DELETE FROM citas WHERE estado = 'X' (❌ borra historial de cancelaciones)
Operación D: Cambiar cita 6 de 'P' a 'C' (✅ el paciente confirmó asistencia)
```

Escribe la transacción para salvar B y D, revertir A y C.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- A: ❌ error
UPDATE citas SET estado = 'P' WHERE id_cita = 1;

-- Revertir A inmediatamente
ROLLBACK;

-- B: ✅ legítima
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado)
VALUES (50, 3, 7, TO_DATE('2024-08-15 09:00', 'YYYY-MM-DD HH24:MI'), 'P');

SAVEPOINT despues_B;

-- C: ❌ desastre
DELETE FROM citas WHERE estado = 'X';

-- Revertir C
ROLLBACK TO SAVEPOINT despues_B;

-- D: ✅ legítima
UPDATE citas SET estado = 'C' WHERE id_cita = 6;

-- Confirmar B y D
COMMIT;

-- Verificación
SELECT id_cita, estado FROM citas ORDER BY id_cita;
-- Cita 1: 'C' (sin cambios, se revirtió A)
-- Cita 5: estado 'X' (sin cambios, se revirtió C)
-- Cita 6: 'C' (actualizada correctamente con D)
-- Cita 50: existe, estado 'P' (insertada con B)
```

```sql
-- Limpiar
DELETE FROM citas WHERE id_cita = 50;
UPDATE citas SET estado = 'P' WHERE id_cita = 6;
COMMIT;
```

</details>

### Tarea 2.2 — Actualización Masiva de Salarios

Recursos Humanos aprobó un aumento del 5% para los traumatólogos y del 3% para los de medicina general. Pero NO para los neurólogos (están en renegociación). El becario aplicó el 5% a TODOS.

Escribe la transacción correcta usando SAVEPOINT para aplicar los aumentos selectivamente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Aumento correcto para Traumatología (5%)
UPDATE medicos SET salario_base = salario_base * 1.05
WHERE id_especialidad = 100;
-- Dra. Elena Ruiz: 3200 → 3360
-- Dr. Carlos Vega: 2800 → 2940

SAVEPOINT trauma_ok;

-- Aumento correcto para Medicina General (3%)
UPDATE medicos SET salario_base = salario_base * 1.03
WHERE id_especialidad = 300;
-- Dr. Luis Moreno: 2600 → 2678

SAVEPOINT medgen_ok;

-- Si por error alguien intenta subir a Neurología:
-- UPDATE medicos SET salario_base = salario_base * 1.05
-- WHERE id_especialidad = 200;
-- ROLLBACK TO SAVEPOINT medgen_ok;

-- Confirmar solo Traumatología y Medicina General
COMMIT;

-- Verificación
SELECT nombre_completo, salario_base, id_especialidad
FROM medicos
ORDER BY id_especialidad;
-- Traumatología: Dra. Elena Ruiz 3360, Dr. Carlos Vega 2940
-- Neurología: Dra. Sarah Adams 4000, Dra. Marta López 4500 (SIN CAMBIOS)
-- Med. General: Dr. Luis Moreno 2678
```

```sql
-- Limpiar
UPDATE medicos SET salario_base = 3200 WHERE id_medico = 4;
UPDATE medicos SET salario_base = 2800 WHERE id_medico = 5;
UPDATE medicos SET salario_base = 2600 WHERE id_medico = 7;
COMMIT;
```

</details>

---

## 🎯 Misión 3 — Aerolínea: Turbulencia en los Datos

Un fallo de sincronización entre sistemas causó datos corruptos en la tabla de vuelos. Tienes que limpiar el desastre.

### Tarea 3.1 — Reasignación de Flota

Mantenimiento reporta que el Airbus A350 (id 30) necesita revisión urgente. Hay que reasignar sus vuelos a otros aviones, pero solo si hay capacidad. Usa SAVEPOINT por si alguna reasignación no es viable.

Datos actuales del Airbus A350:
- Vuelo 1001 (MADLHR) → reasignar a Boeing 777 (id 50) ✅ (tiene capacidad)
- Vuelo 1004 (MADFRA) → reasignar a Boeing 737 (id 20) ❌ (no está certificado para rutas internacionales)

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Reasignar vuelo 1001 al Boeing 777
UPDATE vuelos SET id_avion = 50 WHERE id_vuelo = 1001;

SAVEPOINT vuelo_1001_ok;

-- Intentar reasignar vuelo 1004 al Boeing 737
UPDATE vuelos SET id_avion = 20 WHERE id_vuelo = 1004;

-- ¡Error! Boeing 737 no certificado para rutas internacionales
ROLLBACK TO SAVEPOINT vuelo_1001_ok;

-- El vuelo 1004 necesita otra solución (quizás cancelarse)
-- Por ahora, dejamos solo la reasignación del 1001
COMMIT;

-- Verificación
SELECT v.id_vuelo, a.modelo, r.aeropuerto_origen, r.aeropuerto_destino
FROM vuelos v
JOIN aviones a ON v.id_avion = a.id_avion
JOIN rutas r ON v.id_ruta = r.id_ruta
WHERE v.id_vuelo IN (1001, 1004);
-- Vuelo 1001: Boeing 777 (reasignado) ✅
-- Vuelo 1004: Airbus A350 (sin cambios, reasignación revertida)
```

```sql
-- Limpiar
UPDATE vuelos SET id_avion = 30 WHERE id_vuelo = 1001;
COMMIT;
```

</details>

### Tarea 3.2 — El Plan de Contingencia Completo

Es el escenario final. Las tres bases de datos necesitan operaciones de emergencia coordinadas. Escribe UNA transacción que haga todo lo siguiente, con SAVEPOINT entre cada base de datos para poder revertir selectivamente:

1. **E-commerce:** Poner stock = 0 en Sofá de Cuero (ya está en 0) y registrar un pedido de devolución (id 95, cliente 1, producto 14, cantidad -1, total -405.00).
2. **SAVEPOINT ecommerce_ok**
3. **Hospital:** Cancelar la cita pendiente del paciente David Torres (cita 6, cambiar estado a 'X').
4. **SAVEPOINT hospital_ok**
5. **Aerolínea:** Cancelar todos los vuelos del Boeing 737 (id 20)... ¡pero no tiene vuelos! Verificar y confirmar.
6. **COMMIT** si todo está correcto.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- 1. E-commerce: registrar devolución
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (95, 1, 14, -1, SYSDATE, -405.00);

SAVEPOINT ecommerce_ok;

-- 3. Hospital: cancelar cita de David Torres
UPDATE citas SET estado = 'X' WHERE id_cita = 6;

SAVEPOINT hospital_ok;

-- 5. Aerolínea: cancelar vuelos del Boeing 737
-- Verificar primero:
-- SELECT COUNT(*) FROM vuelos WHERE id_avion = 20;
-- Resultado: 0 (no tiene vuelos, nada que cancelar)

-- No hay operación que hacer, pero el SAVEPOINT se mantiene
SAVEPOINT aerolinea_ok;

-- 6. Todo correcto → COMMIT
COMMIT;

-- Verificación completa
SELECT id_pedido, total FROM pedidos WHERE id_pedido = 95;
-- Resultado: pedido 95, total -405.00 (devolución registrada)

SELECT id_cita, estado FROM citas WHERE id_cita = 6;
-- Resultado: cita 6, estado 'X' (cancelada)

SELECT COUNT(*) AS vuelos_boeing_737 FROM vuelos WHERE id_avion = 20;
-- Resultado: 0 (sin cambios)
```

**Si alguno de los bloques hubiera fallado**, podríamos haber hecho:
- `ROLLBACK TO SAVEPOINT hospital_ok;` → revierte solo la aerolínea.
- `ROLLBACK TO SAVEPOINT ecommerce_ok;` → revierte hospital y aerolínea.
- `ROLLBACK;` → revierte todo.

```sql
-- Limpiar todo
DELETE FROM pedidos WHERE id_pedido = 95;
UPDATE citas SET estado = 'P' WHERE id_cita = 6;
COMMIT;
```

> 💡 Este patrón de **SAVEPOINT por módulo/base de datos** es una práctica real en sistemas empresariales. Permite que un fallo en un módulo no arrastre a los demás.

</details>

---

<div align="center">

⬅️ [**Ejercicios Transacciones**](../ejercicios/ejercicios_transacciones.md) · 🏠 [**Volver al Tema 13**](../README.md) · [**Tema 14: DCL y Seguridad →**](../../14-dcl-seguridad)

</div>
