# 🏋️ Ejercicios — Tema 13: Transacciones

> 💡 **Instrucciones:** Resuelve cada ejercicio razonando paso a paso. En ejercicios de SAVEPOINT/ROLLBACK, lo importante es predecir el estado final de los datos. Todos los ejercicios usan el estado acumulado de la base de datos.

---

## Ejercicio 1 — E-commerce · COMMIT Básico

**Enunciado:** Escribe una transacción que procese un nuevo pedido: el cliente Pedro Ruiz (id 2) compra 2 unidades de Camiseta Básica (id 14, precio 19.99). La transacción debe:

1. Insertar el pedido (id_pedido = 80, total = 39.98).
2. Reducir el stock del producto en 2 unidades.
3. Confirmar con COMMIT.

Luego escribe la verificación.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Transacción: nuevo pedido
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (80, 2, 14, 2, SYSDATE, 39.98);

UPDATE productos SET stock = stock - 2 WHERE id_producto = 14;

COMMIT;

-- Verificación
SELECT * FROM pedidos WHERE id_pedido = 80;
-- Resultado: id 80, cliente 2, producto 14, cantidad 2, total 39.98

SELECT nombre, stock FROM productos WHERE id_producto = 14;
-- Resultado: Camiseta Básica, 98 (era 100, ahora 100-2)
```

> 💡 Si el `UPDATE` fallara (por ejemplo, el producto no existe), deberíamos hacer `ROLLBACK` para que el `INSERT` del pedido también se revierta.

```sql
-- Limpiar
DELETE FROM pedidos WHERE id_pedido = 80;
UPDATE productos SET stock = 100 WHERE id_producto = 14;
COMMIT;
```

</details>

---

## Ejercicio 2 — Hospital · ROLLBACK ante Error

**Enunciado:** El sistema intenta registrar una cita y actualizar el teléfono de un paciente en la misma transacción. Pero al verificar los datos, se detecta un error. Escribe la transacción con ROLLBACK.

1. Insertar cita (id 50, paciente 5, médico 6, fecha '2024-07-01 11:00', estado 'P').
2. Actualizar teléfono del paciente 5 a '699111222'.
3. ¡Error detectado! El médico 6 no trabaja los lunes → ROLLBACK.

¿Qué datos quedan en la base?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Transacción
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado)
VALUES (50, 5, 6, TO_DATE('2024-07-01 11:00', 'YYYY-MM-DD HH24:MI'), 'P');

UPDATE pacientes SET telefono = '699111222' WHERE id_paciente = 5;

-- Error detectado
ROLLBACK;

-- Verificación
SELECT * FROM citas WHERE id_cita = 50;
-- Resultado: no rows selected (la cita se revirtió)

SELECT telefono FROM pacientes WHERE id_paciente = 5;
-- Resultado: valor original (sin cambios)
```

**Ningún dato cambió.** Ambas operaciones (INSERT y UPDATE) se revirtieron completamente.

</details>

---

## Ejercicio 3 — Aerolínea · SAVEPOINT Básico

**Enunciado:** La aerolínea programa 3 nuevos vuelos, pero el tercero tiene un conflicto de puerta de embarque. Usa SAVEPOINT para mantener los dos primeros y revertir solo el tercero.

1. INSERT vuelo 2001 (ruta MADLHR, avión 30, puerta B10).
2. INSERT vuelo 2002 (ruta JFKLAX, avión 50, puerta A01).
3. SAVEPOINT despues_de_dos.
4. INSERT vuelo 2003 (ruta BCNCDG, avión 40, puerta B10) ← ¡conflicto con puerta del vuelo 2001!
5. ROLLBACK TO SAVEPOINT despues_de_dos.
6. COMMIT.

¿Qué vuelos quedan confirmados?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
INSERT INTO vuelos (id_vuelo, id_ruta, id_avion, fecha_salida, puerta_embarque)
VALUES (2001, 'MADLHR', 30, SYSDATE + 10, 'B10');

INSERT INTO vuelos (id_vuelo, id_ruta, id_avion, fecha_salida, puerta_embarque)
VALUES (2002, 'JFKLAX', 50, SYSDATE + 10, 'A01');

SAVEPOINT despues_de_dos;

INSERT INTO vuelos (id_vuelo, id_ruta, id_avion, fecha_salida, puerta_embarque)
VALUES (2003, 'BCNCDG', 40, SYSDATE + 10, 'B10');

