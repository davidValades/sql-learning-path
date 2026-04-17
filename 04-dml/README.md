## 4.1 DML: INSERT (Llenando la tienda)

### 📘 El Concepto

**DML** significa _Data Manipulation Language_ (Lenguaje de Manipulación de Datos). Mientras que el DDL construía la estructura (las tablas), el DML se encarga exclusivamente de la información que va **dentro** de esas tablas.

El primer comando fundamental del DML es **`INSERT`**. Como su nombre indica, sirve para crear nuevas filas (registros) dentro de una tabla que ya existe.

### 🏠 La Analogía

Si usar DDL (CREATE TABLE) era construir un bloque de apartamentos vacío, usar DML (`INSERT`) es **entregar las llaves para que los inquilinos se muden**.

El comando `INSERT` es como el camión de mudanzas. Tú le dices al camión a qué edificio ir (la tabla), qué inquilino llevar y en qué habitación poner cada mueble (las columnas).

### 💻 El Código

Existen dos formas principales de hacer un `INSERT` en Oracle SQL.

**Forma 1: Declarativa (La más profesional y segura)**
Aquí le decimos explícitamente al motor en qué columnas vamos a insertar los datos. Si en el futuro alguien añade una columna nueva a la tabla, este código no se romperá.

```sql
-- Sintaxis: INSERT INTO tabla (columna1, columna2) VALUES (valor1, valor2);
```

INSERT INTO categorias (id_categoria, nombre_categoria)
VALUES (1, 'Electrónica');

Nota: Fíjate que el texto ('Electrónica') va entre comillas simples. Los números (1) van sin comillas.

**Forma 2: Implícita (Rápida pero arriesgada)**

Aquí no escribimos el nombre de las columnas. Oracle asume que vas a meter un dato para CADA UNA de las columnas de la tabla, exactamente en el mismo orden en el que fueron creadas.

```sql
-- Nuestra tabla clientes tiene: id_cliente, nombre, email
INSERT INTO clientes
VALUES (101, 'David López', 'david.lopez@email.com');
```

### 🧠 El Reto y los Proyectos

Tu E-commerce global necesita abrir sus puertas hoy mismo. Necesitamos inaugurar la tienda con datos iniciales reales.

Usando el esquema que acabas de crear, tu tarea es escribir las sentencias SQL para:

Insertar dos categorías nuevas en tu tabla categorias (ej: 'Deportes' y 'Hogar').

Insertar dos clientes nuevos en tu tabla clientes (invéntate los nombres y correos, pero recuerda que los correos no se pueden repetir por tu restricción UNIQUE).

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
-- Insertando Categorías
INSERT INTO categorias (id_categoria, nombre_categoria) VALUES (20, 'Deportes');
INSERT INTO categorias (id_categoria, nombre_categoria) VALUES (21, 'Hogar');

-- Insertando Clientes
INSERT INTO clientes (id_cliente, nombre, email)
VALUES (1, 'Laura Martínez', 'laura.m@ecommerce.com');

INSERT INTO clientes (id_cliente, nombre, email)
VALUES (2, 'Carlos Ruiz', 'carlos.r@ecommerce.com');
```

</details>
