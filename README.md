# 🚀 SQL Learning Path: De Cero a DBA (Oracle SQL)

<div align="center">

![Estado](https://img.shields.io/badge/Estado-En_Proceso-blue?style=for-the-badge)
![Progreso](https://geps.dev/progress/100)

**Una ruta de aprendizaje interactiva y práctica para dominar Bases de Datos Relacionales y SQL.**

</div>

---

¡Bienvenido a mi ruta de aprendizaje de Bases de Datos Relacionales y SQL! 📊

Este repositorio es una bitácora viva de mi evolución como Data Engineer / DBA. Aquí no solo encontrarás teoría, sino **proyectos prácticos, retos gamificados y scripts reales** basados en la sintaxis de Oracle SQL.

> 💡 **¿Cómo usar esta guía?** Sigue los temas en orden. Cada tema tiene temario, ejercicios y/o proyectos. Al final de cada página encontrarás enlaces para avanzar al siguiente paso sin perderte.

## 🛠️ Metodología de Aprendizaje

Cada lección en este repositorio está estructurada de forma rigurosa para garantizar la comprensión profunda de los datos:

| Paso | Componente         | Descripción                                                 |
| :--: | ------------------ | ----------------------------------------------------------- |
|  1️⃣  | **📘 El Concepto** | Explicación técnica directa y sin rodeos                    |
|  2️⃣  | **🏠 La Analogía** | Traslación del concepto a situaciones de la vida cotidiana  |
|  3️⃣  | **💻 El Código**   | Sintaxis real en Oracle SQL                                 |
|  4️⃣  | **🧠 El Reto**     | Ejercicios prácticos con soluciones ocultas (_Hands-On_)    |
|  5️⃣  | **🏆 Proyectos**   | Mini-proyectos para aplicar lo aprendido _(cuando aplique)_ |

---

## 📊 Cobertura actual por tema

> Estado actualizado: los 19 temas ya incluyen temario y práctica (ejercicios + proyecto/s).

| Tema                               | Temario | Ejercicios | Proyectos | Recursos extra |
| ---------------------------------- | ------- | ---------- | --------- | -------------- |
| [Tema 01](./01-fundamentos-bd)     | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 02](./02-tipos-de-datos)     | ✅      | ✅ (1)     | ✅ (1)    | ✅ (1)         |
| [Tema 03](./03-normalizacion)      | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 04](./04-ddl)                | ✅      | ✅ (1)     | ✅ (3)    | ✅ (1)         |
| [Tema 05](./05-dml)                | ✅      | ✅ (1)     | ✅ (2)    | —              |
| [Tema 06](./06-dql)                | ✅      | ✅ (1)     | ✅ (2)    | —              |
| [Tema 07](./07-operadores)         | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 08](./08-funciones)          | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 09](./09-joins)              | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 10](./10-subconsultas)       | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 11](./11-vistas)             | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 12](./12-indexacion)         | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 13](./13-transacciones-acid) | ✅      | ✅ (2)     | ✅ (1)    | —              |
| [Tema 14](./14-dcl-seguridad)      | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 15](./15-sql-avanzado)       | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 16](./16-plsql)              | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 17](./17-optimizacion)       | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 18](./18-ecosistema-sql)     | ✅      | ✅ (1)     | ✅ (1)    | —              |
| [Tema 19](./19-casos-reales)       | ✅      | ✅ (1)     | ✅ (1)    | —              |

---

## 🗺️ Índice del Curso

> _Haz clic en cada sección para desplegar los detalles. Todos los temas tienen cobertura base completa._

### 📚 Bloque I — Los Cimientos

<details>
<summary>[x] <b>📦 Tema 01: Fundamentos de Bases de Datos</b></summary>
<br>
Conceptos clave sobre qué es una Base de Datos, SGBD, SGBDR, el modelo relacional, Tablas, Filas, Columnas y los diferentes tipos de Claves (Primarias, Foráneas, Candidatas, Compuestas).
<br><br>

| Recurso    | Enlace                                     |
| ---------- | ------------------------------------------ |
| 📖 Temario | [**Ir al Tema 01 →**](./01-fundamentos-bd) |

</details>

<details>
<summary>[x] <b>🧩 Tema 02: Tipos de Datos (Oracle)</b></summary>
<br>
Domina cómo almacenar la información de forma eficiente. Diferencias entre <code>NUMBER</code>, <code>VARCHAR2</code>, <code>CHAR</code>, <code>DATE</code> y <code>TIMESTAMP</code> en el ecosistema Oracle.
<br><br>

| Recurso        | Enlace                                                                       |
| -------------- | ---------------------------------------------------------------------------- |
| 📖 Temario     | [**Ir al Tema 02 →**](./02-tipos-de-datos)                                   |
| 📑 Cheat Sheet | [Ver Cheat Sheet Oracle](./02-tipos-de-datos/recursos/cheat-sheet-oracle.md) |

</details>

