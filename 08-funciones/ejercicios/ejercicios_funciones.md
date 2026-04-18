# 🏋️ Ejercicios — Tema 07: Funciones Nativas de Oracle

> 💡 **Instrucciones:** Resuelve cada ejercicio escribiendo la consulta SQL completa. Todos los ejercicios usan el estado acumulado de la base de datos (incluyendo la expansión de datos del Tema 06).

---

## Ejercicio 1 — E-commerce · Funciones de Cadena

**Enunciado:** Genera un listado de productos donde el nombre se muestre en **mayúsculas** y se añada una columna con la **longitud** del nombre. Ordena por longitud descendente.

Muestra: `UPPER(nombre)` como `nombre_mayus`, `LENGTH(nombre)` como `longitud`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT UPPER(nombre) AS nombre_mayus,
       LENGTH(nombre) AS longitud
FROM productos
ORDER BY longitud DESC;
```

Resultado:
1. ZAPATILLAS RUNNING (18)
2. RATÓN INALÁMBRICO (17)
3. TECLADO MECÁNICO (16)
4. CAMISETA BÁSICA (15)
5. SOFÁ DE CUERO (13)
6. LÁMPARA LED (11)
7. MONITOR 4K (10)
8. LAPTOP PRO (10)

</details>

---

## Ejercicio 2 — Hospital · SUBSTR e INSTR

**Enunciado:** Extrae el **título** de cada médico (los caracteres antes del primer espacio: "Dr." o "Dra.") y el **resto del nombre** (después del primer espacio).

Pista: Usa `INSTR` para encontrar la posición del primer espacio y `SUBSTR` para cortar.

Muestra: `titulo`, `nombre_sin_titulo`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT SUBSTR(nombre_completo, 1, INSTR(nombre_completo, ' ') - 1) AS titulo,
       SUBSTR(nombre_completo, INSTR(nombre_completo, ' ') + 1) AS nombre_sin_titulo
FROM medicos;
```

Resultado:
| titulo | nombre_sin_titulo |
|--------|-------------------|
| Dra. | Sarah Adams |
| Dra. | Elena Ruiz |
| Dr. | Carlos Vega |
| Dra. | Marta López |
| Dr. | Luis Moreno |

</details>

---

## Ejercicio 3 — Aerolínea · REPLACE y LPAD

**Enunciado:** Genera un informe de rutas donde el `id_ruta` se muestre con un formato de **código de vuelo**: las primeras 3 letras, un guion, y las últimas 3 letras. Además, muestra la distancia con **ceros a la izquierda** hasta 5 dígitos.

Ejemplo: `MADLHR` → `MAD-LHR`, distancia `1350` → `01350`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT SUBSTR(id_ruta, 1, 3) || '-' || SUBSTR(id_ruta, 4, 3) AS codigo_vuelo,
       LPAD(distancia_km, 5, '0') AS distancia_fmt
FROM rutas;
```

Resultado:
| codigo_vuelo | distancia_fmt |
|-------------|---------------|
| MAD-LHR | 01350 |
| JFK-LAX | 04000 |
| BCN-CDG | 00830 |
| MAD-FRA | 01420 |
| BCN-FCO | 00850 |

</details>

---

## Ejercicio 4 — E-commerce · ROUND y TRUNC

**Enunciado:** El departamento fiscal necesita dos versiones del precio con IVA (21%):
- Una **redondeada** a 2 decimales (`ROUND`).
- Una **truncada** a 2 decimales (`TRUNC`).

Muestra solo los productos cuya diferencia entre el redondeado y el truncado sea **mayor que 0** (es decir, donde el redondeo realmente cambie el valor).

Muestra: `nombre`, `precio_iva_round`, `precio_iva_trunc`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       ROUND(precio * 1.21, 2) AS precio_iva_round,
       TRUNC(precio * 1.21, 2) AS precio_iva_trunc
FROM productos
WHERE ROUND(precio * 1.21, 2) != TRUNC(precio * 1.21, 2);
```

Resultado: Los productos cuyo IVA genera un tercer decimal que causa diferencia entre ROUND y TRUNC. Por ejemplo:
- Ratón Inalámbrico: 45.50 * 1.21 = 55.055 → ROUND=55.06, TRUNC=55.05
- Lámpara LED: 40.50 * 1.21 = 48.405 → ROUND=48.41, TRUNC=48.40

</details>

---

## Ejercicio 5 — Hospital · Funciones de Fecha

**Enunciado:** Genera un informe de pacientes que muestre:
1. El **año de nacimiento** (usando `EXTRACT`).
2. La **edad aproximada en años** (usando `MONTHS_BETWEEN` y `TRUNC`).
3. La fecha en que cumplirán su **próximo aniversario de nacimiento** tras sumar el número correcto de años.

Solo muestra los pacientes nacidos **antes del año 2000**.

