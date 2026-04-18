# 🏆 Proyecto: La Investigación Interna

> 📋 **Contexto:** Se han detectado anomalías en las tres organizaciones y la dirección ha solicitado una investigación interna. Tu trabajo como analista de datos es responder preguntas complejas que requieren subconsultas anidadas, correlacionadas y combinaciones avanzadas. ¡Los datos deben hablar por sí mismos!

---

## 🎯 Misión 1 — E-commerce: Análisis de Patrones de Compra

El departamento de inteligencia de negocio quiere entender los patrones de compra y detectar oportunidades ocultas.

### Tarea 1.1 — Clientes que solo compran en una categoría

Identifica los **clientes que han comprado en exactamente una categoría de productos**. Usa una inline view para contar las categorías distintas por cliente y luego filtra los que solo tienen una.

Muestra: `cliente`, `categoria_unica`, `gasto_total`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT resumen.cliente,
       resumen.categoria_unica,
       resumen.gasto_total
FROM (
    SELECT cl.nombre AS cliente,
           MIN(ca.nombre_categoria) AS categoria_unica,
           COUNT(DISTINCT ca.id_categoria) AS num_categorias,
           SUM(pe.total) AS gasto_total
    FROM pedidos pe
    INNER JOIN clientes cl ON pe.id_cliente = cl.id_cliente
    INNER JOIN productos pr ON pe.id_producto = pr.id_producto
    INNER JOIN categorias ca ON pr.id_categoria = ca.id_categoria
    GROUP BY cl.nombre
) resumen
WHERE resumen.num_categorias = 1
ORDER BY resumen.gasto_total DESC;
```

Resultado:
| cliente | categoria_unica | gasto_total |
|---------|----------------|-------------|
| Pedro Ruiz | Electrónica | 486.50 |
| María García | Ropa | 278.95 |

> 💡 Ana López compra en Electrónica y Hogar (2 categorías), así que queda excluida. Pedro solo en Electrónica y María solo en Ropa → oportunidad de cross-selling.

</details>

### Tarea 1.2 — Productos que generan más ingresos que el promedio

Encuentra los **productos cuyo ingreso total supera el ingreso promedio por producto**. Usa una subconsulta escalar para calcular el promedio y una inline view para el ingreso por producto.

Muestra: `producto`, `ingreso_total`, `ingreso_promedio_global`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT ventas.producto,
       ventas.ingreso_total,
       (SELECT ROUND(AVG(total_prod), 2)
        FROM (
            SELECT SUM(pe.total) AS total_prod
            FROM pedidos pe
            GROUP BY pe.id_producto
        )) AS ingreso_promedio_global
FROM (
    SELECT pr.nombre AS producto,
           SUM(pe.total) AS ingreso_total
    FROM pedidos pe
    INNER JOIN productos pr ON pe.id_producto = pr.id_producto
    GROUP BY pr.nombre
) ventas
WHERE ventas.ingreso_total > (
    SELECT AVG(total_prod)
    FROM (
        SELECT SUM(pe.total) AS total_prod
        FROM pedidos pe
        GROUP BY pe.id_producto
    )
)
ORDER BY ventas.ingreso_total DESC;
```

Ingreso por producto: Laptop Pro(1220), Monitor 4K(350), Zapatillas(179), Ratón(136.50), Camiseta(99.95), Lámpara(81), Teclado(75).
Promedio = (1220+350+179+136.50+99.95+81+75) / 7 = **305.92**

Resultado:
| producto | ingreso_total | ingreso_promedio_global |
|----------|---------------|------------------------|
| Laptop Pro | 1220.00 | 305.92 |
| Monitor 4K | 350.00 | 305.92 |

</details>

### Tarea 1.3 — Categorías sin explorar por cliente (con NOT EXISTS)

Para cada combinación cliente-categoría, identifica las **categorías en las que el cliente NO ha comprado nada**. Usa `NOT EXISTS`.

Muestra: `cliente`, `categoria_sin_explorar`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT cl.nombre AS cliente,
       ca.nombre_categoria AS categoria_sin_explorar
