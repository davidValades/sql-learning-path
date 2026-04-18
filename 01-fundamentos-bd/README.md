## 1.1 ¿Qué es una BD, SGBD y SGBDR? (DB / DBMS / RDBMS)

### 📘 El Concepto

Para entender esto, debemos distinguir tres capas de "inteligencia":

1. **BD (Base de Datos / Database):** Es simplemente el **contenedor**. Es el lugar físico o virtual donde residen los datos organizados.
2. **SGBD (Sistema de Gestión de Bases de Datos / DBMS):** Es el **software** que actúa como intermediario entre tú y los datos. Sin él, los datos serían solo bits ilegibles en un disco duro. El SGBD permite leer, borrar o actualizar la información de forma segura.
3. **SGBDR (Sistema de Gestión de Bases de Datos Relacionales / RDBMS):** Es un tipo específico de SGBD (el más usado en el mundo) que organiza los datos en **tablas que se relacionan entre sí** mediante puntos comunes.

### 🏠 La Analogía

Imagina que quieres montar una biblioteca en casa:

- **La Base de Datos (BD)** son las estanterías físicas y los libros que guardas en ellas.
- **El SGBD (DBMS)** es el **bibliotecario**. Tú no vas directo a los estantes; le pides al bibliotecario: "Búscame el libro de Don Quijote", y él sabe exactamente dónde está y cómo dártelo sin desordenar nada.
- **El SGBDR (RDBMS)** es un sistema de organización donde los libros están conectados. Por ejemplo, el libro tiene un "ID de Autor" y, si vas a la sección de "Autores", ese mismo ID te da la biografía de Cervantes. **Todo está relacionado.**

### 💻 El Código

Aunque SQL es el lenguaje que usamos para hablar con estos sistemas, a este nivel conceptual lo que hacemos es definir qué motor estamos usando. Un ejemplo de cómo "pedimos" algo al RDBMS usando SQL sería:

```sql
-- Esto es una instrucción para que el SGBDR busque en una tabla
SELECT nombre_libro
FROM biblioteca
WHERE autor = 'Miguel de Cervantes';
```

### 🧠 El Reto de la Lección

magina que estás diseñando el sistema de una aplicación como Spotify. Tienes por un lado una lista de "Canciones" y por otro una lista de "Artistas".

**Pregunta:** ¿Qué es exactamente lo que permite que el sistema sepa qué canciones pertenecen a qué artista y por qué decimos que es "Relacional"?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>
Lo que permite esto es un "punto común" entre ambas listas. En bases de datos, le damos a cada Artista un identificador único (ej: ID_Artista = 1 para Rosalía). En la lista de Canciones, añadimos una columna llamada "ID_Artista".

Decimos que es <b>Relacional</b> porque no guardamos toda la biografía de Rosalía en cada una de sus canciones (eso sería repetir datos). Simplemente guardamos su ID, creando una <b>relación</b> entre la tabla "Canciones" y la tabla "Artistas".

</details>

---

---

## 1.2 Tablas, Filas y Columnas

### 📘 El Concepto

En una base de datos relacional, toda la información se guarda de forma cuadriculada.

- **La Tabla (Table / Entity):** Es la estructura principal que guarda información sobre _un solo tipo de cosa_ (ej. tabla de Usuarios, tabla de Pedidos).
- **Las Columnas (Columns / Attributes / Fields):** Definen **qué** información vamos a guardar. Son las características. Una columna de "Edad" solo aceptará números, por ejemplo.
- **Las Filas (Rows / Records / Tuples):** Son los datos en sí. Cada fila representa **un elemento único** y real dentro de esa tabla.

### 🏠 La Analogía

Abre tu mente y piensa en una simple hoja de cálculo de **Excel**.
La pestaña de abajo que dice "Hoja 1", sería tu **Tabla**.
La primera fila de arriba que pones en negrita (Nombre, Apellido, Teléfono) son tus **Columnas**. Definen las reglas.
Cada amigo nuevo que añades a tu lista ocupando una línea horizontal nueva, es una **Fila**.

### 💻 El Código

En SQL, definimos la estructura de la tabla indicando primero el nombre de las columnas.

