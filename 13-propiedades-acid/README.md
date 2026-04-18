# 🧪 Tema 13: Propiedades ACID

> **"Las propiedades ACID son el contrato sagrado que toda base de datos relacional firma con sus usuarios: tus datos siempre serán correctos, completos y confiables."** Entender ACID es entender por qué puedes confiar en Oracle (y en cualquier RDBMS serio) para manejar tu dinero, tu historial médico y tus reservas de vuelo.

## 📋 Índice

- [13.1 ¿Qué son las Propiedades ACID?](#131-qué-son-las-propiedades-acid)
- [13.2 Atomicidad — Todo o Nada](#132-atomicidad--todo-o-nada)
- [13.3 Consistencia — De Estado Válido a Estado Válido](#133-consistencia--de-estado-válido-a-estado-válido)
- [13.4 Aislamiento (Isolation) — Transacciones Independientes](#134-aislamiento-isolation--transacciones-independientes)
- [13.5 Durabilidad — Persistencia ante Fallos](#135-durabilidad--persistencia-ante-fallos)
- [13.6 Niveles de Aislamiento](#136-niveles-de-aislamiento)

---

---

## 13.1 ¿Qué son las Propiedades ACID?

### 📘 El Concepto

**ACID** es un acrónimo que describe las cuatro propiedades fundamentales que garantizan la fiabilidad de las transacciones en una base de datos relacional:

| Letra | Propiedad | Significado |
|:-----:|-----------|-------------|
| **A** | **Atomicidad** | La transacción se ejecuta **completamente** o **no se ejecuta en absoluto**. |
| **C** | **Consistencia** | La base de datos pasa de un **estado válido** a otro **estado válido**. |
| **I** | **Isolation (Aislamiento)** | Las transacciones concurrentes **no interfieren** entre sí. |
| **D** | **Durabilidad** | Una vez confirmados (`COMMIT`), los cambios **persisten** incluso ante fallos del sistema. |

Estas propiedades trabajan juntas como un equipo:

```
┌─────────────────────────────────────────────────────────────┐
│                    TRANSACCIÓN SQL                           │
│                                                             │
│  ATOMICIDAD ──── "Todo o nada"                              │
│       │                                                     │
│  CONSISTENCIA ── "Las reglas siempre se cumplen"            │
│       │                                                     │
│  AISLAMIENTO ─── "Cada transacción trabaja en privado"      │
│       │                                                     │
│  DURABILIDAD ─── "Lo confirmado es permanente"              │
│                                                             │
│  → COMMIT;   (se garantizan las 4 propiedades)              │
│  → ROLLBACK; (atomicidad entra en acción)                   │
└─────────────────────────────────────────────────────────────┘
```

> 📌 **Sin ACID, no podrías confiar en ningún sistema crítico:** bancos, hospitales, aerolíneas, e-commerce… Todos dependen de estas garantías.

### 🏠 La Analogía

Imagina que ACID es un **contrato de mudanza profesional**:

- **Atomicidad:** "Movemos TODOS tus muebles o ninguno. No dejamos la mitad en la calle."
- **Consistencia:** "Tu nueva casa quedará habitable: con cocina, baño y electricidad funcionando."
- **Aislamiento:** "Aunque estemos mudando a otra familia al mismo tiempo, sus muebles no se mezclan con los tuyos."
- **Durabilidad:** "Una vez entregado todo y firmado el recibo, los muebles son tuyos para siempre. Aunque la empresa de mudanzas quiebre mañana."

### 💻 El Código

```sql
-- Ejemplo práctico: procesar un pedido en e-commerce
-- Esta transacción debe cumplir las 4 propiedades ACID

-- ATOMICIDAD: las 3 operaciones se ejecutan juntas o ninguna
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (90, 1, 10, 1, SYSDATE, 1200.00);

UPDATE productos SET stock = stock - 1 WHERE id_producto = 10;

-- Simulación: verificar stock suficiente
-- Si stock < 0, haríamos ROLLBACK (atomicidad)

-- CONSISTENCIA: las constraints (FK, CHECK, NOT NULL) validan
-- que el cliente 1 existe, el producto 10 existe, cantidad > 0

-- AISLAMIENTO: otro usuario que consulte productos en este
-- momento ve el stock ANTERIOR (Oracle usa Read Committed)

-- DURABILIDAD: al hacer COMMIT, se graba en redo logs
COMMIT;
-- Ahora el pedido y el stock actualizado son permanentes

-- Verificar
SELECT p.id_pedido, pr.nombre, pr.stock
FROM pedidos p JOIN productos pr ON p.id_producto = pr.id_producto
WHERE p.id_pedido = 90;

-- Limpiar
DELETE FROM pedidos WHERE id_pedido = 90;
UPDATE productos SET stock = 100 WHERE id_producto = 10;
COMMIT;
```

### 🧠 El Reto

En nuestro sistema de hospital, un doctor agenda una cita y al mismo tiempo el sistema actualiza la disponibilidad. ¿Qué propiedad ACID garantiza que si el sistema se cae justo después del `COMMIT`, la cita no desaparezca? ¿Y cuál garantiza que si falla la actualización de disponibilidad, la inserción de la cita también se revierte?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

- **Durabilidad** garantiza que después del `COMMIT`, la cita persiste aunque el sistema se caiga.
- **Atomicidad** garantiza que si falla una parte de la transacción, toda la transacción se revierte.

```sql
-- Ejemplo de atomicidad protegiendo la operación:
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_cita, estado)
VALUES (50, 1, 1, TO_DATE('2024-08-01 09:00', 'YYYY-MM-DD HH24:MI'), 'P');

-- Si aquí ocurre un error en otra operación de la misma transacción:
ROLLBACK;
-- La cita con id 50 NO existirá — atomicidad protege la integridad
```

</details>

---

---

## 13.2 Atomicidad — Todo o Nada

### 📘 El Concepto

La **atomicidad** garantiza que una transacción es **indivisible**: o se ejecutan **todas** las operaciones que la componen, o no se ejecuta **ninguna**. No existe un estado intermedio.

Oracle implementa la atomicidad mediante:
- **Undo segments (segmentos de rollback):** almacenan la versión anterior de los datos modificados.
- **ROLLBACK:** utiliza los undo segments para restaurar el estado previo.
- **SAVEPOINT:** permite crear puntos de retorno parciales dentro de una transacción.

```
Transacción con 3 operaciones:
┌──────────────────────────────────────────┐
│  OP1: INSERT pedido     ✅ éxito        │
│  OP2: UPDATE stock      ✅ éxito        │
│  OP3: UPDATE saldo      ❌ error        │
│                                          │
│  → ROLLBACK automático de OP1, OP2, OP3 │
│  → Base de datos queda como antes        │
└──────────────────────────────────────────┘
```

### 🏠 La Analogía

Piensa en una **receta de cocina**. Si estás haciendo un pastel y a mitad de la preparación descubres que no tienes huevos, no puedes servir "medio pastel". Tiras toda la mezcla y empiezas de cero. La atomicidad es eso: el pastel se completa entero o no existe.

### 💻 El Código

```sql
-- Demostración de atomicidad con SAVEPOINT y ROLLBACK
-- Escenario: registrar un vuelo nuevo con pasos parciales

-- Paso 1: insertar ruta nueva
INSERT INTO rutas (id_ruta, origen, destino, distancia_km)
VALUES (20, 'Madrid', 'Tokio', 10500);

SAVEPOINT despues_ruta;

-- Paso 2: insertar vuelo en esa ruta
INSERT INTO vuelos (id_vuelo, id_avion, id_ruta, fecha_salida, fecha_llegada, 
                    plazas_disponibles, precio, estado)
VALUES (2010, 1, 20, TO_DATE('2024-12-01 23:00', 'YYYY-MM-DD HH24:MI'),
        TO_DATE('2024-12-02 17:00', 'YYYY-MM-DD HH24:MI'), 300, 850.00, 'P');

SAVEPOINT despues_vuelo;

-- Paso 3: ¡Error! Descubrimos que el avión 1 ya tiene un vuelo ese día
-- Revertimos solo el vuelo, pero mantenemos la ruta
ROLLBACK TO SAVEPOINT despues_ruta;

-- Verificar: la ruta existe pero el vuelo NO
SELECT * FROM rutas WHERE id_ruta = 20;
-- Resultado: Madrid → Tokio, 10500 km ✅

SELECT * FROM vuelos WHERE id_vuelo = 2010;
-- Resultado: no rows selected ✅ (se revirtió)

-- Ahora podemos asignar otro avión y reintentar
-- O revertir todo:
ROLLBACK;
-- Ahora ni la ruta ni el vuelo existen

-- Verificar
SELECT * FROM rutas WHERE id_ruta = 20;
-- Resultado: no rows selected (todo revertido)
```

### 🧠 El Reto

Analiza esta secuencia y determina qué datos quedan en la base al final:

```sql
INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria) 
VALUES (30, 'Monitor 4K', 450.00, 20, 1);

SAVEPOINT sp1;

INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria) 
VALUES (31, 'Teclado Mecánico', 120.00, 50, 1);

SAVEPOINT sp2;

INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria) 
VALUES (32, 'Ratón Gaming', 80.00, 75, 1);

ROLLBACK TO SAVEPOINT sp1;

INSERT INTO productos (id_producto, nombre, precio, stock, id_categoria) 
VALUES (33, 'Auriculares BT', 65.00, 100, 1);

COMMIT;
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Productos que quedan: 30 (Monitor 4K) y 33 (Auriculares BT).**

1. `INSERT 30` (Monitor 4K) — insertado ✅
2. `SAVEPOINT sp1` — marca de control
3. `INSERT 31` (Teclado Mecánico) — insertado ✅
4. `SAVEPOINT sp2` — marca de control
5. `INSERT 32` (Ratón Gaming) — insertado ✅
6. `ROLLBACK TO sp1` — **revierte 31 y 32** (todo después de sp1) ❌❌
7. `INSERT 33` (Auriculares BT) — insertado ✅
8. `COMMIT` — confirma 30 y 33

> 💡 `ROLLBACK TO sp1` también destruye `sp2`. Los savepoints posteriores al punto de restauración desaparecen.

```sql
-- Limpiar
DELETE FROM productos WHERE id_producto IN (30, 33);
COMMIT;
```

</details>

---

---

## 13.3 Consistencia — De Estado Válido a Estado Válido

### 📘 El Concepto

La **consistencia** garantiza que una transacción lleva la base de datos de un **estado válido** a otro **estado válido**. "Válido" significa que se cumplen **todas las reglas de integridad** definidas:

| Regla | Ejemplo | Qué protege |
|-------|---------|-------------|
| `PRIMARY KEY` | `id_pedido` único | No hay duplicados |
| `FOREIGN KEY` | `id_cliente` existe en `clientes` | Relaciones válidas |
| `NOT NULL` | `nombre` obligatorio | Datos completos |
| `CHECK` | `precio > 0` | Datos lógicos |
| `UNIQUE` | `email` no repetido | Unicidad de datos |
| **Triggers** | Validaciones personalizadas | Reglas de negocio |

Si una transacción intenta violar cualquiera de estas reglas, Oracle la **rechaza** y los datos no cambian.

```
Estado VÁLIDO                    Estado VÁLIDO
┌──────────┐    Transacción     ┌──────────┐
│ stock=100 │ ─────────────────→ │ stock=98 │  ✅ Consistente
│ pedidos=5 │    (venta de 2)   │ pedidos=6 │
└──────────┘                    └──────────┘

Estado VÁLIDO                    Estado INVÁLIDO
┌──────────┐    Transacción     ┌──────────┐
│ stock=100 │ ──────── ✗ ──────→ │ stock=-5 │  ❌ Rechazado
│ pedidos=5 │  (si CHECK stock≥0)│ pedidos=6 │
└──────────┘                    └──────────┘
```

### 🏠 La Analogía

Piensa en un **juego de ajedrez**. Cada movimiento debe seguir las reglas: un alfil solo se mueve en diagonal, no puedes mover dos piezas a la vez, no puedes dejar a tu rey en jaque. Si intentas un movimiento ilegal, el árbitro (Oracle) lo rechaza y el tablero vuelve al estado anterior. El tablero siempre está en una posición **legal**.

### 💻 El Código

```sql
-- Demostración de consistencia: las constraints protegen los datos

-- 1. PRIMARY KEY impide duplicados
INSERT INTO clientes (id_cliente, nombre_completo, email, ciudad)
VALUES (1, 'Duplicado', 'dup@test.com', 'Test');
-- ORA-00001: unique constraint violated
-- El cliente con id 1 ya existe → la BD permanece consistente

-- 2. FOREIGN KEY impide referencias inválidas
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (91, 999, 10, 1, SYSDATE, 100);
-- ORA-02291: integrity constraint violated - parent key not found
-- No existe el cliente 999 → rechazado

-- 3. NOT NULL impide datos incompletos
INSERT INTO medicos (id_medico, nombre_completo, id_especialidad)
VALUES (20, NULL, 1);
-- ORA-01400: cannot insert NULL into ("MEDICOS"."NOMBRE_COMPLETO")

-- 4. CHECK constraint en acción
-- Supongamos que tenemos un CHECK en stock:
-- ALTER TABLE productos ADD CONSTRAINT chk_stock CHECK (stock >= 0);

-- Intentar dejar stock negativo:
-- UPDATE productos SET stock = -5 WHERE id_producto = 10;
-- ORA-02290: check constraint (CHK_STOCK) violated

-- 5. Demostración completa: transacción que mantiene consistencia
-- Registrar un pedido correctamente
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (91, 1, 10, 1, SYSDATE, 1200.00);
-- ✅ Cliente 1 existe, producto 10 existe, cantidad > 0 → consistente

-- Verificar
SELECT p.id_pedido, c.nombre_completo, pr.nombre
FROM pedidos p
JOIN clientes c ON p.id_cliente = c.id_cliente
JOIN productos pr ON p.id_producto = pr.id_producto
WHERE p.id_pedido = 91;

-- Limpiar
ROLLBACK;
```

### 🧠 El Reto

¿Qué pasa si ejecutas esta secuencia? ¿Cuántas filas se insertan en `citas`?

```sql
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_cita, estado) 
VALUES (60, 1, 1, TO_DATE('2024-09-01 10:00', 'YYYY-MM-DD HH24:MI'), 'P');

INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_cita, estado) 
VALUES (61, 999, 1, TO_DATE('2024-09-01 11:00', 'YYYY-MM-DD HH24:MI'), 'P');

INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_cita, estado) 
VALUES (62, 2, 1, TO_DATE('2024-09-01 12:00', 'YYYY-MM-DD HH24:MI'), 'P');

COMMIT;
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Se insertan 2 citas: la 60 y la 62.** La cita 61 falla por FOREIGN KEY (el paciente 999 no existe), pero en Oracle cada sentencia es independiente. El error de la segunda sentencia **no revierte la primera ni impide la tercera**.

1. `INSERT 60` — ✅ paciente 1 existe
2. `INSERT 61` — ❌ ORA-02291: paciente 999 no existe (esta sentencia falla, las demás siguen)
3. `INSERT 62` — ✅ paciente 2 existe
4. `COMMIT` — confirma las citas 60 y 62

> 💡 **Importante:** en Oracle, un error en una sentencia individual no aborta toda la transacción (a diferencia de PostgreSQL con su comportamiento por defecto). Solo la sentencia errónea se revierte. Las demás siguen pendientes.

```sql
-- Limpiar
DELETE FROM citas WHERE id_cita IN (60, 62);
COMMIT;
```

</details>

---

---

## 13.4 Aislamiento (Isolation) — Transacciones Independientes

### 📘 El Concepto

El **aislamiento** garantiza que las transacciones concurrentes se ejecutan como si fueran **secuenciales**: una transacción no puede ver los cambios no confirmados de otra.

Oracle implementa el aislamiento usando **MVCC (Multi-Version Concurrency Control)**:

```
Sesión A (transacción abierta)          Sesión B (lectura)
─────────────────────────────           ──────────────────
UPDATE productos                        SELECT * FROM productos
SET precio = 1500                       WHERE id_producto = 10;
WHERE id_producto = 10;                 → Ve precio = 1200 (versión anterior)
-- precio pendiente: 1500              
-- (no ha hecho COMMIT)                -- Oracle lee del UNDO SEGMENT
                                        -- la versión anterior de la fila

COMMIT;                                 SELECT * FROM productos
                                        WHERE id_producto = 10;
                                        → Ahora ve precio = 1500
```

**Mecanismo interno de Oracle (MVCC):**
- Cuando la Sesión A modifica una fila, Oracle guarda la versión anterior en los **undo segments**.
- Cuando la Sesión B lee esa fila, Oracle le entrega la **versión del undo** (la anterior al cambio no confirmado).
- Esto permite que las **lecturas nunca bloqueen escrituras** y las **escrituras nunca bloqueen lecturas**.

> 📌 **Regla de oro de Oracle:** Los lectores no bloquean a los escritores. Los escritores no bloquean a los lectores. Solo los escritores bloquean a otros escritores (sobre la misma fila).

### 🏠 La Analogía

Imagina una **exposición de arte en un museo**. Mientras un artista está pintando un cuadro nuevo en su estudio (transacción abierta), los visitantes del museo ven la **última versión terminada** del cuadro en la galería. Solo cuando el artista firma la obra y la entrega al museo (COMMIT), los visitantes ven la nueva versión. Mientras tanto, el artista trabaja en privado sin molestar a nadie.

### 💻 El Código

```sql
-- SESIÓN A: modificar precio de Laptop Pro
UPDATE productos SET precio = 1500.00 WHERE id_producto = 10;
-- La transacción está abierta, NO hemos hecho COMMIT

-- ¿Qué ve la Sesión A?
SELECT nombre, precio FROM productos WHERE id_producto = 10;
-- Resultado Sesión A: Laptop Pro, 1500.00 (ve SU cambio pendiente)

-- ¿Qué ve la Sesión B (otra conexión)?
-- SELECT nombre, precio FROM productos WHERE id_producto = 10;
-- Resultado Sesión B: Laptop Pro, 1200.00 (ve la versión ANTERIOR)

-- Oracle logra esto usando undo segments (MVCC):
-- La versión 1200.00 se guardó en el undo segment cuando Sesión A
-- ejecutó el UPDATE. Sesión B lee esa versión antigua.

-- Si Sesión A confirma:
COMMIT;
-- Ahora AMBAS sesiones ven: Laptop Pro, 1500.00

-- Restaurar precio original
UPDATE productos SET precio = 1200.00 WHERE id_producto = 10;
COMMIT;

-- Otro ejemplo: aislamiento en el hospital
-- SESIÓN A: cambia el teléfono de un paciente
UPDATE pacientes SET telefono = '600000000' WHERE id_paciente = 1;

-- SESIÓN B intentando leer:
-- SELECT telefono FROM pacientes WHERE id_paciente = 1;
-- Resultado: ve el teléfono ANTERIOR (aislamiento)

-- SESIÓN B intentando escribir en la MISMA fila:
-- UPDATE pacientes SET telefono = '611111111' WHERE id_paciente = 1;
-- → La Sesión B ESPERA hasta que Sesión A haga COMMIT o ROLLBACK

ROLLBACK; -- Restaurar
```

### 🧠 El Reto

Dos agentes de viajes trabajan simultáneamente:

- **Agente A** ejecuta: `UPDATE vuelos SET plazas_disponibles = plazas_disponibles - 1 WHERE id_vuelo = 1001;` (sin COMMIT)
- **Agente B** ejecuta: `SELECT plazas_disponibles FROM vuelos WHERE id_vuelo = 1001;`
- **Agente B** luego ejecuta: `UPDATE vuelos SET plazas_disponibles = plazas_disponibles - 1 WHERE id_vuelo = 1001;`

¿Qué número ve el Agente B en el SELECT? ¿Qué pasa con su UPDATE?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

1. **El Agente B ve el valor ORIGINAL** en el SELECT (por ejemplo, 150 plazas). No ve la reducción del Agente A porque no ha hecho COMMIT (aislamiento MVCC).

2. **El UPDATE del Agente B se BLOQUEA.** Se queda esperando porque el Agente A tiene la fila bloqueada (row lock). El Agente B no puede modificar una fila que otra transacción ya está modificando.

3. **Cuando el Agente A haga COMMIT:**
   - El bloqueo se libera.
   - El UPDATE del Agente B se ejecuta, pero **sobre el valor ya actualizado** por A.
   - Si originalmente había 150 plazas: A resta 1 → 149. B resta 1 → 148.
   - Resultado final correcto: 148 plazas.

> 💡 Oracle garantiza que ambas restas se apliquen correctamente gracias al aislamiento + bloqueo de filas.

</details>

---

---

## 13.5 Durabilidad — Persistencia ante Fallos

### 📘 El Concepto

La **durabilidad** garantiza que una vez que una transacción ha sido confirmada con `COMMIT`, sus cambios **sobreviven** a cualquier fallo posterior: caídas de electricidad, errores de hardware, reinicios del servidor, etc.

Oracle implementa la durabilidad mediante un mecanismo llamado **Write-Ahead Logging (WAL)**:

```
                    Proceso de COMMIT en Oracle
                    ═══════════════════════════

1. Ejecutas: COMMIT;

2. Oracle escribe los cambios en los REDO LOG BUFFERS (memoria)
   ↓
3. El proceso LGWR (Log Writer) escribe los redo logs a DISCO
   ↓
4. Oracle confirma el COMMIT al usuario  ← solo aquí te dice "OK"
   ↓
5. Más tarde, el proceso DBWn escribe los datos reales a disco
   (esto puede tardar, pero los redo logs ya están seguros)

Si hay un fallo entre el paso 4 y el 5:
→ Al reiniciar, Oracle lee los REDO LOGS y RE-APLICA los cambios
→ Tus datos se recuperan automáticamente (Instance Recovery)
```

**Componentes clave:**
- **Redo Log Files:** archivos en disco que registran TODOS los cambios.
- **LGWR (Log Writer):** proceso que escribe redo logs a disco antes de confirmar el COMMIT.
- **SMON (System Monitor):** proceso que realiza la recuperación automática al reiniciar.

### 🏠 La Analogía

Piensa en un **notario público**. Cuando firmas un contrato de compraventa (COMMIT), el notario no solo te da una copia: **registra la escritura en el archivo oficial** (redo logs). Aunque el edificio del notario se incendie mañana, la escritura está registrada en los archivos centrales y se puede recuperar. Tu propiedad es legalmente tuya para siempre.

### 💻 El Código

```sql
-- La durabilidad se demuestra conceptualmente:
-- Oracle escribe en redo logs ANTES de confirmar el COMMIT

-- Paso 1: Insertar un pedido crítico
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (92, 3, 11, 5, SYSDATE, 225.00);

-- Paso 2: COMMIT — aquí Oracle escribe en redo logs a disco
COMMIT;
-- En este momento, aunque se apague el servidor,
-- el pedido 92 se recuperará automáticamente al reiniciar

-- Consultar los redo logs del sistema (solo lectura, requiere permisos DBA)
-- SELECT group#, bytes/1024/1024 AS size_mb, status 
-- FROM v$log;

-- Consultar los archivos de redo log
-- SELECT group#, member FROM v$logfile;

-- Ver el modo de log (ARCHIVELOG permite recuperación completa)
-- SELECT log_mode FROM v$database;
-- ARCHIVELOG = los redo logs se archivan → recuperación point-in-time
-- NOARCHIVELOG = los redo logs se reciclan → solo recuperación de instancia

-- Verificar que el pedido existe
SELECT * FROM pedidos WHERE id_pedido = 92;

-- Limpiar
DELETE FROM pedidos WHERE id_pedido = 92;
COMMIT;
```

### 🧠 El Reto

Un administrador de la aerolínea ejecuta estas sentencias:

```sql
INSERT INTO vuelos (id_vuelo, id_avion, id_ruta, fecha_salida, fecha_llegada, 
                    plazas_disponibles, precio, estado)
VALUES (2020, 2, 3, TO_DATE('2024-11-15 06:00', 'YYYY-MM-DD HH24:MI'),
        TO_DATE('2024-11-15 09:30', 'YYYY-MM-DD HH24:MI'), 180, 200.00, 'P');
COMMIT;

UPDATE vuelos SET precio = 250.00 WHERE id_vuelo = 2020;
-- ← AQUÍ SE CAE EL SERVIDOR (sin COMMIT del UPDATE)
```

Cuando el servidor se reinicia, ¿cuál es el precio del vuelo 2020?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**El precio es 200.00** (el original del INSERT).

1. El `INSERT` + `COMMIT` se ejecutaron correctamente → el vuelo 2020 existe con precio 200.00 (durabilidad: este COMMIT sobrevive al fallo).
2. El `UPDATE` a 250.00 **no tenía COMMIT** cuando el servidor se cayó.
3. Al reiniciar, Oracle ejecuta el **Instance Recovery**:
   - **Roll forward:** re-aplica los cambios de los redo logs que tenían COMMIT (el INSERT).
   - **Roll back:** revierte los cambios sin COMMIT (el UPDATE a 250.00) usando los undo segments.
4. Resultado: el vuelo existe con precio = 200.00.

> 💡 Esta es la diferencia fundamental entre `COMMIT` y no `COMMIT`. Sin COMMIT, los cambios son temporales y se pierden ante cualquier fallo.

```sql
-- Limpiar (si el vuelo existiera)
-- DELETE FROM vuelos WHERE id_vuelo = 2020;
-- COMMIT;
```

</details>

---

---

## 13.6 Niveles de Aislamiento

### 📘 El Concepto

El estándar SQL define **4 niveles de aislamiento**, cada uno con un equilibrio diferente entre **rendimiento** y **protección** contra anomalías de concurrencia:

| Nivel | Lectura Sucia | Lectura No Repetible | Lectura Fantasma | Rendimiento |
|-------|:-------------:|:--------------------:|:----------------:|:-----------:|
| **READ UNCOMMITTED** | ✅ Posible | ✅ Posible | ✅ Posible | ⚡ Máximo |
| **READ COMMITTED** | ❌ Impedido | ✅ Posible | ✅ Posible | ⚡⚡ Alto |
| **REPEATABLE READ** | ❌ Impedido | ❌ Impedido | ✅ Posible | ⚡ Medio |
| **SERIALIZABLE** | ❌ Impedido | ❌ Impedido | ❌ Impedido | 🐢 Bajo |

**Anomalías explicadas:**

| Anomalía | Descripción | Ejemplo |
|----------|-------------|---------|
| **Lectura sucia** (*Dirty Read*) | Leer datos no confirmados de otra transacción | Ves un precio que aún no tiene COMMIT y que podría revertirse |
| **Lectura no repetible** (*Non-repeatable Read*) | Leer la misma fila dos veces y obtener valores diferentes | Consultas el stock, otro usuario lo modifica y confirma, vuelves a consultar y cambió |
| **Lectura fantasma** (*Phantom Read*) | Ejecutar la misma consulta y obtener filas adicionales | Cuentas los pedidos del día, otro usuario inserta uno nuevo y confirma, vuelves a contar y hay uno más |

**Oracle solo soporta 2 niveles:**

| Nivel Oracle | Cómo se activa | Comportamiento |
|-------------|----------------|----------------|
| **READ COMMITTED** (por defecto) | `SET TRANSACTION ISOLATION LEVEL READ COMMITTED;` | Cada SELECT ve los datos confirmados en el momento de su ejecución |
| **SERIALIZABLE** | `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | Cada SELECT ve una "foto" de la BD al inicio de la transacción |

> 📌 Oracle **nunca** permite lecturas sucias. Ni siquiera en READ COMMITTED. Esto es gracias a MVCC (undo segments).

### 🏠 La Analogía

Los niveles de aislamiento son como **niveles de privacidad en una oficina compartida**:

- **READ UNCOMMITTED:** Oficina abierta sin paredes. Puedes ver lo que todos están escribiendo en sus pantallas, incluso borradores sin terminar. (Oracle NO permite esto.)
- **READ COMMITTED** (Oracle por defecto): Cada vez que miras al compañero, ves su **último documento publicado**. Si publican algo nuevo entre tus miradas, ves la nueva versión.
- **REPEATABLE READ:** Antes de empezar tu tarea, sacas una **fotocopia** de los documentos que necesitas. Aunque los originales cambien, tu fotocopia sigue igual. Pero si alguien **añade** un documento nuevo a la carpeta, lo verás.
- **SERIALIZABLE:** Te encierras en una **sala privada** con fotocopias de TODO. No ves ningún cambio de nadie hasta que sales de la sala. Es como si trabajaras completamente solo.

### 💻 El Código

```sql
-- ═══════════════════════════════════════════
-- READ COMMITTED (comportamiento por defecto)
-- ═══════════════════════════════════════════

-- Sesión A: transacción en READ COMMITTED
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT AVG(precio) FROM productos;
-- Resultado: digamos 382.37 (promedio actual)

-- Mientras tanto, Sesión B ejecuta:
-- UPDATE productos SET precio = 2000.00 WHERE id_producto = 10;
-- COMMIT;

-- Sesión A vuelve a ejecutar la misma consulta:
SELECT AVG(precio) FROM productos;
-- Resultado: 482.37 (diferente! — ve el COMMIT de Sesión B)
-- Esto es una "lectura no repetible" — permitida en READ COMMITTED

COMMIT; -- Termina la transacción de Sesión A

-- ═══════════════════════════════════════════
-- SERIALIZABLE (foto congelada)
-- ═══════════════════════════════════════════

-- Sesión A: transacción SERIALIZABLE
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT AVG(precio) FROM productos;
-- Resultado: 382.37

-- Sesión B ejecuta:
-- UPDATE productos SET precio = 2000.00 WHERE id_producto = 10;
-- COMMIT;

-- Sesión A repite la consulta:
SELECT AVG(precio) FROM productos;
-- Resultado: 382.37 (¡IGUAL! — no ve el COMMIT de Sesión B)
-- La foto de datos se congeló al inicio de la transacción

COMMIT; -- Termina la transacción

-- ═══════════════════════════════════════════
-- Limitación de SERIALIZABLE: ORA-08177
-- ═══════════════════════════════════════════

-- Si en modo SERIALIZABLE intentas modificar una fila que otra
-- sesión ya modificó y confirmó desde que empezó tu transacción:

-- SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- UPDATE productos SET precio = 1300 WHERE id_producto = 10;
-- (si Sesión B ya cambió y confirmó el precio de este producto)
-- ORA-08177: can't serialize access for this transaction
-- → Tu transacción se aborta. Debes reintentar.
```

### 🧠 El Reto

Un auditor necesita generar un reporte del hospital que consulte el número total de pacientes, la cantidad de citas pendientes y el médico con más citas. El reporte debe ser **consistente**: si otra persona inserta una cita mientras el reporte se genera, NO debe aparecer en ninguna de las 3 consultas.

¿Qué nivel de aislamiento debe usar? Escribe el código.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

Debe usar **SERIALIZABLE** para que todas las consultas vean la misma "foto" de los datos:

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Consulta 1: total de pacientes
SELECT COUNT(*) AS total_pacientes FROM pacientes;

-- Consulta 2: citas pendientes
SELECT COUNT(*) AS citas_pendientes FROM citas WHERE estado = 'P';

-- Consulta 3: médico con más citas
SELECT m.nombre_completo, COUNT(c.id_cita) AS total_citas
FROM medicos m
JOIN citas c ON m.id_medico = c.id_medico
GROUP BY m.nombre_completo
ORDER BY total_citas DESC
FETCH FIRST 1 ROW ONLY;

COMMIT; -- Libera la transacción SERIALIZABLE
```

> 💡 Con `READ COMMITTED`, si alguien inserta una cita entre la consulta 1 y la consulta 2, la consulta 2 vería esa nueva cita pero la consulta 1 no. El reporte sería inconsistente. Con `SERIALIZABLE`, las 3 consultas ven exactamente el mismo estado de los datos.

</details>

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 13

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de Propiedades ACID](ejercicios/ejercicios_acid.md) |
| 3️⃣ | Avanzar al siguiente tema | ➡️ [Tema 14: Normalización](../14-normalizacion) |

---

<div align="center">

⬅️ [**Tema 12: Transacciones**](../12-transacciones) · 🏠 [**Índice del Curso**](../README.md) · [**Tema 14: Normalización →**](../14-normalizacion)

</div>