FROM clientes cl
CROSS JOIN categorias ca
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos pe
    INNER JOIN productos pr ON pe.id_producto = pr.id_producto
    WHERE pe.id_cliente = cl.id_cliente
      AND pr.id_categoria = ca.id_categoria
)
ORDER BY cl.nombre, ca.nombre_categoria;
```

Resultado:
| cliente | categoria_sin_explorar |
|---------|----------------------|
| Ana López | Ropa |
| María García | Electrónica |
| María García | Hogar |
| Pedro Ruiz | Hogar |
| Pedro Ruiz | Ropa |

> 💡 5 oportunidades de cross-selling detectadas. María García es la que más categorías tiene sin explorar.

</details>

---

## 🎯 Misión 2 — Hospital: Auditoría de Actividad Médica

La dirección del hospital necesita una auditoría profunda de la actividad médica.

### Tarea 2.1 — Médicos sin citas pendientes ni canceladas

Identifica los **médicos que solo tienen citas completadas** (todas sus citas tienen estado `'C'`). No deben tener ni pendientes ni canceladas. Usa `NOT EXISTS`.

Muestra: `medico`, `especialidad`, `citas_completadas`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT m.nombre_completo AS medico,
       e.nombre AS especialidad,
       (SELECT COUNT(*) FROM citas ci WHERE ci.id_medico = m.id_medico AND ci.estado = 'C') AS citas_completadas
FROM medicos m
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
WHERE EXISTS (
    SELECT 1 FROM citas ci WHERE ci.id_medico = m.id_medico
)
AND NOT EXISTS (
    SELECT 1 FROM citas ci
    WHERE ci.id_medico = m.id_medico
      AND ci.estado IN ('P', 'X')
)
ORDER BY citas_completadas DESC;
```

Resultado:
| medico | especialidad | citas_completadas |
|--------|-------------|-------------------|
| Dra. Elena Ruiz | Traumatología | 1 |
| Dr. Carlos Vega | Traumatología | 1 |

> 💡 Dra. Sarah Adams tiene una cancelada (cita 5). Dra. Marta López tiene una pendiente (cita 3). Dr. Luis Moreno tiene una pendiente (cita 6). Solo los dos traumatólogos tienen un historial impecable.

</details>

### Tarea 2.2 — Pacientes atendidos por más de una especialidad

Encuentra los **pacientes cuyas citas abarcan más de una especialidad médica**. Usa una inline view para contar especialidades distintas por paciente.

Muestra: `paciente`, `num_especialidades`, `especialidades` (concatenadas con LISTAGG).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT resumen.paciente,
       resumen.num_especialidades,
       resumen.especialidades
