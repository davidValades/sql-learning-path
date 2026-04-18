# 🏆 Proyecto: El Informe Trimestral

> 📋 **Contexto:** Se acerca el cierre del primer trimestre de 2025. Los directores de las tres organizaciones necesitan informes detallados que crucen información de múltiples tablas. Tu trabajo como analista de datos es construir cada informe usando JOINs complejos, agrupaciones y filtros. ¡Los datos deben contar la historia completa!

---

## 🎯 Misión 1 — E-commerce: Informe Integral de Ventas Q1

El CEO del E-commerce se prepara para la reunión con inversores. Necesita un informe que responda a tres preguntas clave.

### Tarea 1.1 — Top clientes por categoría

Genera un informe que muestre **qué clientes han comprado en cada categoría**, cuántos pedidos hicieron y cuánto gastaron. Cruza las 4 tablas: `pedidos`, `clientes`, `productos` y `categorias`.

Muestra: `cliente`, `categoria`, `num_pedidos`, `gasto_total`. Ordena por cliente y gasto descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT cl.nombre AS cliente,
       ca.nombre_categoria AS categoria,
       COUNT(pe.id_pedido) AS num_pedidos,
       SUM(pe.total) AS gasto_total
FROM pedidos pe
INNER JOIN clientes cl ON pe.id_cliente = cl.id_cliente
INNER JOIN productos pr ON pe.id_producto = pr.id_producto
INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
GROUP BY cl.nombre, ca.nombre_categoria
ORDER BY cl.nombre, gasto_total DESC;
```

Resultado:
| cliente | categoria | num_pedidos | gasto_total |
|---------|-----------|-------------|-------------|
| Ana López | Electrónica | 2 | 1295.00 |
| Ana López | Hogar | 1 | 81.00 |
| María García | Ropa | 2 | 278.95 |
| Pedro Ruiz | Electrónica | 2 | 486.50 |

> 💡 Ana López es la cliente estrella de Electrónica. María García solo compra en Ropa. Pedro Ruiz solo en Electrónica. Nadie ha comprado Hogar excepto Ana (la Lámpara LED).

</details>

### Tarea 1.2 — Productos estrella vs productos dormidos

Genera un informe que muestre **TODOS los productos** con sus ventas. Los que no tienen pedidos deben aparecer como "Sin ventas". Incluye el nombre de la categoría.

Muestra: `producto`, `categoria`, `unidades_vendidas`, `ingreso_generado`, `estado` (`'Vendido'` / `'Sin ventas'`). Ordena por ingreso descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pr.nombre AS producto,
       ca.nombre_categoria AS categoria,
       NVL(SUM(pe.cantidad), 0) AS unidades_vendidas,
       NVL(SUM(pe.total), 0) AS ingreso_generado,
       CASE
           WHEN SUM(pe.id_pedido) IS NULL THEN 'Sin ventas'
           ELSE 'Vendido'
       END AS estado
FROM productos pr
INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
LEFT JOIN pedidos pe ON pr.id_producto = pe.id_producto
GROUP BY pr.nombre, ca.nombre_categoria
ORDER BY ingreso_generado DESC;
```

Resultado:
| producto | categoria | unidades_vendidas | ingreso_generado | estado |
|----------|-----------|-------------------|-----------------|--------|
| Laptop Pro | Electrónica | 1 | 1220.00 | Vendido |
| Monitor 4K | Electrónica | 1 | 350.00 | Vendido |
| Zapatillas Running | Ropa | 2 | 179.00 | Vendido |
| Ratón Inalámbrico | Electrónica | 3 | 136.50 | Vendido |
| Camiseta Básica | Ropa | 5 | 99.95 | Vendido |
| Lámpara LED | Hogar | 2 | 81.00 | Vendido |
| Teclado Mecánico | Electrónica | 1 | 75.00 | Vendido |
| Sofá de Cuero | Hogar | 0 | 0 | Sin ventas |

</details>

### Tarea 1.3 — Matriz cliente × categoría (CROSS JOIN + LEFT JOIN)

El departamento de marketing necesita saber **qué combinaciones cliente-categoría aún NO se han explorado**. Genera todas las combinaciones posibles con `CROSS JOIN` y luego cruza con los pedidos reales para marcar cuáles faltan.

