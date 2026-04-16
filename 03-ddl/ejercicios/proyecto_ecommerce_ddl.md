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
