# 🏋️ Ejercicios — Tema 13: Propiedades ACID

> 💡 **Instrucciones:** Estos ejercicios combinan preguntas teóricas con demostraciones SQL prácticas. Lo importante es que entiendas **por qué** cada propiedad ACID es fundamental y cómo Oracle las implementa. Todos los ejercicios usan el estado acumulado de nuestras tres bases de datos.

---

## Ejercicio 1 — Teoría · Identificar la Propiedad ACID

**Enunciado:** Para cada escenario, identifica cuál de las 4 propiedades ACID es la protagonista. Justifica tu respuesta.

**Escenario A:** Un cliente hace un pedido en la tienda online. El sistema inserta el pedido y resta el stock. Si la resta del stock falla, el pedido también se revierte.

**Escenario B:** Un hacker intenta insertar un pedido con `id_cliente = 9999` (que no existe). Oracle rechaza la operación con `ORA-02291`.

**Escenario C:** Dos recepcionistas del hospital consultan la misma lista de citas simultáneamente. Ninguna ve los cambios sin confirmar de la otra.

**Escenario D:** El servidor de la aerolínea sufre un apagón justo después de que un agente hiciera `COMMIT` en una reserva. Al reiniciar, la reserva sigue ahí.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

| Escenario | Propiedad | Justificación |
|-----------|-----------|---------------|
| **A** | **Atomicidad** | La transacción completa (insertar pedido + restar stock) se ejecuta toda o se revierte toda. No queda un estado intermedio. |
| **B** | **Consistencia** | La constraint FOREIGN KEY impide que la base de datos pase a un estado inválido (un pedido sin cliente real). |
| **C** | **Aislamiento** | Las transacciones concurrentes no ven los datos no confirmados de las demás. Cada recepcionista trabaja con su propia "vista" de los datos. |
| **D** | **Durabilidad** | Una vez confirmado con COMMIT, el cambio se escribió en los redo logs y sobrevive a fallos del sistema. |

</details>

---

## Ejercicio 2 — E-commerce · Atomicidad con SAVEPOINT

**Enunciado:** El departamento de logística necesita procesar 3 pedidos en una sola transacción. Pero el segundo pedido tiene un error (producto con stock insuficiente). Usa SAVEPOINT para confirmar los pedidos correctos y revertir solo el erróneo.

Pedidos a procesar:
1. Cliente Ana García (id 1) compra 1 Laptop Pro (id 10) → total 1200.00 (id_pedido = 93)
2. Cliente Pedro Ruiz (id 2) compra 999 Camiseta Básica (id 16) → ¡stock insuficiente! (id_pedido = 94)
3. Cliente María López (id 3) compra 2 Zapatillas Running (id 17) → total 179.00 (id_pedido = 95)

Escribe la transacción completa con SAVEPOINT, ROLLBACK TO y COMMIT. Verifica el estado final.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Pedido 1: correcto ✅
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (93, 1, 10, 1, SYSDATE, 1200.00);
UPDATE productos SET stock = stock - 1 WHERE id_producto = 10;

SAVEPOINT despues_pedido1;

-- Pedido 2: ¡error! 999 unidades > stock disponible (500) ❌
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (94, 2, 16, 999, SYSDATE, 19990.01);
UPDATE productos SET stock = stock - 999 WHERE id_producto = 16;
-- stock quedaría en -499 → detectamos el error

-- Revertir solo el pedido 2
ROLLBACK TO SAVEPOINT despues_pedido1;

-- Pedido 3: correcto ✅
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (95, 3, 17, 2, SYSDATE, 179.00);
UPDATE productos SET stock = stock - 2 WHERE id_producto = 17;

COMMIT;

-- Verificación
SELECT id_pedido, id_cliente, total FROM pedidos WHERE id_pedido IN (93, 94, 95);
-- Resultado: pedidos 93 y 95 existen; pedido 94 NO existe

SELECT nombre, stock FROM productos WHERE id_producto IN (10, 16, 17);
-- Laptop Pro: 99 (100-1)
-- Camiseta Básica: 500 (sin cambios — se revirtió)
-- Zapatillas Running: 498 (500-2)
```

> 💡 La atomicidad parcial con SAVEPOINT nos permitió salvar los pedidos buenos sin perder toda la transacción.

```sql
-- Limpiar
DELETE FROM pedidos WHERE id_pedido IN (93, 95);
UPDATE productos SET stock = 100 WHERE id_producto = 10;
UPDATE productos SET stock = 500 WHERE id_producto = 17;
COMMIT;
```

</details>

---

## Ejercicio 3 — Hospital · Consistencia y Constraints

**Enunciado:** Un desarrollador junior intenta ejecutar estas operaciones en la base de datos del hospital. Para cada una, indica si tendrá **éxito o error**, y qué tipo de constraint lo protege.

```sql
-- Operación A
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado) 
VALUES (70, 1, 2, TO_DATE('2024-10-01 09:00', 'YYYY-MM-DD HH24:MI'), 'P');