-- Conflicto detectado
ROLLBACK TO SAVEPOINT despues_de_dos;

COMMIT;
```

**Vuelos confirmados: 2001 y 2002.** El vuelo 2003 se revirtió.

```sql
-- Limpiar
DELETE FROM vuelos WHERE id_vuelo IN (2001, 2002);
COMMIT;
```

</details>

---

## Ejercicio 4 — E-commerce · Múltiples SAVEPOINT

**Enunciado:** Predice el estado final del stock del Monitor 4K (id 16, stock inicial = 25) después de estas operaciones:

```sql
UPDATE productos SET stock = 20 WHERE id_producto = 16;
SAVEPOINT sp1;
UPDATE productos SET stock = 15 WHERE id_producto = 16;
SAVEPOINT sp2;
UPDATE productos SET stock = 10 WHERE id_producto = 16;
SAVEPOINT sp3;
UPDATE productos SET stock = 5 WHERE id_producto = 16;
ROLLBACK TO SAVEPOINT sp2;
COMMIT;
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**El stock final es 15.**

Paso a paso:
1. `UPDATE stock = 20` → stock = 20
2. `SAVEPOINT sp1` → marca en stock = 20
3. `UPDATE stock = 15` → stock = 15
4. `SAVEPOINT sp2` → marca en stock = 15
5. `UPDATE stock = 10` → stock = 10
6. `SAVEPOINT sp3` → marca en stock = 10
7. `UPDATE stock = 5` → stock = 5
8. `ROLLBACK TO sp2` → revierte al estado de sp2 → **stock = 15**
   (sp3 también desaparece)
9. `COMMIT` → confirma stock = **15**

```sql
-- Restaurar
UPDATE productos SET stock = 25 WHERE id_producto = 16;
COMMIT;
```

</details>

---

## Ejercicio 5 — Hospital · Transacción con Verificación

**Enunciado:** Escribe una transacción que cambie el estado de la cita 3 de 'P' (Pendiente) a 'C' (Completada), pero solo si el médico asociado tiene un salario superior a 3000. Si no cumple la condición, haz ROLLBACK.

Pista: La cita 3 es de Dra. Marta López (salario 4500).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Verificar la condición primero
-- Cita 3: id_medico = 6 (Dra. Marta López, salario 4500)

-- Actualizar la cita
UPDATE citas SET estado = 'C' WHERE id_cita = 3;

-- Verificar condición (en PL/SQL sería automático, aquí es manual)
-- Salario de Dra. Marta López = 4500 > 3000 ✅ → COMMIT
COMMIT;

-- Si el salario fuera <= 3000, haríamos:
-- ROLLBACK;

-- Verificación
SELECT ci.id_cita, ci.estado, m.nombre_completo, m.salario_base
FROM citas ci
JOIN medicos m ON ci.id_medico = m.id_medico
WHERE ci.id_cita = 3;
-- Resultado: cita 3, estado 'C', Dra. Marta López, 4500
```

```sql
-- Restaurar estado original
UPDATE citas SET estado = 'P' WHERE id_cita = 3;
COMMIT;
```

</details>

---

## Ejercicio 6 — E-commerce · DDL y Auto-Commit

**Enunciado:** ¿Qué queda en la base de datos después de ejecutar este bloque?

```sql
INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria)
VALUES (99, 'Producto A', 10.00, 5, 1);

INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria)
VALUES (98, 'Producto B', 20.00, 10, 2);

CREATE TABLE temp_test (id NUMBER);

ROLLBACK;

SELECT nombre FROM productos WHERE id_producto IN (98, 99);
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Ambos productos (98 y 99) existen. La tabla temp_test también existe.**

Explicación:
1. `INSERT 99` — no confirmado (transacción abierta).
2. `INSERT 98` — no confirmado (misma transacción).
3. `CREATE TABLE` — es DDL → **auto-commit implícito** antes de ejecutarse. Los dos INSERTs se confirman permanentemente.
4. `ROLLBACK` — no hay nada que revertir (la transacción ya se confirmó).

Resultado del SELECT: `Producto A` y `Producto B`.

```sql
-- Limpiar
DELETE FROM productos WHERE id_producto IN (98, 99);
DROP TABLE temp_test;
COMMIT;
```

> 💡 Los DDL son "trampas silenciosas" que confirman cambios que no querías guardar. Nunca mezcles DDL con DML en una transacción.

</details>

---

