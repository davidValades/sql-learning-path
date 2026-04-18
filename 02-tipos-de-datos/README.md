## 2.1 Tipos Numéricos (El poder de NUMBER en Oracle)

### 📘 El Concepto

Cuando creamos una columna, la base de datos no es adivina; tenemos que decirle exactamente qué tipo de información va a guardar. Si es un número, debemos especificar si es un número entero (sin decimales) o un número exacto (con decimales).

En el estándar SQL existen muchos tipos (`INT`, `BIGINT`, `DECIMAL`, `FLOAT`). Sin embargo, **Oracle es especial**. Oracle simplificó todo esto creando un tipo de dato todopoderoso llamado **`NUMBER`**.

La magia de `NUMBER` es que tú le dices su tamaño usando dos valores: **Precisión** (total de dígitos) y **Escala** (cuántos de esos dígitos son decimales).
Se escribe así: `NUMBER(p, s)`

### 🏠 La Analogía

Imagina que vas a comprar cajas para una mudanza.

- Si vas a guardar un anillo, compras una caja diminuta.
- Si vas a guardar una televisión, compras una caja enorme.

En las bases de datos pasa lo mismo. Si vas a guardar la edad de una persona (que como mucho tendrá 3 cifras, ej: 105), no reservas el espacio de un número de 20 cifras. Le dices al motor: "Dame una caja pequeña". Eso ahorra millones de gigabytes en empresas grandes.

### 💻 El Código (Sintaxis Oracle)

Vamos a ver cómo usaríamos el todopoderoso `NUMBER` al crear una tabla:

```sql
CREATE TABLE empleados (
    id_empleado NUMBER(5),      -- 'Caja' para números de hasta 5 cifras (ej: 99999). Sin decimales.
    edad NUMBER(3),             -- 'Caja' para números de hasta 3 cifras (ej: 120).
    salario NUMBER(7, 2)        -- ¡Atención aquí! 7 cifras EN TOTAL, de las cuales 2 son decimales.
                                -- El máximo sería: 99999.99
);
```

(Nota sobre estándar SQL: Si no estuvieras en Oracle, en lugar de NUMBER(5) usarías INT, y en lugar de NUMBER(7,2) usarías DECIMAL(7,2)).

### 🧠 El Reto de la Lección

Acabas de entrar a trabajar en Amazon y te piden que diseñes la tabla Inventario. Tienes que crear una columna para el Peso de los paquetes (en kilos).

Te dan las siguientes reglas de negocio:

El paquete más pesado que Amazon puede enviar pesa 950.50 kilos.

Necesitamos exactamente dos decimales de precisión para calcular bien los costes de envío de cosas ligeras.

**Pregunta:** Teniendo en cuenta que estás trabajando en Oracle... ¿Cómo escribirías exactamente el tipo de dato de la columna Peso para que sea lo más eficiente posible, cumpliendo con la regla del peso máximo 950.50?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>
<br>
<b>Respuesta del Profesor:</b><br>
La opción más eficiente en Oracle es <code>NUMBER(5,2)</code>. Esto significa que el número tendrá 5 dígitos en total (precisión), de los cuales 2 serán decimales (escala). Esto permite guardar valores hasta <code>999.99</code>, cubriendo perfectamente el peso máximo de 950.50 kilos sin desperdiciar espacio en memoria.
</details>

---

---

## 2.2 Cadenas de Texto (CHAR vs VARCHAR2)

### 📘 El Concepto

En SQL existen dos formas principales de guardar texto. Es vital elegir bien para no desperdiciar espacio:

1. **`VARCHAR2(n)`**: La "2" es específica de Oracle. Es de **longitud variable**. Si defines `VARCHAR2(100)` pero solo escribes "Pepe" (4 letras), Oracle solo gasta espacio para esas 4 letras.
2. **`CHAR(n)`**: Es de **longitud fija**. Si defines `CHAR(100)` y escribes "Pepe", Oracle guarda las 4 letras y añade **96 espacios en blanco** para rellenar.

### 🏠 La Analogía

Imagina que vas de viaje:

- **`VARCHAR2`** es una **bolsa de plástico**: se encoge o se estira según la ropa que metas dentro. No ocupa espacio extra en la maleta.
- **`CHAR`** es una **caja de madera rígida**: aunque metas solo un calcetín, la caja ocupa el mismo espacio enorme en el maletero. Solo la usamos cuando sabemos que el objeto siempre mide lo mismo (como una matrícula o un DNI).

### 💻 El Código