<details>
<summary>[x] <b>📐 Tema 03: Normalización y Modelado ER</b></summary>
<br>
Diseño eficiente de bases de datos para evitar redundancias (1NF, 2NF, 3NF y BCNF). Modelado Entidad-Relación: el plano arquitectónico que dibujas <b>antes</b> de construir tus tablas.
<br><br>

| Recurso    | Enlace                                    |
| ---------- | ----------------------------------------- |
| 📖 Temario | [**Ir al Tema 03 →**](./03-normalizacion) |

</details>

### 🏗️ Bloque II — Construir y Manipular

<details>
<summary>[x] <b>🏗️ Tema 04: DDL (Data Definition Language)</b></summary>
<br>
El arte de construir los cimientos. Creación, modificación y destrucción de estructuras con <code>CREATE</code>, <code>ALTER</code>, <code>DROP</code> y <code>TRUNCATE</code>.
<br><br>

| Recurso       | Enlace                                                                |
| ------------- | --------------------------------------------------------------------- |
| 📖 Temario    | [**Ir al Tema 04 →**](./04-ddl)                                       |
| 🏆 Proyecto 1 | [E-commerce Global](./04-ddl/proyectos/proyecto_ecommerce_ddl.md)     |
| 🏆 Proyecto 2 | [Sistema Hospitalario](./04-ddl/proyectos/proyecto_hospital_ddl.md)   |
| 🏆 Proyecto 3 | [Gestión de Aerolíneas](./04-ddl/proyectos/proyecto_aerolinea_ddl.md) |

</details>

<details>
<summary>[x] <b>📝 Tema 05: DML (Data Manipulation Language)</b></summary>
<br>
Cómo inyectar vida a las tablas. Inserción, actualización y borrado de registros usando <code>INSERT</code>, <code>UPDATE</code> y <code>DELETE</code>.
<br><br>

| Recurso       | Enlace                                                            |
| ------------- | ----------------------------------------------------------------- |
| 📖 Temario    | [**Ir al Tema 05 →**](./05-dml)                                   |
| 🏆 Proyecto 1 | [El Día Cero](./05-dml/proyectos/proyecto_triatlon_dml.md)        |
| 🏆 Proyecto 2 | [Día 30 en Producción](./05-dml/proyectos/proyecto_DML_parte2.md) |

</details>

### 🔍 Bloque III — Consultar y Filtrar

<details>
<summary>[ ] <b>🔍 Tema 06: DQL Básico (Data Query Language)</b></summary>
<br>
El poder de la extracción de datos. Primeros pasos con <code>SELECT</code>, <code>FROM</code> y buenas prácticas de consulta.
<br><br>

| Recurso       | Enlace                                                                  |
| ------------- | ----------------------------------------------------------------------- |
| 📖 Temario    | [**Ir al Tema 06 →**](./06-dql)                                         |
| 🏆 Proyecto 1 | [Extracción de Inteligencia](./06-dql/proyectos/proyecto_dql_basico.md) |
| 🏆 Proyecto 2 | [Reportes Gerenciales](./06-dql/proyectos/proyecto_dql_medio.md)        |

</details>

<details>
<summary>[ ] <b>🎛️ Tema 07: Operadores Lógicos y Filtros</b></summary>
<br>
Afinando las búsquedas con la cláusula <code>WHERE</code>, operadores lógicos (<code>AND</code>, <code>OR</code>, <code>NOT</code>) y ordenación con <code>ORDER BY</code>.
<br><br>

| Recurso    | Enlace                                 |
| ---------- | -------------------------------------- |
| 📖 Temario | [**Ir al Tema 07 →**](./07-operadores) |

</details>

<details>
<summary>[ ] <b>🛠️ Tema 08: Funciones Nativas de Oracle</b></summary>
<br>
Manipulación avanzada en vuelo: funciones de texto, fechas, conversión (<code>TO_CHAR</code>, <code>TO_DATE</code>) y manejo de nulos (<code>NVL</code>, <code>COALESCE</code>).
<br><br>

| Recurso    | Enlace                                |
| ---------- | ------------------------------------- |
| 📖 Temario | [**Ir al Tema 08 →**](./08-funciones) |

</details>

### 🔗 Bloque IV — Relaciones y Consultas Avanzadas

<details>
<summary>[ ] <b>🔗 Tema 09: Relaciones y JOINs</b></summary>
<br>
Cruzando datos de múltiples tablas. Dominio absoluto de <code>INNER JOIN</code>, <code>LEFT/RIGHT JOIN</code>, <code>FULL OUTER JOIN</code> y <code>CROSS JOIN</code>.
<br><br>

| Recurso    | Enlace                            |
| ---------- | --------------------------------- |
| 📖 Temario | [**Ir al Tema 09 →**](./09-joins) |

</details>

<details>
<summary>[ ] <b>🪆 Tema 10: Subconsultas</b></summary>
<br>
Consultas anidadas dentro de otras consultas. Uso de subconsultas escalares, correlacionadas y operadores <code>IN</code>, <code>EXISTS</code>, <code>ANY</code>, <code>ALL</code>.
<br><br>

| Recurso    | Enlace                                   |
| ---------- | ---------------------------------------- |
| 📖 Temario | [**Ir al Tema 10 →**](./10-subconsultas) |