-- Operación B
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado) 
VALUES (70, 2, 2, TO_DATE('2024-10-02 10:00', 'YYYY-MM-DD HH24:MI'), 'P');

-- Operación C
DELETE FROM especialidades WHERE id_especialidad = 100;

-- Operación D
INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base) 
VALUES (20, 'Dr. Nuevo', 99, 3000);

-- Operación E
UPDATE pacientes SET nombre = NULL WHERE id_paciente = 1;
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

| Operación | Resultado | Constraint | Explicación |
|-----------|-----------|------------|-------------|
| **A** | ✅ Éxito | — | Todos los valores son válidos: paciente 1 y médico 2 existen. |
| **B** | ❌ Error | `PRIMARY KEY` | El `id_cita = 70` ya fue insertado por la operación A. ORA-00001: unique constraint violated. |
| **C** | ❌ Error | `FOREIGN KEY` | Hay médicos que referencian la especialidad 100 (Traumatología). ORA-02292: integrity constraint violated - child record found. |
| **D** | ❌ Error | `FOREIGN KEY` | La especialidad 99 no existe en la tabla especialidades. ORA-02291: parent key not found. |
| **E** | ❌ Error | `NOT NULL` | El campo nombre no permite valores nulos. ORA-01407: cannot update to NULL. |

> 💡 De las 5 operaciones, solo la A tiene éxito. Las constraints protegen la **consistencia** de la base de datos impidiendo 4 estados inválidos diferentes.

```sql
-- Limpiar
ROLLBACK; -- Revierte la operación A si no se hizo COMMIT
```

</details>

---

## Ejercicio 4 — Aerolínea · Aislamiento entre Sesiones

**Enunciado:** Analiza esta secuencia de eventos entre dos agentes de la aerolínea. ¿Cuántas plazas disponibles tiene el vuelo 1001 en cada momento para cada agente? Asume que el vuelo 1001 tiene 150 plazas al inicio y Oracle usa READ COMMITTED.

```
Tiempo  Agente A                                    Agente B
──────  ─────────────────────────────────────────   ─────────────────────────────────────
T1      SELECT plazas_disponibles                   
        FROM vuelos WHERE id_vuelo = 1001;          
                                                    
T2                                                  UPDATE vuelos 
                                                    SET plazas_disponibles = 149 
                                                    WHERE id_vuelo = 1001;
                                                    
T3      SELECT plazas_disponibles                   
        FROM vuelos WHERE id_vuelo = 1001;          
                                                    
T4                                                  COMMIT;
                                                    
T5      SELECT plazas_disponibles                   
        FROM vuelos WHERE id_vuelo = 1001;          
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

| Tiempo | Agente A ve | Agente B ve | Explicación |
|--------|:-----------:|:-----------:|-------------|
| **T1** | **150** | — | Valor original. |
| **T2** | — | 149 (su cambio pendiente) | B modifica pero no confirma. |
| **T3** | **150** | — | A **no ve** el cambio de B porque no tiene COMMIT. Aislamiento MVCC: Oracle lee del undo segment. |
| **T4** | — | — | B confirma. El cambio 149 es ahora permanente. |
| **T5** | **149** | — | **Ahora** A ve 149 porque el COMMIT de B ya ocurrió. En READ COMMITTED, cada SELECT ve el estado confirmado más reciente. |

> 💡 Si el Agente A estuviera en modo SERIALIZABLE, T5 seguiría mostrando 150 (la foto se congeló al inicio de su transacción).

</details>

---

## Ejercicio 5 — E-commerce · Durabilidad y Recuperación

**Enunciado:** Analiza esta secuencia de eventos y determina el estado final de la base de datos después de un fallo y reinicio del servidor.

```sql
-- 10:00 AM
INSERT INTO clientes (id_cliente, nombre, email) 
VALUES (10, 'Roberto Test', 'roberto@test.com');
COMMIT;  -- ✅ Confirmado

-- 10:05 AM
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total) 
VALUES (96, 10, 11, 1, SYSDATE, 45.50);
COMMIT;  -- ✅ Confirmado

-- 10:10 AM
UPDATE clientes SET email = 'roberto@nuevo.com' WHERE id_cliente = 10;
-- ⚠️ NO se hizo COMMIT

