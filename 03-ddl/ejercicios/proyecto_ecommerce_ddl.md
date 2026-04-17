## 🏆 Mini-Proyecto DDL: E-commerce Global

### 📘 El Concepto

Para dominar SQL, necesitamos un entorno de pruebas realista. Vamos a diseñar la arquitectura básica de una tienda en línea. Este esquema nos servirá para practicar inserciones (DML), consultas complejas (DQL) y cruces de datos (JOINs) en los próximos temas.

### 🏗️ La Arquitectura (El Reto)

Tu objetivo es actuar como el Arquitecto de Datos y escribir un script SQL consecutivo que levante esta base de datos desde cero y realice algunos ajustes.

Necesitas traducir las siguientes reglas de negocio a código DDL:

**1. Tabla `clientes`**:

- `id_cliente`: Número entero, será la clave primaria.
- `nombre`: Texto de hasta 50 caracteres, no puede estar vacío.
- `email`: Texto de hasta 100 caracteres, debe ser único y no puede estar vacío.

**2. Tabla `categorias`**:

- `id_categoria`: Número entero, será la clave primaria.
- `nombre_categoria`: Texto de hasta 50 caracteres, no puede estar vacío.

**3. Tabla `productos`**:

- `id_producto`: Número entero, será la clave primaria.
- `nombre`: Texto de hasta 100 caracteres, no puede estar vacío.
- `precio`: Número decimal (formato para manejar precios como 99.99), no puede estar vacío.
- `id_categoria`: Número entero. Esta debe ser una **Clave Foránea (Foreign Key)** que referencie a la tabla `categorias`.

**4. El Ajuste (ALTER)**:

- El equipo de logística acaba de llamar. Necesitan controlar el inventario. Escribe el comando para añadir una columna llamada `stock` (número entero, por defecto 0) a tu tabla `productos`.

**5. La Limpieza (DROP)**:

- Ayer estuviste haciendo pruebas y quedó una tabla residual llamada `categorias_backup`. Escribe el comando para fulminarla del sistema.

<summary>Ver respuesta</summary>

```sql
CREATE TABLE clientes (
    id_cliente NUMBER(10) PRIMARY KEY,
    nombre VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL
);

CREATE TABLE categorias (
    id_categoria NUMBER(10) PRIMARY KEY,
    nombre_categoria VARCHAR2(50) NOT NULL
);

CREATE TABLE productos (
    id_producto NUMBER(10) PRIMARY KEY,
    nombre VARCHAR2(100) NOT NULL,
    precio NUMBER(5,2) NOT NULL,
    id_categoria NUMBER(10) REFERENCES categorias(id_categoria)
);

ALTER TABLE productos ADD (stock number(5) default 0);

drop TABLE categorias_backup;
```

</details>