## Ejercicio 7 — Aerolínea · Cadena de SAVEPOINT

**Enunciado:** Sigue la ejecución paso a paso y determina qué vuelos existen al final:

```sql
INSERT INTO vuelos VALUES (3001, 'MADLHR', 30, SYSDATE+1, 'A01');
SAVEPOINT alpha;

INSERT INTO vuelos VALUES (3002, 'BCNCDG', 40, SYSDATE+2, 'B05');
SAVEPOINT beta;

INSERT INTO vuelos VALUES (3003, 'MADFRA', 30, SYSDATE+3, 'C10');
ROLLBACK TO SAVEPOINT alpha;

INSERT INTO vuelos VALUES (3004, 'BCNFCO', 40, SYSDATE+4, 'D20');
SAVEPOINT gamma;

INSERT INTO vuelos VALUES (3005, 'JFKLAX', 50, SYSDATE+5, 'E15');
ROLLBACK TO SAVEPOINT gamma;

COMMIT;
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Vuelos que existen: 3001 y 3004.**

| Paso | Operación | Vuelos en memoria |
|------|-----------|-------------------|
| 1 | INSERT 3001 | 3001 |
| 2 | SAVEPOINT alpha | 3001 (marcado) |
| 3 | INSERT 3002 | 3001, 3002 |
| 4 | SAVEPOINT beta | 3001, 3002 (marcado) |
| 5 | INSERT 3003 | 3001, 3002, 3003 |
| 6 | ROLLBACK TO alpha | **3001** (se revierten 3002 y 3003, beta desaparece) |
| 7 | INSERT 3004 | 3001, 3004 |
| 8 | SAVEPOINT gamma | 3001, 3004 (marcado) |
| 9 | INSERT 3005 | 3001, 3004, 3005 |
| 10 | ROLLBACK TO gamma | **3001, 3004** (se revierte 3005) |
| 11 | COMMIT | 3001, 3004 confirmados ✅ |

```sql
-- Limpiar
DELETE FROM vuelos WHERE id_vuelo IN (3001, 3004);
COMMIT;
```

</details>

---

## Ejercicio 8 — Combinado · Transacción Multi-Tabla

**Enunciado:** Escribe una transacción completa para este escenario:

> Ana López (id 1) compra un Monitor 4K (id 16) y paga con un descuento del 10%. Además, se le actualiza el email a 'ana.lopez@premium.com'. Use SAVEPOINT para poder revertir solo la actualización del email si falla la validación.

1. Insertar pedido (id 85, cantidad 1, total = 350 × 0.90 = 315.00).
2. Reducir stock del Monitor 4K en 1.
3. SAVEPOINT pedido_ok.
4. Actualizar email de Ana López.
5. Si todo OK → COMMIT. Si el email es inválido → ROLLBACK TO SAVEPOINT pedido_ok, luego COMMIT.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Paso 1: insertar pedido con descuento
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (85, 1, 16, 1, SYSDATE, 315.00);

-- Paso 2: reducir stock
UPDATE productos SET stock = stock - 1 WHERE id_producto = 16;

-- Paso 3: punto de control
SAVEPOINT pedido_ok;

-- Paso 4: actualizar email
UPDATE clientes SET email = 'ana.lopez@premium.com' WHERE id_cliente = 1;

-- Escenario A: todo OK
COMMIT;
-- El pedido, el stock y el email se confirman

-- Escenario B: email inválido
-- ROLLBACK TO SAVEPOINT pedido_ok;
-- COMMIT;
-- Solo se confirman el pedido y el stock. El email no cambia.

-- Verificación
SELECT * FROM pedidos WHERE id_pedido = 85;
SELECT stock FROM productos WHERE id_producto = 16;
SELECT email FROM clientes WHERE id_cliente = 1;
```

```sql
-- Limpiar
DELETE FROM pedidos WHERE id_pedido = 85;
UPDATE productos SET stock = 25 WHERE id_producto = 16;
UPDATE clientes SET email = 'ana@email.com' WHERE id_cliente = 1;
COMMIT;
```

> 💡 El patrón SAVEPOINT permite hacer **commits parciales**: confirmar la parte crítica (el pedido) sin arriesgar que un error en una parte secundaria (el email) revierta todo.

</details>

---

<div align="center">

⬅️ [**Tema 13: Transacciones y Propiedades ACID**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 13 →**](../proyectos/proyecto_transacciones.md)

</div>