Muestra: `nombre`, `anio_nacimiento`, `edad_aprox`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       EXTRACT(YEAR FROM fecha_nacimiento) AS anio_nacimiento,
       TRUNC(MONTHS_BETWEEN(SYSDATE, fecha_nacimiento) / 12) AS edad_aprox
FROM pacientes
WHERE EXTRACT(YEAR FROM fecha_nacimiento) < 2000;
```

Resultado (aproximado en 2025):
- Laura Martinez (1990, ~35 años)
- Carlos Gomez (1985, ~40 años)
- Ana Ruiz (1978, ~46 años)
- David Torres (1995, ~29 años)

</details>

---

## Ejercicio 6 — E-commerce · TO_CHAR con formato

**Enunciado:** Genera un catálogo de precios formateado para impresión. Cada producto debe mostrar:
- El nombre en formato `INITCAP` (primera letra mayúscula).
- El precio formateado como moneda con separador de miles y 2 decimales, seguido de ` €`.

Ordena por precio descendente.

Muestra: `nombre_formato`, `precio_formato`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT INITCAP(nombre) AS nombre_formato,
       TO_CHAR(precio, 'FM999,999.00') || ' €' AS precio_formato
FROM productos
ORDER BY precio DESC;
```

Resultado:
1. Laptop Pro → 1,220.00 €
2. Sofá De Cuero → 405.00 €
3. Monitor 4K → 350.00 €
4. Zapatillas Running → 89.50 €
5. Teclado Mecánico → 75.00 €
6. Ratón Inalámbrico → 45.50 €
7. Lámpara Led → 40.50 €
8. Camiseta Básica → 19.99 €

</details>

---

## Ejercicio 7 — Hospital · NVL y NVL2

**Enunciado:** Genera un directorio de pacientes que muestre:
1. El nombre del paciente.
2. El teléfono, o `'PENDIENTE'` si es NULL (usa `NVL`).
3. Una columna `estado_contacto` que diga `'COMPLETO'` si tiene teléfono o `'INCOMPLETO'` si no (usa `NVL2`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       NVL(telefono, 'PENDIENTE') AS telefono_mostrado,
       NVL2(telefono, 'COMPLETO', 'INCOMPLETO') AS estado_contacto
FROM pacientes;
```

Resultado:
| nombre | telefono_mostrado | estado_contacto |
|--------|-------------------|-----------------|
| Laura Martinez | 555-9999 | COMPLETO |
| Carlos Gomez | 555-0002 | COMPLETO |
| Pedro Sánchez | 555-0003 | COMPLETO |
| Ana Ruiz | PENDIENTE | INCOMPLETO |
| David Torres | 555-0005 | COMPLETO |

</details>

---

## Ejercicio 8 — E-commerce · COALESCE y NULLIF

**Enunciado:** El equipo de logística necesita calcular el **precio por unidad en stock** (`precio / stock`). El problema es que el Sofá de Cuero tiene stock 0, lo que causaría un error de **división por cero**.

Usa `NULLIF` para convertir el stock 0 en NULL, y luego `COALESCE` para que el resultado de la división sea `0` cuando no se pueda calcular.

Muestra: `nombre`, `precio`, `stock`, `precio_por_unidad`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre,
       precio,
       stock,
       COALESCE(ROUND(precio / NULLIF(stock, 0), 2), 0) AS precio_por_unidad
FROM productos;
```

Resultado:
| nombre | precio | stock | precio_por_unidad |
|--------|--------|-------|-------------------|
| Laptop Pro | 1220.00 | 50 | 24.40 |
| Ratón Inalámbrico | 45.50 | 200 | 0.23 |
| Sofá de Cuero | 405.00 | 0 | 0 |
| Lámpara LED | 40.50 | 30 | 1.35 |
| Camiseta Básica | 19.99 | 100 | 0.20 |
| Zapatillas Running | 89.50 | 45 | 1.99 |
| Monitor 4K | 350.00 | 25 | 14.00 |
| Teclado Mecánico | 75.00 | 80 | 0.94 |

</details>

---

## Ejercicio 9 — Hospital · GROUP BY y COUNT

**Enunciado:** Genera un informe que muestre **cuántos médicos hay por especialidad** y el **salario medio** de cada especialidad, redondeado a 2 decimales. Ordena por número de médicos descendente.

Muestra: `id_especialidad`, `num_medicos`, `salario_medio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT id_especialidad,
       COUNT(*) AS num_medicos,
       ROUND(AVG(salario_base), 2) AS salario_medio
FROM medicos
GROUP BY id_especialidad
ORDER BY num_medicos DESC;
```

Resultado:
| id_especialidad | num_medicos | salario_medio |
|-----------------|-------------|---------------|
| 100 (Traumatología) | 2 | 3000.00 |
| 200 (Neurología) | 2 | 4250.00 |
| 300 (Med. General) | 1 | 2600.00 |

</details>

---

## Ejercicio 10 — E-commerce · GROUP BY con HAVING

**Enunciado:** El director financiero quiere ver las categorías donde el **stock total supere las 100 unidades**. Muestra el id de la categoría, la cantidad total de stock y el número de productos.

Muestra: `id_categoria`, `stock_total`, `num_productos`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT id_categoria,
       SUM(stock) AS stock_total,
       COUNT(*) AS num_productos
FROM productos
GROUP BY id_categoria
HAVING SUM(stock) > 100;
```

Resultado:
| id_categoria | stock_total | num_productos |
|-------------|-------------|---------------|
| 1 (Electrónica) | 355 | 4 |
| 3 (Ropa) | 145 | 2 |

Cat 2 (Hogar) tiene stock total = 30, no supera 100.

</details>

---

## Ejercicio 11 — Aerolínea · Funciones combinadas

**Enunciado:** Genera un informe de la flota que muestre:
1. El modelo del avión en **mayúsculas**.
2. El año de fabricación.
3. La **antigüedad en años** (año actual menos año de fabricación; usa `EXTRACT(YEAR FROM SYSDATE)`).
4. La capacidad formateada con texto: `'Cap: XXX pax'` (usa concatenación `||`).

Ordena por antigüedad descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT UPPER(modelo) AS modelo_mayus,
       anio_fabricacion,
       EXTRACT(YEAR FROM SYSDATE) - anio_fabricacion AS antiguedad_anios,
       'Cap: ' || capacidad_pasajeros || ' pax' AS capacidad_fmt
FROM aviones
ORDER BY antiguedad_anios DESC;
```

Resultado (en 2025):
| modelo_mayus | anio_fabricacion | antiguedad_anios | capacidad_fmt |
|-------------|------------------|------------------|---------------|
| BOEING 777 | 2016 | 9 | Cap: 350 pax |
| BOEING 737 | 2018 | 7 | Cap: 180 pax |
| EMBRAER E195 | 2020 | 5 | Cap: 120 pax |
| AIRBUS A350 | 2022 | 3 | Cap: 300 pax |

</details>

---

## Ejercicio 12 — Combinación total · Las tres bases de datos

**Enunciado:** Resuelve estas tres consultas independientes combinando funciones de distintos tipos:

**a)** E-commerce: Muestra cada producto con su nombre en mayúsculas, precio con IVA redondeado a 2 decimales, y el código de barras reemplazando NULL por `'SIN CÓDIGO'`. Agrupa por categoría mostrando cuántos productos hay y la suma de precios con IVA. Solo categorías con suma de precios con IVA **mayor a 100**.