FROM (
    SELECT pa.nombre_completo AS paciente,
           COUNT(DISTINCT m.id_especialidad) AS num_especialidades,
           LISTAGG(DISTINCT e.nombre, ', ') WITHIN GROUP (ORDER BY e.nombre) AS especialidades
    FROM citas ci
    INNER JOIN pacientes pa ON ci.id_paciente = pa.id_paciente
    INNER JOIN medicos m ON ci.id_medico = m.id_medico
    INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
    GROUP BY pa.nombre_completo
) resumen
WHERE resumen.num_especialidades > 1
ORDER BY resumen.num_especialidades DESC;
```

Resultado:
| paciente | num_especialidades | especialidades |
|----------|--------------------|---------------|
| Carlos Gomez | 2 | Neurología, Traumatología |
| Laura Martinez | 2 | Neurología, Traumatología |

> 💡 Laura tuvo citas con Neurología (Dra. Sarah Adams) y Traumatología (Dr. Carlos Vega). Carlos con Traumatología (Dra. Elena Ruiz) y Neurología (Dra. Marta López).

</details>

### Tarea 2.3 — Especialidad más demandada vs menos demandada

Usa subconsultas escalares con `ALL` para encontrar la **especialidad con más citas** y la **especialidad con menos citas** en una sola consulta.

Muestra: `especialidad`, `total_citas`, `clasificacion` (`'Más demandada'` / `'Menos demandada'`).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT e.nombre AS especialidad,
       COUNT(ci.id_cita) AS total_citas,
       CASE
           WHEN COUNT(ci.id_cita) >= ALL (
               SELECT COUNT(ci2.id_cita)
               FROM citas ci2
               INNER JOIN medicos m2 ON ci2.id_medico = m2.id_medico
               GROUP BY m2.id_especialidad
           ) THEN 'Más demandada'
           WHEN COUNT(ci.id_cita) <= ALL (
               SELECT COUNT(ci2.id_cita)
               FROM citas ci2
               INNER JOIN medicos m2 ON ci2.id_medico = m2.id_medico
               GROUP BY m2.id_especialidad
           ) THEN 'Menos demandada'
       END AS clasificacion
FROM citas ci
INNER JOIN medicos m ON ci.id_medico = m.id_medico
INNER JOIN especialidades e ON m.id_especialidad = e.id_especialidad
GROUP BY e.nombre
HAVING COUNT(ci.id_cita) >= ALL (
    SELECT COUNT(ci2.id_cita)
    FROM citas ci2
    INNER JOIN medicos m2 ON ci2.id_medico = m2.id_medico
    GROUP BY m2.id_especialidad
)
OR COUNT(ci.id_cita) <= ALL (
    SELECT COUNT(ci2.id_cita)
    FROM citas ci2
    INNER JOIN medicos m2 ON ci2.id_medico = m2.id_medico
    GROUP BY m2.id_especialidad
)
ORDER BY total_citas DESC;
```

Citas por especialidad: Neurología (citas 1,3,5,7 = 4), Traumatología (citas 2,4 = 2), Med. General (cita 6 = 1).

Resultado:
| especialidad | total_citas | clasificacion |
|-------------|-------------|---------------|
| Neurología | 4 | Más demandada |
| Medicina General | 1 | Menos demandada |

</details>

---

## 🎯 Misión 3 — Aerolínea: Inteligencia Operativa

El director de operaciones necesita análisis profundos de la utilización de la flota.

### Tarea 3.1 — Aviones que solo vuelan rutas cortas

Encuentra los **aviones cuyos vuelos TODOS tienen rutas con distancia inferior a 1000 km**. Usa `NOT EXISTS` para verificar que no hay ningún vuelo con ruta >= 1000 km.

Muestra: `modelo`, `capacidad`, `num_vuelos`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT a.modelo,
       a.capacidad,
       (SELECT COUNT(*) FROM vuelos v WHERE v.id_avion = a.id_avion) AS num_vuelos
FROM aviones a
WHERE EXISTS (
    SELECT 1 FROM vuelos v WHERE v.id_avion = a.id_avion
)
AND NOT EXISTS (
    SELECT 1
    FROM vuelos v
    INNER JOIN rutas r ON v.id_ruta = r.id_ruta
    WHERE v.id_avion = a.id_avion
      AND r.distancia_km >= 1000
);
```

Rutas del Embraer E195 (id=40): BCNCDG (830km) y BCNFCO (850km) → ambas < 1000.
Rutas del Airbus A350 (id=30): MADLHR (1350km) y MADFRA (1420km) → ambas >= 1000.
Rutas del Boeing 777 (id=50): JFKLAX (4000km) y MADLHR (1350km) → ambas >= 1000.

Resultado:
| modelo | capacidad | num_vuelos |
|--------|-----------|-----------|
| Embraer E195 | 120 | 2 |

> 💡 El Embraer E195 solo opera rutas cortas desde Barcelona. Podría optimizarse asignándole rutas más largas si su alcance lo permite.

</details>

### Tarea 3.2 — Ruta con mayor capacidad total de pasajeros

Calcula la **capacidad total de pasajeros por ruta** (sumando la capacidad de cada avión que opera en esa ruta). Encuentra la ruta con la mayor capacidad. Usa una inline view.

Muestra: `ruta`, `num_vuelos`, `capacidad_total_pasajeros`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT ruta_cap.ruta,
       ruta_cap.num_vuelos,
       ruta_cap.capacidad_total_pasajeros
FROM (
    SELECT r.origen || ' → ' || r.destino AS ruta,
           COUNT(v.id_vuelo) AS num_vuelos,
           SUM(a.capacidad) AS capacidad_total_pasajeros
    FROM vuelos v
    INNER JOIN rutas r ON v.id_ruta = r.id_ruta
    INNER JOIN aviones a ON v.id_avion = a.id_avion
    GROUP BY r.origen, r.destino
) ruta_cap
WHERE ruta_cap.capacidad_total_pasajeros = (
    SELECT MAX(cap_total)
    FROM (
        SELECT SUM(a.capacidad) AS cap_total
        FROM vuelos v
        INNER JOIN aviones a ON v.id_avion = a.id_avion
        GROUP BY v.id_ruta
    )
);
```

