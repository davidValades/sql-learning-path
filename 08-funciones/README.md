# 🛠️ Tema 08: Funciones Nativas de Oracle

> **"Los datos crudos son diamantes sin pulir."** Las funciones de Oracle son tus herramientas de joyero: transforman, formatean, calculan y resumen la información para que brille con todo su potencial.

## 📋 Índice

- [8.1 Funciones de Cadena](#81-funciones-de-cadena)
- [8.2 Funciones Numéricas](#82-funciones-numéricas)
- [8.3 Funciones de Fecha](#83-funciones-de-fecha)
- [8.4 Funciones de Conversión](#84-funciones-de-conversión)
- [8.5 Manejo de Nulos](#85-manejo-de-nulos)
- [8.6 Funciones de Agregación y GROUP BY](#86-funciones-de-agregación-y-group-by)

---

---

## 8.1 Funciones de Cadena

### 📘 El Concepto

Las funciones de cadena permiten manipular textos (valores `VARCHAR2` y `CHAR`). No modifican los datos almacenados en la tabla, solo transforman la salida de la consulta.

| Función | Descripción | Ejemplo | Resultado |
|---------|-------------|---------|-----------|
| `UPPER(cadena)` | Convierte a mayúsculas | `UPPER('hola')` | `'HOLA'` |
| `LOWER(cadena)` | Convierte a minúsculas | `LOWER('HOLA')` | `'hola'` |
| `INITCAP(cadena)` | Primera letra de cada palabra en mayúscula | `INITCAP('hola mundo')` | `'Hola Mundo'` |
| `SUBSTR(cadena, inicio, longitud)` | Extrae una subcadena | `SUBSTR('Oracle',1,3)` | `'Ora'` |
| `LENGTH(cadena)` | Devuelve la longitud | `LENGTH('SQL')` | `3` |
| `TRIM(cadena)` | Elimina espacios al inicio y final | `TRIM('  hola  ')` | `'hola'` |
| `LTRIM / RTRIM` | Elimina espacios a la izquierda / derecha | `LTRIM('  hola')` | `'hola'` |
| `REPLACE(cadena, buscar, reemplazo)` | Reemplaza ocurrencias | `REPLACE('SQL','S','P')` | `'PQL'` |
| `INSTR(cadena, subcadena)` | Posición de la primera aparición | `INSTR('Oracle','ac')` | `3` |
| `LPAD(cadena, longitud, relleno)` | Rellena por la izquierda | `LPAD('42',5,'0')` | `'00042'` |
| `RPAD(cadena, longitud, relleno)` | Rellena por la derecha | `RPAD('A',4,'*')` | `'A***'` |
| `CONCAT(a, b)` | Concatena dos cadenas (también `\|\|`) | `CONCAT('Ho','la')` | `'Hola'` |

> 📌 En Oracle se usa preferentemente el operador `||` para concatenar: `'Ho' || 'la'` → `'Hola'`.

### 🏠 La Analogía

Piensa en un **taller de camisetas personalizadas**. El cliente te da un texto cualquiera ("hOlA mUnDo") y tú tienes herramientas para:
- Ponerlo todo en MAYÚSCULAS (`UPPER`) → estampado gritón.
- Solo la primera letra en mayúscula (`INITCAP`) → estampado elegante.
- Recortar solo las primeras 5 letras (`SUBSTR`) → diseño minimalista.
- Medir cuántas letras tiene (`LENGTH`) → para saber si cabe en la camiseta.

### 💻 El Código

```sql
-- E-commerce: nombre de productos en mayúsculas y su longitud
SELECT UPPER(nombre) AS nombre_mayusculas,
       LENGTH(nombre) AS longitud_nombre
FROM productos;

-- Hospital: extraer las iniciales del nombre completo del médico
SELECT nombre_completo,
       SUBSTR(nombre_completo, 1, 1) || '.' AS inicial
FROM medicos;

-- Aerolínea: formatear el id_ruta para mostrar origen → destino
SELECT id_ruta,
       SUBSTR(id_ruta, 1, 3) || ' → ' || SUBSTR(id_ruta, 4, 3) AS ruta_formateada
FROM rutas;

-- Hospital: buscar si el nombre del médico contiene 'López'
SELECT nombre_completo,
       INSTR(nombre_completo, 'López') AS posicion_lopez
FROM medicos;

-- E-commerce: generar códigos de producto rellenados con ceros
SELECT nombre,
       LPAD(id_producto, 6, '0') AS codigo_formateado
FROM productos;
-- Resultado: Laptop Pro → 000010
```

### 🧠 El Reto

El departamento de marketing del E-commerce necesita generar etiquetas para productos. Cada etiqueta debe mostrar el nombre del producto en **mayúsculas**, seguido de un guion y las **primeras 3 letras** del nombre de la categoría en **minúsculas**.

**Pregunta:** Escribe una consulta que muestre: `UPPER(p.nombre)` como `etiqueta_nombre` y `LOWER(SUBSTR(c.nombre_categoria, 1, 3))` como `prefijo_cat`, uniendo `productos p` y `categorias c` por `id_categoria`. _(Si aún no conoces JOIN, usa una subconsulta o simplemente genera la etiqueta solo con la tabla productos)_.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Versión sin JOIN (solo productos):
SELECT UPPER(nombre) AS etiqueta_nombre,
       LPAD(id_producto, 6, '0') AS codigo
FROM productos;

-- Versión anticipando JOIN (Tema 08):
SELECT UPPER(p.nombre) AS etiqueta_nombre,
       LOWER(SUBSTR(c.nombre_categoria, 1, 3)) AS prefijo_cat
FROM productos p, categorias c
WHERE p.id_categoria = c.id_categoria;
```

</details>

---

---

## 8.2 Funciones Numéricas

### 📘 El Concepto

Estas funciones operan sobre valores numéricos (`NUMBER`) y son esenciales para cálculos financieros, estadísticos y de presentación.

| Función | Descripción | Ejemplo | Resultado |
|---------|-------------|---------|-----------|
| `ROUND(n, decimales)` | Redondea al número de decimales | `ROUND(45.678, 1)` | `45.7` |
| `TRUNC(n, decimales)` | Trunca (corta sin redondear) | `TRUNC(45.678, 1)` | `45.6` |
| `MOD(n, divisor)` | Resto de la división (módulo) | `MOD(10, 3)` | `1` |
| `ABS(n)` | Valor absoluto | `ABS(-42)` | `42` |
| `CEIL(n)` | Redondea hacia arriba (entero) | `CEIL(4.1)` | `5` |
| `FLOOR(n)` | Redondea hacia abajo (entero) | `FLOOR(4.9)` | `4` |
| `POWER(base, exp)` | Potencia | `POWER(2, 3)` | `8` |
| `SQRT(n)` | Raíz cuadrada | `SQRT(16)` | `4` |

> 📌 `ROUND` vs `TRUNC`: `ROUND(45.678, 1)` = 45.7 (redondea), `TRUNC(45.678, 1)` = 45.6 (simplemente corta). La diferencia importa mucho en finanzas.

### 🏠 La Analogía

Imagina que vas al cajero automático y tu saldo es **127.856€**:
- El cajero te muestra **127.86€** → usó `ROUND` (redondeó).
- Tu extracto bancario muestra **127.85€** → usó `TRUNC` (cortó sin redondear).
- Quieres saber si un número es par o impar → `MOD(numero, 2)`: si da 0 es par, si da 1 es impar.

### 💻 El Código

```sql
-- E-commerce: precio con IVA (21%) redondeado a 2 decimales
SELECT nombre,
       precio,
       ROUND(precio * 1.21, 2) AS precio_con_iva
FROM productos;
-- Laptop Pro: 1220 * 1.21 = 1476.20

-- Hospital: salarios truncados a enteros (sin céntimos)
SELECT nombre_completo,
       salario_base,
       TRUNC(salario_base, 0) AS salario_entero
FROM medicos;

-- Aerolínea: distancia en millas (1 km = 0.621371 millas), redondeado
SELECT id_ruta,
       distancia_km,
       ROUND(distancia_km * 0.621371, 0) AS distancia_millas
FROM rutas;

-- Hospital: identificar médicos con id par o impar
SELECT nombre_completo,
       id_medico,
       MOD(id_medico, 2) AS es_impar
FROM medicos;
```

### 🧠 El Reto

El E-commerce quiere mostrar precios redondeados al euro más cercano (**sin decimales**) para una campaña de "precios redondos". Pero el Sofá de Cuero (405.00) y la Laptop Pro (1220.00) ya son redondos, así que no los incluyas.

**Pregunta:** Muestra `nombre`, `precio` original y `ROUND(precio, 0)` como `precio_redondo` de los productos cuyo precio tenga decimales (es decir, donde `precio != ROUND(precio, 0)`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       precio,
       ROUND(precio, 0) AS precio_redondo
FROM productos
WHERE precio != ROUND(precio, 0);
```

Resultado: Ratón Inalámbrico (45.50→46), Lámpara LED (40.50→41), Camiseta Básica (19.99→20), Zapatillas Running (89.50→90).

</details>

---

---

## 8.3 Funciones de Fecha

### 📘 El Concepto

Oracle trata las fechas como un tipo especial (`DATE` y `TIMESTAMP`) con su propio conjunto de funciones. Estas funciones son críticas para informes, cálculos de plazos y análisis temporal.

| Función | Descripción | Ejemplo |
|---------|-------------|---------|
| `SYSDATE` | Fecha y hora actual del servidor | `SELECT SYSDATE FROM DUAL` |
| `ADD_MONTHS(fecha, n)` | Suma n meses a una fecha | `ADD_MONTHS(SYSDATE, 3)` |
| `MONTHS_BETWEEN(f1, f2)` | Meses entre dos fechas | `MONTHS_BETWEEN(f1, f2)` |
| `EXTRACT(parte FROM fecha)` | Extrae año, mes o día | `EXTRACT(YEAR FROM SYSDATE)` |
| `LAST_DAY(fecha)` | Último día del mes | `LAST_DAY(SYSDATE)` |
| `NEXT_DAY(fecha, 'día')` | Próximo día de la semana | `NEXT_DAY(SYSDATE, 'LUNES')` |
| `TRUNC(fecha)` | Trunca a medianoche (00:00:00) | `TRUNC(SYSDATE)` |
| `ROUND(fecha)` | Redondea al día más cercano | `ROUND(SYSDATE)` |

> 📌 **Aritmética de fechas:** En Oracle puedes sumar/restar **días** directamente: `SYSDATE + 7` = dentro de 7 días. Para meses, usa `ADD_MONTHS`.
>
> 📌 **DUAL:** Es una tabla especial de Oracle con una sola fila. Se usa para evaluar expresiones: `SELECT SYSDATE FROM DUAL`.

### 🏠 La Analogía

Piensa en un **calendario inteligente**:
- `SYSDATE` es mirar el reloj ahora mismo.
- `ADD_MONTHS` es pasar las páginas del calendario: _"¿Qué fecha será dentro de 6 meses?"_.
- `MONTHS_BETWEEN` es contar las páginas entre dos fechas marcadas: _"¿Cuántos meses pasaron desde mi cumpleaños?"_.
- `EXTRACT` es arrancar solo un dato del calendario: _"Solo me interesa el año, no el mes ni el día"_.

### 💻 El Código

```sql
-- Hospital: edad aproximada de cada paciente (en años)
SELECT nombre,
       fecha_nacimiento,
       TRUNC(MONTHS_BETWEEN(SYSDATE, fecha_nacimiento) / 12) AS edad_aprox
FROM pacientes;

-- Hospital: extraer el año de nacimiento de cada paciente
SELECT nombre,
       EXTRACT(YEAR FROM fecha_nacimiento) AS anio_nacimiento
FROM pacientes;

-- Aerolínea: calcular la fecha de la próxima revisión (cada 6 meses desde hoy)
SELECT modelo,
       SYSDATE AS hoy,
       ADD_MONTHS(SYSDATE, 6) AS proxima_revision
FROM aviones;

-- Hospital: último día del mes de nacimiento de cada paciente
SELECT nombre,
       fecha_nacimiento,
       LAST_DAY(fecha_nacimiento) AS ultimo_dia_mes_nac
FROM pacientes;
```

### 🧠 El Reto

El hospital necesita saber cuántos **meses** han pasado desde el nacimiento de cada paciente hasta hoy. Muestra el resultado redondeado a **entero**.

**Pregunta:** Escribe una consulta que muestre `nombre`, `fecha_nacimiento` y `ROUND(MONTHS_BETWEEN(SYSDATE, fecha_nacimiento), 0)` como `meses_de_vida`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       fecha_nacimiento,
       ROUND(MONTHS_BETWEEN(SYSDATE, fecha_nacimiento), 0) AS meses_de_vida
FROM pacientes;
```

El resultado variará según la fecha actual. Por ejemplo, para Laura Martinez (1990-01-01), a mediados de 2025 serían aprox. 426 meses.

</details>

---

---

## 8.4 Funciones de Conversión

### 📘 El Concepto

Oracle es muy estricto con los tipos de datos. Las funciones de conversión permiten transformar un tipo en otro de forma controlada, usando **máscaras de formato**.

| Función | Convierte | Ejemplo |
|---------|-----------|---------|
| `TO_CHAR(valor, formato)` | Número/Fecha → Texto | `TO_CHAR(SYSDATE, 'DD/MM/YYYY')` |
| `TO_DATE(texto, formato)` | Texto → Fecha | `TO_DATE('15/03/2025','DD/MM/YYYY')` |
| `TO_NUMBER(texto, formato)` | Texto → Número | `TO_NUMBER('1.234,56','9G999D99','NLS_NUMERIC_CHARACTERS=,.')` |

#### Máscaras de formato para fechas más comunes:

| Máscara | Significado | Ejemplo |
|---------|-------------|---------|
| `YYYY` | Año con 4 dígitos | `2025` |
| `MM` | Mes con 2 dígitos | `03` |
| `DD` | Día con 2 dígitos | `15` |
| `HH24:MI:SS` | Hora (24h), minutos, segundos | `14:30:00` |
| `DAY` | Nombre del día de la semana | `LUNES` |
| `MONTH` | Nombre del mes | `MARZO` |
| `Q` | Trimestre (1-4) | `1` |

#### Máscaras de formato para números:

| Máscara | Significado | Ejemplo |
|---------|-------------|---------|
| `9` | Dígito (suprime ceros a la izquierda) | `TO_CHAR(42,'9999')` → `'  42'` |
| `0` | Dígito (muestra ceros a la izquierda) | `TO_CHAR(42,'0000')` → `'0042'` |
| `FM` | Elimina espacios de relleno | `TO_CHAR(42,'FM9999')` → `'42'` |
| `.` | Punto decimal | `TO_CHAR(3.14,'9.99')` → `'3.14'` |
| `,` | Separador de miles | `TO_CHAR(1220,'9,999')` → `'1,220'` |
| `L` | Símbolo de moneda local | `TO_CHAR(50,'L999')` → `'€50'` |

### 🏠 La Analogía

Piensa en un **traductor simultáneo** en una conferencia internacional:
- `TO_CHAR` traduce el idioma de Oracle (fecha/número) al idioma humano (texto legible).
- `TO_DATE` traduce del idioma humano (texto) al idioma de Oracle (fecha interna).
- `TO_NUMBER` traduce un texto que parece un número a un número de verdad.

Sin el traductor, Oracle y el humano hablan idiomas diferentes y no se entienden.

### 💻 El Código

```sql
-- Hospital: mostrar fecha de nacimiento en formato español DD/MM/YYYY
SELECT nombre,
       TO_CHAR(fecha_nacimiento, 'DD/MM/YYYY') AS fecha_esp
FROM pacientes;
-- Laura Martinez → '01/01/1990'

-- E-commerce: precios formateados con símbolo de moneda
SELECT nombre,
       TO_CHAR(precio, 'FM999,999.00') || ' €' AS precio_formateado
FROM productos;
-- Laptop Pro → '1,220.00 €'

-- Hospital: convertir un texto a fecha para comparar
SELECT nombre
FROM pacientes
WHERE fecha_nacimiento > TO_DATE('01/01/1995', 'DD/MM/YYYY');
-- Resultado: Pedro Sánchez (2000-03-15)

-- Aerolínea: mostrar el año de fabricación como texto con prefijo
SELECT modelo,
       'Año: ' || TO_CHAR(anio_fabricacion) AS info_fabricacion
FROM aviones;
```

### 🧠 El Reto

El departamento financiero del E-commerce necesita un informe donde el precio se muestre como texto formateado con **dos decimales, separador de miles y el símbolo €** al final. Por ejemplo: `1,220.00 €`.

**Pregunta:** Escribe una consulta que muestre `nombre` y el precio formateado como se describe. Usa `TO_CHAR` con la máscara apropiada.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       TO_CHAR(precio, 'FM999,999.00') || ' €' AS precio_formateado
FROM productos;
```

Resultado:
- Laptop Pro → `1,220.00 €`
- Ratón Inalámbrico → `45.50 €`
- Sofá de Cuero → `405.00 €`
- etc.

</details>

---

---

## 8.5 Manejo de Nulos

### 📘 El Concepto

Los valores `NULL` son el talón de Aquiles de muchas consultas. Oracle proporciona varias funciones para **manejarlos con elegancia** en lugar de dejar que arruinen tus resultados.

| Función | Descripción | Ejemplo |
|---------|-------------|---------|
| `NVL(expr, valor_si_null)` | Si `expr` es NULL, devuelve `valor_si_null` | `NVL(telefono, 'Sin teléfono')` |
| `NVL2(expr, si_no_null, si_null)` | Si NO es NULL devuelve el 2º; si es NULL, el 3º | `NVL2(telefono, 'Sí', 'No')` |
| `COALESCE(e1, e2, e3, ...)` | Devuelve el **primer valor no NULL** de la lista | `COALESCE(tel1, tel2, 'N/A')` |
| `NULLIF(e1, e2)` | Devuelve NULL si `e1 = e2`, si no devuelve `e1` | `NULLIF(stock, 0)` |

> 📌 **`NVL` vs `COALESCE`**: `NVL` solo acepta 2 parámetros. `COALESCE` acepta muchos y es estándar SQL (más portable). `COALESCE` también es más eficiente: evalúa de izquierda a derecha y se detiene en el primer no-NULL.

### 🏠 La Analogía

Piensa en un **menú de restaurante con platos agotados**:
- `NVL`: _"Si el plato principal está agotado (NULL), tráeme la alternativa"_.
- `NVL2`: _"Si el plato está disponible (no NULL), ponle salsa especial. Si está agotado (NULL), tráeme el postre"_.
- `COALESCE`: _"Tráeme el primer plato que esté disponible de esta lista: salmón, pollo, ensalada, pan"_.
- `NULLIF`: _"Si el stock es 0, considéralo como si no existiera (NULL)"_. Útil para evitar divisiones por cero.

### 💻 El Código

```sql
-- Hospital: reemplazar teléfonos NULL por un texto descriptivo
SELECT nombre,
       NVL(telefono, 'Sin teléfono registrado') AS telefono_mostrado
FROM pacientes;
-- Ana Ruiz → 'Sin teléfono registrado'

-- Hospital: indicar si el paciente tiene o no teléfono
SELECT nombre,
       NVL2(telefono, '✅ Sí', '❌ No') AS tiene_telefono
FROM pacientes;

-- E-commerce: usar COALESCE para mostrar código de barras o un código por defecto
SELECT nombre,
       COALESCE(codigo_barras, 'PEND-' || LPAD(id_producto, 4, '0')) AS codigo_final
FROM productos;
-- Monitor 4K → 'PEND-0016' (porque codigo_barras es NULL)

-- E-commerce: convertir stock 0 en NULL con NULLIF (útil para evitar división por cero)
SELECT nombre,
       stock,
       NULLIF(stock, 0) AS stock_o_null
FROM productos;
-- Sofá de Cuero: stock=0, stock_o_null=NULL
```

### 🧠 El Reto

El equipo de atención al cliente del hospital necesita un directorio de pacientes con un **número de contacto garantizado**. Si el paciente tiene `telefono`, se muestra ese. Si no, se debe mostrar el texto `'CONTACTAR POR CORREO'`.

**Pregunta:** Escribe la consulta usando `NVL` que muestre `nombre` y el teléfono o el texto alternativo como `contacto`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       NVL(telefono, 'CONTACTAR POR CORREO') AS contacto
FROM pacientes;
```

Resultado:
- Laura Martinez → 555-9999
- Carlos Gomez → 555-0002
- Pedro Sánchez → 555-0003
- Ana Ruiz → CONTACTAR POR CORREO
- David Torres → 555-0005

</details>

---

---

## 8.6 Funciones de Agregación y GROUP BY

### 📘 El Concepto

Las funciones de agregación operan sobre **conjuntos de filas** y devuelven un **único valor resumen**. Son la base de cualquier informe o dashboard.

| Función | Descripción | Ejemplo |
|---------|-------------|---------|
| `COUNT(*)` | Cuenta todas las filas | `SELECT COUNT(*) FROM productos` |
| `COUNT(columna)` | Cuenta filas donde la columna NO es NULL | `COUNT(codigo_barras)` |
| `SUM(columna)` | Suma los valores | `SUM(precio)` |
| `AVG(columna)` | Promedio (media aritmética) | `AVG(salario_base)` |
| `MIN(columna)` | Valor mínimo | `MIN(precio)` |
| `MAX(columna)` | Valor máximo | `MAX(distancia_km)` |

#### GROUP BY: Agrupar para resumir

`GROUP BY` divide las filas en grupos según los valores de una o más columnas, y aplica la función de agregación **a cada grupo por separado**.

```
SELECT columna_grupo, FUNCION_AGREGACION(columna_valor)
FROM tabla
GROUP BY columna_grupo;
```

> ⚠️ **Regla de oro del GROUP BY:** Toda columna que aparezca en el `SELECT` y que **no** esté dentro de una función de agregación, **DEBE** estar en el `GROUP BY`.

#### HAVING: Filtrar grupos

`HAVING` es el `WHERE` de los grupos. Mientras que `WHERE` filtra **filas individuales** antes de agrupar, `HAVING` filtra **grupos** después de agregar.

```
SELECT columna_grupo, COUNT(*)
FROM tabla
GROUP BY columna_grupo
HAVING COUNT(*) > 2;
```

| Cláusula | ¿Cuándo filtra? | ¿Qué filtra? |
|----------|-----------------|---------------|
| `WHERE` | **Antes** de agrupar | Filas individuales |
| `HAVING` | **Después** de agrupar | Grupos completos |

### 🏠 La Analogía

Imagina que eres el director de un colegio y quieres saber **cuántos alumnos hay por clase**:
1. Primero **agrupas** a todos los alumnos por su clase → `GROUP BY clase`.
2. Luego **cuentas** cuántos hay en cada grupo → `COUNT(*)`.
3. Si solo quieres ver las clases con más de 30 alumnos → `HAVING COUNT(*) > 30`.

`WHERE` sería el filtro **antes** de agrupar: _"Solo cuenta a los alumnos que han aprobado"_.
`HAVING` sería el filtro **después** de agrupar: _"Solo muéstrame las clases con más de 30 aprobados"_.

### 💻 El Código

```sql
-- E-commerce: estadísticas generales de productos
SELECT COUNT(*) AS total_productos,
       ROUND(AVG(precio), 2) AS precio_medio,
       MIN(precio) AS precio_minimo,
       MAX(precio) AS precio_maximo,
       SUM(stock) AS stock_total
FROM productos;
-- 8 productos, precio medio ~280.69, min 19.99, max 1220.00, stock total 530

-- E-commerce: número de productos y precio medio POR CATEGORÍA
SELECT id_categoria,
       COUNT(*) AS num_productos,
       ROUND(AVG(precio), 2) AS precio_medio
FROM productos
GROUP BY id_categoria;
-- Cat 1(Electrónica): 4 productos, precio medio 422.63
-- Cat 2(Hogar): 2 productos, precio medio 222.75
-- Cat 3(Ropa): 2 productos, precio medio 54.75

-- Hospital: salario medio por especialidad, solo las que superen 3000
SELECT id_especialidad,
       COUNT(*) AS num_medicos,
       ROUND(AVG(salario_base), 2) AS salario_medio
FROM medicos
GROUP BY id_especialidad
HAVING AVG(salario_base) > 3000;
-- Especialidad 200 (Neurología): 2 médicos, salario medio 4250.00

-- Aerolínea: aviones por década de fabricación
SELECT TRUNC(anio_fabricacion, -1) AS decada,
       COUNT(*) AS num_aviones
FROM aviones
GROUP BY TRUNC(anio_fabricacion, -1);
-- 2010: 2 aviones, 2020: 2 aviones

-- E-commerce: categorías con más de 2 productos
SELECT id_categoria,
       COUNT(*) AS total
FROM productos
GROUP BY id_categoria
HAVING COUNT(*) > 2;
-- Cat 1 (Electrónica): 4 productos

-- Hospital: contar pacientes con y sin teléfono
SELECT NVL2(telefono, 'Con teléfono', 'Sin teléfono') AS estado_contacto,
       COUNT(*) AS cantidad
FROM pacientes
GROUP BY NVL2(telefono, 'Con teléfono', 'Sin teléfono');
-- Con teléfono: 4, Sin teléfono: 1
```

### 🧠 El Reto

El CEO del E-commerce quiere un informe que muestre, **por cada categoría**, el número de productos y la suma total del valor del inventario (`precio * stock`). Solo debe mostrar las categorías cuyo valor total de inventario **supere los 5000 euros**.

**Pregunta:** Escribe la consulta usando `GROUP BY` y `HAVING`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT id_categoria,
       COUNT(*) AS num_productos,
       SUM(precio * stock) AS valor_inventario
FROM productos
GROUP BY id_categoria
HAVING SUM(precio * stock) > 5000;
```

Resultado:
- Cat 1 (Electrónica): 4 productos, valor = (1220*50) + (45.50*200) + (350*25) + (75*80) = 61000 + 9100 + 8750 + 6000 = 84850.00
- Cat 3 (Ropa): 2 productos, valor = (19.99*100) + (89.50*45) = 1999 + 4027.50 = 6026.50

Cat 2 (Hogar) no supera 5000: (405*0) + (40.50*30) = 0 + 1215 = 1215.00

</details>

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 08

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Completar los ejercicios | 🏋️ [Ir a Ejercicios →](./ejercicios/ejercicios_funciones.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Ir a Proyecto →](./proyectos/proyecto_funciones.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 09: Relaciones y JOINs](../09-joins) |

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 07**](../07-operadores/proyectos/proyecto_operadores.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 08 →**](ejercicios/ejercicios_funciones.md)

</div>
