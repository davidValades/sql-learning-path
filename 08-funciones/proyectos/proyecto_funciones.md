# 🏆 Proyecto: El Dashboard del CEO

> 📋 **Contexto:** El CEO de cada organización necesita un panel de control ejecutivo (dashboard) con métricas clave resumidas. Tu trabajo como analista senior es generar cada informe usando funciones de agregación, agrupación y todas las funciones nativas que has aprendido. ¡Los datos deben estar perfectamente formateados y listos para presentar!

---

## 🎯 Misión 1 — E-commerce: Dashboard de Ventas

El CEO del E-commerce necesita tres informes clave para la reunión con inversores.

### Tarea 1.1 — Resumen ejecutivo de inventario

Genera un **resumen por categoría** que muestre:
1. El `id_categoria`.
2. El número total de productos (`num_productos`).
3. El precio medio formateado con 2 decimales (`precio_medio`).
4. El stock total (`stock_total`).
5. El **valor total del inventario** (`SUM(precio * stock)`) formateado con `TO_CHAR` como moneda con separador de miles y símbolo `€`.

Ordena por valor total de inventario descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT id_categoria,
       COUNT(*) AS num_productos,
       ROUND(AVG(precio), 2) AS precio_medio,
       SUM(stock) AS stock_total,
       TO_CHAR(SUM(precio * stock), 'FM999,999,999.00') || ' €' AS valor_inventario
FROM productos
GROUP BY id_categoria
ORDER BY SUM(precio * stock) DESC;
```

Resultado:
| id_categoria | num_productos | precio_medio | stock_total | valor_inventario |
|-------------|---------------|-------------|-------------|------------------|
| 1 | 4 | 422.63 | 355 | 84,850.00 € |
| 3 | 2 | 54.75 | 145 | 6,026.50 € |
| 2 | 2 | 222.75 | 30 | 1,215.00 € |

</details>

### Tarea 1.2 — Productos sin código de barras: impacto económico

El CEO quiere saber cuántos productos **no tienen código de barras** vs los que sí tienen, y el **valor de inventario** de cada grupo. Usa `NVL2` para crear la columna de agrupación.

Muestra: `estado_codigo` (`'Con código'` / `'Sin código'`), `num_productos`, `valor_inventario`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT NVL2(codigo_barras, 'Con código', 'Sin código') AS estado_codigo,
       COUNT(*) AS num_productos,
       TO_CHAR(SUM(precio * stock), 'FM999,999,999.00') || ' €' AS valor_inventario
FROM productos
GROUP BY NVL2(codigo_barras, 'Con código', 'Sin código');
```

Resultado:
| estado_codigo | num_productos | valor_inventario |
|---------------|---------------|------------------|
| Con código | 4 | 71,315.00 € |
| Sin código | 4 | 20,776.50 € |

Detalle Con código: Laptop Pro(1220×50=61000) + Ratón(45.50×200=9100) + Sofá(405×0=0) + Lámpara(40.50×30=1215) = 71,315.
Detalle Sin código: Camiseta(19.99×100=1999) + Zapatillas(89.50×45=4027.50) + Monitor(350×25=8750) + Teclado(75×80=6000) = 20,776.50.

</details>

### Tarea 1.3 — Análisis de rango de precios por categoría

Genera un informe que muestre por cada categoría:
1. El precio **máximo** y **mínimo**.
2. El **rango** (diferencia entre máximo y mínimo).
3. Solo muestra categorías con un rango de precios **mayor a 50**.

Muestra: `id_categoria`, `precio_max`, `precio_min`, `rango_precio`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT id_categoria,
       MAX(precio) AS precio_max,
       MIN(precio) AS precio_min,
       MAX(precio) - MIN(precio) AS rango_precio
FROM productos
GROUP BY id_categoria
HAVING MAX(precio) - MIN(precio) > 50;
```

Resultado:
| id_categoria | precio_max | precio_min | rango_precio |
|-------------|-----------|-----------|--------------|
| 1 (Electrónica) | 1220.00 | 45.50 | 1174.50 |
| 2 (Hogar) | 405.00 | 40.50 | 364.50 |
| 3 (Ropa) | 89.50 | 19.99 | 69.51 |

Todas superan 50, así que aparecen las 3.

</details>

---

## 🎯 Misión 2 — Hospital: Dashboard de Gestión Sanitaria

La gerencia del hospital necesita métricas clave de personal y pacientes.

### Tarea 2.1 — Informe salarial por especialidad

Genera un informe ejecutivo por especialidad que muestre:
1. El `id_especialidad`.
2. El número de médicos.
3. El salario total del departamento (`SUM`).
4. El salario medio redondeado a 2 decimales.
5. El salario formateado del médico mejor pagado: `TO_CHAR(MAX(salario_base), 'FM99,999.00') || ' €'`.

Solo muestra especialidades donde el salario medio **sea mayor a 2700**.

Ordena por salario medio descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT id_especialidad,
       COUNT(*) AS num_medicos,
       SUM(salario_base) AS salario_total,
       ROUND(AVG(salario_base), 2) AS salario_medio,
       TO_CHAR(MAX(salario_base), 'FM99,999.00') || ' €' AS mejor_salario
FROM medicos
GROUP BY id_especialidad
HAVING AVG(salario_base) > 2700
ORDER BY salario_medio DESC;
```

