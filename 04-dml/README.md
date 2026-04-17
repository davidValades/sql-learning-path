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
