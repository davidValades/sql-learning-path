# 🎛️ Tema 07: Operadores Lógicos y Filtros

> **"Preguntar es fácil. Preguntar lo correcto es un arte."** En este tema dominarás el arte de filtrar, buscar y ordenar datos con la precisión de un cirujano.

## 📋 Índice

- [📦 Expansión de Datos](#-expansión-de-datos)
- [7.1 WHERE y operadores de comparación](#71-where-y-operadores-de-comparación)
- [7.2 BETWEEN para rangos](#72-between-para-rangos)
- [7.3 IN para listas](#73-in-para-listas)
- [7.4 LIKE y wildcards](#74-like-y-wildcards)
- [7.5 IS NULL / IS NOT NULL](#75-is-null--is-not-null)
- [7.6 AND, OR, NOT y precedencia](#76-and-or-not-y-precedencia-con-paréntesis)
- [7.7 ORDER BY](#77-order-by-asc-desc-múltiples-columnas)

---

## 📦 Expansión de Datos

Antes de empezar con la teoría, necesitamos más datos en nuestras tres bases de datos para que los ejercicios sean significativos. Ejecuta los siguientes `INSERT` en tu sesión de Oracle:

```sql
-- ============================================
-- E-COMMERCE: más productos y clientes
-- ============================================
INSERT INTO clientes VALUES (1, 'Ana López', 'ana@email.com');
INSERT INTO clientes VALUES (2, 'Pedro Ruiz', 'pedro@email.com');
INSERT INTO clientes VALUES (3, 'María García', 'maria@email.com');
INSERT INTO categorias VALUES (3, 'Ropa');
INSERT INTO productos VALUES (14, 'Camiseta Básica', 19.99, 3, 100, NULL);
INSERT INTO productos VALUES (15, 'Zapatillas Running', 89.50, 3, 45, NULL);
INSERT INTO productos VALUES (16, 'Monitor 4K', 350.00, 1, 25, NULL);
INSERT INTO productos VALUES (17, 'Teclado Mecánico', 75.00, 1, 80, NULL);

-- ============================================
-- HOSPITAL: más médicos y pacientes
-- ============================================
INSERT INTO medicos VALUES (5, 'Dr. Carlos Vega', 100, 2800.00);
INSERT INTO medicos VALUES (6, 'Dra. Marta López', 200, 4500.00);
INSERT INTO medicos VALUES (7, 'Dr. Luis Moreno', 300, 2600.00);
INSERT INTO pacientes VALUES (3, '33333333C', 'Pedro Sánchez', TO_DATE('2000-03-15','YYYY-MM-DD'), '555-0003');
INSERT INTO pacientes VALUES (4, '44444444D', 'Ana Ruiz', TO_DATE('1978-11-20','YYYY-MM-DD'), NULL);
INSERT INTO pacientes VALUES (5, '55555555E', 'David Torres', TO_DATE('1995-07-08','YYYY-MM-DD'), '555-0005');

-- ============================================
-- AEROLÍNEA: más aviones y rutas
-- ============================================
INSERT INTO aviones VALUES (40, 'Embraer E195', 120, 2020);
INSERT INTO aviones VALUES (50, 'Boeing 777', 350, 2016);
INSERT INTO rutas VALUES ('BCNCDG', 'BCN', 'CDG', 830);
INSERT INTO rutas VALUES ('MADFRA', 'MAD', 'FRA', 1420);
INSERT INTO rutas VALUES ('BCNFCO', 'BCN', 'FCO', 850);
COMMIT;
```

> ✅ **Estado acumulado tras la expansión:**
>
> - **E-commerce**: 3 clientes · 3 categorías · 8 productos (4 con `codigo_barras` NULL)
> - **Hospital**: 5 médicos · 5 pacientes (1 con `telefono` NULL) · especialidades sin cambio
> - **Aerolínea**: 4 aviones · 5 rutas · vuelos sin cambio

---

---

## 7.1 WHERE y operadores de comparación

### 📘 El Concepto

La cláusula `WHERE` es el **filtro universal** de SQL. Se coloca después del `FROM` y evalúa cada fila de la tabla: si la condición es verdadera, la fila pasa al resultado; si no, se descarta.

Los **operadores de comparación** son las herramientas que usamos dentro del `WHERE` para construir esas condiciones:

| Operador | Significado | Ejemplo |
|----------|-------------|---------|
| `=` | Igual a | `WHERE precio = 45.50` |
| `!=` o `<>` | Diferente de | `WHERE stock != 0` |
| `>` | Mayor que | `WHERE precio > 100` |
| `<` | Menor que | `WHERE capacidad_pasajeros < 200` |
| `>=` | Mayor o igual que | `WHERE salario_base >= 3000` |
| `<=` | Menor o igual que | `WHERE distancia_km <= 1000` |

> ⚠️ **Regla de oro:** Para comparar textos en Oracle, el valor debe ir entre comillas simples (`'...'`) y **respeta mayúsculas/minúsculas**: `'Neurología'` ≠ `'neurología'`.

### 🏠 La Analogía

Piensa en el `WHERE` como el **filtro de búsqueda de una tienda online**. Entras a Amazon con 10.000 productos disponibles, pero tú ajustas el deslizador de precio: _"Solo quiero ver productos de más de 50€ y menos de 200€"_. Amazon descarta los que no cumplen y te muestra solo los supervivientes. Eso es exactamente lo que hace `WHERE`.

### 💻 El Código

```sql
-- E-commerce: productos que cuestan más de 100€
SELECT nombre, precio
FROM productos
WHERE precio > 100;
-- Resultado: Laptop Pro(1220), Sofá de Cuero(405), Monitor 4K(350)

-- Hospital: médicos con salario base de exactamente 3200
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base = 3200;
-- Resultado: Dra. Elena Ruiz

-- Aerolínea: rutas con distancia menor o igual a 1000 km
SELECT id_ruta, aeropuerto_origen, aeropuerto_destino, distancia_km
FROM rutas
WHERE distancia_km <= 1000;
-- Resultado: BCNCDG(830), BCNFCO(850)
```

### 🧠 El Reto

El jefe de almacén del E-commerce quiere identificar los productos que **no** están agotados, es decir, aquellos cuyo `stock` sea **diferente de 0**.

**Pregunta:** Escribe la consulta que devuelva el `nombre` y el `stock` de todos los productos con stock distinto de 0.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT nombre, stock
FROM productos
WHERE stock != 0;
```

Resultado: 7 filas (todos menos Sofá de Cuero con stock 0).

</details>

---

---

## 7.2 BETWEEN para rangos

### 📘 El Concepto

El operador `BETWEEN` permite filtrar valores dentro de un **rango inclusivo** (incluye ambos extremos). Es equivalente a combinar `>=` y `<=`, pero mucho más legible.

```
WHERE columna BETWEEN valor_bajo AND valor_alto
```

> 📌 **Importante**: `BETWEEN` funciona con números, fechas y texto (orden alfabético). Siempre es **inclusivo**: `BETWEEN 100 AND 500` incluye el 100 y el 500.

También existe `NOT BETWEEN` para excluir un rango.

### 🏠 La Analogía

Imagina que vas al supermercado y le dices al dependiente: _"Quiero ver los vinos que cuestan **entre 10 y 30 euros**"_. No te interesan los de 9.99€ ni los de 30.01€. Ese rango exacto es tu `BETWEEN`.

### 💻 El Código

```sql
-- E-commerce: productos con precio entre 40 y 100 euros (inclusive)
SELECT nombre, precio
FROM productos
WHERE precio BETWEEN 40 AND 100;
-- Resultado: Ratón Inalámbrico(45.50), Sofá... NO (405), Lámpara LED(40.50),
--            Zapatillas Running(89.50), Teclado Mecánico(75.00)

-- Hospital: médicos con salario entre 3000 y 4200
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base BETWEEN 3000 AND 4200;
-- Resultado: Dra. Sarah Adams(4000), Dra. Elena Ruiz(3200)

-- Aerolínea: rutas con distancia entre 800 y 1400 km
SELECT id_ruta, distancia_km
FROM rutas
WHERE distancia_km BETWEEN 800 AND 1400;
-- Resultado: MADLHR(1350), BCNCDG(830), BCNFCO(850)
```

### 🧠 El Reto

El departamento de RRHH del hospital quiere localizar a los pacientes nacidos **entre el 1 de enero de 1980 y el 31 de diciembre de 1999** (la generación millennial del hospital).

**Pregunta:** Escribe la consulta que devuelva `nombre` y `fecha_nacimiento` de esos pacientes usando `BETWEEN` y `TO_DATE`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT nombre, fecha_nacimiento
FROM pacientes
WHERE fecha_nacimiento BETWEEN TO_DATE('1980-01-01','YYYY-MM-DD')
                            AND TO_DATE('1999-12-31','YYYY-MM-DD');
```

Resultado: Carlos Gomez (1985-05-05), Laura Martinez (1990-01-01), David Torres (1995-07-08).

</details>

---

---

## 7.3 IN para listas

### 📘 El Concepto

El operador `IN` permite comprobar si un valor coincide con **cualquiera de los valores de una lista**. Es un atajo elegante para no encadenar múltiples `OR`.

```
WHERE columna IN (valor1, valor2, valor3)
```

Es equivalente a:

```
WHERE columna = valor1 OR columna = valor2 OR columna = valor3
```

También existe `NOT IN` para excluir valores de una lista.

> ⚠️ **Cuidado con NULL**: Si la lista de `NOT IN` contiene un `NULL`, el resultado será vacío. Esto es un error clásico de Oracle.

### 🏠 La Analogía

Estás en un restaurante y el camarero te pregunta qué quieres beber. Tú dices: _"Lo que sea, siempre que sea **de esta lista**: agua, cerveza o zumo"_. Si tiene vino pero no está en tu lista, no lo quieres. Eso es `IN`: una lista de opciones aceptables.

### 💻 El Código

```sql
-- E-commerce: productos de las categorías Electrónica (1) o Ropa (3)
SELECT nombre, precio, id_categoria
FROM productos
WHERE id_categoria IN (1, 3);
-- Resultado: Laptop Pro, Ratón Inalámbrico, Camiseta Básica,
--            Zapatillas Running, Monitor 4K, Teclado Mecánico

-- Hospital: médicos que NO sean de Traumatología (100) ni Neurología (200)
SELECT nombre_completo, id_especialidad
FROM medicos
WHERE id_especialidad NOT IN (100, 200);
-- Resultado: Dr. Luis Moreno (300 - Medicina General)

-- Aerolínea: aviones fabricados en 2018, 2020 o 2022
SELECT modelo, anio_fabricacion
FROM aviones
WHERE anio_fabricacion IN (2018, 2020, 2022);
-- Resultado: Boeing 737(2018), Airbus A350(2022), Embraer E195(2020)
```

### 🧠 El Reto

La aerolínea necesita un informe de las rutas que **salen de Madrid (MAD) o de Barcelona (BCN)**.

**Pregunta:** Escribe la consulta usando `IN` que devuelva todas las columnas de las rutas cuyo `aeropuerto_origen` sea `'MAD'` o `'BCN'`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT *
FROM rutas
WHERE aeropuerto_origen IN ('MAD', 'BCN');
```

Resultado: MADLHR, MADFRA, BCNCDG, BCNFCO (4 rutas).

</details>

---

---

## 7.4 LIKE y wildcards

### 📘 El Concepto

El operador `LIKE` permite buscar patrones dentro de textos. Es la herramienta perfecta cuando **no conoces el valor exacto** pero sabes parte del texto que buscas.

Utiliza dos **comodines** (wildcards):

| Comodín | Significado | Ejemplo |
|---------|-------------|---------|
| `%` | Cualquier cantidad de caracteres (0 o más) | `'A%'` → empieza por A |
| `_` | Exactamente **un** carácter | `'_a%'` → segunda letra es 'a' |

Combinaciones comunes:

| Patrón | Significado |
|--------|-------------|
| `'%Pro%'` | Contiene "Pro" en cualquier posición |
| `'Dr.%'` | Empieza por "Dr." |
| `'%ción'` | Termina en "ción" |
| `'___'` | Exactamente 3 caracteres |

También existe `NOT LIKE` para excluir un patrón.

> ⚠️ En Oracle, `LIKE` **distingue mayúsculas de minúsculas**. Si necesitas ignorarlas, combina con `UPPER()` o `LOWER()` (lo veremos en el Tema 07).

### 🏠 La Analogía

Piensa en cuando buscas una canción en Spotify pero solo recuerdas una parte del título. Escribes _"Bohemian"_ y Spotify te devuelve "Bohemian Rhapsody", "Bohemian Like You"... Eso es `LIKE '%Bohemian%'`: no sabes el nombre completo, pero sabes un trozo.

### 💻 El Código

```sql
-- E-commerce: productos cuyo nombre empieza por 'L'
SELECT nombre, precio
FROM productos
WHERE nombre LIKE 'L%';
-- Resultado: Laptop Pro, Lámpara LED

-- Hospital: médicos cuyo nombre contiene 'Dr.' (doctores, no doctoras)
SELECT nombre_completo
FROM medicos
WHERE nombre_completo LIKE 'Dr. %';
-- Resultado: Dr. Carlos Vega, Dr. Luis Moreno

-- Hospital: pacientes cuyo DNI termina en 'C'
SELECT nombre, dni
FROM pacientes
WHERE dni LIKE '%C';
-- Resultado: Pedro Sánchez (33333333C)

-- Aerolínea: rutas cuyo id_ruta tiene 'BCN' en cualquier posición
SELECT id_ruta, aeropuerto_origen, aeropuerto_destino
FROM rutas
WHERE id_ruta LIKE '%BCN%';
-- Resultado: BCNCDG, BCNFCO
```

### 🧠 El Reto

El departamento de marketing del E-commerce quiere enviar un email promocional a todos los clientes cuyo email contenga la palabra `"email"` en su dirección.

**Pregunta:** Escribe la consulta que devuelva `nombre` y `email` de los clientes cuyo email contenga `'email'`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT nombre, email
FROM clientes
WHERE email LIKE '%email%';
```

Resultado: Ana López, Pedro Ruiz, María García (los 3 clientes tienen '@email.com').

</details>

---

---

## 7.5 IS NULL / IS NOT NULL

### 📘 El Concepto

`NULL` en Oracle representa la **ausencia de valor**. No es cero, no es una cadena vacía, no es nada. Y aquí viene la trampa: **no puedes comparar `NULL` con `=` ni con `!=`**. Hacer `WHERE telefono = NULL` **no funciona**. Siempre devolverá vacío.

La forma correcta de trabajar con NULL es:

```
WHERE columna IS NULL       -- La columna NO tiene valor
WHERE columna IS NOT NULL   -- La columna SÍ tiene valor
```

> 🧪 **¿Por qué `= NULL` no funciona?** Porque en la lógica de SQL existe un tercer estado: `UNKNOWN`. La comparación `NULL = NULL` no es verdadera ni falsa, es desconocida. Por eso necesitamos `IS NULL`.

### 🏠 La Analogía

Imagina un formulario en papel. Hay una casilla para "Número de teléfono". Si alguien **deja la casilla en blanco**, eso es `NULL`. Si escribiera un 0, eso sería un valor (incorrecto, pero un valor). Si el formulario pregunta: _"¿El teléfono es igual a [vacío]?"_... la pregunta no tiene sentido. Lo correcto es preguntar: _"¿La casilla está vacía?"_ → `IS NULL`.

### 💻 El Código

```sql
-- E-commerce: productos que NO tienen código de barras asignado
SELECT nombre, codigo_barras
FROM productos
WHERE codigo_barras IS NULL;
-- Resultado: Camiseta Básica, Zapatillas Running, Monitor 4K, Teclado Mecánico

-- Hospital: pacientes que SÍ tienen teléfono registrado
SELECT nombre, telefono
FROM pacientes
WHERE telefono IS NOT NULL;
-- Resultado: Laura Martinez, Carlos Gomez, Pedro Sánchez, David Torres

-- Hospital: pacientes SIN teléfono
SELECT nombre, telefono
FROM pacientes
WHERE telefono IS NULL;
-- Resultado: Ana Ruiz
```

### 🧠 El Reto

El equipo de logística del E-commerce necesita identificar los productos que **sí** tienen código de barras asignado para generar etiquetas de envío.

**Pregunta:** Escribe la consulta que devuelva `nombre` y `codigo_barras` de los productos cuyo código de barras **no sea nulo**.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT nombre, codigo_barras
FROM productos
WHERE codigo_barras IS NOT NULL;
```

Resultado: Laptop Pro, Ratón Inalámbrico, Sofá de Cuero, Lámpara LED (los 4 productos originales que sí tienen código).

</details>

---

---

## 7.6 AND, OR, NOT y precedencia con paréntesis

### 📘 El Concepto

Los operadores lógicos permiten **combinar múltiples condiciones** en un solo `WHERE`:

| Operador | Significado | Ejemplo |
|----------|-------------|---------|
| `AND` | **Todas** las condiciones deben ser verdaderas | `WHERE precio > 50 AND stock > 0` |
| `OR` | **Al menos una** condición debe ser verdadera | `WHERE id_categoria = 1 OR id_categoria = 2` |
| `NOT` | **Niega** (invierte) una condición | `WHERE NOT id_categoria = 3` |

#### ⚡ La precedencia (orden de evaluación)

Oracle evalúa las condiciones en este orden:

1. `NOT` (se evalúa primero)
2. `AND` (se evalúa segundo)
3. `OR` (se evalúa último)

Esto significa que `A OR B AND C` se evalúa como `A OR (B AND C)`, ¡NO como `(A OR B) AND C`!

> 🛡️ **Regla de supervivencia**: Siempre usa **paréntesis** para hacer explícita tu intención. No confíes en la precedencia por defecto.

### 🏠 La Analogía

Piensa en un portero de discoteca con reglas complejas:

- _"Entran los que tengan **entrada VIP** O los que **sean mayores de 25 Y lleven traje**"_.
- Sin paréntesis, es ambiguo. ¿Quién entra?
  - `VIP OR (mayor_25 AND traje)` → Los VIP entran siempre. Los demás necesitan cumplir las dos.
  - `(VIP OR mayor_25) AND traje` → Todos necesitan traje, pero puedes ser VIP o mayor de 25.

¡Son reglas completamente diferentes! Los paréntesis son tu salvavidas.

### 💻 El Código

```sql
-- E-commerce: productos de Electrónica (cat 1) con precio mayor a 100 Y con stock
SELECT nombre, precio, stock
FROM productos
WHERE id_categoria = 1 AND precio > 100 AND stock > 0;
-- Resultado: Laptop Pro(1220, stock 50), Monitor 4K(350, stock 25)

-- Hospital: médicos de Traumatología O con salario mayor a 4000
SELECT nombre_completo, id_especialidad, salario_base
FROM medicos
WHERE id_especialidad = 100 OR salario_base > 4000;
-- Resultado: Dra. Elena Ruiz(100,3200), Dr. Carlos Vega(100,2800),
--            Dra. Marta López(200,4500)

-- E-commerce: productos de Ropa o Hogar, PERO solo los que cuesten menos de 50
SELECT nombre, precio, id_categoria
FROM productos
WHERE (id_categoria = 2 OR id_categoria = 3) AND precio < 50;
-- Resultado: Lámpara LED(40.50, cat2), Camiseta Básica(19.99, cat3)

-- Aerolínea: aviones que NO sean Boeing
SELECT modelo, capacidad_pasajeros
FROM aviones
WHERE NOT modelo LIKE 'Boeing%';
-- Resultado: Airbus A350, Embraer E195
```

### 🧠 El Reto

El director financiero del hospital necesita un listado de médicos que cumplan **una de estas dos condiciones**:
1. Sean de **Neurología** (id_especialidad = 200) con salario mayor a 4000.
2. Sean de **Medicina General** (id_especialidad = 300).

**Pregunta:** Escribe la consulta que devuelva `nombre_completo`, `id_especialidad` y `salario_base`. Usa paréntesis para asegurar la precedencia correcta.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT nombre_completo, id_especialidad, salario_base
FROM medicos
WHERE (id_especialidad = 200 AND salario_base > 4000)
   OR id_especialidad = 300;
```

Resultado: Dra. Marta López (200, 4500), Dr. Luis Moreno (300, 2600).

</details>

---

---

## 7.7 ORDER BY (ASC, DESC, múltiples columnas)

### 📘 El Concepto

`ORDER BY` controla el **orden de presentación** de los resultados. Se coloca siempre al **final** de la consulta (después del `WHERE`, si existe).

Reglas clave:

| Regla | Descripción |
|-------|-------------|
| `ASC` | Orden ascendente (A→Z, 1→99). Es el **valor por defecto**. |
| `DESC` | Orden descendente (Z→A, 99→1). |
| Múltiples columnas | Se puede ordenar por varias columnas separadas por coma. |
| Posición numérica | Puedes usar el número de la columna en el `SELECT`: `ORDER BY 2 DESC`. |
| NULLs | En Oracle, los `NULL` aparecen **al final** en orden `ASC` y **al principio** en `DESC`. Puedes controlar esto con `NULLS FIRST` / `NULLS LAST`. |

### 🏠 La Analogía

Imagina que tienes una pila de currículos y necesitas organizarlos. Primero los ordenas por **años de experiencia** (de más a menos). Si hay empate, los desempatas por **orden alfabético del apellido** (de A a Z). Eso es exactamente:

```sql
ORDER BY experiencia DESC, apellido ASC
```

### 💻 El Código

```sql
-- E-commerce: productos ordenados del más caro al más barato
SELECT nombre, precio
FROM productos
ORDER BY precio DESC;

-- Hospital: médicos ordenados por especialidad, y dentro de cada especialidad
-- por salario de mayor a menor
SELECT nombre_completo, id_especialidad, salario_base
FROM medicos
ORDER BY id_especialidad ASC, salario_base DESC;
-- Resultado:
-- Dra. Elena Ruiz    (100, 3200)
-- Dr. Carlos Vega    (100, 2800)
-- Dra. Marta López   (200, 4500)
-- Dra. Sarah Adams   (200, 4000)
-- Dr. Luis Moreno    (300, 2600)

-- Aerolínea: rutas ordenadas por distancia, NULLs al final (por defecto en ASC)
SELECT id_ruta, distancia_km
FROM rutas
ORDER BY distancia_km ASC;

-- E-commerce: productos con código de barras NULL al principio
SELECT nombre, codigo_barras
FROM productos
ORDER BY codigo_barras NULLS FIRST;
```

### 🧠 El Reto

El CEO de la aerolínea quiere un informe de flota ordenado de la siguiente forma: **primero los aviones con mayor capacidad de pasajeros**; si dos aviones tienen la misma capacidad, **el más nuevo primero**.

**Pregunta:** Escribe la consulta que devuelva `modelo`, `capacidad_pasajeros` y `anio_fabricacion` con el orden solicitado.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT modelo, capacidad_pasajeros, anio_fabricacion
FROM aviones
ORDER BY capacidad_pasajeros DESC, anio_fabricacion DESC;
```

Resultado:
1. Boeing 777 (350, 2016)
2. Airbus A350 (300, 2022)
3. Boeing 737 (180, 2018)
4. Embraer E195 (120, 2020)

</details>

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 07

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Completar los ejercicios | 🏋️ [Ir a Ejercicios →](./ejercicios/ejercicios_operadores.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Ir a Proyecto →](./proyectos/proyecto_operadores.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 08: Funciones Nativas de Oracle](../08-funciones) |

---

<div align="center">

⬅️ [**🏆 Proyecto Anterior**](../06-dql/proyectos/proyecto_dql_medio.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 07 →**](ejercicios/ejercicios_operadores.md)

</div>
