# 🏋️ Ejercicios — Tema 07: Operadores Lógicos y Filtros

> 💡 **Instrucciones:** Resuelve cada ejercicio escribiendo la consulta SQL completa. Luego compara tu respuesta con la solución oficial. Todos los ejercicios usan el estado acumulado de la base de datos (incluyendo la expansión de datos del Tema 07).

---

## Ejercicio 1 — E-commerce · Comparación simple

**Enunciado:** El departamento de precios quiere ver todos los productos que cuesten **estrictamente menos de 50 euros**. Muestra el `nombre` y el `precio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
WHERE precio < 50;
```

Resultado: Ratón Inalámbrico (45.50), Lámpara LED (40.50), Camiseta Básica (19.99).

</details>

---

## Ejercicio 2 — Hospital · BETWEEN con fechas

**Enunciado:** Recursos Humanos necesita un listado de pacientes nacidos **entre el 1 de enero de 1990 y el 31 de diciembre de 2000** (inclusive). Muestra `nombre` y `fecha_nacimiento`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, fecha_nacimiento
FROM pacientes
WHERE fecha_nacimiento BETWEEN TO_DATE('1990-01-01','YYYY-MM-DD')
                            AND TO_DATE('2000-12-31','YYYY-MM-DD');
```

Resultado: Laura Martinez (1990-01-01), David Torres (1995-07-08), Pedro Sánchez (2000-03-15).

</details>

---

## Ejercicio 3 — Aerolínea · IN con texto

**Enunciado:** El equipo de operaciones necesita las rutas cuyo `aeropuerto_destino` sea **Londres (LHR), París (CDG) o Roma (FCO)**. Muestra todas las columnas.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT *
FROM rutas
WHERE aeropuerto_destino IN ('LHR', 'CDG', 'FCO');
```

Resultado: MADLHR (MAD→LHR, 1350), BCNCDG (BCN→CDG, 830), BCNFCO (BCN→FCO, 850).

</details>

---

## Ejercicio 4 — E-commerce · LIKE con comodines

**Enunciado:** Marketing quiere encontrar todos los productos cuyo nombre **contenga la palabra "Cuero" o "Mecánico"**. Usa dos condiciones `LIKE` combinadas con `OR`. Muestra `nombre` y `precio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
WHERE nombre LIKE '%Cuero%' OR nombre LIKE '%Mecánico%';
```

Resultado: Sofá de Cuero (405.00), Teclado Mecánico (75.00).

</details>

---

## Ejercicio 5 — Hospital · IS NULL

**Enunciado:** El departamento de comunicaciones necesita llamar a todos los pacientes, pero primero quiere identificar a los que **NO tienen teléfono registrado** (valor NULL). Muestra `nombre`, `dni` y `telefono`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, dni, telefono
FROM pacientes
WHERE telefono IS NULL;
```

Resultado: Ana Ruiz (44444444D, NULL).

</details>

---

## Ejercicio 6 — E-commerce · AND + comparación

**Enunciado:** El jefe de almacén necesita productos de la categoría **Electrónica (id_categoria = 1)** que además tengan un stock **mayor o igual a 50**. Muestra `nombre`, `stock` y `precio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, stock, precio
FROM productos
WHERE id_categoria = 1 AND stock >= 50;
```

Resultado: Laptop Pro (stock 50, 1220.00), Ratón Inalámbrico (stock 200, 45.50), Teclado Mecánico (stock 80, 75.00).

</details>

---

## Ejercicio 7 — Aerolínea · NOT BETWEEN

**Enunciado:** El director de operaciones quiere ver los aviones cuyo año de fabricación **NO esté entre 2018 y 2020** (inclusive). Muestra `modelo` y `anio_fabricacion`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT modelo, anio_fabricacion
FROM aviones
WHERE anio_fabricacion NOT BETWEEN 2018 AND 2020;
```

Resultado: Airbus A350 (2022), Boeing 777 (2016).

</details>

---