```sql
-- En Oracle siempre preferimos VARCHAR2 para casi todo
CREATE TABLE contactos (
    nombre VARCHAR2(50),   -- Variable: eficiente
    pais_iso CHAR(2)       -- Fija: siempre serán 2 letras (ES, MX, US...)
);
```

### 🧠 El Reto de la Lección

Estás configurando la tabla Usuarios en Oracle para una red social. Tienes que definir el tipo de dato para dos columnas:

Bio_Presentacion: Un texto donde el usuario cuenta su vida (máximo 500 caracteres).

Codigo_Postal: En tu país, los códigos postales siempre tienen exactamente 5 dígitos.

Pregunta: ¿Qué tipo de dato y qué longitud le pondrías a cada una de estas dos columnas para ser el mejor Ingeniero de Datos de la clase?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

Para la <code>Bio_Presentacion</code> usaríamos <b><code>VARCHAR2(500)</code></b>, ya que no todos los usuarios escribirán 500 caracteres y no queremos desperdiciar espacio con los que escriban poco.

Para el <code>Codigo_Postal</code> usaríamos <b><code>CHAR(5)</code></b>, porque sabemos con absoluta certeza que siempre, sin excepción, va a medir 5 caracteres. Al ser de longitud fija, el motor de la base de datos lo procesará más rápido.

</details>

---

---

## 2.3 Fecha y Hora (DATE y TIMESTAMP)

### 📘 El Concepto

En las bases de datos no guardamos las fechas como si fueran texto (como "15 de mayo"). Necesitamos tipos especiales para poder hacer cálculos (ej: ¿cuántos días han pasado entre dos fechas?).

En Oracle, los dos protagonistas son:

1. **`DATE`**: Es el tipo más usado. Guarda **siglo, año, mes, día, hora, minuto y segundo**.
2. **`TIMESTAMP`**: Es una evolución de DATE. Se usa cuando necesitas **precisión quirúrgica**, ya que guarda fracciones de segundo (milisegundos) y puede gestionar zonas horarias (_Time Zones_).

### 🏠 La Analogía

- **`DATE`** es como un **reloj de pulsera clásico**: te da la hora y el segundo exacto en el que estás. Es suficiente para casi todo (saber cuándo se hizo un pedido, la fecha de nacimiento de alguien, etc.).
- **`TIMESTAMP`** es como el **cronómetro de la Fórmula 1**: te dice exactamente cuántas milésimas de segundo han pasado. Se usa en sistemas financieros donde cada milisegundo cuenta o en logs de servidores de alta velocidad.

### 💻 El Código

```sql
CREATE TABLE registros_vuelos (
    id_vuelo NUMBER(10) PRIMARY KEY,
    fecha_salida DATE,                    -- Fecha y hora normal
    momento_exacto_aterrizaje TIMESTAMP(6) -- Con 6 decimales de segundo
);
```

### 🧠 El Reto de la Lección

Trabajas en el departamento de datos de un Banco Central. Tienes que diseñar la tabla Transacciones_Bancarias. El Director de Seguridad te pide que la base de datos sea capaz de registrar el momento exacto de una transferencia con fracciones de segundo, para evitar fraudes que ocurren en millonésimas de segundo entre cuentas.

**Pregunta:** ¿Qué tipo de dato elegirías para la columna Fecha_Transferencia y por qué no usarías el tipo DATE en este caso específico?

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

Utilizaríamos <b><code>TIMESTAMP</code></b> (ej. <code>TIMESTAMP(6)</code>). El tipo <code>DATE</code> en Oracle se detiene en el nivel de los segundos, por lo que si dos transacciones ocurren en el mismo segundo exacto, registrarían la misma hora. <code>TIMESTAMP</code> guarda fracciones de segundo, garantizando la precisión necesaria para detectar el orden de operaciones ultrarrápidas.

(Nota sobre Booleanos: Oracle no tiene un tipo de dato BOOLEAN estándar para las tablas. La práctica profesional es usar NUMBER(1) guardando 0 o 1, o CHAR(1) guardando 'Y' o 'N' y aplicando una restricción CHECK).

</details>

---

---
---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 02

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Avanzar al siguiente tema | ➡️ [Tema 03: DDL (Data Definition Language)](../03-ddl) |

---

<div align="center">

⬅️ [**Tema 01: Fundamentos de Bases de Datos**](../01-fundamentos-bd) · 🏠 [**Índice del Curso**](../README.md) · [**Tema 03: DDL (Data Definition Language) →**](../03-ddl)

</div>