Muestra: `cliente`, `categoria`, `tiene_pedidos` (`'Sí'` / `'No'`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT cl.nombre AS cliente,
       ca.nombre_categoria AS categoria,
       CASE
           WHEN COUNT(pe.id_pedido) > 0 THEN 'Sí'
           ELSE 'No'
       END AS tiene_pedidos
FROM clientes cl
CROSS JOIN categorias ca
LEFT JOIN pedidos pe
    ON cl.id_cliente = pe.id_cliente
LEFT JOIN productos pr
    ON pe.id_producto = pr.id_producto
   AND pr.id_categoria = ca.id_categoria
GROUP BY cl.nombre, ca.nombre_categoria
ORDER BY cl.nombre, ca.nombre_categoria;
```

Resultado:
| cliente | categoria | tiene_pedidos |
|---------|-----------|---------------|
| Ana López | Electrónica | Sí |
| Ana López | Hogar | Sí |
| Ana López | Ropa | No |
| María García | Electrónica | No |
| María García | Hogar | No |
| María García | Ropa | Sí |
| Pedro Ruiz | Electrónica | Sí |
| Pedro Ruiz | Hogar | No |
| Pedro Ruiz | Ropa | No |

> 💡 ¡5 de 9 combinaciones están sin explotar! El marketing puede dirigir campañas personalizadas: Ropa para Ana, Electrónica/Hogar para María, etc.

</details>

---

## 🎯 Misión 2 — Hospital: Panel de Actividad Médica

El director del hospital necesita evaluar la carga de trabajo de cada médico y la actividad por especialidad.

### Tarea 2.1 — Carga de trabajo por médico

Genera un informe que muestre **cada médico** con su especialidad y cuántas citas ha atendido (todas, independientemente del estado). Incluye los estados desglosados.

Muestra: `medico`, `especialidad`, `total_citas`, `completadas`, `pendientes`, `canceladas`. Ordena por total de citas descendente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT m.nombre_completo AS medico,
       e.nombre AS especialidad,
       COUNT(ci.id_cita) AS total_citas,
       COUNT(CASE WHEN ci.estado = 'C' THEN 1 END) AS completadas,
       COUNT(CASE WHEN ci.estado = 'P' THEN 1 END) AS pendientes,
       COUNT(CASE WHEN ci.estado = 'X' THEN 1 END) AS canceladas
FROM medicos m
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
LEFT JOIN citas ci ON m.id_medico = ci.id_medico
GROUP BY m.nombre_completo, e.nombre
ORDER BY total_citas DESC;
```

Resultado:
| medico | especialidad | total_citas | completadas | pendientes | canceladas |
|--------|-------------|-------------|-------------|------------|------------|
| Dra. Sarah Adams | Neurología | 2 | 1 | 0 | 1 |
| Dra. Marta López | Neurología | 2 | 1 | 1 | 0 |
| Dra. Elena Ruiz | Traumatología | 1 | 1 | 0 | 0 |
| Dr. Carlos Vega | Traumatología | 1 | 1 | 0 | 0 |
| Dr. Luis Moreno | Medicina General | 1 | 0 | 1 | 0 |

</details>

### Tarea 2.2 — Historial completo de pacientes

Genera un informe con el **historial de cada paciente**: todas sus citas con el nombre del médico, la especialidad y el estado. Usa 4 tablas.

Muestra: `paciente`, `dni`, `medico`, `especialidad`, `fecha_cita`, `estado`. Ordena por paciente y fecha.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT pa.nombre_completo AS paciente,
       pa.dni,
       m.nombre_completo AS medico,
       e.nombre AS especialidad,
       TO_CHAR(ci.fecha_hora, 'DD/MM/YYYY HH24:MI') AS fecha_cita,
       DECODE(ci.estado, 'C', 'Completada', 'P', 'Pendiente', 'X', 'Cancelada') AS estado
FROM citas ci
INNER JOIN pacientes pa ON ci.id_paciente = pa.id_paciente
INNER JOIN medicos m ON ci.id_medico = m.id_medico
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
ORDER BY pa.nombre_completo, ci.fecha_hora;
```

Resultado:
| paciente | dni | medico | especialidad | fecha_cita | estado |
|----------|-----|--------|-------------|------------|--------|
| Ana Ruiz | 44444444D | Dra. Sarah Adams | Neurología | 01/02/2025 14:00 | Cancelada |
| Carlos Gomez | 22222222B | Dra. Elena Ruiz | Traumatología | 10/01/2025 10:30 | Completada |
| Carlos Gomez | 22222222B | Dra. Marta López | Neurología | 10/02/2025 16:00 | Completada |
| David Torres | 55555555E | Dr. Luis Moreno | Medicina General | 05/02/2025 08:00 | Pendiente |
| Laura Martinez | 11111111A | Dra. Sarah Adams | Neurología | 10/01/2025 09:00 | Completada |
| Laura Martinez | 11111111A | Dr. Carlos Vega | Traumatología | 20/01/2025 09:30 | Completada |
| Pedro Sánchez | 33333333C | Dra. Marta López | Neurología | 15/01/2025 11:00 | Pendiente |

</details>

---

## 🎯 Misión 3 — Aerolínea: Informe Operativo de Flota

El director de operaciones necesita evaluar el uso de la flota y la cobertura de rutas.

### Tarea 3.1 — Uso de flota con detalle de rutas

Genera un informe que muestre **cada avión** con el detalle de sus vuelos y rutas asignadas. Los aviones sin vuelos deben aparecer marcados como `'Disponible'`.

Muestra: `avion`, `capacidad`, `anio_fabricacion`, `ruta`, `distancia_km`, `fecha_salida`, `estado_flota` (`'En servicio'` / `'Disponible'`). Ordena por avión y fecha.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT a.modelo AS avion,
       a.capacidad,
       a.anio_fabricacion,
       r.origen || ' → ' || r.destino AS ruta,
       r.distancia_km,
       v.fecha_hora_salida AS fecha_salida,
       CASE
           WHEN v.id_vuelo IS NULL THEN 'Disponible'
           ELSE 'En servicio'
       END AS estado_flota
FROM aviones a
LEFT JOIN vuelos v ON a.id_avion = v.id_avion
LEFT JOIN rutas r ON v.id_ruta = r.id_ruta
ORDER BY a.modelo, v.fecha_hora_salida;
```

Resultado:
| avion | capacidad | anio_fabricacion | ruta | distancia_km | fecha_salida | estado_flota |
|-------|-----------|-----------------|------|-------------|-------------|-------------|
| Airbus A350 | 300 | 2022 | MAD → LHR | 1350 | 2025-03-01 07:30 | En servicio |
| Airbus A350 | 300 | 2022 | MAD → FRA | 1420 | 2025-03-02 11:15 | En servicio |
| Boeing 737 | 180 | 2018 | NULL | NULL | NULL | Disponible |
| Boeing 777 | 350 | 2016 | JFK → LAX | 4000 | 2025-03-01 14:00 | En servicio |
| Boeing 777 | 350 | 2016 | MAD → LHR | 1350 | 2025-03-03 07:30 | En servicio |
| Embraer E195 | 120 | 2020 | BCN → CDG | 830 | 2025-03-02 06:00 | En servicio |
| Embraer E195 | 120 | 2020 | BCN → FCO | 850 | 2025-03-04 09:00 | En servicio |

</details>

### Tarea 3.2 — Resumen operativo por avión

Genera un **resumen agrupado** por avión con: número de vuelos asignados, distancia total que cubrirá (km), y si está infrautilizado (`num_vuelos < 2`).

Muestra: `avion`, `capacidad`, `num_vuelos`, `distancia_total_km`, `infrautilizado` (`'Sí'` / `'No'`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT a.modelo AS avion,
       a.capacidad,
       COUNT(v.id_vuelo) AS num_vuelos,
       NVL(SUM(r.distancia_km), 0) AS distancia_total_km,
       CASE
           WHEN COUNT(v.id_vuelo) < 2 THEN 'Sí'
           ELSE 'No'
       END AS infrautilizado
FROM aviones a
LEFT JOIN vuelos v ON a.id_avion = v.id_avion
LEFT JOIN rutas r ON v.id_ruta = r.id_ruta
GROUP BY a.modelo, a.capacidad
ORDER BY num_vuelos DESC;
```

Resultado:
| avion | capacidad | num_vuelos | distancia_total_km | infrautilizado |
|-------|-----------|-----------|-------------------|----------------|
| Airbus A350 | 300 | 2 | 2770 | No |
| Boeing 777 | 350 | 2 | 5350 | No |
| Embraer E195 | 120 | 2 | 1680 | No |
| Boeing 737 | 180 | 0 | 0 | Sí |

> 💡 El Boeing 737 está completamente disponible — ¡oportunidad de asignarle rutas o programar mantenimiento!

</details>

### Tarea 3.3 — Frecuencia por ruta

Genera un informe de **frecuencia de vuelos por ruta**, mostrando cuántos vuelos tiene cada ruta y qué aviones la operan.

Muestra: `ruta`, `distancia_km`, `num_vuelos`, `aviones_asignados` (concatenación de modelos con `LISTAGG`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT r.origen || ' → ' || r.destino AS ruta,
       r.distancia_km,
       COUNT(v.id_vuelo) AS num_vuelos,
       LISTAGG(a.modelo, ', ') WITHIN GROUP (ORDER BY a.modelo) AS aviones_asignados
FROM rutas r
LEFT JOIN vuelos v ON r.id_ruta = v.id_ruta
LEFT JOIN aviones a ON v.id_avion = a.id_avion
GROUP BY r.origen, r.destino, r.distancia_km
ORDER BY num_vuelos DESC;
```

Resultado:
| ruta | distancia_km | num_vuelos | aviones_asignados |
|------|-------------|-----------|-------------------|
| MAD → LHR | 1350 | 2 | Airbus A350, Boeing 777 |
| BCN → CDG | 830 | 1 | Embraer E195 |
| BCN → FCO | 850 | 1 | Embraer E195 |
| JFK → LAX | 4000 | 1 | Boeing 777 |
| MAD → FRA | 1420 | 1 | Airbus A350 |

> 💡 MAD → LHR es la ruta más frecuente con 2 vuelos operados por 2 aviones diferentes.

</details>

---

<div align="center">

⬅️ [**Ejercicios JOINs**](../ejercicios/ejercicios_joins.md) · 🏠 [**Volver al Tema 08**](../README.md) · [**Tema 09: Subconsultas →**](../../09-subconsultas)

</div>
