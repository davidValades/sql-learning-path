# ⚙️ Tema 16: PL/SQL (Programación Procedural)

> **"SQL te permite hablar con los datos; PL/SQL te permite enseñarle a la base de datos a tomar decisiones por sí misma."** Con PL/SQL puedes crear bloques de lógica, procedimientos reutilizables, funciones, cursores, manejo de errores y triggers — todo viviendo dentro del motor de Oracle.

## 📋 Índice

- [16.1 Bloques Anónimos PL/SQL (El Hola Mundo)](#161-bloques-anónimos-plsql-el-hola-mundo)
- [16.2 Variables y Tipos de Datos PL/SQL](#162-variables-y-tipos-de-datos-plsql)
- [16.3 Estructuras de Control (IF, CASE, LOOP)](#163-estructuras-de-control-if-case-loop)
- [16.4 Cursores Explícitos e Implícitos](#164-cursores-explícitos-e-implícitos)
- [16.5 Manejo de Excepciones](#165-manejo-de-excepciones)
- [16.6 Procedimientos Almacenados (STORED PROCEDURES)](#166-procedimientos-almacenados-stored-procedures)
- [16.7 Funciones PL/SQL](#167-funciones-plsql)
- [16.8 Triggers (Disparadores)](#168-triggers-disparadores)

---

---

## 16.1 Bloques Anónimos PL/SQL (El Hola Mundo)

### 📘 El Concepto

**PL/SQL** significa _Procedural Language / SQL_. Es la extensión de Oracle que convierte SQL de un "idioma de peticiones" a un **lenguaje de programación completo** dentro de la base de datos. Mientras que con SQL puro solo puedes decir "dame estos datos" o "actualiza esta fila", con PL/SQL puedes decir "si pasa esto, haz aquello; si no, haz lo otro; y si algo falla, haz esto otro".

La unidad mínima de PL/SQL es el **bloque anónimo**. Se llama "anónimo" porque no tiene nombre, no se guarda en la base de datos y se ejecuta una sola vez. Es el equivalente a ejecutar un script temporal.

Todo bloque PL/SQL tiene **tres secciones**:

| Sección | Obligatoria | Descripción |
|---------|:-----------:|-------------|
| `DECLARE` | ❌ | Aquí se declaran variables, constantes y cursores |
| `BEGIN ... END;` | ✅ | El código ejecutable — la lógica del programa |
| `EXCEPTION` | ❌ | Manejo de errores (lo veremos en 16.5) |

### 🏠 La Analogía

Imagina que SQL es como un **walkie-talkie**: tú dices una orden y la base de datos responde. Pero solo puedes dar una orden a la vez.

PL/SQL es como darle a la base de datos una **lista de instrucciones escrita en un post-it**: "Primero mira cuántos pedidos hay. Si hay más de 100, avisa al jefe. Si hay menos de 10, manda una alerta. Y si el sistema falla al contar, anota el error en el cuaderno". La base de datos lee el post-it completo y lo ejecuta sin que tú tengas que estar dando órdenes una por una.

### 💻 El Código

```sql
-- El bloque PL/SQL más simple del mundo
BEGIN
  DBMS_OUTPUT.PUT_LINE('¡Hola desde PL/SQL!');
END;
/
```

> 💡 **`DBMS_OUTPUT.PUT_LINE`** es la función de Oracle para imprimir mensajes por consola (equivalente a `print` en Python o `console.log` en JavaScript). La barra `/` al final le dice a Oracle "ejecuta este bloque ahora".

Un bloque más completo con las tres secciones:

```sql
DECLARE
  v_total NUMBER;
BEGIN
  SELECT COUNT(*) INTO v_total FROM productos;
  DBMS_OUTPUT.PUT_LINE('Total de productos: ' || v_total);
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error inesperado: ' || SQLERRM);
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Total de productos: 8
```

### 🧠 El Reto

Escribe un bloque anónimo PL/SQL que cuente el número total de médicos en la tabla `medicos` y muestre el resultado por consola con el formato: `'El hospital tiene X médicos en plantilla'`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
DECLARE
  v_total_medicos NUMBER;
BEGIN
  SELECT COUNT(*) INTO v_total_medicos FROM medicos;
  DBMS_OUTPUT.PUT_LINE('El hospital tiene ' || v_total_medicos || ' médicos en plantilla');
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
El hospital tiene 2 médicos en plantilla
```

> 💡 Recordemos que en el Tema 05 se borró al médico con ID 3, por lo que quedan 2 médicos en la tabla.

</details>

---

---

## 16.2 Variables y Tipos de Datos PL/SQL

### 📘 El Concepto

En PL/SQL, las **variables** se declaran en la sección `DECLARE` y sirven para almacenar valores temporales durante la ejecución del bloque. Cada variable necesita un **nombre** y un **tipo de dato**.

Oracle ofrece dos formas especiales de declarar tipos que evitan errores y hacen tu código más robusto:

| Declaración | Qué hace | Ejemplo |
|-------------|----------|---------|
| `tipo explícito` | Tú defines el tipo manualmente | `v_precio NUMBER(7,2);` |
| `%TYPE` | Copia el tipo de una columna existente | `v_precio productos.precio%TYPE;` |
| `%ROWTYPE` | Copia la estructura de una fila entera | `v_producto productos%ROWTYPE;` |

**Constantes** se declaran con la palabra `CONSTANT` y no se pueden cambiar después:

```sql
c_iva CONSTANT NUMBER := 0.21;
```

### 🏠 La Analogía

Piensa en las variables como **cajas etiquetadas** en un almacén:
- Una variable `NUMBER` es una caja que solo acepta números.
- `%TYPE` es como decir: "Dame una caja exactamente del mismo tamaño que la que usan en el estante de productos para guardar el precio". Si algún día el estante cambia de tamaño, tu caja se adapta sola.
- `%ROWTYPE` es como pedir una **bandeja con compartimentos** que tiene exactamente la misma forma que una fila completa de la mesa del almacén.

### 💻 El Código

```sql
-- Variables con tipos explícitos
DECLARE
  v_nombre   VARCHAR2(50) := 'Laptop Pro';
  v_precio   NUMBER(7,2)  := 1220.00;
  v_en_stock BOOLEAN      := TRUE;  -- BOOLEAN solo existe en PL/SQL, no en SQL puro
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_nombre || ' cuesta ' || v_precio || '€');
END;
/

-- Variables con %TYPE (heredan el tipo de la columna)
DECLARE
  v_nombre productos.nombre%TYPE;
  v_precio productos.precio%TYPE;
BEGIN
  SELECT nombre, precio INTO v_nombre, v_precio
  FROM productos
  WHERE id_producto = 10;

  DBMS_OUTPUT.PUT_LINE('Producto: ' || v_nombre || ' | Precio: ' || v_precio);
END;
/

-- Variable con %ROWTYPE (toda una fila)
DECLARE
  v_avion aviones%ROWTYPE;
BEGIN
  SELECT * INTO v_avion
  FROM aviones
  WHERE id_avion = 10;

  DBMS_OUTPUT.PUT_LINE('Modelo: ' || v_avion.modelo || ' | Capacidad: ' || v_avion.capacidad_pasajeros);
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Laptop Pro cuesta 1220€
Producto: Laptop Pro | Precio: 1220
Modelo: Airbus A320 | Capacidad: 150
```

### 🧠 El Reto

Escribe un bloque PL/SQL que use `%ROWTYPE` para cargar toda la información del paciente con `id_paciente = 1` y muestre por consola: `'Paciente: Laura Martinez | DNI: 11111111A'`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
DECLARE
  v_paciente pacientes%ROWTYPE;
BEGIN
  SELECT * INTO v_paciente
  FROM pacientes
  WHERE id_paciente = 1;

  DBMS_OUTPUT.PUT_LINE('Paciente: ' || v_paciente.nombre || ' | DNI: ' || v_paciente.dni);
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Paciente: Laura Martinez | DNI: 11111111A
```

> 💡 Con `%ROWTYPE` no necesitas declarar una variable por cada columna. `v_paciente` tiene automáticamente todos los campos de la tabla `pacientes` y accedes a ellos con notación de punto.

</details>

---

---

## 16.3 Estructuras de Control (IF, CASE, LOOP)

### 📘 El Concepto

PL/SQL incluye las estructuras de control clásicas de cualquier lenguaje de programación. Son las que le dan el "cerebro" al código:

| Estructura | Uso | Equivalente |
|------------|-----|-------------|
| `IF / ELSIF / ELSE` | Decisiones condicionales | `if / else if / else` en otros lenguajes |
| `CASE` | Selección entre múltiples opciones | `switch` en Java/JS |
| `LOOP` | Bucle infinito (necesita `EXIT`) | `while(true)` |
| `WHILE ... LOOP` | Bucle con condición al inicio | `while` |
| `FOR ... LOOP` | Bucle con contador | `for` |

### 🏠 La Analogía

Las estructuras de control son como el **semáforo de tráfico** de tu programa:
- **IF** es el semáforo básico: rojo o verde, izquierda o derecha.
- **CASE** es la rotonda con múltiples salidas: según tu destino, tomas una u otra.
- **LOOP / FOR / WHILE** son las vueltas que le das al circuito de carreras. Un `FOR` es "da exactamente 10 vueltas". Un `WHILE` es "sigue dando vueltas mientras te quede gasolina". Y un `LOOP` básico es "da vueltas hasta que yo te diga que pares (`EXIT`)".

### 💻 El Código

**IF / ELSIF / ELSE:**
```sql
DECLARE
  v_stock productos.stock%TYPE;
BEGIN
  SELECT stock INTO v_stock
  FROM productos
  WHERE id_producto = 12;  -- Sofá de Cuero

  IF v_stock = 0 THEN
    DBMS_OUTPUT.PUT_LINE('ALERTA: Producto agotado');
  ELSIF v_stock < 20 THEN
    DBMS_OUTPUT.PUT_LINE('AVISO: Stock bajo (' || v_stock || ' unidades)');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Stock OK (' || v_stock || ' unidades)');
  END IF;
END;
/
```

Salida esperada:
```
ALERTA: Producto agotado
```

> 💡 El Sofá de Cuero (id 12) tiene stock 0 según los datos cargados en el Tema 05.

**CASE:**
```sql
DECLARE
  v_segmento clientes.segmento_cliente%TYPE;
  v_nombre   clientes.nombre%TYPE;
BEGIN
  SELECT nombre, segmento_cliente INTO v_nombre, v_segmento
  FROM clientes
  WHERE id_cliente = 1;

  CASE v_segmento
    WHEN 'ALTO_VALOR'  THEN DBMS_OUTPUT.PUT_LINE(v_nombre || ': Cliente VIP — Descuento 20%');
    WHEN 'MEDIO_VALOR' THEN DBMS_OUTPUT.PUT_LINE(v_nombre || ': Cliente Frecuente — Descuento 10%');
    WHEN 'BASE'        THEN DBMS_OUTPUT.PUT_LINE(v_nombre || ': Cliente Estándar — Sin descuento');
    ELSE DBMS_OUTPUT.PUT_LINE(v_nombre || ': Segmento desconocido');
  END CASE;
END;
/
```

**FOR LOOP (con rango numérico):**
```sql
BEGIN
  FOR i IN 1..5 LOOP
    DBMS_OUTPUT.PUT_LINE('Iteración nº ' || i);
  END LOOP;
END;
/
```

**WHILE LOOP:**
```sql
DECLARE
  v_contador NUMBER := 1;
BEGIN
  WHILE v_contador <= 3 LOOP
    DBMS_OUTPUT.PUT_LINE('Vuelta ' || v_contador);
    v_contador := v_contador + 1;
  END LOOP;
END;
/
```

**LOOP con EXIT WHEN:**
```sql
DECLARE
  v_intentos NUMBER := 0;
BEGIN
  LOOP
    v_intentos := v_intentos + 1;
    DBMS_OUTPUT.PUT_LINE('Intento ' || v_intentos);
    EXIT WHEN v_intentos >= 3;
  END LOOP;
  DBMS_OUTPUT.PUT_LINE('Fin: ' || v_intentos || ' intentos realizados');
END;
/
```

### 🧠 El Reto

Escribe un bloque PL/SQL que consulte la capacidad del avión con `id_avion = 10` y clasifique el avión usando `IF/ELSIF/ELSE`:
- Si la capacidad es mayor a 250: `'Avión de fuselaje ancho'`
- Si está entre 100 y 250: `'Avión de fuselaje estrecho'`
- Si es menor a 100: `'Avión regional'`

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
DECLARE
  v_capacidad aviones.capacidad_pasajeros%TYPE;
  v_modelo    aviones.modelo%TYPE;
BEGIN
  SELECT modelo, capacidad_pasajeros INTO v_modelo, v_capacidad
  FROM aviones
  WHERE id_avion = 10;

  IF v_capacidad > 250 THEN
    DBMS_OUTPUT.PUT_LINE(v_modelo || ': Avión de fuselaje ancho');
  ELSIF v_capacidad >= 100 THEN
    DBMS_OUTPUT.PUT_LINE(v_modelo || ': Avión de fuselaje estrecho');
  ELSE
    DBMS_OUTPUT.PUT_LINE(v_modelo || ': Avión regional');
  END IF;
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Airbus A320: Avión de fuselaje estrecho
```

> 💡 El Airbus A320 tiene capacidad de 150 pasajeros, por lo que cae en el rango de 100-250 (fuselaje estrecho).

</details>

---

---

## 16.4 Cursores Explícitos e Implícitos

### 📘 El Concepto

Un **cursor** es un puntero a un conjunto de resultados devuelto por una consulta. Es la forma en la que PL/SQL te deja recorrer filas **una por una**, procesando cada registro individualmente.

| Tipo | Qué es | Cuándo usarlo |
|------|--------|---------------|
| **Cursor implícito** | Oracle lo crea automáticamente en cada `SELECT INTO`, `INSERT`, `UPDATE`, `DELETE` | Cuando esperas **una sola fila** o no necesitas iterar |
| **Cursor explícito** | Tú lo declaras, abres, recorres y cierras manualmente | Cuando necesitas recorrer **múltiples filas** una por una |

**Atributos de cursores** (funcionan en ambos tipos):

| Atributo | Significado |
|----------|-------------|
| `%FOUND` | `TRUE` si la última operación encontró/afectó filas |
| `%NOTFOUND` | `TRUE` si la última operación NO encontró/afectó filas |
| `%ROWCOUNT` | Número de filas encontradas/afectadas hasta el momento |
| `%ISOPEN` | `TRUE` si el cursor está abierto (solo explícitos) |

### 🏠 La Analogía

Imagina que tienes un archivador lleno de fichas de pacientes:
- Un **cursor implícito** es como decir "dame la ficha del paciente nº 1". Abres el cajón, sacas UNA ficha y lo cierras. Rápido y directo.
- Un **cursor explícito** es como decir "voy a revisar TODAS las fichas de pacientes con citas pendientes". Abres el cajón (`OPEN`), vas sacando fichas una por una (`FETCH`), las revisas, y cuando llegas al final cierras el cajón (`CLOSE`).

La forma moderna y elegante de recorrer un cursor explícito es con un **FOR LOOP de cursor**, que abre, recorre y cierra automáticamente. Es como tener un asistente que va pasándote las fichas sin que tú tengas que gestionar el cajón.

### 💻 El Código

**Cursor implícito (con atributos SQL%):**
```sql
BEGIN
  UPDATE productos
  SET stock = stock - 1
  WHERE id_producto = 10 AND stock > 0;

  IF SQL%FOUND THEN
    DBMS_OUTPUT.PUT_LINE('Stock actualizado. Filas afectadas: ' || SQL%ROWCOUNT);
  ELSE
    DBMS_OUTPUT.PUT_LINE('No se pudo actualizar: producto agotado o inexistente');
  END IF;

  ROLLBACK;  -- Revertimos para no alterar el estado del curso
END;
/
```

Salida esperada:
```
Stock actualizado. Filas afectadas: 1
```

**Cursor explícito (forma clásica: OPEN / FETCH / CLOSE):**
```sql
DECLARE
  CURSOR c_aviones IS
    SELECT modelo, capacidad_pasajeros
    FROM aviones
    ORDER BY capacidad_pasajeros DESC;

  v_modelo    aviones.modelo%TYPE;
  v_capacidad aviones.capacidad_pasajeros%TYPE;
BEGIN
  OPEN c_aviones;
  LOOP
    FETCH c_aviones INTO v_modelo, v_capacidad;
    EXIT WHEN c_aviones%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(v_modelo || ' — ' || v_capacidad || ' pasajeros');
  END LOOP;
  CLOSE c_aviones;
END;
/
```

Salida esperada:
```
Airbus A350 — 300 pasajeros
Boeing 737 — 180 pasajeros
Airbus A320 — 150 pasajeros
```

**Cursor explícito (forma moderna: FOR LOOP — la recomendada):**
```sql
DECLARE
  CURSOR c_productos_electronicos IS
    SELECT nombre, precio, stock
    FROM productos
    WHERE id_categoria = 1
    ORDER BY precio DESC;
BEGIN
  FOR r IN c_productos_electronicos LOOP
    DBMS_OUTPUT.PUT_LINE(r.nombre || ' | ' || r.precio || '€ | Stock: ' || r.stock);
  END LOOP;
END;
/
```

Salida esperada:
```
Laptop Pro | 1220€ | Stock: 50
Monitor 4K | 350€ | Stock: 25
Teclado Mecánico | 75€ | Stock: 80
Ratón Inalámbrico | 45.5€ | Stock: 200
```

> 💡 El `FOR r IN cursor LOOP` es la forma **recomendada** porque Oracle gestiona automáticamente el `OPEN`, `FETCH` y `CLOSE`. Menos código, menos errores.

### 🧠 El Reto

Escribe un bloque PL/SQL con un cursor explícito (usando `FOR LOOP`) que recorra todas las especialidades médicas y muestre: `'Especialidad: <nombre>'` para cada una.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
DECLARE
  CURSOR c_especialidades IS
    SELECT nombre_especialidad
    FROM especialidades
    ORDER BY nombre_especialidad;
BEGIN
  FOR r IN c_especialidades LOOP
    DBMS_OUTPUT.PUT_LINE('Especialidad: ' || r.nombre_especialidad);
  END LOOP;
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Especialidad: Medicina General
Especialidad: Neurología
Especialidad: Traumatología
```

> 💡 Aunque el médico de Medicina General fue eliminado en el Tema 05, la especialidad sigue existiendo en la tabla `especialidades` porque solo se borró el registro del médico, no la especialidad.

</details>

---

---

## 16.5 Manejo de Excepciones

### 📘 El Concepto

En programación, los errores ocurren. Un dato que no existe, una división entre cero, una inserción duplicada... PL/SQL te permite **atrapar y manejar estos errores** con la sección `EXCEPTION`, evitando que tu bloque explote y se detenga sin control.

Las excepciones más comunes predefinidas por Oracle:

| Excepción | Cuándo ocurre |
|-----------|---------------|
| `NO_DATA_FOUND` | Un `SELECT INTO` no devuelve ninguna fila |
| `TOO_MANY_ROWS` | Un `SELECT INTO` devuelve más de una fila |
| `ZERO_DIVIDE` | División entre cero |
| `DUP_VAL_ON_INDEX` | Insertas un valor duplicado en una columna `UNIQUE` o `PRIMARY KEY` |
| `VALUE_ERROR` | Error de conversión o truncamiento de datos |
| `OTHERS` | Comodín que atrapa cualquier excepción no capturada |

**Funciones de diagnóstico:**
- `SQLCODE`: Devuelve el código numérico del error.
- `SQLERRM`: Devuelve el mensaje descriptivo del error.

### 🏠 La Analogía

Piensa en las excepciones como la **red de seguridad de un trapecista**. El trapecista (tu código) hace sus acrobacias arriba (`BEGIN`). Si todo sale bien, aterriza perfecto. Pero si se resbala (un error), la red (`EXCEPTION`) lo atrapa y le permite caer de forma controlada en lugar de estrellarse contra el suelo.

Sin la sección `EXCEPTION`, tu bloque PL/SQL se detiene abruptamente y Oracle lanza el error al usuario. Con ella, tú decides qué hacer: mostrar un mensaje amigable, registrar el error en una tabla de log, o intentar una operación alternativa.

### 💻 El Código

**Capturar `NO_DATA_FOUND`:**
```sql
DECLARE
  v_nombre productos.nombre%TYPE;
BEGIN
  SELECT nombre INTO v_nombre
  FROM productos
  WHERE id_producto = 99999;  -- No existe

  DBMS_OUTPUT.PUT_LINE('Producto: ' || v_nombre);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('ERROR: El producto solicitado no existe en el catálogo');
END;
/
```

Salida esperada:
```
ERROR: El producto solicitado no existe en el catálogo
```

**Capturar `TOO_MANY_ROWS`:**
```sql
DECLARE
  v_nombre productos.nombre%TYPE;
BEGIN
  -- Esto fallará si hay más de un producto en categoría 1
  SELECT nombre INTO v_nombre
  FROM productos
  WHERE id_categoria = 1;

  DBMS_OUTPUT.PUT_LINE(v_nombre);
EXCEPTION
  WHEN TOO_MANY_ROWS THEN
    DBMS_OUTPUT.PUT_LINE('ERROR: La consulta devolvió múltiples filas. Usa un cursor.');
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('ERROR: No se encontraron productos en esa categoría.');
END;
/
```

Salida esperada:
```
ERROR: La consulta devolvió múltiples filas. Usa un cursor.
```

**Capturar `DUP_VAL_ON_INDEX` y usar `OTHERS` como red final:**
```sql
BEGIN
  -- Intentamos insertar una categoría que ya existe
  INSERT INTO categorias (id_categoria, nombre_categoria)
  VALUES (1, 'Electrónica Duplicada');

  DBMS_OUTPUT.PUT_LINE('Categoría insertada correctamente');
EXCEPTION
  WHEN DUP_VAL_ON_INDEX THEN
    DBMS_OUTPUT.PUT_LINE('ERROR: Ya existe una categoría con ese ID');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('ERROR INESPERADO [' || SQLCODE || ']: ' || SQLERRM);
END;
/
```

Salida esperada:
```
ERROR: Ya existe una categoría con ese ID
```

### 🧠 El Reto

Escribe un bloque PL/SQL que intente obtener el modelo del avión con `id_avion = 777`. Como no existe, debe capturar el error `NO_DATA_FOUND` y mostrar: `'No existe ningún avión con ese identificador'`. Además, añade un bloque `WHEN OTHERS` que muestre el código y mensaje del error para cualquier excepción inesperada.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
DECLARE
  v_modelo aviones.modelo%TYPE;
BEGIN
  SELECT modelo INTO v_modelo
  FROM aviones
  WHERE id_avion = 777;

  DBMS_OUTPUT.PUT_LINE('Modelo: ' || v_modelo);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('No existe ningún avión con ese identificador');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error inesperado [' || SQLCODE || ']: ' || SQLERRM);
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
No existe ningún avión con ese identificador
```

> 💡 El `WHEN OTHERS` actúa como la última línea de defensa. Si Oracle lanzara un error distinto a `NO_DATA_FOUND` (por ejemplo, un problema de permisos), quedaría registrado con su código y mensaje.

</details>

---

---

## 16.6 Procedimientos Almacenados (STORED PROCEDURES)

### 📘 El Concepto

Un **procedimiento almacenado** (Stored Procedure) es un bloque PL/SQL con nombre que se **guarda en la base de datos** y se puede ejecutar tantas veces como quieras. A diferencia de un bloque anónimo (que desaparece tras ejecutarse), un procedimiento vive permanentemente en el esquema de la base de datos.

Los procedimientos pueden recibir **parámetros**:

| Modo | Descripción |
|------|-------------|
| `IN` | Parámetro de entrada (valor que le pasas al procedimiento). Es el modo **por defecto**. |
| `OUT` | Parámetro de salida (el procedimiento devuelve un valor a través de él). |
| `IN OUT` | Parámetro de entrada y salida (entra con un valor y sale modificado). |

**Sintaxis base:**
```sql
CREATE OR REPLACE PROCEDURE nombre_procedimiento (
  p_parametro1 IN tipo_dato,
  p_parametro2 OUT tipo_dato
) AS
  -- Sección de declaración de variables locales
BEGIN
  -- Lógica
END nombre_procedimiento;
/
```

### 🏠 La Analogía

Si un bloque anónimo es un **post-it** con instrucciones que tiras a la basura después de leerlo, un procedimiento almacenado es una **receta guardada en tu libro de cocina**.

La escribes una vez (`CREATE PROCEDURE`), le pones un nombre ("Receta de Paella"), y luego la ejecutas siempre que quieras con `EXECUTE nombre_receta`. Además puedes pasarle "ingredientes" (parámetros): hoy la haces para 4 personas (`p_comensales => 4`), mañana para 8.

### 💻 El Código

**Procedimiento simple sin parámetros:**
```sql
CREATE OR REPLACE PROCEDURE sp_listar_aviones AS
  CURSOR c_aviones IS
    SELECT modelo, capacidad_pasajeros, anio_fabricacion
    FROM aviones
    ORDER BY anio_fabricacion DESC;
BEGIN
  DBMS_OUTPUT.PUT_LINE('=== FLOTA DE AVIONES ===');
  FOR r IN c_aviones LOOP
    DBMS_OUTPUT.PUT_LINE(r.modelo || ' | Cap: ' || r.capacidad_pasajeros || ' | Año: ' || r.anio_fabricacion);
  END LOOP;
END sp_listar_aviones;
/

-- Ejecución:
EXECUTE sp_listar_aviones;
```

Salida esperada:
```
=== FLOTA DE AVIONES ===
Airbus A350 | Cap: 300 | Año: 2022
Boeing 737 | Cap: 180 | Año: 2018
Airbus A320 | Cap: 150 | Año: 2015
```

**Procedimiento con parámetros IN y OUT:**
```sql
CREATE OR REPLACE PROCEDURE sp_info_producto (
  p_id_producto IN productos.id_producto%TYPE,
  p_nombre      OUT productos.nombre%TYPE,
  p_precio      OUT productos.precio%TYPE
) AS
BEGIN
  SELECT nombre, precio INTO p_nombre, p_precio
  FROM productos
  WHERE id_producto = p_id_producto;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    p_nombre := 'NO ENCONTRADO';
    p_precio := 0;
END sp_info_producto;
/

-- Ejecución con bloque anónimo para ver los resultados:
DECLARE
  v_nombre productos.nombre%TYPE;
  v_precio productos.precio%TYPE;
BEGIN
  sp_info_producto(10, v_nombre, v_precio);
  DBMS_OUTPUT.PUT_LINE('Producto: ' || v_nombre || ' | Precio: ' || v_precio || '€');

  sp_info_producto(99999, v_nombre, v_precio);
  DBMS_OUTPUT.PUT_LINE('Producto: ' || v_nombre || ' | Precio: ' || v_precio || '€');
END;
/
```

Salida esperada:
```
Producto: Laptop Pro | Precio: 1220€
Producto: NO ENCONTRADO | Precio: 0€
```

**Procedimiento para refrescar KPIs (conectando con el Tema 15):**
```sql
CREATE OR REPLACE PROCEDURE sp_refrescar_kpis_mensuales AS
  v_filas NUMBER;
BEGIN
  MERGE INTO kpi_ventas_mensuales k
  USING (
    SELECT TRUNC(fecha_pedido, 'MM') AS mes,
           COUNT(*) AS total_pedidos,
           SUM(total) AS facturacion_total,
           ROUND(AVG(total), 2) AS ticket_medio,
           COUNT(DISTINCT id_cliente) AS clientes_activos
    FROM pedidos
    GROUP BY TRUNC(fecha_pedido, 'MM')
  ) s
  ON (k.mes = s.mes)
  WHEN MATCHED THEN UPDATE SET
    k.total_pedidos     = s.total_pedidos,
    k.facturacion_total = s.facturacion_total,
    k.ticket_medio      = s.ticket_medio,
    k.clientes_activos  = s.clientes_activos,
    k.actualizado_en    = SYSDATE
  WHEN NOT MATCHED THEN INSERT (
    mes, total_pedidos, facturacion_total, ticket_medio, clientes_activos
  ) VALUES (
    s.mes, s.total_pedidos, s.facturacion_total, s.ticket_medio, s.clientes_activos
  );

  v_filas := SQL%ROWCOUNT;
  DBMS_OUTPUT.PUT_LINE('KPIs actualizados: ' || v_filas || ' meses procesados');
  COMMIT;
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error al refrescar KPIs: ' || SQLERRM);
    ROLLBACK;
END sp_refrescar_kpis_mensuales;
/
```

### 🧠 El Reto

Crea un procedimiento almacenado llamado `sp_buscar_medico` que reciba como parámetro `IN` un `id_medico` y muestre por consola el nombre completo y su especialidad (haciendo JOIN con la tabla `especialidades`). Si el médico no existe, debe mostrar `'Médico no encontrado'`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
CREATE OR REPLACE PROCEDURE sp_buscar_medico (
  p_id_medico IN medicos.id_medico%TYPE
) AS
  v_nombre       medicos.nombre_completo%TYPE;
  v_especialidad especialidades.nombre_especialidad%TYPE;
BEGIN
  SELECT m.nombre_completo, e.nombre_especialidad
  INTO v_nombre, v_especialidad
  FROM medicos m
  JOIN especialidades e ON m.id_especialidad = e.id_especialidad
  WHERE m.id_medico = p_id_medico;

  DBMS_OUTPUT.PUT_LINE('Médico: ' || v_nombre || ' | Especialidad: ' || v_especialidad);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('Médico no encontrado');
END sp_buscar_medico;
/

-- Pruebas:
EXECUTE sp_buscar_medico(1);
EXECUTE sp_buscar_medico(999);
```

Salida esperada por consola (DBMS_OUTPUT):
```
Médico: Dr. John Smith | Especialidad: Traumatología
Médico no encontrado
```

</details>

---

---

## 16.7 Funciones PL/SQL

### 📘 El Concepto

Una **función** PL/SQL es similar a un procedimiento, pero tiene una diferencia clave: siempre **devuelve un valor** usando la cláusula `RETURN`. Esto permite que las funciones se puedan usar directamente dentro de sentencias SQL, cosa que los procedimientos no pueden hacer.

| Característica | Procedimiento | Función |
|----------------|:------------:|:------:|
| Devuelve un valor | ❌ (usa `OUT`) | ✅ (con `RETURN`) |
| Se puede usar en un `SELECT` | ❌ | ✅ |
| Puede modificar datos (`INSERT/UPDATE/DELETE`) | ✅ | ⚠️ No recomendado si se usa en `SELECT` |
| Ejecutar desde SQL*Plus | `EXECUTE proc(...)` | `SELECT func(...) FROM DUAL;` |

### 🏠 La Analogía

Si el procedimiento almacenado es una **receta de cocina** que haces paso a paso, la función es una **máquina expendedora**: metes una moneda (parámetro de entrada), le das al botón, y siempre sale un producto (el valor de retorno). No necesitas ver las tripas de la máquina, solo te importa el resultado.

Y lo mejor es que puedes poner esa máquina expendedora en cualquier sitio, incluso dentro de una consulta `SELECT`, porque siempre produce un resultado limpio.

### 💻 El Código

**Función simple: calcular el precio con IVA:**
```sql
CREATE OR REPLACE FUNCTION fn_precio_con_iva (
  p_precio IN NUMBER
) RETURN NUMBER AS
  c_iva CONSTANT NUMBER := 0.21;
BEGIN
  RETURN ROUND(p_precio * (1 + c_iva), 2);
END fn_precio_con_iva;
/

-- Uso directo en un SELECT (¡esto NO se puede hacer con procedimientos!)
SELECT nombre, precio, fn_precio_con_iva(precio) AS precio_con_iva
FROM productos
WHERE id_categoria = 1
ORDER BY precio DESC;
```

Resultado:
| nombre | precio | precio_con_iva |
|--------|--------|----------------|
| Laptop Pro | 1220 | 1476.20 |
| Monitor 4K | 350 | 423.50 |
| Teclado Mecánico | 75 | 90.75 |
| Ratón Inalámbrico | 45.5 | 55.06 |

**Función que consulta la base de datos:**
```sql
CREATE OR REPLACE FUNCTION fn_nombre_especialidad (
  p_id_especialidad IN especialidades.id_especialidad%TYPE
) RETURN VARCHAR2 AS
  v_nombre especialidades.nombre_especialidad%TYPE;
BEGIN
  SELECT nombre_especialidad INTO v_nombre
  FROM especialidades
  WHERE id_especialidad = p_id_especialidad;

  RETURN v_nombre;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RETURN 'Especialidad no registrada';
END fn_nombre_especialidad;
/

-- Usarla en una consulta real:
SELECT nombre_completo, fn_nombre_especialidad(id_especialidad) AS especialidad
FROM medicos;
```

Resultado:
| nombre_completo | especialidad |
|-----------------|-------------|
| Dr. John Smith | Traumatología |
| Dra. Sarah Adams | Neurología |

**Función para clasificar aviones:**
```sql
CREATE OR REPLACE FUNCTION fn_tipo_avion (
  p_capacidad IN NUMBER
) RETURN VARCHAR2 AS
BEGIN
  IF p_capacidad > 250 THEN
    RETURN 'Fuselaje Ancho';
  ELSIF p_capacidad >= 100 THEN
    RETURN 'Fuselaje Estrecho';
  ELSE
    RETURN 'Regional';
  END IF;
END fn_tipo_avion;
/

SELECT modelo, capacidad_pasajeros, fn_tipo_avion(capacidad_pasajeros) AS tipo
FROM aviones
ORDER BY capacidad_pasajeros DESC;
```

Resultado:
| modelo | capacidad_pasajeros | tipo |
|--------|---------------------|------|
| Airbus A350 | 300 | Fuselaje Ancho |
| Boeing 737 | 180 | Fuselaje Estrecho |
| Airbus A320 | 150 | Fuselaje Estrecho |

### 🧠 El Reto

Crea una función llamada `fn_valor_stock` que reciba un `id_producto` y devuelva el valor total del stock de ese producto (`precio * stock`). Luego úsala en un `SELECT` sobre la tabla `productos` para ver el valor de inventario de cada producto de categoría 1.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
CREATE OR REPLACE FUNCTION fn_valor_stock (
  p_id_producto IN productos.id_producto%TYPE
) RETURN NUMBER AS
  v_valor NUMBER;
BEGIN
  SELECT precio * stock INTO v_valor
  FROM productos
  WHERE id_producto = p_id_producto;

  RETURN ROUND(v_valor, 2);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RETURN 0;
END fn_valor_stock;
/

-- Consulta:
SELECT nombre, precio, stock, fn_valor_stock(id_producto) AS valor_inventario
FROM productos
WHERE id_categoria = 1
ORDER BY valor_inventario DESC;
```

Resultado:
| nombre | precio | stock | valor_inventario |
|--------|--------|-------|------------------|
| Laptop Pro | 1220 | 50 | 61000.00 |
| Monitor 4K | 350 | 25 | 8750.00 |
| Teclado Mecánico | 75 | 80 | 6000.00 |
| Ratón Inalámbrico | 45.5 | 200 | 9100.00 |

> 💡 La función puede usarse directamente en el `SELECT` como si fuera una columna calculada. Esto es imposible con un procedimiento.

</details>

---

---

## 16.8 Triggers (Disparadores)

### 📘 El Concepto

Un **trigger** (disparador) es un bloque PL/SQL que se ejecuta **automáticamente** cuando ocurre un evento determinado en una tabla. Tú no lo llamas manualmente; Oracle lo dispara solo.

Los triggers se definen en función de **tres ejes**:

| Eje | Opciones |
|-----|----------|
| **Evento** | `INSERT`, `UPDATE`, `DELETE` (o combinaciones) |
| **Momento** | `BEFORE` (antes del evento) o `AFTER` (después del evento) |
| **Nivel** | `FOR EACH ROW` (se ejecuta una vez por cada fila afectada) o nivel de sentencia (se ejecuta una sola vez por sentencia) |

Dentro de un trigger a nivel de fila, puedes acceder a los valores con:
- `:NEW.columna` — el valor nuevo que se va a insertar o actualizar.
- `:OLD.columna` — el valor viejo que existía antes del cambio.

| Evento | `:OLD` disponible | `:NEW` disponible |
|--------|:-----------------:|:-----------------:|
| `INSERT` | ❌ | ✅ |
| `UPDATE` | ✅ | ✅ |
| `DELETE` | ✅ | ❌ |

### 🏠 La Analogía

Un trigger es como una **cámara de seguridad con alarma** en una tienda. Tú no tienes que activarla manualmente cada vez que entra alguien. Está programada para que, **automáticamente**, cuando alguien cruza la puerta (`INSERT`), cuando alguien mueve un producto de sitio (`UPDATE`), o cuando alguien saca algo (`DELETE`), la cámara grabe el evento y, si detecta algo raro, haga sonar la alarma.

Lo potente es que el trigger funciona **sin que nadie se acuerde de activarlo**. Un desarrollador nuevo que haga un `INSERT` en la tabla ni siquiera necesita saber que el trigger existe — se ejecutará igual.

### 💻 El Código

**Trigger de auditoría: registrar cambios de precio:**

Primero creamos la tabla de auditoría:
```sql
CREATE TABLE log_cambios_precio (
  id_log        NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  id_producto   NUMBER NOT NULL,
  precio_antiguo NUMBER(7,2),
  precio_nuevo   NUMBER(7,2),
  fecha_cambio   DATE DEFAULT SYSDATE,
  usuario        VARCHAR2(50) DEFAULT USER
);
```

Ahora el trigger:
```sql
CREATE OR REPLACE TRIGGER trg_auditoria_precio
AFTER UPDATE OF precio ON productos
FOR EACH ROW
WHEN (OLD.precio != NEW.precio)
BEGIN
  INSERT INTO log_cambios_precio (id_producto, precio_antiguo, precio_nuevo)
  VALUES (:OLD.id_producto, :OLD.precio, :NEW.precio);
END trg_auditoria_precio;
/
```

Probamos el trigger:
```sql
UPDATE productos SET precio = 1300 WHERE id_producto = 10;

SELECT * FROM log_cambios_precio;
```

Resultado:
| id_log | id_producto | precio_antiguo | precio_nuevo | fecha_cambio | usuario |
|--------|------------|----------------|--------------|-------------|---------|
| 1 | 10 | 1220 | 1300 | 2025-XX-XX | SYSTEM |

```sql
ROLLBACK;  -- Revertimos para no alterar el estado del curso
```

**Trigger BEFORE para validar datos antes de insertar:**
```sql
CREATE OR REPLACE TRIGGER trg_validar_capacidad_avion
BEFORE INSERT OR UPDATE OF capacidad_pasajeros ON aviones
FOR EACH ROW
BEGIN
  IF :NEW.capacidad_pasajeros <= 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'La capacidad del avión debe ser mayor que cero');
  ELSIF :NEW.capacidad_pasajeros > 850 THEN
    RAISE_APPLICATION_ERROR(-20002, 'La capacidad máxima permitida es 850 pasajeros (A380)');
  END IF;
END trg_validar_capacidad_avion;
/
```

Probamos:
```sql
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros, anio_fabricacion)
VALUES (99, 'Avión Fantasma', -5, 2024);
```

Error esperado:
```
ORA-20001: La capacidad del avión debe ser mayor que cero
```

> 💡 `RAISE_APPLICATION_ERROR` permite lanzar errores personalizados con códigos entre -20000 y -20999.

**Trigger para mantener estado automáticamente:**
```sql
CREATE OR REPLACE TRIGGER trg_alerta_stock_cero
AFTER UPDATE OF stock ON productos
FOR EACH ROW
WHEN (NEW.stock = 0 AND OLD.stock > 0)
BEGIN
  INSERT INTO alertas_negocio (id_alerta, tipo_alerta, referencia, detalle)
  VALUES (
    (SELECT NVL(MAX(id_alerta), 0) + 1 FROM alertas_negocio),
    'STOCK_AGOTADO',
    'Producto ID: ' || :NEW.id_producto,
    'El producto "' || :NEW.nombre || '" se ha quedado sin stock'
  );
END trg_alerta_stock_cero;
/
```

### 🧠 El Reto

Crea un trigger llamado `trg_log_nuevo_paciente` que se dispare **después** de cada `INSERT` en la tabla `pacientes` y que registre un mensaje en `DBMS_OUTPUT` con el formato: `'Nuevo paciente registrado: <nombre> (DNI: <dni>)'`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
CREATE OR REPLACE TRIGGER trg_log_nuevo_paciente
AFTER INSERT ON pacientes
FOR EACH ROW
BEGIN
  DBMS_OUTPUT.PUT_LINE('Nuevo paciente registrado: ' || :NEW.nombre || ' (DNI: ' || :NEW.dni || ')');
END trg_log_nuevo_paciente;
/

-- Prueba:
INSERT INTO pacientes (id_paciente, dni, nombre, fecha_nacimiento, telefono)
VALUES (99, '99999999Z', 'Test Trigger', TO_DATE('2000-01-01', 'YYYY-MM-DD'), '555-9999');
```

Salida esperada por consola (DBMS_OUTPUT):
```
Nuevo paciente registrado: Test Trigger (DNI: 99999999Z)
```

```sql
ROLLBACK;  -- Revertimos la inserción de prueba
```

> 💡 El trigger se ejecuta automáticamente sin que el usuario que hizo el `INSERT` tenga que hacer nada especial. Es invisible para quien inserta datos, pero registra el evento.

</details>

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 16

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de PL/SQL](./ejercicios/ejercicios_plsql.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Proyecto: Automatización Operativa con PL/SQL](./proyectos/proyecto_plsql.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 17: Optimización de Consultas](../17-optimizacion) |

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 15**](../15-sql-avanzado/proyectos/proyecto_sql_avanzado.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 16 →**](ejercicios/ejercicios_plsql.md)

</div>
