## 4.1 DDL: CREATE (Construyendo los Cimientos)

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

Salida esperada:
```
Table created.
```

Verificación:
```sql
DESCRIBE app_productos;
```
| Name | Null? | Type |
|------|-------|------|
| ID_PRODUCTO | NOT NULL | NUMBER(5) |
| NOMBRE_PRODUCTO | NOT NULL | VARCHAR2(100) |
| PRECIO_EUR | | NUMBER(6,2) |

</details>

---

---

## 4.2 ALTER TABLE (Modificar estructuras)

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

Salida esperada:
```
Table altered.
Table altered.
```

Verificación:
```sql
DESCRIBE libros;
```
La columna `genero` ahora aparece en la lista y `titulo` muestra `VARCHAR2(255)`.

</details>

---

---

## 4.3 DDL: DROP (Eliminando Estructuras)

### 📘 El Concepto

El comando `DROP` se utiliza para eliminar completamente un objeto estructural de la base de datos, como una tabla entera, un índice o incluso la propia base de datos. Es una operación DDL pura.

Es fundamental entender que `DROP` no solo borra la información que contiene la tabla, sino que **destruye la estructura misma**. Una vez ejecutado, la tabla deja de existir en el esquema y, por lo general, es una acción irreversible.

### 🏗️ La Analogía

Si `CREATE` es construir un edificio desde los cimientos y `ALTER` es llamar al carpintero para añadir una ventana, `DROP` es llamar al equipo de demolición con la bola de acero.

No estás simplemente vaciando el edificio de muebles e inquilinos; estás echando abajo las paredes, el techo y los pilares. El espacio vuelve a quedar como un solar vacío.

### 💻 El Código

La sintaxis es directa, lo cual la hace muy fácil de escribir pero peligrosa si no se usa con precaución:

```sql
-- Eliminar una tabla por completo
DROP TABLE nombre_de_la_tabla;
```

### 🎯 El Reto

Imagina que ayer estuviste haciendo pruebas y creaste una tabla temporal llamada autores_borrador en tu base de datos. Hoy ya has terminado tus pruebas y necesitas mantener tu entorno limpio y ordenado.

Tu tarea: Escribe la sentencia SQL exacta para demoler y hacer desaparecer por completo la tabla autores_borrador.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
DROP TABLE autores_borrador;
```

Salida esperada:
```
Table dropped.
```

Verificación:
```sql
SELECT table_name FROM user_tables WHERE table_name = 'AUTORES_BORRADOR';
```
Resultado: no rows selected (la tabla ya no existe en el esquema).

</details>

---

---

## 4.4 DDL: TRUNCATE (Vaciado Express)

### 📘 El Concepto

El comando `TRUNCATE` se utiliza para eliminar **todos los registros (filas)** de una tabla de forma inmediata, pero **conservando la estructura** de la tabla intacta (columnas, tipos de datos, restricciones).

Aunque parezca que estamos manipulando datos y debería ser DML (como veremos con `DELETE`), `TRUNCATE` está clasificado como DDL en la mayoría de los motores. Esto se debe a cómo funciona internamente: no borra fila por fila, sino que desasigna las páginas de datos de la tabla de golpe. Por eso es extremadamente rápido, pero a menudo no se puede deshacer (no genera un registro de transacciones completo).

### 🧹 La Analogía

Volvamos a nuestra biblioteca.

- `DROP` era llamar al equipo de demolición y destruir la estantería entera.
- `TRUNCATE` es acercarte a la estantería y, con un solo movimiento de brazo, barrer absolutamente todos los libros y tirarlos a la basura de golpe. La estantería (la tabla) sigue ahí, vacía e intacta, lista para que pongas libros nuevos.

### 💻 El Código

La sintaxis es tan sencilla como la de `DROP`, pero el resultado es muy diferente:

```sql
-- Vaciar todos los datos de una tabla, manteniendo su estructura
TRUNCATE TABLE nombre_de_la_tabla;
```

### 🎯 El Reto

En tu base de datos tienes una tabla llamada logs_sistema que registra cada clic que hacen los usuarios. Ha estado acumulando datos durante un año y está saturando el disco. Quieres borrar toda esa información antigua para empezar de cero, pero necesitas que la tabla siga existiendo para que el sistema siga registrando los clics de mañana.

Tu tarea: Escribe la sentencia SQL para limpiar completamente esa tabla sin destruirla.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
TRUNCATE TABLE logs_sistema;
```

Salida esperada:
```
Table truncated.
```

Verificación:
```sql
SELECT COUNT(*) FROM logs_sistema;
```
| COUNT(*) |
|----------|
| 0 |

> 💡 La tabla sigue existiendo (puedes hacer `DESCRIBE logs_sistema`), pero está completamente vacía.

</details>
---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 04

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de DDL](./ejercicios/ejercicios.md) |
| 3️⃣ | Completar los proyectos | 🏆 [Proyecto 1: E-Commerce](./proyectos/proyecto_ecommerce_ddl.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 05: DML (Data Manipulation Language)](../05-dml) |

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 03**](../03-normalizacion/proyectos/proyecto_normalizacion.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 04 →**](ejercicios/ejercicios.md)

</div>