Resultado:
| id_especialidad | num_medicos | salario_total | salario_medio | mejor_salario |
|-----------------|-------------|---------------|---------------|---------------|
| 200 | 2 | 8500.00 | 4250.00 | 4,500.00 € |
| 100 | 2 | 6000.00 | 3000.00 | 3,200.00 € |

Especialidad 300 tiene media=2600 → no supera 2700.

</details>

### Tarea 2.2 — Demografía de pacientes

Genera un análisis demográfico que muestre:
1. La **década de nacimiento** (usa `TRUNC(EXTRACT(YEAR FROM fecha_nacimiento), -1)`).
2. El número de pacientes nacidos en esa década.
3. El paciente más joven de la década (fecha más reciente: `MAX(fecha_nacimiento)`) formateado como `DD/MM/YYYY`.
4. Una columna `tiene_todos_telefonos` que diga `'SÍ'` si `COUNT(telefono) = COUNT(*)`, o `'NO'` en caso contrario.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT TRUNC(EXTRACT(YEAR FROM fecha_nacimiento), -1) AS decada,
       COUNT(*) AS num_pacientes,
       TO_CHAR(MAX(fecha_nacimiento), 'DD/MM/YYYY') AS mas_joven,
       CASE WHEN COUNT(telefono) = COUNT(*) THEN 'SÍ' ELSE 'NO' END AS tiene_todos_telefonos
FROM pacientes
GROUP BY TRUNC(EXTRACT(YEAR FROM fecha_nacimiento), -1)
ORDER BY decada;
```

Resultado:
| decada | num_pacientes | mas_joven | tiene_todos_telefonos |
|--------|---------------|-----------|----------------------|
| 1970 | 1 | 20/11/1978 | NO |
| 1980 | 1 | 05/05/1985 | SÍ |
| 1990 | 2 | 08/07/1995 | SÍ |
| 2000 | 1 | 15/03/2000 | SÍ |

Nota: La década 1970 tiene a Ana Ruiz que no tiene teléfono → 'NO'.

</details>

### Tarea 2.3 — Directorio formateado de médicos

Genera un directorio elegante de médicos con:
1. El nombre en formato `UPPER`.
2. El salario formateado con separador de miles y `€`.
3. Una columna `nivel` basada en el salario: `'SENIOR'` si salario >= 4000, `'ESTÁNDAR'` si no (usa `CASE` o puedes simularlo con funciones).

Ordena por salario descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT UPPER(nombre_completo) AS nombre_mayus,
       TO_CHAR(salario_base, 'FM99,999.00') || ' €' AS salario_fmt,
       CASE WHEN salario_base >= 4000 THEN 'SENIOR' ELSE 'ESTÁNDAR' END AS nivel
FROM medicos
ORDER BY salario_base DESC;
```

Resultado:
| nombre_mayus | salario_fmt | nivel |
|-------------|-------------|-------|
| DRA. MARTA LÓPEZ | 4,500.00 € | SENIOR |
| DRA. SARAH ADAMS | 4,000.00 € | SENIOR |
| DRA. ELENA RUIZ | 3,200.00 € | ESTÁNDAR |
| DR. CARLOS VEGA | 2,800.00 € | ESTÁNDAR |
| DR. LUIS MORENO | 2,600.00 € | ESTÁNDAR |

</details>

---

## 🎯 Misión 3 — Aerolínea: Dashboard de Operaciones

El director de operaciones necesita métricas de flota y rutas para la planificación estratégica.

### Tarea 3.1 — Análisis de flota por antigüedad

Clasifica los aviones en grupos de antigüedad y genera estadísticas:
1. Grupo de antigüedad: `'0-5 años'` si `(EXTRACT(YEAR FROM SYSDATE) - anio_fabricacion) <= 5`, o `'Más de 5 años'`.
2. Número de aviones en cada grupo.
3. La capacidad media redondeada a entero.
4. La capacidad total.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT CASE
         WHEN (EXTRACT(YEAR FROM SYSDATE) - anio_fabricacion) <= 5 THEN '0-5 años'
         ELSE 'Más de 5 años'
       END AS grupo_antiguedad,
       COUNT(*) AS num_aviones,
       ROUND(AVG(capacidad_pasajeros), 0) AS capacidad_media,
       SUM(capacidad_pasajeros) AS capacidad_total