```sql
-- Creamos el 'molde' o la tabla vacía
CREATE TABLE clientes (
    id_cliente INT,      -- Esto es una columna
    nombre VARCHAR,      -- Esto es una columna
    pais VARCHAR         -- Esto es una columna
);
```

### 🧠 El Reto de la Lección

Imagina que en una tienda online creamos una tabla llamada Productos. En esa tabla decidimos que queremos guardar el Código del producto, el Nombre del producto, el Precio y el Stock disponible. Después de un mes, la tienda ha registrado 150 productos diferentes en su inventario.

**Pregunta:** ¿Cuántas columnas y cuántas filas tiene exactamente nuestra tabla Productos en este momento?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>
La tabla tiene exactamente <b>4 columnas</b> (Código, Nombre, Precio, Stock) y <b>150 filas</b> (una fila por cada producto registrado en el inventario).

</details>

---

---

## 1.3 Clave Primaria (Primary Key)

### 📘 El Concepto

Una Clave Primaria (abreviada como PK) es una columna (o un grupo de columnas) que **identifica de forma única y exclusiva a cada fila** dentro de una tabla.
Para que una columna pueda ser coronada como "Clave Primaria", debe cumplir dos reglas sagradas en el mundo de los datos:

1.  **NUNCA puede estar vacía** (tiene que tener un valor sí o sí).
2.  **NUNCA puede repetirse** (el valor tiene que ser único en toda la tabla).

### 🏠 La Analogía

Piensa en los ciudadanos de un país. Pueden existir miles de personas que se llamen "David García" y que hayan nacido el mismo día. Si el gobierno necesita buscarte, buscar por nombre sería un caos.
¿Cuál es la solución? **El DNI o Pasaporte**.
Ese número es tu Clave Primaria en la "tabla" de ciudadanos. Nadie más en el país tiene tu mismo número de DNI, y no puedes existir legalmente sin tener uno.

### 💻 El Código

Cuando creamos una tabla en SQL, podemos decirle al motor cuál será nuestra columna especial usando las palabras mágicas `PRIMARY KEY`.

```sql
CREATE TABLE alumnos (
    dni_alumno VARCHAR PRIMARY KEY,  -- ¡Esta es nuestra clave principal!
    nombre VARCHAR,
    fecha_nacimiento DATE
);
```

### 🧠 El Reto de la Lección

Estás diseñando la base de datos para una nueva red social. Tienes una tabla llamada Usuarios_App con las siguientes columnas: Nombre_Completo, Ciudad, Email_Registro y Edad.

**Pregunta:** Si tuvieras que elegir una de estas columnas para que sea tu Clave Primaria... ¿Cuál elegirías y por qué?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>
La única opción válida de esa lista es el <b>Email_Registro</b>. Esto se debe a que es la única columna donde los datos nunca se van a repetir (dos personas no pueden compartir el mismo email) y no puede estar vacía al registrarse. A esto se le llama "Clave Natural".

<i>Nota Pro: En la vida real, los ingenieros suelen inventar una columna nueva (ej. ID_Usuario numérico) para que las búsquedas sean más rápidas, lo que se conoce como "Clave Sustituta".</i>

</details>

---

---

## 1.4 Clave Foránea (Foreign Key)

### 📘 El Concepto

La Clave Foránea (abreviada como FK) es la columna que **crea el puente o la relación entre dos tablas distintas**.
Dicho de forma muy simple: una Clave Foránea es simplemente la Clave Primaria de _otra_ tabla que ha venido de "visita" a la nuestra.

### 🏠 La Analogía

Imagina un bloque de apartamentos. Cada puerta de apartamento tiene un número único grabado en bronce (Apartamento 1, Apartamento 2...). Esa es la **Clave Primaria** del apartamento.

Tú llevas en tu bolsillo una llave física. Esa llave es tu **Clave Foránea**. La llave que llevas en el bolsillo tiene que tener exactamente la misma forma que la cerradura del apartamento para poder entrar. Tu llave "hace referencia" a la puerta de ese apartamento específico.

### 💻 El Código

En SQL, cuando creamos una tabla, le decimos "Oye, esta columna no es mía, viene de otra tabla". Lo hacemos con `FOREIGN KEY` y `REFERENCES` (hace referencia a).