-- 10:11 AM
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total) 
VALUES (97, 10, 12, 2, SYSDATE, 150.00);
-- ⚠️ NO se hizo COMMIT

-- 10:12 AM — ¡FALLO DEL SERVIDOR! ⚡
```

Preguntas:
1. ¿Existe el cliente Roberto Test después del reinicio?
2. ¿Qué email tiene?
3. ¿Cuántos pedidos tiene Roberto?
4. ¿Qué pedidos son?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Estado tras la recuperación:**

1. ✅ **Sí existe** el cliente Roberto Test — se insertó con COMMIT a las 10:00.
2. Email: **roberto@test.com** — el UPDATE a 'roberto@nuevo.com' no tenía COMMIT → se revierte (durabilidad protege solo lo confirmado).
3. **1 pedido** — solo el pedido 96 tenía COMMIT.
4. **Pedido 96** (producto 11, cantidad 1, total 45.50) — el pedido 97 no tenía COMMIT → se revierte.

**Proceso de Instance Recovery de Oracle:**

```
Roll Forward (redo): re-aplica los cambios confirmados
  ✅ INSERT cliente 10 (COMMIT a las 10:00)
  ✅ INSERT pedido 96 (COMMIT a las 10:05)

Roll Back (undo): revierte los cambios sin confirmar
  ❌ UPDATE email a 'roberto@nuevo.com' (sin COMMIT)
  ❌ INSERT pedido 97 (sin COMMIT)
```

> 💡 La lección clave: **COMMIT = seguro. Sin COMMIT = temporal.** Los redo logs solo protegen los cambios confirmados.

```sql
-- Limpiar (si ejecutaste las operaciones)
DELETE FROM pedidos WHERE id_pedido IN (96, 97);
DELETE FROM clientes WHERE id_cliente = 10;
COMMIT;
```

</details>

---

## Ejercicio 6 — Hospital · Niveles de Aislamiento

**Enunciado:** El director del hospital necesita generar dos tipos de reportes. Para cada uno, indica qué nivel de aislamiento es más apropiado (READ COMMITTED o SERIALIZABLE) y escribe el código SQL.

**Reporte A — Consulta rápida:** "¿Cuántos pacientes hay registrados ahora mismo?" (una sola consulta, el dato puede cambiar entre ejecuciones).

**Reporte B — Auditoría mensual:** "Total de pacientes, total de citas del mes y promedio de citas por médico." (tres consultas que DEBEN ser coherentes entre sí — si hay 10 citas, las tres consultas deben reflejar ese mismo número).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Reporte A — READ COMMITTED** (suficiente para una consulta aislada):

```sql
-- READ COMMITTED es el nivel por defecto, no necesita SET TRANSACTION
SELECT COUNT(*) AS total_pacientes FROM pacientes;
-- Si alguien inserta un paciente entre dos ejecuciones
-- de esta consulta, veremos el nuevo dato. Aceptable.
```

**Reporte B — SERIALIZABLE** (necesitamos consistencia entre consultas):

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Consulta 1: total de pacientes
SELECT COUNT(*) AS total_pacientes FROM pacientes;

-- Consulta 2: total de citas del mes actual
SELECT COUNT(*) AS citas_mes
FROM citas
WHERE fecha_hora_cita >= TRUNC(SYSDATE, 'MM')
  AND fecha_hora_cita < ADD_MONTHS(TRUNC(SYSDATE, 'MM'), 1);

-- Consulta 3: promedio de citas por médico
SELECT m.nombre_completo, COUNT(c.id_cita) AS citas
FROM medicos m
LEFT JOIN citas c ON m.id_medico = c.id_medico
  AND c.fecha_hora_cita >= TRUNC(SYSDATE, 'MM')
  AND c.fecha_hora_cita < ADD_MONTHS(TRUNC(SYSDATE, 'MM'), 1)
GROUP BY m.nombre_completo
ORDER BY citas DESC;

COMMIT; -- Liberar la transacción SERIALIZABLE
```

> 💡 **¿Por qué SERIALIZABLE para el Reporte B?** Si usáramos READ COMMITTED y alguien insertara una cita entre la consulta 2 y la 3, el total de citas no coincidiría con la suma de citas por médico. SERIALIZABLE congela la "foto" de los datos al inicio de la transacción, garantizando coherencia.

> ⚠️ **Cuidado:** no abuses de SERIALIZABLE. Puede causar errores `ORA-08177` si otra sesión modifica datos que tu transacción necesita actualizar. Úsalo para **reportes de lectura**, no para transacciones de escritura intensiva.

</details>

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 13**](../proyectos/proyecto_transacciones.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 14: DCL y Seguridad →**](../../14-dcl-seguridad/README.md)

</div>
