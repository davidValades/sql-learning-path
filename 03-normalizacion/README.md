# 📐 Tema 03: Normalización y Modelado ER

> **"Una base de datos mal diseñada es como una casa con los cimientos torcidos: por fuera puede parecer bonita, pero tarde o temprano se agrieta."** La normalización es el arte de organizar tus tablas para eliminar redundancias, evitar anomalías y construir bases de datos sólidas.

## 📋 Índice

- [3.1 ¿Qué es la Normalización y por qué Importa?](#31-qué-es-la-normalización-y-por-qué-importa)
- [3.2 Primera Forma Normal (1NF)](#32-primera-forma-normal-1nf)
- [3.3 Segunda Forma Normal (2NF)](#33-segunda-forma-normal-2nf)
- [3.4 Tercera Forma Normal (3NF)](#34-tercera-forma-normal-3nf)
- [3.5 Forma Normal de Boyce-Codd (BCNF)](#35-forma-normal-de-boyce-codd-bcnf)
- [3.6 Desnormalización: Cuándo y por qué Romper las Reglas](#36-desnormalización-cuándo-y-por-qué-romper-las-reglas)
- [3.7 Modelado Entidad-Relación (ER)](#37-modelado-entidad-relación-er)

---

---

## 3.1 ¿Qué es la Normalización y por qué Importa?

### 📘 El Concepto

La **normalización** es el proceso de organizar las tablas y columnas de una base de datos para:

1. **Eliminar redundancia** — no almacenar el mismo dato en múltiples lugares.
2. **Evitar anomalías** — problemas que surgen al insertar, actualizar o eliminar datos.
3. **Garantizar integridad** — cada dato tiene un único "dueño" y un único lugar.

**Las 3 anomalías que la normalización previene:**

| Anomalía | Problema | Ejemplo |
|----------|----------|---------|
| **Inserción** | No puedes añadir datos sin información innecesaria | No puedes registrar una categoría nueva sin tener un producto |
| **Actualización** | Cambiar un dato obliga a actualizarlo en muchos sitios | Si "Electrónica" cambia a "Tech", hay que actualizar 50 filas |
| **Eliminación** | Borrar un dato destruye información no relacionada | Borrar el último producto de "Hogar" elimina la categoría |

**Ejemplo concreto — tabla MAL diseñada (sin normalizar):**

```
PEDIDOS_COMPLETOS (todo en una tabla)
┌──────────┬─────────────┬──────────────┬──────────┬────────────┬────────┬───────┐
│ id_pedido│ cliente_nombre│ cliente_email│ producto │ categoria  │ precio │ cant. │
├──────────┼─────────────┼──────────────┼──────────┼────────────┼────────┼───────┤
│ 1        │ Ana García   │ ana@mail.com │ Laptop   │ Electrónica│ 1200   │ 1     │
│ 2        │ Ana García   │ ana@mail.com │ Ratón    │ Electrónica│ 45.50  │ 2     │
│ 3        │ Pedro Ruiz   │ pedro@mail.com│ Laptop  │ Electrónica│ 1200   │ 1     │
└──────────┴─────────────┴──────────────┴──────────┴────────────┴────────┴───────┘
  ↑ "Ana García" repetido 2 veces
  ↑ "Electrónica" repetido 3 veces
  ↑ "Laptop/1200" repetido 2 veces
```

**Mismos datos BIEN diseñados (normalizado — nuestro esquema actual):**

```
CLIENTES          CATEGORÍAS       PRODUCTOS              PEDIDOS
┌────┬──────┐    ┌────┬───────┐   ┌────┬───────┬────┐   ┌────┬───────┬───────┬─────┐
│ id │nombre│    │ id │nombre │   │ id │nombre │cat.│   │ id │cli_id │prod_id│cant.│
├────┼──────┤    ├────┼───────┤   ├────┼───────┼────┤   ├────┼───────┼───────┼─────┤
│ 1  │Ana   │    │ 1  │Electr.│   │ 10 │Laptop │ 1  │   │ 1  │ 1     │ 10    │ 1   │
│ 2  │Pedro │    │ 2  │Ropa   │   │ 11 │Ratón  │ 1  │   │ 2  │ 1     │ 11    │ 2   │
└────┴──────┘    └────┴───────┘   └────┴───────┴────┘   │ 3  │ 2     │ 10    │ 1   │
                                                         └────┴───────┴───────┴─────┘
  ✅ Cada dato almacenado UNA sola vez
```

### 🏠 La Analogía

Imagina una **biblioteca**. En una biblioteca mal organizada, cada libro tiene pegada una nota con el nombre completo del autor, su biografía, su editorial y su dirección. Si el autor cambia de editorial, hay que ir libro por libro cambiando la nota (anomalía de actualización). Con normalización, la información del autor está en una **ficha única** en el fichero de autores, y cada libro solo tiene una referencia al autor. Cambias la ficha una vez y todos los libros están actualizados.

### 💻 El Código

```sql
-- Ejemplo de tabla SIN normalizar (MAL diseño)
CREATE TABLE pedidos_mal_disenio (
    id_pedido       NUMBER PRIMARY KEY,
    cliente_nombre  VARCHAR2(100),
    cliente_email   VARCHAR2(100),
    cliente_ciudad  VARCHAR2(50),
    producto_nombre VARCHAR2(100),
    categoria_nombre VARCHAR2(50),
    precio          NUMBER(10,2),
    cantidad        NUMBER,
    fecha_pedido    DATE
);

-- Problema: si Ana García cambia de email, hay que actualizar
-- TODAS las filas donde aparece. Si hay 100 pedidos de Ana: 100 UPDATEs.
-- UPDATE pedidos_mal_disenio SET cliente_email = 'ana.nueva@mail.com'
-- WHERE cliente_nombre = 'Ana García';
-- ↑ Peligroso: ¿y si hay otra "Ana García"? Anomalía de actualización.

-- VS. Nuestro diseño normalizado actual:
-- UPDATE clientes SET email = 'ana.nueva@mail.com' WHERE id_cliente = 1;
-- ↑ UN solo UPDATE. Todos los pedidos de Ana reflejan el cambio automáticamente
-- porque acceden al email a través del JOIN con clientes.

-- Limpiar
DROP TABLE pedidos_mal_disenio;
```

### 🧠 El Reto

¿Por qué nuestra tabla `productos` tiene una columna `id_categoria` (FK) en lugar de almacenar directamente el nombre de la categoría? ¿Qué anomalía se evita?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

Se evita la **anomalía de actualización**. Si almacenáramos `categoria_nombre = 'Electrónica'` directamente en cada producto, cambiar el nombre de la categoría (por ejemplo, a "Tecnología") requeriría actualizar TODAS las filas de productos de esa categoría. Con `id_categoria` como FK:

```sql
-- Un solo UPDATE en la tabla categorías:
UPDATE categorias SET nombre = 'Tecnología' WHERE id_categoria = 1;
-- Todos los productos "ven" el nuevo nombre automáticamente vía JOIN
```

</details>

---

---

## 3.2 Primera Forma Normal (1NF)

### 📘 El Concepto

Una tabla está en **1NF** si cumple:

1. **Cada columna contiene valores atómicos** — un solo valor por celda (no listas, no conjuntos).
2. **No hay grupos repetitivos** — no hay columnas como `telefono1`, `telefono2`, `telefono3`.
3. **Cada fila es única** — existe una clave primaria.

**Ejemplo de tabla que VIOLA 1NF:**

```
MEDICOS_MAL (viola 1NF)
┌────┬─────────────┬──────────────────────────┬───────────────────────┐
│ id │ nombre      │ telefonos                │ especialidades        │
├────┼─────────────┼──────────────────────────┼───────────────────────┤
│ 1  │ Dra. López  │ 600111222, 911222333     │ Cardiología, Interna  │  ← ❌ múltiples valores
│ 2  │ Dr. García  │ 644555666                │ Pediatría             │
│ 3  │ Dra. Martín │ 677888999, 915444555,    │ Dermatología          │
│    │             │ 600333444                │                       │  ← ❌ múltiples valores
└────┴─────────────┴──────────────────────────┴───────────────────────┘
```

**Problemas:**
- ¿Cómo buscas médicos por un teléfono específico? Necesitas `LIKE '%600111222%'` (lento, sin índice).
- ¿Cómo añades un cuarto teléfono? Hay que modificar el valor de la celda.
- No puedes hacer JOIN por teléfono ni por especialidad.

### 🏠 La Analogía

Es como un **formulario de registro** donde el campo "Teléfono" dice: "Escribe todos tus teléfonos separados por comas." El resultado es un caos imposible de buscar. En cambio, un buen formulario tiene campos separados o, mejor aún, un botón "Añadir otro teléfono" que crea una lista limpia.

### 💻 El Código

```sql
-- ❌ Tabla que VIOLA 1NF (valores no atómicos)
CREATE TABLE medicos_mal_1nf (
    id_medico      NUMBER PRIMARY KEY,
    nombre         VARCHAR2(100),
    telefonos      VARCHAR2(200),  -- '600111222, 911222333'
    especialidades VARCHAR2(200)   -- 'Cardiología, Interna'
);

INSERT INTO medicos_mal_1nf VALUES (1, 'Dra. López', '600111222, 911222333', 'Cardiología, Interna');
INSERT INTO medicos_mal_1nf VALUES (2, 'Dr. García', '644555666', 'Pediatría');

-- Buscar médicos con teléfono 600111222:
SELECT * FROM medicos_mal_1nf WHERE telefonos LIKE '%600111222%';
-- Funciona, pero es LENTO y propenso a errores (¿y si busco '600'?)

-- ✅ Misma información en 1NF (valores atómicos):
CREATE TABLE medicos_1nf (
    id_medico NUMBER,
    nombre    VARCHAR2(100),
    CONSTRAINT pk_med_1nf PRIMARY KEY (id_medico)
);

CREATE TABLE medicos_telefonos (
    id_medico NUMBER,
    telefono  VARCHAR2(20),
    CONSTRAINT pk_med_tel PRIMARY KEY (id_medico, telefono),
    CONSTRAINT fk_med_tel FOREIGN KEY (id_medico) REFERENCES medicos_1nf(id_medico)
);

CREATE TABLE medicos_especialidades_1nf (
    id_medico      NUMBER,
    especialidad   VARCHAR2(50),
    CONSTRAINT pk_med_esp PRIMARY KEY (id_medico, especialidad),
    CONSTRAINT fk_med_esp FOREIGN KEY (id_medico) REFERENCES medicos_1nf(id_medico)
);

INSERT INTO medicos_1nf VALUES (1, 'Dra. López');
INSERT INTO medicos_telefonos VALUES (1, '600111222');
INSERT INTO medicos_telefonos VALUES (1, '911222333');
INSERT INTO medicos_especialidades_1nf VALUES (1, 'Cardiología');
INSERT INTO medicos_especialidades_1nf VALUES (1, 'Interna');

-- Buscar por teléfono exacto (rápido, indexable):
SELECT m.nombre, t.telefono
FROM medicos_1nf m
JOIN medicos_telefonos t ON m.id_medico = t.id_medico
WHERE t.telefono = '600111222';

-- Limpiar
DROP TABLE medicos_especialidades_1nf;
DROP TABLE medicos_telefonos;
DROP TABLE medicos_1nf;
DROP TABLE medicos_mal_1nf;
```

### 🧠 El Reto

Esta tabla registra rutas de vuelo. ¿Viola 1NF? Si es así, ¿cómo la normalizarías?

```
RUTAS_AMPLIADAS
┌──────────┬────────┬─────────┬─────────────────────────┐
│ id_ruta  │ origen │ destino │ dias_operacion           │
├──────────┼────────┼─────────┼─────────────────────────┤
│ 1        │ Madrid │ Barcelona│ Lunes, Miércoles, Viernes│
│ 2        │ Madrid │ Sevilla │ Todos los días           │
└──────────┴────────┴─────────┴─────────────────────────┘
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Sí, viola 1NF.** La columna `dias_operacion` contiene múltiples valores (no es atómica).

```sql
-- Normalizada a 1NF:
-- Tabla rutas (sin cambios)
-- id_ruta, origen, destino

-- Nueva tabla para los días de operación:
-- CREATE TABLE rutas_dias (
--     id_ruta        NUMBER,
--     dia_operacion  VARCHAR2(15),
--     CONSTRAINT pk_ruta_dia PRIMARY KEY (id_ruta, dia_operacion),
--     CONSTRAINT fk_ruta_dia FOREIGN KEY (id_ruta) REFERENCES rutas(id_ruta)
-- );
-- INSERT INTO rutas_dias VALUES (1, 'Lunes');
-- INSERT INTO rutas_dias VALUES (1, 'Miércoles');
-- INSERT INTO rutas_dias VALUES (1, 'Viernes');
-- INSERT INTO rutas_dias VALUES (2, 'Lunes');
-- INSERT INTO rutas_dias VALUES (2, 'Martes');
-- ... etc.
```

Ahora cada celda tiene un solo valor y puedes buscar rutas por día:
`SELECT * FROM rutas_dias WHERE dia_operacion = 'Lunes';`

</details>

---

---

## 3.3 Segunda Forma Normal (2NF)

### 📘 El Concepto

Una tabla está en **2NF** si:

1. Está en **1NF**.
2. **Cada columna no-clave depende de TODA la clave primaria**, no solo de una parte.

> ⚠️ 2NF solo aplica a tablas con **clave primaria compuesta** (de 2+ columnas). Si la PK es una sola columna, ya estás en 2NF automáticamente (no puede haber dependencia parcial).

**Ejemplo de tabla que VIOLA 2NF:**

```
DETALLE_PEDIDOS (PK compuesta: id_pedido + id_producto)
┌──────────┬────────────┬──────────┬──────────────┬────────────┐
│ id_pedido│ id_producto│ cantidad │ producto_nombre│ cat_nombre │
├──────────┼────────────┼──────────┼──────────────┼────────────┤
│ 1        │ 10         │ 1        │ Laptop Pro    │ Electrónica│
│ 1        │ 11         │ 2        │ Ratón Inalám. │ Electrónica│
│ 2        │ 10         │ 1        │ Laptop Pro    │ Electrónica│
└──────────┴────────────┴──────────┴──────────────┴────────────┘

PK = (id_pedido, id_producto)

Dependencias:
- cantidad         → depende de (id_pedido, id_producto) ✅ completa
- producto_nombre  → depende SOLO de id_producto ❌ parcial
- cat_nombre       → depende SOLO de id_producto ❌ parcial
```

**Problema:** `producto_nombre` y `cat_nombre` se repiten cada vez que ese producto aparece en un pedido. Si "Laptop Pro" cambia de nombre, hay que actualizar múltiples filas.

### 🏠 La Analogía

Imagina un **registro de asistencia escolar** donde la PK es (alumno, fecha) y las columnas incluyen "asistió" (depende de alumno + fecha) pero también "dirección del alumno" (depende solo del alumno). La dirección se repite en todas las fechas. Eso es una dependencia parcial: la dirección debería estar en la tabla de alumnos, no en la de asistencia.

### 💻 El Código

```sql
-- ❌ Tabla que VIOLA 2NF (dependencia parcial)
CREATE TABLE detalle_pedidos_mal (
    id_pedido        NUMBER,
    id_producto      NUMBER,
    cantidad         NUMBER,
    producto_nombre  VARCHAR2(100),  -- depende solo de id_producto ❌
    categoria_nombre VARCHAR2(50),   -- depende solo de id_producto ❌
    CONSTRAINT pk_det_mal PRIMARY KEY (id_pedido, id_producto)
);

INSERT INTO detalle_pedidos_mal VALUES (1, 10, 1, 'Laptop Pro', 'Electrónica');
INSERT INTO detalle_pedidos_mal VALUES (1, 11, 2, 'Ratón Inalámbrico', 'Electrónica');
INSERT INTO detalle_pedidos_mal VALUES (2, 10, 1, 'Laptop Pro', 'Electrónica');

-- Anomalía de actualización: renombrar "Laptop Pro"
-- Hay que actualizar TODAS las filas donde aparece:
UPDATE detalle_pedidos_mal SET producto_nombre = 'Laptop Pro Max'
WHERE id_producto = 10;
-- ¿Y si falla a mitad? Datos inconsistentes.

-- ✅ Normalizado a 2NF: separar lo que depende parcialmente
-- Tabla productos: (id_producto → nombre, categoría) — ya existe en nuestro esquema
-- Tabla detalles: (id_pedido, id_producto → cantidad) — solo datos que dependen de toda la PK

-- Nuestro esquema actual ya está en 2NF:
-- pedidos tiene su propia PK simple (id_pedido)
-- productos tiene id_producto → nombre, precio, stock, id_categoria
-- No hay dependencias parciales

SELECT p.id_pedido, pr.nombre AS producto, p.cantidad, c.nombre AS categoria
FROM pedidos p
JOIN productos pr ON p.id_producto = pr.id_producto
JOIN categorias c ON pr.id_categoria = c.id_categoria
WHERE p.id_pedido <= 3;
-- Los nombres vienen por JOIN, no se almacenan repetidos

-- Limpiar
DROP TABLE detalle_pedidos_mal;
```

### 🧠 El Reto

Esta tabla de citas del hospital tiene PK compuesta. ¿Está en 2NF?

```
CITAS_AMPLIADAS (PK: id_cita)
┌─────────┬────────────┬───────────┬──────────────┬────────────────┬──────────┐
│ id_cita │ id_paciente│ id_medico │ medico_nombre │ especialidad   │ estado   │
├─────────┼────────────┼───────────┼──────────────┼────────────────┼──────────┤
│ 1       │ 1          │ 1         │ Dra. López   │ Cardiología    │ C        │
│ 2       │ 2          │ 1         │ Dra. López   │ Cardiología    │ P        │
│ 3       │ 3          │ 2         │ Dr. García   │ Pediatría      │ C        │
└─────────┴────────────┴───────────┴──────────────┴────────────────┴──────────┘
```

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Sí, está en 2NF** porque la PK es simple (`id_cita`, una sola columna). Con una PK simple no puede haber dependencias parciales, así que 2NF se cumple automáticamente.

**Sin embargo, NO está en 3NF** (siguiente sección). Las columnas `medico_nombre` y `especialidad` dependen de `id_medico`, no de `id_cita`. Es una **dependencia transitiva**: `id_cita → id_medico → medico_nombre, especialidad`.

La solución es eliminar esas columnas y obtenerlas vía JOIN (que es exactamente lo que hace nuestro esquema real).

</details>

---

---

## 3.4 Tercera Forma Normal (3NF)

### 📘 El Concepto

Una tabla está en **3NF** si:

1. Está en **2NF**.
2. **No hay dependencias transitivas** — ninguna columna no-clave depende de otra columna no-clave.

**Dependencia transitiva:** `A → B → C` donde A es la PK, B y C son no-clave. C depende de A, pero a través de B. La solución es mover C a una tabla propia con B como clave.

**Ejemplo de tabla que VIOLA 3NF:**

```
PRODUCTOS_MAL (PK: id_producto)
┌─────────────┬──────────┬────────┬──────────────┬─────────────────┐
│ id_producto  │ nombre   │ precio │ id_categoria │ nombre_categoria│
├─────────────┼──────────┼────────┼──────────────┼─────────────────┤
│ 10           │ Laptop   │ 1200   │ 1            │ Electrónica     │
│ 11           │ Ratón    │ 45.50  │ 1            │ Electrónica     │
│ 16           │ Camiseta │ 19.99  │ 2            │ Ropa            │
└─────────────┴──────────┴────────┴──────────────┴─────────────────┘

Dependencia transitiva:
id_producto → id_categoria → nombre_categoria
                    ↑              ↑
              no-clave        no-clave ❌
```

`nombre_categoria` depende de `id_categoria`, no directamente de `id_producto`.

### 🏠 La Analogía

Es como una **tarjeta de empleado** que incluye: nombre del empleado, departamento y nombre del jefe del departamento. El nombre del jefe depende del departamento, no del empleado. Si el jefe cambia, hay que actualizar la tarjeta de TODOS los empleados de ese departamento. Mejor tener una tabla de departamentos con su jefe.

### 💻 El Código

```sql
-- ❌ Tabla que VIOLA 3NF (dependencia transitiva)
CREATE TABLE productos_mal_3nf (
    id_producto      NUMBER PRIMARY KEY,
    nombre           VARCHAR2(100),
    precio           NUMBER(10,2),
    id_categoria     NUMBER,
    nombre_categoria VARCHAR2(50),   -- depende de id_categoria, no de id_producto ❌
    desc_categoria   VARCHAR2(200)   -- también depende de id_categoria ❌
);

INSERT INTO productos_mal_3nf VALUES (10, 'Laptop Pro', 1200, 1, 'Electrónica', 'Gadgets y dispositivos');
INSERT INTO productos_mal_3nf VALUES (11, 'Ratón', 45.50, 1, 'Electrónica', 'Gadgets y dispositivos');
INSERT INTO productos_mal_3nf VALUES (16, 'Camiseta', 19.99, 2, 'Ropa', 'Ropa y accesorios');

-- Anomalía: renombrar categoría "Electrónica" → "Tecnología"
-- Hay que actualizar CADA producto de esa categoría:
UPDATE productos_mal_3nf SET nombre_categoria = 'Tecnología'
WHERE id_categoria = 1;
-- Si hay 1000 productos electrónicos: 1000 UPDATEs

-- ✅ Normalizado a 3NF (nuestro esquema actual):
-- categorias: (id_categoria → nombre) — tabla propia
-- productos: (id_producto → nombre, precio, stock, id_categoria) — sin nombre_categoria

-- Renombrar categoría: UN solo UPDATE
-- UPDATE categorias SET nombre = 'Tecnología' WHERE id_categoria = 1;

-- Verificar nuestro esquema actual (ya está en 3NF):
SELECT p.id_producto, p.nombre, p.precio, c.nombre AS categoria
FROM productos p
JOIN categorias c ON p.id_categoria = c.id_categoria;

-- Limpiar
DROP TABLE productos_mal_3nf;
```

### 🧠 El Reto

Analiza nuestras 3 bases de datos actuales. ¿Están todas en 3NF? Verifica la tabla `vuelos` — ¿alguna columna depende transitivamente de otra columna no-clave?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Sí, nuestras 3 bases de datos están en 3NF.** Verifiquemos:

**E-commerce:**
- `productos`: id_producto → nombre, precio, stock, id_categoria. No hay dep. transitiva (id_categoria es FK, no arrastra nombre_categoria).
- `pedidos`: id_pedido → id_cliente, id_producto, cantidad, fecha, total. Todas dependen directamente de id_pedido.

**Hospital:**
- `medicos`: id_medico → nombre, id_especialidad, salario. No arrastra nombre_especialidad.
- `citas`: id_cita → id_paciente, id_medico, fecha, estado. No arrastra nombre_medico ni nombre_paciente.

**Aerolínea:**
- `vuelos`: id_vuelo → id_avion, id_ruta, fechas, plazas, precio, estado. No arrastra datos del avión (modelo, capacidad) ni de la ruta (origen, destino). Todo se accede vía JOIN con las FK.

> 💡 Nuestro diseño es un ejemplo de 3NF correcta. Cada tabla almacena solo los datos que dependen directamente de SU clave primaria.

</details>

---

---

## 3.5 Forma Normal de Boyce-Codd (BCNF)

### 📘 El Concepto

**BCNF** es una versión más estricta de 3NF. Una tabla está en BCNF si:

> **Cada determinante es una clave candidata.**

Un **determinante** es una columna (o conjunto) que determina funcionalmente a otra. Una **clave candidata** es un conjunto mínimo de columnas que puede ser PK.

**Diferencia entre 3NF y BCNF:**
- 3NF permite que un atributo no-clave determine a otro atributo que sea parte de una clave candidata.
- BCNF no lo permite.

**Ejemplo de tabla en 3NF pero NO en BCNF:**

```
CLASES_PROFESOR
┌──────────┬─────────┬───────────┐
│ alumno   │ materia │ profesor  │
├──────────┼─────────┼───────────┤
│ Ana      │ SQL     │ Prof. López│
│ Pedro    │ SQL     │ Prof. López│
│ Ana      │ Java    │ Prof. García│
│ María    │ SQL     │ Prof. Martín│
└──────────┴─────────┴───────────┘

PK: (alumno, materia)
Regla de negocio: cada profesor enseña UNA sola materia (pero una materia puede tener varios profesores)

Dependencias:
- (alumno, materia) → profesor  ✅ PK completa
- profesor → materia            ❌ profesor determina materia, pero profesor NO es clave candidata

Viola BCNF porque "profesor" es determinante pero no es clave candidata.
```

### 🏠 La Analogía

Piensa en un **parking con plazas asignadas**. La regla es: cada empleado estaciona en un edificio, y cada edificio tiene una zona de parking específica. Si la tabla tiene PK = (empleado, edificio) y una columna "zona_parking" que depende solo del edificio, está en 3NF pero no en BCNF. La zona de parking debería ir en la tabla de edificios.

### 💻 El Código

```sql
-- ❌ Tabla en 3NF pero NO en BCNF
CREATE TABLE clases_profesor (
    alumno    VARCHAR2(50),
    materia   VARCHAR2(50),
    profesor  VARCHAR2(50),
    CONSTRAINT pk_clases PRIMARY KEY (alumno, materia)
);

INSERT INTO clases_profesor VALUES ('Ana', 'SQL', 'Prof. López');
INSERT INTO clases_profesor VALUES ('Pedro', 'SQL', 'Prof. López');
INSERT INTO clases_profesor VALUES ('Ana', 'Java', 'Prof. García');
INSERT INTO clases_profesor VALUES ('María', 'SQL', 'Prof. Martín');

-- Problema: si Prof. López cambia de materia (de SQL a Python),
-- hay que actualizar TODAS sus filas. Y si no las actualizamos
-- todas, la regla "cada profesor enseña una materia" se rompe.

-- ✅ Normalizado a BCNF: separar la dependencia profesor → materia
CREATE TABLE profesores_materias (
    profesor VARCHAR2(50) PRIMARY KEY,
    materia  VARCHAR2(50)
);

CREATE TABLE inscripciones_bcnf (
    alumno   VARCHAR2(50),
    profesor VARCHAR2(50),
    CONSTRAINT pk_inscr PRIMARY KEY (alumno, profesor),
    CONSTRAINT fk_inscr_prof FOREIGN KEY (profesor) REFERENCES profesores_materias(profesor)
);

INSERT INTO profesores_materias VALUES ('Prof. López', 'SQL');
INSERT INTO profesores_materias VALUES ('Prof. García', 'Java');
INSERT INTO profesores_materias VALUES ('Prof. Martín', 'SQL');

INSERT INTO inscripciones_bcnf VALUES ('Ana', 'Prof. López');
INSERT INTO inscripciones_bcnf VALUES ('Pedro', 'Prof. López');
INSERT INTO inscripciones_bcnf VALUES ('Ana', 'Prof. García');
INSERT INTO inscripciones_bcnf VALUES ('María', 'Prof. Martín');

-- Ahora, cambiar la materia del profesor es UN solo UPDATE:
-- UPDATE profesores_materias SET materia = 'Python' WHERE profesor = 'Prof. López';

-- Limpiar
DROP TABLE inscripciones_bcnf;
DROP TABLE profesores_materias;
DROP TABLE clases_profesor;
```

### 🧠 El Reto

¿Nuestras tablas actuales (clientes, productos, pedidos, médicos, citas, vuelos, etc.) están en BCNF?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Sí, todas nuestras tablas están en BCNF** porque todas tienen claves primarias simples (una sola columna) y no hay determinantes que no sean claves candidatas.

BCNF solo es problemática cuando hay claves compuestas y reglas de negocio especiales (como "cada profesor enseña una sola materia"). En nuestro esquema:

- Todas las PK son simples: `id_cliente`, `id_producto`, `id_pedido`, `id_medico`, `id_cita`, `id_vuelo`, etc.
- Las FK apuntan a tablas separadas.
- No hay dependencias funcionales ocultas.

> 💡 En la práctica, la mayoría de bases de datos bien diseñadas con PKs simples están automáticamente en BCNF. Los problemas de BCNF surgen principalmente con PKs compuestas y reglas de negocio complejas.

</details>

---

---

## 3.6 Desnormalización: Cuándo y por qué Romper las Reglas

### 📘 El Concepto

La **desnormalización** es el proceso deliberado de introducir redundancia en una base de datos normalizada para **mejorar el rendimiento de lectura**.

| Aspecto | Normalizado (3NF) | Desnormalizado |
|---------|-------------------|----------------|
| **Redundancia** | Mínima | Controlada |
| **JOINs necesarios** | Muchos | Pocos |
| **Velocidad de lectura** | Más lenta (JOINs) | Más rápida |
| **Velocidad de escritura** | Rápida | Más lenta (actualizar copias) |
| **Integridad** | Automática | Requiere disciplina |
| **Espacio en disco** | Mínimo | Mayor |

**Cuándo desnormalizar:**

| Escenario | Ejemplo | Técnica |
|-----------|---------|---------|
| **Data warehouses** | Reportes analíticos que consultan millones de filas | Tablas de hechos con columnas redundantes |
| **Campos calculados** | `total` en pedidos (precio × cantidad) | Columna calculada persistida |
| **Consultas críticas de rendimiento** | Dashboard que muestra resumen en < 1 segundo | Tabla resumen pre-calculada |
| **Datos históricos** | Precio del producto AL MOMENTO del pedido | Copiar precio a tabla de pedidos |

> ⚠️ **Regla de oro:** Normaliza primero (3NF), mide el rendimiento, y desnormaliza **solo lo necesario** con justificación.

### 🏠 La Analogía

Piensa en un **menú de restaurante**. Un menú normalizado diría: "Plato #15 — ver ingredientes en tabla aparte, ver alérgenos en otra tabla, ver precio en la tabla de precios." Correcto pero impracticable. El menú real (desnormalizado) repite el nombre del plato, su descripción, el precio y los alérgenos **en cada página**. Es redundante, pero permite al cliente elegir rápidamente.

### 💻 El Código

```sql
-- Ejemplo de desnormalización JUSTIFICADA:
-- Nuestra tabla 'pedidos' tiene una columna 'total'
-- En un diseño estrictamente normalizado, 'total' no existiría
-- porque se puede calcular: precio × cantidad

-- Versión normalizada pura (sin total):
SELECT p.id_pedido, p.cantidad, pr.precio,
       (p.cantidad * pr.precio) AS total_calculado
FROM pedidos p
JOIN productos pr ON p.id_producto = pr.id_producto
WHERE p.id_pedido = 1;

-- Versión desnormalizada (con total almacenado):
SELECT id_pedido, cantidad, total
FROM pedidos
WHERE id_pedido = 1;
-- Más rápido: no necesita JOIN

-- ¿Por qué 'total' es una buena desnormalización?
-- 1. El precio del producto puede CAMBIAR después del pedido
-- 2. El total refleja el precio AL MOMENTO de la compra (dato histórico)
-- 3. Evita JOIN para reportes de ventas (rendimiento)

-- Ejemplo de tabla resumen desnormalizada para reporting:
CREATE TABLE resumen_ventas_mensual (
    anio              NUMBER,
    mes               NUMBER,
    total_pedidos     NUMBER,
    ingresos_totales  NUMBER(12,2),
    ticket_promedio   NUMBER(10,2),
    CONSTRAINT pk_resumen PRIMARY KEY (anio, mes)
);

-- Poblar con datos pre-calculados:
INSERT INTO resumen_ventas_mensual
SELECT EXTRACT(YEAR FROM fecha_pedido) AS anio,
       EXTRACT(MONTH FROM fecha_pedido) AS mes,
       COUNT(*) AS total_pedidos,
       SUM(total) AS ingresos_totales,
       ROUND(AVG(total), 2) AS ticket_promedio
FROM pedidos
GROUP BY EXTRACT(YEAR FROM fecha_pedido), EXTRACT(MONTH FROM fecha_pedido);

-- Consulta de reporting ultra-rápida:
SELECT * FROM resumen_ventas_mensual ORDER BY anio, mes;
-- Sin JOINs, sin GROUP BY, datos pre-calculados

-- Limpiar
DROP TABLE resumen_ventas_mensual;
```

### 🧠 El Reto

El equipo de analítica de la aerolínea necesita un dashboard que muestre para cada ruta: el origen, destino, número total de vuelos, promedio de ocupación y precio promedio. La consulta normalizada tarda 3 segundos con millones de filas. ¿Cómo desnormalizarías para que tarde milisegundos?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

Crear una **tabla resumen desnormalizada** que se actualice periódicamente:

```sql
CREATE TABLE dashboard_rutas (
    id_ruta         NUMBER PRIMARY KEY,
    origen          VARCHAR2(50),    -- redundante (viene de rutas)
    destino         VARCHAR2(50),    -- redundante (viene de rutas)
    total_vuelos    NUMBER,
    avg_ocupacion   NUMBER(5,2),
    avg_precio      NUMBER(10,2),
    ultima_actualizacion DATE
);

-- Poblar/actualizar (ejecutar periódicamente, ej: cada hora):
MERGE INTO dashboard_rutas d
USING (
    SELECT r.id_ruta, r.aeropuerto_origen, r.aeropuerto_destino,
           COUNT(v.id_vuelo) AS total_vuelos,
           ROUND(AVG(v.plazas_disponibles), 2) AS avg_ocupacion,
           ROUND(AVG(v.precio), 2) AS avg_precio
    FROM rutas r
    LEFT JOIN vuelos v ON r.id_ruta = v.id_ruta
    GROUP BY r.id_ruta, r.aeropuerto_origen, r.aeropuerto_destino
) src ON (d.id_ruta = src.id_ruta)
WHEN MATCHED THEN UPDATE SET
    d.total_vuelos = src.total_vuelos,
    d.avg_ocupacion = src.avg_ocupacion,
    d.avg_precio = src.avg_precio,
    d.ultima_actualizacion = SYSDATE
WHEN NOT MATCHED THEN INSERT VALUES (
    src.id_ruta, src.aeropuerto_origen, src.aeropuerto_destino,
    src.total_vuelos, src.avg_ocupacion, src.avg_precio, SYSDATE
);
COMMIT;

-- Dashboard ultra-rápido (sin JOINs, sin GROUP BY):
SELECT * FROM dashboard_rutas ORDER BY total_vuelos DESC;
```

> 💡 El trade-off: los datos del dashboard pueden estar "atrasados" hasta la próxima actualización. Para un dashboard, eso es aceptable.

```sql
-- Limpiar
DROP TABLE dashboard_rutas;
```

</details>

---

## 3.7 Modelado Entidad-Relación (ER)

### 📘 El Concepto

El **Modelado Entidad-Relación (ER)** es una técnica de diseño de bases de datos que permite representar visualmente la estructura de los datos antes de implementarlos en SQL. Es el **plano arquitectónico** que dibujas antes de construir la casa.

Los componentes fundamentales de un diagrama ER son:

| Componente | Representación | Descripción |
|------------|:-:|-------------|
| **Entidad** | Rectángulo | Un objeto del mundo real (Cliente, Producto, Pedido) |
| **Atributo** | Óvalo | Una propiedad de la entidad (nombre, email, precio) |
| **Relación** | Rombo | La conexión entre entidades (compra, trabaja_en) |
| **Clave Primaria** | Subrayado | El atributo que identifica de forma única a cada entidad |

**Tipos de relaciones (cardinalidad):**

| Cardinalidad | Ejemplo | Significado |
|:------------:|---------|-------------|
| **1:1** | Persona ↔ Pasaporte | Cada persona tiene un único pasaporte y viceversa |
| **1:N** | Departamento → Empleados | Un departamento tiene muchos empleados, pero cada empleado pertenece a un solo departamento |
| **N:M** | Estudiantes ↔ Cursos | Un estudiante puede inscribirse en muchos cursos y un curso puede tener muchos estudiantes |

### 🏠 La Analogía

Imagina que vas a construir una casa. **No empiezas poniendo ladrillos** — primero dibujas los planos:
- ¿Cuántas habitaciones hay? → **Entidades** (tablas)
- ¿Qué hay en cada habitación? → **Atributos** (columnas)
- ¿Cómo se conectan las habitaciones? → **Relaciones** (claves foráneas)
- ¿Cuál es la puerta principal de cada habitación? → **Clave primaria**

El diagrama ER es tu plano. La normalización es el inspector que verifica que los planos cumplen las normas de construcción. Juntos, garantizan que tu base de datos será sólida.

### 💻 El Proceso

```
Paso 1: Identificar Entidades
   → ¿Qué "cosas" necesito almacenar? (Clientes, Productos, Pedidos...)

Paso 2: Definir Atributos
   → ¿Qué información necesito de cada entidad? (nombre, email, precio...)

Paso 3: Identificar Relaciones
   → ¿Cómo se conectan las entidades? (un cliente HACE pedidos...)

Paso 4: Determinar Cardinalidad
   → ¿Es 1:1, 1:N o N:M?

Paso 5: Identificar Claves
   → ¿Qué atributo identifica de forma única a cada entidad?

Paso 6: Normalizar
   → Aplicar 1NF, 2NF, 3NF para eliminar redundancias
```

> 💡 **Regla de oro:** Siempre modela primero en un diagrama ER y normaliza el diseño **antes** de escribir tu primera sentencia `CREATE TABLE`. Corregir un mal diseño después es mucho más costoso.

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 03

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de Normalización](ejercicios/ejercicios_normalizacion.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Proyecto: Rescatando una BD Legacy](proyectos/proyecto_normalizacion.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 04: DDL (Data Definition Language)](../04-ddl) |

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 02**](../02-tipos-de-datos/proyectos/proyecto_tipos_datos.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 03 →**](ejercicios/ejercicios_normalizacion.md)

</div>