```sql
CREATE TABLE pedidos (
    id_pedido INT PRIMARY KEY,       -- Mi propia clave primaria
    fecha DATE,
    id_cliente INT,                  -- Esta columna viene de visita
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente)
    -- ^ Aquí le decimos que id_cliente es la llave de la tabla 'clientes'
);
```

### 🧠 El Reto de la Lección

Vamos a volver a nuestro ejemplo favorito: la aplicación de Spotify.

Tenemos la tabla Artistas (donde su Clave Primaria es ID_Artista).
Tenemos la tabla Canciones (donde su Clave Primaria es ID_Cancion).

Para saber qué canción canta Rosalía, decidimos meter la columna ID_Artista dentro de la tabla Canciones.

**Pregunta:** Dentro de la tabla Canciones... ¿Qué título o "cargo" ostenta exactamente esa columna ID_Artista?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>
Esa columna actúa como <b>Clave Foránea (Foreign Key)</b>. Es la clave primaria de otra tabla que está "de visita" en nuestra tabla de canciones para establecer la relación entre ambas.

</details>

---

---

## 1.5 Clave Candidata (Candidate Key)

### 📘 El Concepto

Ya sabemos que una Clave Primaria (PK) identifica de forma única a una fila. Pero, ¿qué pasa si en una tabla tenemos **varias columnas que cumplen las reglas** para ser Clave Primaria (no se repiten y no están vacías)?

A todas esas columnas que "podrían" ser la Clave Primaria se les llama **Claves Candidatas**. El Diseñador de la Base de Datos (tú) tiene que elegir **solo una** para coronarla como la reina (la Clave Primaria). Las demás que no fueron elegidas se quedan con el título de "Claves Candidatas" (o claves alternativas).

### 🏠 La Analogía

Piensa en una entrevista de trabajo para un puesto de "Director" (Clave Primaria).
A la entrevista final llegan tres personas (columnas) increíbles:

1.  Una tiene muchísima experiencia.
2.  Otra tiene tres másters.
3.  Otra conoce la empresa desde dentro.

Las tres personas son **Candidatas** perfectas para el puesto porque cumplen todos los requisitos. Sin embargo, Recursos Humanos solo puede contratar a **una**. La persona contratada se convierte en el "Director", y las otras dos simplemente se quedan como "Candidatas" altamente cualificadas.

### 💻 El Código

En SQL no existe un comando llamado `CANDIDATE KEY`. Lo que hacemos es nombrar a nuestra ganadora como `PRIMARY KEY`, y a las otras candidatas que no ganaron, les ponemos la restricción `UNIQUE` para asegurar que sigan sin repetirse (como buenas candidatas que son).

```sql
CREATE TABLE ciudadanos (
    dni VARCHAR PRIMARY KEY,         -- ¡La ganadora! (Clave Primaria)
    numero_pasaporte VARCHAR UNIQUE, -- ¡La finalista! (Clave Candidata alternativa)
    nombre VARCHAR
);
```

### 🧠 El Reto de la Lección

Estamos creando la base de datos de los empleados de nuestra empresa. Diseñamos la tabla Empleados con las siguientes columnas:

- ID_Empleado (Un número interno único, ej: 1045)

- Numero_Seguridad_Social (El número único del gobierno)

- Email_Corporativo (ej: david@miempresa.com)

- Color_Pelo

**Pregunta 1:** De esa lista, ¿Cuáles serían tus Claves Candidatas? (Las que cumplen el requisito de no repetirse nunca).
**Pregunta 2:** De esas candidatas... ¿A cuál le darías tú el puesto de Clave Primaria y por qué?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

<b>1. Claves Candidatas:</b> <code>ID_Empleado</code>, <code>Numero_Seguridad_Social</code> y <code>Email_Corporativo</code>. Las tres son únicas y no nulas. (<code>Color_Pelo</code> se descarta porque se repite).

<b>2. Clave Primaria Ideal:</b> <code>ID_Empleado</code>. Es la mejor práctica (Clave Sustituta) porque los números son más rápidos de procesar que los textos (Email), no cambia nunca (a diferencia de un email por cambio de apellido) y no expone datos sensibles (como la Seguridad Social).

</details>

---

---

## 1.6 Clave Compuesta (Composite Key)