FROM aviones
GROUP BY CASE
           WHEN (EXTRACT(YEAR FROM SYSDATE) - anio_fabricacion) <= 5 THEN '0-5 años'
           ELSE 'Más de 5 años'
         END;
```

Resultado (en 2025):
| grupo_antiguedad | num_aviones | capacidad_media | capacidad_total |
|-----------------|-------------|-----------------|-----------------|
| 0-5 años | 2 | 210 | 420 |
| Más de 5 años | 2 | 265 | 530 |

(0-5: Airbus A350 2022 + Embraer E195 2020; Más de 5: Boeing 737 2018 + Boeing 777 2016)

</details>

### Tarea 3.2 — Rutas por aeropuerto de origen: informe completo

Genera un informe por cada `aeropuerto_origen` con:
1. El número de rutas que salen de ese aeropuerto.
2. La distancia media redondeada a entero.
3. La distancia mínima y máxima.
4. La distancia formateada del trayecto más largo: `TO_CHAR(MAX(distancia_km), 'FM99,999') || ' km'`.

Solo muestra aeropuertos con **más de 1 ruta**.

Ordena por distancia media descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT aeropuerto_origen,
       COUNT(*) AS num_rutas,
       ROUND(AVG(distancia_km), 0) AS distancia_media,
       MIN(distancia_km) AS dist_min,
       MAX(distancia_km) AS dist_max,
       TO_CHAR(MAX(distancia_km), 'FM99,999') || ' km' AS mayor_trayecto
FROM rutas
GROUP BY aeropuerto_origen
HAVING COUNT(*) > 1
ORDER BY AVG(distancia_km) DESC;
```

Resultado:
| aeropuerto_origen | num_rutas | distancia_media | dist_min | dist_max | mayor_trayecto |
|------------------|-----------|-----------------|----------|----------|----------------|
| MAD | 2 | 1385 | 1350 | 1420 | 1,420 km |
| BCN | 2 | 840 | 830 | 850 | 850 km |

</details>

### Tarea 3.3 — Ficha técnica de cada avión

Genera una ficha técnica formateada para cada avión con:
1. Modelo en **mayúsculas**.
2. Un código técnico: primeras 3 letras del modelo en mayúsculas + `'-'` + id_avion rellenado a 4 dígitos con ceros (`LPAD`).
3. Capacidad formateada: `RPAD(TO_CHAR(capacidad_pasajeros), 5, '.') || ' PAX'`.
4. Antigüedad en años.

Ordena por antigüedad descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT UPPER(modelo) AS modelo_mayus,
       UPPER(SUBSTR(modelo, 1, 3)) || '-' || LPAD(id_avion, 4, '0') AS codigo_tecnico,
       RPAD(TO_CHAR(capacidad_pasajeros), 5, '.') || ' PAX' AS capacidad_fmt,
       EXTRACT(YEAR FROM SYSDATE) - anio_fabricacion AS antiguedad
FROM aviones
ORDER BY antiguedad DESC;
```

Resultado (en 2025):
| modelo_mayus | codigo_tecnico | capacidad_fmt | antiguedad |
|-------------|----------------|---------------|------------|
| BOEING 777 | BOE-0050 | 350.. PAX | 9 |
| BOEING 737 | BOE-0020 | 180.. PAX | 7 |
| EMBRAER E195 | EMB-0040 | 120.. PAX | 5 |
| AIRBUS A350 | AIR-0030 | 300.. PAX | 3 |

</details>

---

## ✅ Checklist de Completado

- [ ] Misión 1: Las 3 tareas del E-commerce completadas (GROUP BY, NVL2, HAVING, TO_CHAR)
- [ ] Misión 2: Las 3 tareas del Hospital completadas (agregación, EXTRACT, TRUNC, CASE, TO_CHAR)
- [ ] Misión 3: Las 3 tareas de la Aerolínea completadas (CASE en GROUP BY, SUBSTR, LPAD, RPAD)
- [ ] Todas las consultas usan al menos una función nativa de Oracle
- [ ] Los resultados están formateados para presentación ejecutiva

---

<div align="center">

### 🗺️ Navegación

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Repasar la teoría | 📖 [Volver al Temario](../README.md) |
| 2️⃣ | Completar los ejercicios | 🏋️ [Ir a Ejercicios](../ejercicios/ejercicios_funciones.md) |
| 3️⃣ | Completar el proyecto | 🏆 _Estás aquí_ |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 09: Relaciones y JOINs](../../09-joins) |

---

<div align="center">

⬅️ [**📝 Ejercicios del Tema 08**](../ejercicios/ejercicios_funciones.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 09: Relaciones y JOINs →**](../../09-joins/README.md)

</div>