**b)** Hospital: Para cada especialidad, muestra el número de médicos, el salario máximo y el salario mínimo. Añade una columna `rango_salarial` que sea la diferencia entre el máximo y el mínimo. Solo especialidades con **más de 1 médico**.

**c)** Aerolínea: Cuenta cuántas rutas salen de cada `aeropuerto_origen`, muestra la distancia media redondeada a entero y la distancia máxima. Solo orígenes con **más de 1 ruta**.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**a) E-commerce:**
```sql
SELECT id_categoria,
       COUNT(*) AS num_productos,
       ROUND(SUM(precio * 1.21), 2) AS suma_precio_iva
FROM productos
GROUP BY id_categoria
HAVING SUM(precio * 1.21) > 100;
```
Resultado:
- Cat 1: 4 productos, suma IVA = 2041.41
- Cat 2: 2 productos, suma IVA = 539.07
- Cat 3: 2 productos, suma IVA = 132.48 (supera 100, así que entra)

**b) Hospital:**
```sql
SELECT id_especialidad,
       COUNT(*) AS num_medicos,
       MAX(salario_base) AS salario_max,
       MIN(salario_base) AS salario_min,
       MAX(salario_base) - MIN(salario_base) AS rango_salarial
FROM medicos
GROUP BY id_especialidad
HAVING COUNT(*) > 1;
```
Resultado:
- Especialidad 100: 2 médicos, max=3200, min=2800, rango=400
- Especialidad 200: 2 médicos, max=4500, min=4000, rango=500

**c) Aerolínea:**
```sql
SELECT aeropuerto_origen,
       COUNT(*) AS num_rutas,
       ROUND(AVG(distancia_km), 0) AS distancia_media,
       MAX(distancia_km) AS distancia_max
FROM rutas
GROUP BY aeropuerto_origen
HAVING COUNT(*) > 1;
```
Resultado:
- MAD: 2 rutas, distancia media = 1385, max = 1420
- BCN: 2 rutas, distancia media = 840, max = 850

</details>

---

<div align="center">

### 🗺️ Navegación

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Repasar la teoría | 📖 [Volver al Temario](../README.md) |
| 2️⃣ | Completar los ejercicios | 🏋️ _Estás aquí_ |
| 3️⃣ | Completar el proyecto | 🏆 [Ir a Proyecto →](../proyectos/proyecto_funciones.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 08: Relaciones y JOINs](../../08-joins) |

---

<div align="center">

⬅️ [**Volver al Temario**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto →**](../proyectos/proyecto_funciones.md)

</div>