</details>

<details>
<summary>[ ] <b>🖼️ Tema 11: Vistas y Objetos de BD</b></summary>
<br>
Creación de Vistas (<code>VIEWS</code>), Secuencias (<code>SEQUENCES</code>) para autoincrementales, y Sinónimos para simplificar la arquitectura.
<br><br>

| Recurso    | Enlace                             |
| ---------- | ---------------------------------- |
| 📖 Temario | [**Ir al Tema 11 →**](./11-vistas) |

</details>

### ⚙️ Bloque V — Rendimiento y Seguridad

<details>
<summary>[ ] <b>⚡ Tema 12: Indexación</b></summary>
<br>
Optimizando la velocidad de lectura. Creación y funcionamiento de Índices (B-Tree, Bitmap) y cuándo no usarlos.
<br><br>

| Recurso    | Enlace                                 |
| ---------- | -------------------------------------- |
| 📖 Temario | [**Ir al Tema 12 →**](./12-indexacion) |

</details>

<details>
<summary>[ ] <b>🛡️ Tema 13: Transacciones y Propiedades ACID</b></summary>
<br>
Gestión de bloques de operaciones seguras con <code>COMMIT</code>, <code>ROLLBACK</code> y <code>SAVEPOINT</code>. Teoría avanzada: Atomicidad, Consistencia, Aislamiento (Isolation) y Durabilidad.
<br><br>

| Recurso    | Enlace                                         |
| ---------- | ---------------------------------------------- |
| 📖 Temario | [**Ir al Tema 13 →**](./13-transacciones-acid) |

</details>

<details>
<summary>[ ] <b>🔒 Tema 14: DCL y Seguridad (Data Control Language)</b></summary>
<br>
Control de acceso a la base de datos. <code>GRANT</code>, <code>REVOKE</code>, roles, privilegios de sistema y de objeto, y el principio de mínimo privilegio.
<br><br>

| Recurso    | Enlace                                    |
| ---------- | ----------------------------------------- |
| 📖 Temario | [**Ir al Tema 14 →**](./14-dcl-seguridad) |

</details>

### 🚀 Bloque VI — SQL de Élite

<details>
<summary>[ ] <b>🚀 Tema 15: SQL Avanzado (Window Functions y CTEs)</b></summary>
<br>
Consultas de nivel analítico: Funciones de ventana (<code>ROW_NUMBER</code>, <code>RANK</code>, <code>OVER</code>), CTEs con la cláusula <code>WITH</code> y operadores de conjuntos (<code>UNION</code>, <code>INTERSECT</code>, <code>MINUS</code>).
<br><br>

| Recurso    | Enlace                                   |
| ---------- | ---------------------------------------- |
| 📖 Temario | [**Ir al Tema 15 →**](./15-sql-avanzado) |

</details>

### 🧠 Bloque VII — Programación en la Base de Datos

<details>
<summary>[ ] <b>⚙️ Tema 16: PL/SQL (Programación Procedural)</b></summary>
<br>
Programación dentro de la base de datos. Bloques anónimos, procedimientos almacenados (<code>STORED PROCEDURES</code>), funciones, cursores, manejo de excepciones y triggers.
<br><br>

| Recurso    | Enlace                            |
| ---------- | --------------------------------- |
| 📖 Temario | [**Ir al Tema 16 →**](./16-plsql) |

</details>

### 🌐 Bloque VIII — El Mundo Real (Capstone)

<details>
<summary>[ ] <b>🏎️ Tema 17: Optimización de Consultas</b></summary>
<br>
Planes de ejecución (Explain Plan), cuellos de botella y mejores prácticas para que el SQL "vuele".
<br><br>

| Recurso    | Enlace                                   |
| ---------- | ---------------------------------------- |
| 📖 Temario | [**Ir al Tema 17 →**](./17-optimizacion) |

</details>

<details>
<summary>[ ] <b>🌐 Tema 18: Ecosistema SQL</b></summary>
<br>
Diferencias entre motores (Oracle, PostgreSQL, SQL Server, MySQL) y herramientas de mercado.
<br><br>

| Recurso    | Enlace                                     |
| ---------- | ------------------------------------------ |
| 📖 Temario | [**Ir al Tema 18 →**](./18-ecosistema-sql) |

</details>

<details>
<summary>[ ] <b>💼 Tema 19: Casos Reales y Proyectos Finales</b></summary>
<br>
Desafíos complejos que simulan el día a día de un Data Engineer y un DBA en producción. El <b>capstone</b> que integra todo lo aprendido.
<br><br>

| Recurso    | Enlace                                   |
| ---------- | ---------------------------------------- |
| 📖 Temario | [**Ir al Tema 19 →**](./19-casos-reales) |

</details>

---

<div align="center">

### 🎯 ¿Listo para empezar?

[**📦 Comienza con el Tema 01: Fundamentos de Bases de Datos →**](./01-fundamentos-bd)

</div>

---

_Este repositorio es un proyecto personal en constante crecimiento. ¡Siente libre de explorar!_
