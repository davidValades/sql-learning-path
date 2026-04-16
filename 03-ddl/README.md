## 3.1 DDL: CREATE (Construyendo los Cimientos)

### 📘 El Concepto

**DDL** significa _Data Definition Language_ (Lenguaje de Definición de Datos). Son los comandos que usamos para crear, modificar o destruir la **estructura** de la base de datos (tablas, columnas, restricciones). No tocan los datos de los clientes, solo el "molde".

El rey del DDL es el comando **`CREATE TABLE`**.

### 🏠 La Analogía

Usar comandos DDL es como ser un **arquitecto y un albañil**. Tú no metes los muebles (datos) en la casa; tú levantas las paredes, pones las puertas y dices: "Esta habitación será el baño y solo caben cosas de baño" (crear columnas con tipos de datos).

### 💻 El Código

Vamos a construir la base de datos de nuestra aplicación, empezando por los cimientos: la tabla de Usuarios.

**Copia este código y pégalo en el panel izquierdo de tu DB Fiddle:**

```sql
-- Creamos nuestra primera tabla en Oracle
CREATE TABLE app_usuarios (
    id_usuario NUMBER(5) PRIMARY KEY,
    nombre VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    fecha_registro DATE DEFAULT SYSDATE,  -- SYSDATE es la función de Oracle para la fecha actual
    estado CHAR(1) DEFAULT 'A' CHECK (estado IN ('A', 'I')) -- 'A'ctivo o 'I'nactivo
);
`
```

### 🧠 El Reto de la Lección (Práctica)

Ahora te toca a ti ser el arquitecto. En el mismo panel izquierdo de tu DB Fiddle, justo debajo de la tabla que acabamos de crear, quiero que escribas tú solo el código para crear una segunda tabla llamada app_productos.

Debe tener las siguientes columnas con las reglas (restricciones) adecuadas:

1.  **id_producto:** Un número de hasta 5 cifras. Debe ser la Clave Primaria.

2.  **nombre_producto:** Un texto variable de hasta 100 letras. No puede estar vacío.

3.  **precio_eur:** Un número de hasta 6 cifras en total, con 2 decimales. No puede ser negativo.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
CREATE TABLE app_productos (
  id_producto NUMBER(5) PRIMARY KEY,
  nombre_producto VARCHAR2(100) NOT NULL,
  precio_eur NUMBER(6,2) CHECK (precio_eur >= 0)
);
```

</details>

---

---

## 3.2 ALTER TABLE (Modificar estructuras)

### 📖 El Concepto

El comando `ALTER` nos permite modificar la estructura de una tabla que ya existe sin perder los datos que contiene. Es una operación DDL pura. Con él podemos añadir nuevas columnas, modificar el tipo de dato o las restricciones de las columnas existentes, y eliminar columnas que ya no necesitamos.

### 💡 La Analogía

Siguiendo con nuestra **biblioteca** 📚:
Imagina que ya tienes construido el estante de `libros` y lleno de ejemplares. De repente, el bibliotecario te dice: _"Necesitamos registrar en qué idioma está cada libro"_.
No vas a prenderle fuego al estante (y a los libros) para construir uno nuevo desde cero con ese espacio extra. En lugar de eso, llamas al carpintero (`ALTER`) para que le añada una balda adicional al estante existente.

### 💻 Ejemplo de Código (Sintaxis Oracle)

```sql
-- 1. Añadir una nueva columna
ALTER TABLE autores ADD (email VARCHAR2(100));

-- 2. Modificar el tipo de dato o tamaño de una columna
ALTER TABLE autores MODIFY (nacionalidad VARCHAR2(80));

-- 3. Eliminar una columna
ALTER TABLE autores DROP COLUMN email;
```

### 🎯 El Reto

Ya tenemos nuestra tabla `libros` creada, pero tu jefe acaba de entrar a la oficina y te ha dado dos nuevos requisitos:

1. Necesitamos saber el **género literario** del libro (por ejemplo: "Ficción", "Terror", "Ensayo").
2. Se ha dado cuenta de que un texto de 150 caracteres para el `titulo` podría quedarse corto para algunos libros antiguos con nombres larguísimos, y quiere ampliarlo a **255 caracteres**.

**Tu tarea:** Escribe las sentencias SQL para:

1. Añadir una nueva columna llamada `genero` a tu tabla `libros` (piensa qué tipo de dato sería el adecuado).
2. Modificar la columna `titulo` para que ahora soporte hasta 255 caracteres.

¿Cómo le darías estas instrucciones al "carpintero" en SQL?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
-- 1. Añadimos la columna para el género literario
ALTER TABLE libros ADD (genero VARCHAR2(100));

-- 2. Ampliamos el tamaño del título para soportar nombres más largos
ALTER TABLE libros MODIFY (titulo VARCHAR2(255));
```

</details>
