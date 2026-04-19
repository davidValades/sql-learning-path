# 🛡️ Tema 13: Transacciones y Propiedades ACID

> **"En una base de datos, o todo sale bien, o nada cambia."** Las transacciones son el mecanismo que garantiza la integridad de los datos: si una operación falla a mitad de camino, puedes revertir **todo** como si nada hubiera pasado. Son la red de seguridad del mundo SQL.

## 📋 Índice

- [13.1 ¿Qué es una Transacción?](#131-qué-es-una-transacción)
- [13.2 COMMIT](#132-commit)
- [13.3 ROLLBACK](#133-rollback)
- [13.4 SAVEPOINT](#134-savepoint)
- [13.5 Autocommit vs. Transacciones Explícitas](#135-autocommit-vs-transacciones-explícitas)
- [13.6 Bloqueos y Concurrencia](#136-bloqueos-y-concurrencia)
- [Parte B: Propiedades ACID](#-parte-b-propiedades-acid)

---

---

## 13.1 ¿Qué es una Transacción?

### 📘 El Concepto

Una **transacción** es una **unidad atómica de trabajo** compuesta por una o más sentencias SQL que deben ejecutarse **todas o ninguna**. Si alguna falla, se deshacen todos los cambios realizados hasta ese momento.

En Oracle, una transacción comienza implícitamente con la primera sentencia DML (`INSERT`, `UPDATE`, `DELETE`) y termina con:
- `COMMIT` — confirma todos los cambios.
- `ROLLBACK` — revierte todos los cambios.
- Una sentencia DDL (`CREATE`, `ALTER`, `DROP`) — hace **auto-commit** implícito.
- Fin de la sesión — `COMMIT` implícito si la sesión termina normalmente, `ROLLBACK` si termina anormalmente.

```
Transacción
┌──────────────────────────────────────────┐
│  INSERT INTO pedidos ...;                │
│  UPDATE productos SET stock = stock - 1; │
│  INSERT INTO log_actividad ...;          │
│                                          │
│  → COMMIT;   (todo se confirma)          │
│  → ROLLBACK; (todo se revierte)          │
└──────────────────────────────────────────┘
```

### 🏠 La Analogía

Piensa en una **transferencia bancaria**. Cuando transfieres 500€ de tu cuenta a la de un amigo, hay dos operaciones:
1. Restar 500€ de tu cuenta.
2. Sumar 500€ a la cuenta de tu amigo.

Si la operación 1 se ejecuta pero la 2 falla (por ejemplo, se cae el sistema), **desaparecerían 500€**. Una transacción garantiza que ambas operaciones se ejecutan juntas. Si la segunda falla, la primera se revierte automáticamente. Tu dinero nunca desaparece.

### 💻 El Código

```sql
-- Ejemplo: procesar un pedido completo
-- Paso 1: insertar el pedido
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (50, 1, 11, 2, SYSDATE, 91.00);

-- Paso 2: reducir el stock del producto
UPDATE productos SET stock = stock - 2 WHERE id_producto = 11;

-- Paso 3: todo salió bien → confirmar
COMMIT;
-- Ahora ambos cambios son permanentes

-- Si algo hubiera fallado en el paso 2:
-- ROLLBACK;
-- El INSERT del paso 1 también se revertiría

-- Limpiar el ejemplo para mantener el estado original
DELETE FROM pedidos WHERE id_pedido = 50;
UPDATE productos SET stock = 200 WHERE id_producto = 11;
COMMIT;
```

### 🧠 El Reto

¿Qué pasa si ejecutas estas sentencias en orden?

```sql
INSERT INTO pacientes (id_paciente, nombre, telefono) VALUES (10, 'Test', '000');
CREATE TABLE prueba (id NUMBER);    -- ← sentencia DDL
ROLLBACK;
SELECT * FROM pacientes WHERE id_paciente = 10;
```

¿El paciente con id 10 existe después del `ROLLBACK`?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Sí, el paciente existe.** El `CREATE TABLE` es una sentencia DDL que provoca un **auto-commit implícito** antes de ejecutarse. Por lo tanto:

1. `INSERT` — inserta el paciente (transacción abierta).
2. `CREATE TABLE` — Oracle hace `COMMIT` implícito antes del DDL → el INSERT se confirma permanentemente.
3. `ROLLBACK` — no hay nada que revertir, la transacción ya se confirmó.

> 💡 **Cuidado con los DDL en medio de transacciones.** Cualquier `CREATE`, `ALTER` o `DROP` confirma implícitamente todos los cambios pendientes. Es un error común y peligroso.

```sql
-- Limpiar
DELETE FROM pacientes WHERE id_paciente = 10;
DROP TABLE prueba;
COMMIT;
```

</details>

---

---

## 13.2 COMMIT

### 📘 El Concepto

`COMMIT` **confirma permanentemente** todos los cambios realizados en la transacción actual. Una vez hecho, los cambios:
- Son **visibles** para todas las demás sesiones.
- Son **permanentes** — no se pueden revertir.
- Se **escriben en disco** (en los redo logs de Oracle).

```
COMMIT;
-- O con comentario de auditoría:
COMMIT COMMENT 'Procesamiento de pedido #50 completado';
```

**Antes del COMMIT:**
- Los cambios solo son visibles para **tu sesión**.
- Otras sesiones ven los datos **anteriores** (aislamiento de lectura).
- Los cambios se pueden deshacer con `ROLLBACK`.

**Después del COMMIT:**
- Los cambios son visibles para **todos**.
- No hay vuelta atrás.

### 🏠 La Analogía

Imagina que estás editando un documento compartido. Mientras escribes, tus cambios están en un "borrador" que solo tú ves. Tus compañeros ven la versión anterior. Cuando haces clic en **"Guardar y publicar"** (COMMIT), todos ven tus cambios y no puedes deshacerlos. Si cierras sin guardar (ROLLBACK), el documento vuelve a la versión anterior.

### 💻 El Código

```sql
-- Sesión 1: insertar un nuevo producto
INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria)
VALUES (20, 'Webcam HD', 55.00, 30, 1);

-- En este momento, solo Sesión 1 ve el producto 20
-- Sesión 2 ejecutando: SELECT * FROM productos WHERE id_producto = 20;
-- Resultado en Sesión 2: no rows selected

COMMIT;

-- Ahora Sesión 2 también ve el producto 20
-- Sesión 2 ejecutando: SELECT * FROM productos WHERE id_producto = 20;
-- Resultado: Webcam HD, 55.00, 30, Electrónica

-- Limpiar ejemplo
DELETE FROM productos WHERE id_producto = 20;
COMMIT;

-- Hospital: confirmar una nueva cita
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado)
VALUES (20, 1, 7, TO_DATE('2024-06-01 10:00', 'YYYY-MM-DD HH24:MI'), 'P');

COMMIT;  -- La cita es oficial y visible para todo el sistema

-- Limpiar ejemplo
DELETE FROM citas WHERE id_cita = 20;
COMMIT;
```

### 🧠 El Reto

En una sesión, ejecutas:

```sql
UPDATE medicos SET salario_base = 5000 WHERE id_medico = 7;
```

Otro usuario, en una sesión diferente, ejecuta al mismo tiempo:

```sql
SELECT salario_base FROM medicos WHERE id_medico = 7;
```

¿Qué valor ve el otro usuario? ¿Y después de que hagas `COMMIT`?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

- **Antes del COMMIT:** El otro usuario ve **2600** (el valor original). Oracle usa **lectura consistente**: cada sesión ve los datos como estaban al inicio de su consulta, sin ver cambios no confirmados de otras sesiones.
- **Después del COMMIT:** El otro usuario ve **5000** en su siguiente consulta.

```sql
-- Revertir el cambio para mantener el estado original
UPDATE medicos SET salario_base = 2600 WHERE id_medico = 7;
COMMIT;
```

> 💡 Este comportamiento se llama **aislamiento de lectura** y es fundamental para evitar que un usuario vea datos "a medias" de otra transacción.

</details>

---

---

## 13.3 ROLLBACK

### 📘 El Concepto

`ROLLBACK` **revierte todos los cambios** realizados desde el último `COMMIT` (o desde el inicio de la transacción si no ha habido ningún COMMIT).

```
ROLLBACK;
```

**Después del ROLLBACK:**
- Todos los `INSERT`, `UPDATE` y `DELETE` de la transacción actual se deshacen.
- Los datos vuelven al estado del último `COMMIT`.
- Los valores de secuencia (`NEXTVAL`) **NO se revierten** — esos números se pierden.

### 🏠 La Analogía

`ROLLBACK` es como pulsar **Ctrl+Z** en un editor de texto, pero a lo grande: deshace **todo** lo que has hecho desde la última vez que guardaste (COMMIT). No importa si escribiste una línea o cien páginas — todo desaparece y el documento vuelve a su estado guardado anterior.

### 💻 El Código

```sql
-- E-commerce: un pedido que sale mal
-- Verificar estado inicial
SELECT stock FROM productos WHERE id_producto = 12;
-- Resultado: 25

-- Inicio de transacción implícita
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (60, 2, 12, 1, SYSDATE, 350.00);

UPDATE productos SET stock = stock - 1 WHERE id_producto = 12;

-- ¡Detectamos un error! El cliente 2 tiene la tarjeta rechazada
-- Revertir todo
ROLLBACK;

-- Verificar: el stock no cambió
SELECT stock FROM productos WHERE id_producto = 12;
-- Resultado: 25 (sin cambios)

-- El pedido 60 tampoco existe
SELECT * FROM pedidos WHERE id_pedido = 60;
-- Resultado: no rows selected

-- Hospital: cita creada por error
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado)
VALUES (30, 3, 2, TO_DATE('2024-04-01 08:00', 'YYYY-MM-DD HH24:MI'), 'P');

-- Ups, el médico 2 está de vacaciones ese día
ROLLBACK;
-- La cita 30 nunca existió

-- Aerolínea: cambio de avión abortado
UPDATE vuelos SET id_avion = 20 WHERE id_vuelo = 1001;
-- Cambiar de Airbus A350 a Boeing 737

-- Nos damos cuenta de que el Boeing 737 no está certificado para esa ruta
ROLLBACK;
-- El vuelo 1001 sigue con el Airbus A350
```

### 🧠 El Reto

¿Qué valor tiene el stock del Ratón Inalámbrico (id 11) después de este bloque?

```sql
UPDATE productos SET stock = 50 WHERE id_producto = 11;
UPDATE productos SET stock = 25 WHERE id_producto = 11;
COMMIT;
UPDATE productos SET stock = 0 WHERE id_producto = 11;
ROLLBACK;
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**El stock es 25.**

1. `UPDATE stock = 50` — stock pasa a 50 (no confirmado).
2. `UPDATE stock = 25` — stock pasa a 25 (no confirmado).
3. `COMMIT` — se confirma stock = **25** permanentemente.
4. `UPDATE stock = 0` — stock pasa a 0 (no confirmado).
5. `ROLLBACK` — revierte al último COMMIT → stock vuelve a **25**.

```sql
-- Restaurar valor original
UPDATE productos SET stock = 100 WHERE id_producto = 11;
COMMIT;
```

> 💡 El `ROLLBACK` solo revierte hasta el último `COMMIT`. Todo lo que se confirmó con `COMMIT` es irreversible.

</details>

---

---

## 13.4 SAVEPOINT

### 📘 El Concepto

Un `SAVEPOINT` es un **punto de control intermedio** dentro de una transacción. Te permite hacer `ROLLBACK` parcial — revertir solo hasta el savepoint en vez de deshacer toda la transacción.

```
SAVEPOINT nombre_punto;

-- ... más operaciones ...

-- Revertir solo hasta el savepoint (mantiene lo anterior)
ROLLBACK TO SAVEPOINT nombre_punto;

-- O revertir todo (ignora savepoints)
ROLLBACK;
```

**Esquema visual:**

```
COMMIT anterior
    │
    ├── INSERT A
    ├── SAVEPOINT sp1    ◄── punto de recuperación
    ├── UPDATE B
    ├── INSERT C
    ├── SAVEPOINT sp2    ◄── otro punto
    ├── DELETE D
    │
    ├── ROLLBACK TO sp2  → deshace DELETE D, mantiene A, B, C
    ├── ROLLBACK TO sp1  → deshace B, C, D, mantiene solo A
    ├── ROLLBACK         → deshace TODO (A, B, C, D)
    └── COMMIT           → confirma todo lo que quede
```

### 🏠 La Analogía

Imagina que estás escalando una montaña y vas clavando **banderines de seguridad** (savepoints) en puntos estratégicos. Si resbalas, no caes hasta abajo — caes solo hasta el último banderín. Puedes elegir a qué banderín volver: al más reciente o a uno anterior. Pero si decides bajar del todo (ROLLBACK completo), vuelves al campamento base.

### 💻 El Código

```sql
-- E-commerce: procesamiento de múltiples pedidos con puntos de control

-- Pedido 1: exitoso
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (70, 1, 16, 3, SYSDATE, 59.97);
UPDATE productos SET stock = stock - 3 WHERE id_producto = 16;

SAVEPOINT pedido1_ok;

-- Pedido 2: exitoso
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (71, 2, 17, 1, SYSDATE, 89.50);
UPDATE productos SET stock = stock - 1 WHERE id_producto = 17;

SAVEPOINT pedido2_ok;

-- Pedido 3: ¡falla! (stock insuficiente para Sofá de Cuero)
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (72, 3, 14, 1, SYSDATE, 405.00);
-- Sofá tiene stock = 0, no deberíamos haberlo vendido

-- Revertir SOLO el pedido 3, mantener pedidos 1 y 2
ROLLBACK TO SAVEPOINT pedido2_ok;
-- Pedido 72 desapareció, pero 70 y 71 siguen intactos

-- Confirmar pedidos 1 y 2
COMMIT;

-- Limpiar ejemplo
DELETE FROM pedidos WHERE id_pedido IN (70, 71);
UPDATE productos SET stock = 500 WHERE id_producto = 16;
UPDATE productos SET stock = 75 WHERE id_producto = 17;
COMMIT;

-- Hospital: registro de citas con savepoint
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado)
VALUES (40, 1, 4, TO_DATE('2024-05-10 09:00', 'YYYY-MM-DD HH24:MI'), 'P');

SAVEPOINT cita_registrada;

-- Intentar actualizar el teléfono de un paciente (operación secundaria)
UPDATE pacientes SET telefono = '699000001' WHERE id_paciente = 4;
-- ¡Error detectado! No era el teléfono correcto

ROLLBACK TO SAVEPOINT cita_registrada;
-- El teléfono de Ana Ruiz sigue como estaba (NULL)
-- Pero la cita 40 sigue registrada

COMMIT;  -- Confirmar solo la cita

-- Limpiar
DELETE FROM citas WHERE id_cita = 40;
COMMIT;
```

### 🧠 El Reto

¿Cuáles de estos vuelos existen en la tabla después de ejecutar el bloque completo?

```sql
INSERT INTO vuelos VALUES (2001, 'MADLHR', 30, SYSDATE, 'B12');
SAVEPOINT sp_a;
INSERT INTO vuelos VALUES (2002, 'JFKLAX', 50, SYSDATE, 'A01');
SAVEPOINT sp_b;
INSERT INTO vuelos VALUES (2003, 'BCNCDG', 40, SYSDATE, 'C05');
ROLLBACK TO SAVEPOINT sp_a;
INSERT INTO vuelos VALUES (2004, 'MADFRA', 30, SYSDATE, 'B15');
COMMIT;
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Vuelos que existen: 2001 y 2004.**

1. `INSERT 2001` — insertado ✅
2. `SAVEPOINT sp_a` — marca de control
3. `INSERT 2002` — insertado ✅
4. `SAVEPOINT sp_b` — marca de control
5. `INSERT 2003` — insertado ✅
6. `ROLLBACK TO sp_a` — **revierte 2002 y 2003** (todo después de sp_a) ❌❌
7. `INSERT 2004` — insertado ✅
8. `COMMIT` — confirma 2001 y 2004

> 💡 `ROLLBACK TO sp_a` elimina también el `sp_b` — los savepoints posteriores al punto de restauración dejan de existir.

</details>

---

---

## 13.5 Autocommit vs. Transacciones Explícitas

### 📘 El Concepto

| Modo | Comportamiento | Herramientas |
|------|---------------|-------------|
| **Autocommit ON** | Cada sentencia DML se confirma automáticamente al ejecutarse. No hay forma de hacer `ROLLBACK`. | MySQL por defecto, muchos clientes SQL |
| **Autocommit OFF** | Las sentencias se acumulan en una transacción abierta hasta que hagas `COMMIT` o `ROLLBACK` manualmente. | **Oracle por defecto**, PostgreSQL |

> 📌 **Oracle NO tiene autocommit por defecto.** Cada sentencia DML abre una transacción que permanece abierta hasta que haces `COMMIT` o `ROLLBACK`. Esto es más seguro pero requiere disciplina.

**Excepciones que causan auto-commit en Oracle:**
- Sentencias DDL: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.
- Sentencias DCL: `GRANT`, `REVOKE`.
- Desconexión normal de la sesión.

> ⚠️ **TRUNCATE vs DELETE:** `TRUNCATE TABLE` es DDL (auto-commit, no se puede revertir). `DELETE FROM` es DML (parte de la transacción, se puede revertir con ROLLBACK).

### 🏠 La Analogía

- **Autocommit ON** es como un bolígrafo que escribe directamente en tinta permanente. Cada letra que escribes queda para siempre. Si te equivocas, no puedes borrar.
- **Autocommit OFF** (Oracle) es como escribir con lápiz primero. Puedes borrar (ROLLBACK) las veces que quieras. Solo cuando estás seguro, repasas con bolígrafo (COMMIT) y queda permanente.

### 💻 El Código

```sql
-- Oracle: autocommit OFF por defecto
-- Podemos verificar y cambiar en SQL*Plus:
-- SHOW AUTOCOMMIT;    → autocommit OFF
-- SET AUTOCOMMIT ON;  → cada DML se confirma inmediatamente
-- SET AUTOCOMMIT OFF; → volver al comportamiento normal

-- Demostración de por qué autocommit OFF es más seguro:
DELETE FROM productos WHERE precio < 50;
-- ¡Ups! Eliminé 3 productos sin querer

-- Con autocommit OFF:
ROLLBACK;
-- ¡Salvado! Los 3 productos siguen ahí

-- Con autocommit ON, no habría forma de recuperarlos

-- DDL causa auto-commit (¡peligro!)
INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria)
VALUES (99, 'Producto Temporal', 10.00, 5, 1);
-- La transacción está abierta...

-- Esta sentencia DDL provoca auto-commit:
CREATE TABLE temp_prueba (id NUMBER);
-- El INSERT de arriba YA se confirmó permanentemente

ROLLBACK;  -- No revierte nada, el COMMIT ya ocurrió

-- Limpiar
DELETE FROM productos WHERE id_producto = 99;
DROP TABLE temp_prueba;
COMMIT;
```

### 🧠 El Reto

Un desarrollador ejecuta:

```sql
SET AUTOCOMMIT OFF;
INSERT INTO clientes VALUES (10, 'Test', 'test@test.com', '000');
DELETE FROM productos WHERE id_producto = 16;
-- En este punto, el servidor se apaga inesperadamente
```

¿Se guardó el INSERT? ¿Se guardó el DELETE?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Ninguno de los dos se guardó.**

Con autocommit OFF, ambas sentencias forman parte de una transacción abierta que **nunca recibió COMMIT**. Cuando el servidor se apaga inesperadamente (crash), Oracle ejecuta un **ROLLBACK automático** al reiniciar (proceso de recovery). Todos los cambios no confirmados se revierten.

> 💡 Esta es exactamente la razón por la que las transacciones existen: protegen contra fallos del sistema. Solo los datos confirmados con `COMMIT` sobreviven a un crash.

</details>

---

---

## 13.6 Bloqueos y Concurrencia

### 📘 El Concepto

Cuando múltiples usuarios modifican datos simultáneamente, Oracle usa **bloqueos (locks)** para evitar conflictos.

**Tipos de bloqueo en Oracle:**

| Bloqueo | Nivel | Cuándo ocurre |
|---------|-------|---------------|
| **Row Lock** (TX) | Fila | Al hacer `UPDATE` o `DELETE` de una fila |
| **Table Lock** (TM) | Tabla | Al hacer DDL o ciertos DML masivos |

**Comportamiento de los bloqueos:**
- Si la Sesión A hace `UPDATE` de la fila 5 (sin COMMIT), la fila queda bloqueada.
- Si la Sesión B intenta hacer `UPDATE` de la **misma fila 5**, la Sesión B **espera** hasta que la Sesión A haga `COMMIT` o `ROLLBACK`.
- Si la Sesión B consulta la fila 5 con `SELECT`, **no espera** — ve la versión anterior (lectura consistente).

**Deadlock (interbloqueo):**

Un deadlock ocurre cuando dos sesiones se bloquean mutuamente:

```
Sesión A: bloquea fila 1, necesita fila 2
Sesión B: bloquea fila 2, necesita fila 1
→ Ambas esperan eternamente → Oracle detecta el deadlock
  y aborta una de las transacciones con ORA-00060
```

### 🏠 La Analogía

Piensa en un **baño público con un solo inodoro** (la fila de datos). Si alguien entra y cierra con pestillo (hace UPDATE sin COMMIT), tú tienes que esperar en la cola. Cuando sale (COMMIT o ROLLBACK), puedes entrar.

Un **deadlock** es como dos personas en un pasillo estrecho que vienen en sentidos opuestos: ninguna quiere retroceder y ambas se quedan bloqueadas. Oracle actúa como un guardia que obliga a una de ellas a retroceder (ROLLBACK).

### 💻 El Código

```sql
-- SESIÓN A: actualizar precio del Laptop Pro
UPDATE productos SET precio = 1300.00 WHERE id_producto = 10;
-- La fila 10 está ahora BLOQUEADA por Sesión A
-- Sesión A no ha hecho COMMIT todavía

-- SESIÓN B (simultáneamente): intentar actualizar la misma fila
-- UPDATE productos SET precio = 1100.00 WHERE id_producto = 10;
-- → La Sesión B ESPERA (se queda "colgada")
-- No hay error, simplemente espera

-- SESIÓN A: confirmar sus cambios
COMMIT;
-- → La Sesión B se desbloquea y ejecuta su UPDATE
-- → El precio final es 1100.00 (el de Sesión B)

-- Restaurar
UPDATE productos SET precio = 1220.00 WHERE id_producto = 10;
COMMIT;

-- Consultar sesiones bloqueadas (como DBA)
-- SELECT blocking_session, sid, serial#, wait_class
-- FROM v$session
-- WHERE blocking_session IS NOT NULL;

-- Ejemplo de cómo EVITAR deadlocks:
-- Regla: siempre bloquear recursos en el MISMO ORDEN
-- Si necesitas modificar productos y pedidos:
-- 1. Primero UPDATE productos (tabla "mayor")
-- 2. Luego UPDATE pedidos (tabla "menor")
-- Si todas las sesiones siguen este orden, no hay deadlocks

-- Timeout para no esperar indefinidamente (Oracle 11g+):
-- ALTER SESSION SET ddl_lock_timeout = 10;  -- espera máximo 10 segundos
```

### 🧠 El Reto

Dos recepcionistas del hospital intentan actualizar la misma cita al mismo tiempo:

- **Recepcionista A:** `UPDATE citas SET estado = 'C' WHERE id_cita = 3;`
- **Recepcionista B:** `UPDATE citas SET estado = 'X' WHERE id_cita = 3;`

Si A ejecuta primero (sin COMMIT), ¿qué le pasa a B? ¿Cuál es el estado final de la cita si A hace COMMIT y luego B hace COMMIT?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

1. **B espera.** Al intentar actualizar la misma fila que A tiene bloqueada, la Sesión B se queda en espera (no error, solo espera).

2. **Secuencia:**
   - A ejecuta UPDATE → estado = 'C' (no confirmado, fila bloqueada)
   - B ejecuta UPDATE → **espera** (la fila está bloqueada por A)
   - A hace COMMIT → estado = 'C' confirmado, fila desbloqueada
   - B se desbloquea → su UPDATE se ejecuta: estado = 'X'
   - B hace COMMIT → **estado final = 'X'**

3. **El estado final es 'X'** — el UPDATE de B "gana" porque se ejecutó después del COMMIT de A.

> 💡 Este es el problema de la "última escritura gana" (*last write wins*). Para evitarlo, las aplicaciones usan **control de concurrencia optimista** (verificar que los datos no cambiaron antes de actualizar).

</details>

---

## 📚 Parte B: Propiedades ACID

> Para el contenido completo sobre las Propiedades ACID (Atomicidad, Consistencia, Aislamiento y Durabilidad), consulta los ejercicios específicos.

📝 [Ejercicios de Propiedades ACID](ejercicios/ejercicios_acid.md)

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 13

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios de Transacciones | 📝 [Ejercicios de Transacciones](ejercicios/ejercicios_transacciones.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Proyecto: El Día del Desastre](proyectos/proyecto_transacciones.md) |
| 4️⃣ | Practicar ejercicios de ACID | 📝 [Ejercicios de Propiedades ACID](ejercicios/ejercicios_acid.md) |
| 5️⃣ | Avanzar al siguiente tema | ➡️ [Tema 14: DCL y Seguridad](../14-dcl-seguridad) |

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 12**](../12-indexacion/proyectos/proyecto_indexacion.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 13 →**](ejercicios/ejercicios_transacciones.md)

</div>