### 📘 El Concepto

A veces, por mucho que mires una tabla, **no hay ninguna columna que por sí sola sea capaz de identificar a una fila** de forma única.
Cuando esto pasa, la solución es coger dos (o más) columnas y "fusionarlas". A la combinación de ambas la llamamos **Clave Compuesta**. Es decir, es una Clave Primaria que necesita de varias columnas trabajando en equipo para garantizar que el dato no se repita.

### 🏠 La Analogía

Piensa en una entrada de cine.
Si solo miras el número de butaca, por ejemplo "Butaca 5", no sabes dónde sentarte. Hay una "Butaca 5" en todas las filas de la sala.
Si solo miras la letra de la fila, por ejemplo "Fila F", tampoco te sirve. Hay muchas butacas en la Fila F.
Ningún dato por separado es único. Pero si los unes: **"Fila F + Butaca 5"**, ¡bingo\! Esa combinación crea un asiento único y exclusivo en todo el cine que nadie más puede ocupar en esa sesión.

### 💻 El Código

En SQL, cuando queremos hacer que nuestra Clave Primaria esté formada por más de una columna, se lo indicamos al final de la creación de la tabla, metiendo los nombres entre paréntesis separados por comas.

```sql
CREATE TABLE butacas_cine (
    letra_fila CHAR(1),
    numero_butaca INT,
    tipo_asiento VARCHAR,
    -- Aquí creamos la Clave Compuesta:
    PRIMARY KEY (letra_fila, numero_butaca)
);
```

### 🧠 El Reto de la Lección

Estamos diseñando la base de datos de una Universidad. Tenemos una tabla especial llamada Notas_Examenes que registra las calificaciones de los alumnos. Las columnas que hemos pensado son:

- ID_Alumno (Ej: A-100)

- ID_Asignatura (Ej: MAT-101)

- Calificacion_Final (Ej: 8.5)

Ten en cuenta esta regla de negocio de la universidad: Un alumno solo puede tener una calificación final por cada asignatura.

**Pregunta:** Si miras esas tres columnas por separado, ninguna sirve como Clave Primaria (un alumno puede estar en varias asignaturas, y una asignatura tiene muchos alumnos). Para que esta tabla funcione perfectamente... ¿Qué columnas unirías para formar tu Clave Compuesta?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

La combinación perfecta sería <code>ID_Alumno</code> + <code>ID_Asignatura</code>. Así aseguramos que un mismo alumno no pueda tener dos notas finales en la misma materia, pero le permitimos tener notas en distintas materias, y que distintos alumnos tengan nota en la misma materia.

</details>

---

---

## 1.7 Restricciones (Constraints)

### 📘 El Concepto

Las restricciones o _Constraints_ son **reglas estrictas** que aplicamos a las columnas de una tabla. Su trabajo es vigilar cada dato que intenta entrar. Si el dato cumple la regla, entra. Si el dato rompe la regla, la base de datos lo bloquea, lanza un error de inmediato y protege la tabla.
Hay 6 restricciones principales:

1. **NOT NULL:** Obliga a que la celda no pueda estar vacía.
2. **UNIQUE:** Obliga a que el dato no se repita en toda la columna.
3. **PRIMARY KEY:** Es la fusión de NOT NULL + UNIQUE.
4. **FOREIGN KEY:** Obliga a que el dato que intentas meter exista previamente en otra tabla.
5. **CHECK:** Aplica una condición lógica o matemática (ej. que un número sea mayor que cero).
6. **DEFAULT:** Si al insertar una fila no proporcionas ningún dato para esa columna, el sistema pondrá un valor por "defecto" en lugar de dejarlo nulo.

### 🏠 La Analogía

Piensa en las restricciones como **el portero de seguridad de una discoteca** muy estricta:

- `NOT NULL`: "Si no llevas tu DNI, no entras."
- `UNIQUE`: "No puedes entrar si llevas la misma camiseta que alguien que ya está dentro."
- `CHECK`: "Solo entras si tienes 18 años o más."
- `DEFAULT`: "Si al pedir en la barra no me dices qué quieres beber, te pongo agua por defecto."
- `FOREIGN KEY`: "Solo entras si estás en la lista VIP de invitados (la otra tabla)."

