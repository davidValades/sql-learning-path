## 6.1 DQL Básico: El arte de preguntar (`SELECT` y `FROM`)

### 📘 El Concepto

**DQL** significa _Data Query Language_ (Lenguaje de Consulta de Datos). Mientras que el DML altera la información, el DQL es de **solo lectura**. Su objetivo es extraer la información exacta que necesitas, de la forma en la que la necesitas, sin tocar un solo dato original de la base de datos.

La herramienta absoluta y rey indiscutible del DQL es el comando **`SELECT`**.
Para hacer una consulta básica necesitas obligatoriamente dos partes:

1. **`SELECT`**: ¿Qué columnas quiero ver? (Si quieres todas, usamos el comodín `*`).
2. **`FROM`**: ¿De qué tabla voy a sacar esos datos?

### 🏠 La Analogía

Imagina que estás en un restaurante de lujo y llamas al camarero:

- **`SELECT nombre_plato, precio`**: Le dices al camarero: _"Solo me interesa saber el nombre del plato y cuánto cuesta, no me leas la lista de ingredientes"_.
- **`FROM menu_cenas`**: _"Búscalo en la carta de cenas, no me traigas la de desayunos"_.

El camarero (el motor de Oracle) va a la cocina (la base de datos) y te trae una bandeja (el resultado) solo con lo que pediste.

### 💻 El Código

Vamos a ver cómo se ve esto con la sintaxis de SQL:

```sql
-- Opción 1: Traer absolutamente TODAS las columnas y filas de la tabla
SELECT * FROM productos;

-- Opción 2: Traer solo columnas específicas (Mucho más eficiente)
SELECT nombre, precio
FROM productos;
```

### 🧠 El Reto de la Lección

Eres el encargado de inventario del E-commerce. Necesitas hacer un conteo rápido, por lo que los precios y las categorías no te interesan en absoluto para esta tarea.

**Pregunta:** Escribe la consulta SQL exacta para obtener un listado que muestre **únicamente** el nombre del producto y su cantidad en stock.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT nombre, stock
FROM productos;
```

Resultado:
| nombre | stock |
|--------|-------|
| Laptop Pro | 50 |
| Ratón Inalámbrico | 200 |
| Sofá de Cuero | 0 |
| Lámpara LED | 30 |

</details>

---

---

## 6.2 Filtros con `WHERE` y Operadores Lógicos

### 📘 El Concepto

Rara vez queremos ver _todos_ los registros de una tabla. Para limitar las filas que nos devuelve el `SELECT`, utilizamos la cláusula **`WHERE`**. Esta cláusula actúa como un colador, permitiendo que solo pasen las filas que cumplen una condición verdadera.

Para crear estas condiciones usamos **Operadores Relacionales**:

- `=`: Igual a.
- `!=` o `<>`: Diferente de.
- `>`, `<`, `>=`, `<=`: Mayor que, menor que, etc.

Y **Operadores Lógicos** (para combinar varias condiciones):

- `AND`: Se deben cumplir TODAS las condiciones.
- `OR`: Se debe cumplir AL MENOS UNA de las condiciones.

### 🏠 La Analogía

Piensa en el `WHERE` como el **portero de una discoteca VIP**.
El portero tiene instrucciones estrictas:

- _"Solo entran los que tengan entrada VIP"_ (`WHERE tipo_entrada = 'VIP'`).
- _"Oye, que entren los que tengan entrada VIP **Y** sean mayores de 18 años"_ (`WHERE tipo_entrada = 'VIP' AND edad >= 18`).

Si no cumples la regla del portero, te quedas fuera del resultado de la consulta.

### 💻 El Código

```sql
-- Filtro simple usando =
SELECT nombre_completo, salario_base
FROM medicos
WHERE id_especialidad = 100;

-- Filtro compuesto usando >= y AND
SELECT nombre, precio
FROM productos
WHERE id_categoria = 1 AND precio >= 1000;
```

### 🧠 El Reto de la Lección

El director del Hospital Central necesita contactar urgentemente a los pacientes nacidos antes del año 1990, pero solo si no tienen el teléfono vacío.
_(Nota: Asume que si el teléfono no está vacío, es diferente de null o tiene algún valor, pero para simplificar, busquemos a la paciente que sabemos que nació antes de 1990 en nuestros datos)_.

**Pregunta:** Escribe la consulta SQL para obtener el nombre y el teléfono de los pacientes cuya fecha de nacimiento sea anterior (`<`) al 1 de enero de 1990. Recuerda usar `TO_DATE()`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT nombre, telefono
FROM pacientes
WHERE fecha_nacimiento < TO_DATE('1990-01-01', 'YYYY-MM-DD');
```

Resultado:
| nombre | telefono |
|--------|----------|
| Carlos Gomez | 555-0002 |

> 💡 Laura Martinez nació el 1990-01-01, pero la condición es `<` (estrictamente menor), así que NO se incluye.

</details>

---

---

## 6.3 Ordenando los resultados con `ORDER BY`

### 📘 El Concepto

Por defecto, la base de datos te escupe las filas en el orden en el que las encuentra en el disco duro (lo cual suele ser un caos). Para presentar la información de forma humana y estructurada, usamos **`ORDER BY`**. Esta siempre será la **última cláusula** de tu consulta `SELECT`.

Puedes ordenar por una o varias columnas, en dos direcciones:

- **`ASC`**: Ascendente (De A-Z, de menor a mayor). Es el valor por defecto si no pones nada.
- **`DESC`**: Descendente (De Z-A, de mayor a menor).

### 🏠 La Analogía

Es igual que cuando entras a buscar un hotel en Booking o Amazon. Has buscado "Hoteles en Madrid" (`SELECT` y `FROM`) que tengan "Piscina" (`WHERE`). Tienes 100 resultados desordenados. Al final, haces clic en el desplegable de arriba a la derecha y seleccionas: **"Ordenar por: Precio, del más barato al más caro"** (`ORDER BY precio ASC`).

### 💻 El Código

```sql
-- Ordenar un listado de aviones desde el más nuevo al más antiguo
SELECT modelo, anio_fabricacion
FROM aviones
ORDER BY anio_fabricacion DESC;

-- Ordenar por múltiples columnas (Primero por categoría, luego por precio de más caro a barato)
SELECT nombre, id_categoria, precio
FROM productos
ORDER BY id_categoria ASC, precio DESC;
```

### 🧠 El Reto de la Lección

El CEO de la Aerolínea quiere un informe para ver qué rutas gastan más combustible.

**Pregunta:** Genera una consulta que devuelva todas las columnas de la tabla `rutas`, pero ordenadas de forma que la ruta con mayor `distancia_km` aparezca en la primera fila.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
SELECT * FROM rutas
ORDER BY distancia_km DESC;
```

Resultado:
| id_ruta | aeropuerto_origen | aeropuerto_destino | distancia_km |
|---------|------------------|-------------------|-------------|
| JFKLAX | JFK | LAX | 4000 |
| MADLHR | MAD | LHR | 1350 |

</details>
---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 06

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de DQL](./ejercicios/ejercicios_dql.md) |
| 3️⃣ | Completar los proyectos | 🏆 [Ir a Proyectos →](./proyectos) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 07: Operadores Lógicos y Filtros](../07-operadores) |

---

<div align="center">

⬅️ [**Tema 05: DML (Data Manipulation Language)**](../05-dml) · 🏠 [**Índice del Curso**](../README.md) · [**🏆 Proyectos →**](./proyectos/proyecto_dql_basico.md)

</div>
