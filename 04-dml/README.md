## 4.1 DML: INSERT (Poblando las Tablas)

### 📘 El Concepto

**DML** significa _Data Manipulation Language_ (Lenguaje de Manipulación de Datos). A diferencia del DDL (que toca la estructura), el DML interactúa exclusivamente con los **datos** que viven dentro de las tablas.

El comando **`INSERT`** es la puerta de entrada. Se utiliza para añadir nuevas filas (registros) a una tabla existente.

**Reglas de Oro del INSERT en Oracle:**

1. Los textos (`VARCHAR2`, `CHAR`) y las fechas (`DATE`) SIEMPRE van entre **comillas simples** (`'texto'`).
2. Los números (`NUMBER`) NUNCA llevan comillas.
3. El orden de los valores que insertas debe coincidir exactamente con el orden de las columnas que declaras.

### 🏠 La Analogía

Imagina que el **DDL** fue el proceso de construir una enorme estantería de madera (la tabla) y etiquetar cada balda: "Aquí van novelas", "Aquí van cómics".
El **DML (`INSERT`)** es el momento en el que llegas con una caja llena de libros (datos) y empiezas a colocarlos, uno por uno, en su balda correspondiente respetando las etiquetas.

### 💻 El Código

La forma más segura y profesional de insertar datos es nombrando explícitamente las columnas. De esta manera, si la tabla cambia en el futuro, tu código no se romperá.

```sql
-- Sintaxis profesional: INSERT INTO nombre_tabla (columnas) VALUES (datos);

-- Usando la tabla 'especialidades' de nuestro Hospital:
INSERT INTO especialidades (id_especialidad, nombre_especialidad)
VALUES (101, 'Cardiología');

INSERT INTO especialidades (id_especialidad, nombre_especialidad)
VALUES (102, 'Pediatría');
```

Nota Pro: En Oracle, cuando haces un INSERT, los datos se guardan temporalmente en tu sesión. Para que se guarden de forma definitiva en el disco duro del servidor para que todos los demás usuarios puedan verlos, necesitas ejecutar el comando COMMIT; (lo veremos en detalle muy pronto, pero es bueno que te suene).

### 🧠 El Reto de la Lección

Vamos a estrenar la base de datos de nuestra Aerolínea. Necesito que insertes el primer avión en tu tabla aviones.

Los datos del avión son:

ID del avión: 1

Modelo: 'Boeing 787 Dreamliner'

Capacidad: 296 pasajeros

Año de fabricación: 2022 (Recuerda que añadiste esta columna con un ALTER).

Tu Tarea: Escribe la sentencia SQL exacta para insertar esta fila en la tabla aviones.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros, anio_fabricacion)
VALUES (1, 'Boeing 787 Dreamliner', 296, 2022);
```

</details>

---

---

## 3.2 DML: UPDATE (Modificando el presente)

### 📘 El Concepto

A veces, los datos cambian. Un cliente se muda de casa, un producto sube de precio o, como suele pasar, alguien comete un error de teclado al insertar un dato. Para arreglarlo, no borramos la fila y la volvemos a crear; usamos el comando **`UPDATE`**.

**La Regla de Oro, de Platino y de Diamante del UPDATE:**
NUNCA, jamás, bajo ninguna circunstancia, ejecutes un `UPDATE` sin la cláusula **`WHERE`** (a menos que realmente quieras cambiar TODAS las filas de la tabla al mismo tiempo). El `WHERE` es tu filtro; le dice a Oracle exactamente qué fila o filas deben ser modificadas.

### 🏠 La Analogía

Imagina que estás en clase y la profesora te pide que corrijas una falta de ortografía en tu cuaderno.

- Usar un `UPDATE` **CON `WHERE`** es como coger una goma de borrar, ir a la línea 5, borrar solo la palabra equivocada y escribir la correcta.
- Usar un `UPDATE` **SIN `WHERE`** es como coger un bote de tipex y pintar el cuaderno entero escribiendo la misma palabra en todas las líneas. ¡Un desastre absoluto!

### 💻 El Código

La sintaxis sigue el orden de: "Actualiza ESTA tabla, pon ESTOS valores, pero SÓLO donde se cumpla ESTA condición".

```sql
-- Sintaxis profesional:
-- UPDATE nombre_tabla SET columna = nuevo_valor WHERE condicion;

-- Ejemplo en nuestro E-commerce: Un producto sube de precio
UPDATE productos
SET precio = 105.50
WHERE id_producto = 42;
-- ^^^ ¡Ese WHERE ha salvado tu empleo! Solo el producto 42 cambiará de precio.

-- Puedes actualizar varias columnas a la vez separándolas por comas:
UPDATE pacientes
SET telefono = '555-1234', nombre = 'David García'
WHERE id_paciente = 1005;
```

### 🧠 El Reto de la Lección

Ha ocurrido un pequeño contratiempo en la Aerolínea. El ingeniero jefe te acaba de llamar sudando frío.

Resulta que el avión que insertaste en la lección anterior (ID 1, modelo 'boeing 787 Dreamliner') en realidad es una versión especial modificada. No tiene capacidad para 296 pasajeros, sino que le han quitado asientos para poner un bar de lujo, por lo que su capacidad real ahora es de 250 pasajeros.

Tu Tarea: Escribe la sentencia SQL exacta para actualizar solo la capacidad de este avión en tu tabla aviones, asegurándote de no afectar a ningún otro avión que pudiéramos añadir en el futuro.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
UPDATE aviones
SET capacidad_pasajeros = 250
WHERE id_avion = 1;
```

</details>