### 💻 El Código

En SQL, estas reglas se ponen justo al lado del tipo de dato al crear la tabla.

```sql
CREATE TABLE usuarios_app (
    id_usuario INT PRIMARY KEY,                   -- Regla 3: Único y no vacío
    email VARCHAR UNIQUE NOT NULL,                -- Regla 2 y 1: Único y no vacío
    edad INT CHECK (edad >= 18),                  -- Regla 5: Matemáticamente mayor o igual a 18
    estado_cuenta VARCHAR DEFAULT 'Activo'        -- Regla 6: Si no digo nada, pon 'Activo'
);

### 🧠 El Reto de la Lección

Estás diseñando la base de datos de una tienda online. Tienes una tabla llamada Productos con una columna para el Precio.

En las primeras semanas de pruebas, te das cuenta de que a veces los empleados se equivocan al teclear y, en lugar de poner 50.00, ponen -50.00 (un precio negativo). Esto rompe la contabilidad de la tienda.

**Pregunta:** De las 6 restricciones que hemos visto... ¿Cuál utilizarías exactamente en la columna Precio para evitar que vuelva a entrar un precio negativo, y cómo escribirías la regla?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>


<b>Respuesta del Profesor:</b>


La restricción ideal es <b>CHECK</b>. La escribiríamos así: <code>CHECK (Precio >= 0)</code>. De esta forma, el motor de la base de datos rechazará automáticamente cualquier número negativo (como -50.00) antes de que se guarde en la tabla, protegiendo así la contabilidad.
</details>

```

## 1.8 Integridad de los Datos

### 📘 El Concepto

La Integridad de los Datos no es un comando ni una herramienta, es **el objetivo final** de todo buen diseñador de bases de datos. Significa garantizar que la información almacenada sea precisa, válida, consistente y confiable a lo largo de todo su ciclo de vida.

Existen tres tipos principales de integridad que hemos aprendido a proteger con las herramientas anteriores:

1. **Integridad de Entidad:** Nos aseguramos de que no haya filas duplicadas y que cada registro sea único (Lo logramos usando _Primary Keys_).
2. **Integridad Referencial:** Nos aseguramos de que las relaciones entre tablas sean válidas y no apunten a datos que no existen, evitando registros "huérfanos" (Lo logramos usando _Foreign Keys_).
3. **Integridad de Dominio:** Nos aseguramos de que los datos de una columna cumplan con las reglas de negocio y formato correctos (Lo logramos usando _Constraints_ como `CHECK`, `NOT NULL` y los tipos de datos).

### 🏠 La Analogía

Imagina un hospital. La "Integridad de los Datos" significa que:

1. No hay dos historiales médicos distintos para el mismo paciente (Integridad de Entidad).
2. Si un historial indica que el paciente está en la "Cama 404", la Cama 404 realmente existe en el hospital (Integridad Referencial).
3. En el campo "Grupo Sanguíneo" solo puede poner A, B, AB o O, nunca puede poner la palabra "Gato" o un número negativo (Integridad de Dominio).

### 🧠 El Reto de la Lección

En una app de envíos a domicilio, descubres que el sistema ha colapsado. Al investigar, ves que un cliente tiene un pedido registrado, pero el ID del restaurante al que apunta ese pedido ya no existe en la tabla `Restaurantes` (alguien lo borró). Ahora tienes un pedido "huérfano".

**Pregunta:** De los tres tipos de integridad (Entidad, Referencial o de Dominio)... ¿Cuál se ha roto en este caso?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>
<br>
<b>Respuesta del Profesor:</b><br>
Se ha roto la <b>Integridad Referencial</b>. La tabla de Pedidos estaba haciendo referencia (mediante una Clave Foránea) a un restaurante que ya no existe. Un buen diseño habría bloqueado el borrado del restaurante o habría archivado el registro en lugar de eliminarlo.
</details>
---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 01

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Avanzar al siguiente tema | ➡️ [Tema 02: Tipos de Datos (Oracle)](../02-tipos-de-datos) |

---

<div align="center">

🏠 [**Índice del Curso**](../README.md) · [**Tema 02: Tipos de Datos (Oracle) →**](../02-tipos-de-datos)

</div>
