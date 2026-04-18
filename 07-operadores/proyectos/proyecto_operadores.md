# 🏆 Proyecto: La Auditoría del Trimestre

> 📋 **Contexto:** Es final de trimestre y cada departamento necesita informes filtrados y ordenados para presentar ante la dirección. Tu trabajo como analista de datos es generar los informes exactos que te piden, combinando todos los operadores que has aprendido.

---

## 🎯 Misión 1 — E-commerce: Informe de Inventario Crítico

El director de logística necesita **tres informes** para la reunión trimestral:

### Tarea 1.1 — Productos en riesgo

Genera un listado de productos que cumplan **todas** estas condiciones:
- Stock **menor o igual a 50** unidades.
- Precio **mayor a 30** euros.
- Que **no** sean de la categoría Ropa (id_categoria = 3).

Ordena el resultado por stock ascendente (los más críticos primero).

Muestra: `nombre`, `precio`, `stock`, `id_categoria`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio, stock, id_categoria
FROM productos
WHERE stock <= 50
  AND precio > 30
  AND id_categoria != 3
ORDER BY stock ASC;
```

Resultado:
1. Sofá de Cuero (405.00, stock 0, cat 2)
2. Monitor 4K (350.00, stock 25, cat 1)
3. Lámpara LED (40.50, stock 30, cat 2)
4. Laptop Pro (1220.00, stock 50, cat 1)

</details>

### Tarea 1.2 — Catálogo sin código de barras

Identifica los productos que **no tienen código de barras** asignado y cuyo nombre **no contenga la palabra "Básica"**. Ordena alfabéticamente por nombre.

Muestra: `nombre`, `precio`, `codigo_barras`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio, codigo_barras
FROM productos
WHERE codigo_barras IS NULL
  AND nombre NOT LIKE '%Básica%'
ORDER BY nombre ASC;
```

Resultado:
1. Monitor 4K (350.00, NULL)
2. Teclado Mecánico (75.00, NULL)
3. Zapatillas Running (89.50, NULL)

</details>

### Tarea 1.3 — Rango de precios para folleto promocional

Marketing quiere un folleto con productos en un rango de precio **entre 20 y 100 euros**, de las categorías **Electrónica (1) o Ropa (3)**. Ordena por precio descendente.

Muestra: `nombre`, `precio`, `id_categoria`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, precio, id_categoria
FROM productos
WHERE precio BETWEEN 20 AND 100
  AND id_categoria IN (1, 3)
ORDER BY precio DESC;
```

Resultado:
1. Zapatillas Running (89.50, cat 3)
2. Teclado Mecánico (75.00, cat 1)
3. Ratón Inalámbrico (45.50, cat 1)

</details>

---

## 🎯 Misión 2 — Hospital: Informe de Personal y Pacientes

La gerencia del hospital necesita dos informes para la auditoría sanitaria:

### Tarea 2.1 — Médicos para plan de incentivos

RRHH necesita identificar a los médicos candidatos al bono trimestral. Los candidatos son aquellos que:
- Sean de **Traumatología (100) o Medicina General (300)**.
- Tengan un salario **menor a 3000**.

También necesitan, en la misma consulta, a cualquier médico de **Neurología (200)** con salario **mayor o igual a 4500** (candidatos a promoción).

Ordena por `salario_base` ascendente.

Muestra: `nombre_completo`, `id_especialidad`, `salario_base`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre_completo, id_especialidad, salario_base
FROM medicos
WHERE (id_especialidad IN (100, 300) AND salario_base < 3000)
   OR (id_especialidad = 200 AND salario_base >= 4500)
ORDER BY salario_base ASC;
```

Resultado:
1. Dr. Luis Moreno (300, 2600)
2. Dr. Carlos Vega (100, 2800)
3. Dra. Marta López (200, 4500)

</details>

### Tarea 2.2 — Pacientes con datos incompletos

El departamento de calidad necesita un informe de pacientes que cumplan **al menos una** de estas condiciones:
- **No tienen teléfono** registrado.
- Su DNI **termina en 'A' o 'B'** (posible error de formato antiguo).

Ordena por `nombre` ascendente.

Muestra: `nombre`, `dni`, `telefono`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT nombre, dni, telefono
FROM pacientes
WHERE telefono IS NULL
   OR dni LIKE '%A'
   OR dni LIKE '%B';
```

Resultado:
1. Ana Ruiz (44444444D, NULL) — NO aparece porque su DNI termina en D, pero SÍ por NULL.
   Corrección: Ana Ruiz aparece por telefono IS NULL.
2. Carlos Gomez (22222222B) — aparece por DNI terminado en B.
3. Laura Martinez (11111111A) — aparece por DNI terminado en A.

Ordenado:
1. Ana Ruiz (44444444D, NULL)
2. Carlos Gomez (22222222B, 555-0002)
3. Laura Martinez (11111111A, 555-9999)

</details>

---

## 🎯 Misión 3 — Aerolínea: Planificación de Rutas

El director de operaciones necesita informes para la planificación del próximo trimestre:

### Tarea 3.1 — Rutas europeas cortas y largas

Genera dos listados en una sola consulta:
- Rutas con distancia **menor a 900 km** (cortas) O rutas con distancia **mayor a 1400 km** (largas).
- Excluye las rutas que pasen por **JFK** (ni como origen ni como destino).

Ordena por `distancia_km` ascendente.

Muestra: `id_ruta`, `aeropuerto_origen`, `aeropuerto_destino`, `distancia_km`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT id_ruta, aeropuerto_origen, aeropuerto_destino, distancia_km
FROM rutas
WHERE (distancia_km < 900 OR distancia_km > 1400)
  AND aeropuerto_origen != 'JFK'
  AND aeropuerto_destino != 'JFK'
ORDER BY distancia_km ASC;
```

Resultado:
1. BCNCDG (BCN→CDG, 830)
2. BCNFCO (BCN→FCO, 850)
3. MADFRA (MAD→FRA, 1420)

</details>

### Tarea 3.2 — Flota para renovación

Identifica los aviones que podrían necesitar renovación: aquellos fabricados **antes de 2019** con capacidad **entre 150 y 400 pasajeros**. Ordena por `anio_fabricacion` ascendente.

Muestra: `modelo`, `capacidad_pasajeros`, `anio_fabricacion`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT modelo, capacidad_pasajeros, anio_fabricacion
FROM aviones
WHERE anio_fabricacion < 2019
  AND capacidad_pasajeros BETWEEN 150 AND 400
ORDER BY anio_fabricacion ASC;
```

Resultado:
1. Boeing 777 (350, 2016)
2. Boeing 737 (180, 2018)

</details>

---

## ✅ Checklist de Completado

- [ ] Misión 1: Las 3 tareas del E-commerce completadas
- [ ] Misión 2: Las 2 tareas del Hospital completadas
- [ ] Misión 3: Las 2 tareas de la Aerolínea completadas
- [ ] Todas las consultas usan al menos 2 operadores distintos
- [ ] `ORDER BY` utilizado en todas las consultas

---

<div align="center">

### 🗺️ Navegación

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Repasar la teoría | 📖 [Volver al Temario](../README.md) |
| 2️⃣ | Completar los ejercicios | 🏋️ [Ir a Ejercicios](../ejercicios/ejercicios_operadores.md) |
| 3️⃣ | Completar el proyecto | 🏆 _Estás aquí_ |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 08: Funciones Nativas de Oracle](../../08-funciones) |

---

<div align="center">

⬅️ [**Volver a Ejercicios**](../ejercicios/ejercicios_operadores.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 08 →**](../../08-funciones)

</div>