## Ejercicio 8 — Hospital · OR + AND con paréntesis

**Enunciado:** Se necesita un listado de médicos que cumplan **una** de estas condiciones:
- Sean de **Traumatología** (100) con salario **mayor a 3000**.
- Sean de **Neurología** (200) con salario **mayor a 4000**.

Muestra `nombre_completo`, `id_especialidad` y `salario_base`. ¡Cuidado con la precedencia!

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre_completo, id_especialidad, salario_base
FROM medicos
WHERE (id_especialidad = 100 AND salario_base > 3000)
   OR (id_especialidad = 200 AND salario_base > 4000);
```

Resultado: Dra. Elena Ruiz (100, 3200), Dra. Marta López (200, 4500).

</details>

---

## Ejercicio 9 — E-commerce · ORDER BY múltiple

**Enunciado:** Genera un catálogo de productos ordenado: primero por `id_categoria` en orden **ascendente**; dentro de cada categoría, por `precio` en orden **descendente**. Muestra `nombre`, `id_categoria` y `precio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, id_categoria, precio
FROM productos
ORDER BY id_categoria ASC, precio DESC;
```

Resultado:
1. Laptop Pro (1, 1220.00)
2. Monitor 4K (1, 350.00)
3. Teclado Mecánico (1, 75.00)
4. Ratón Inalámbrico (1, 45.50)
5. Sofá de Cuero (2, 405.00)
6. Lámpara LED (2, 40.50)
7. Zapatillas Running (3, 89.50)
8. Camiseta Básica (3, 19.99)

</details>

---

## Ejercicio 10 — Combinación total · Las tres bases de datos

**Enunciado:** Resuelve estas **tres consultas independientes**, cada una combinando al menos 2 operadores diferentes:

**a)** E-commerce: Productos con precio **entre 40 y 400**, que **no** sean de la categoría Hogar (2), ordenados por precio descendente.

**b)** Hospital: Pacientes cuyo nombre **empiece por la letra 'A' o 'D'**, que **tengan teléfono** registrado, ordenados alfabéticamente por nombre.

**c)** Aerolínea: Rutas con distancia **mayor a 800 km** que salgan de **'MAD' o 'BCN'**, ordenadas por distancia ascendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**a) E-commerce:**
```sql
SELECT nombre, precio, id_categoria
FROM productos
WHERE precio BETWEEN 40 AND 400
  AND id_categoria != 2
ORDER BY precio DESC;
```
Resultado: Monitor 4K (350), Zapatillas Running (89.50), Teclado Mecánico (75), Ratón Inalámbrico (45.50).

**b) Hospital:**
```sql
SELECT nombre, telefono
FROM pacientes
WHERE (nombre LIKE 'A%' OR nombre LIKE 'D%')
  AND telefono IS NOT NULL
ORDER BY nombre ASC;
```
Resultado: David Torres (555-0005). (Ana Ruiz empieza por A pero no tiene teléfono).

**c) Aerolínea:**
```sql
SELECT id_ruta, aeropuerto_origen, distancia_km
FROM rutas
WHERE distancia_km > 800
  AND aeropuerto_origen IN ('MAD', 'BCN')
ORDER BY distancia_km ASC;
```
Resultado: BCNCDG (BCN, 830), BCNFCO (BCN, 850), MADLHR (MAD, 1350), MADFRA (MAD, 1420).

</details>

---

<div align="center">

### 🗺️ Navegación

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Repasar la teoría | 📖 [Volver al Temario](../README.md) |
| 2️⃣ | Completar los ejercicios | 🏋️ _Estás aquí_ |
| 3️⃣ | Completar el proyecto | 🏆 [Ir a Proyecto →](../proyectos/proyecto_operadores.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 08: Funciones Nativas de Oracle](../../08-funciones) |

---

<div align="center">

⬅️ [**Tema 07: Operadores Lógicos y Filtros**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 07 →**](../proyectos/proyecto_operadores.md)

</div>