Capacidades por ruta:
- MAD → LHR: Airbus A350(300) + Boeing 777(350) = **650**
- JFK → LAX: Boeing 777(350) = 350
- BCN → CDG: Embraer E195(120) = 120
- MAD → FRA: Airbus A350(300) = 300
- BCN → FCO: Embraer E195(120) = 120

Resultado:
| ruta | num_vuelos | capacidad_total_pasajeros |
|------|-----------|--------------------------|
| MAD → LHR | 2 | 650 |

</details>

### Tarea 3.3 — Análisis completo: aviones vs promedio de la flota

Para cada avión con vuelos, calcula su **distancia total volada** y compárala con la **distancia promedio** de toda la flota activa. Clasifica cada avión como `'Por encima del promedio'` o `'Por debajo del promedio'`. Usa subconsultas correlacionadas e inline views.

Muestra: `avion`, `distancia_total`, `distancia_promedio_flota`, `clasificacion`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT stats.avion,
       stats.distancia_total,
       (SELECT ROUND(AVG(dist_total), 2)
        FROM (
            SELECT SUM(r.distancia_km) AS dist_total
            FROM vuelos v
            INNER JOIN rutas r ON v.id_ruta = r.id_ruta
            GROUP BY v.id_avion
        )) AS distancia_promedio_flota,
       CASE
           WHEN stats.distancia_total > (
               SELECT AVG(dist_total)
               FROM (
                   SELECT SUM(r.distancia_km) AS dist_total
                   FROM vuelos v
                   INNER JOIN rutas r ON v.id_ruta = r.id_ruta
                   GROUP BY v.id_avion
               )
           ) THEN 'Por encima del promedio'
           ELSE 'Por debajo del promedio'
       END AS clasificacion
FROM (
    SELECT a.modelo AS avion,
           a.id_avion,
           SUM(r.distancia_km) AS distancia_total
    FROM aviones a
    INNER JOIN vuelos v ON a.id_avion = v.id_avion
    INNER JOIN rutas r ON v.id_ruta = r.id_ruta
    GROUP BY a.modelo, a.id_avion
) stats
ORDER BY stats.distancia_total DESC;
```

Distancias: Airbus A350=2770, Boeing 777=5350, Embraer E195=1680.
Promedio flota activa = (2770+5350+1680) / 3 = **3266.67**

Resultado:
| avion | distancia_total | distancia_promedio_flota | clasificacion |
|-------|----------------|--------------------------|---------------|
| Boeing 777 | 5350 | 3266.67 | Por encima del promedio |
| Airbus A350 | 2770 | 3266.67 | Por debajo del promedio |
| Embraer E195 | 1680 | 3266.67 | Por debajo del promedio |

> 💡 Solo el Boeing 777 supera el promedio de la flota, principalmente por la ruta transatlántica JFK → LAX (4000 km).

</details>

---

<div align="center">

⬅️ [**Ejercicios Subconsultas**](../ejercicios/ejercicios_subconsultas.md) · 🏠 [**Volver al Tema 10**](../README.md) · [**Tema 11: Vistas →**](../../11-vistas)

</div>
