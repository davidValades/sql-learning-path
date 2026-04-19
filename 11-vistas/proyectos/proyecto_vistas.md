# 🏆 Proyecto: El Panel de Control Empresarial

> 📋 **Contexto:** La dirección general de las tres organizaciones (e-commerce, hospital y aerolínea) necesita un **panel de control (dashboard)** que permita a los ejecutivos consultar métricas clave sin escribir SQL complejo. Tu trabajo es diseñar un sistema de vistas que actúe como capa de presentación sobre las bases de datos existentes.

---

## 🎯 Misión 1 — E-commerce: Dashboard Comercial

El director comercial necesita tres vistas que alimenten su panel de control diario.

### Tarea 1.1 — Vista: Top Productos por Ingresos

Crea una vista `v_top_productos` que muestre cada producto con su ingreso total por ventas, número de unidades vendidas y el porcentaje que representa sobre el ingreso total de la tienda. Ordena por ingresos descendente.

Muestra: `producto`, `categoria`, `unidades_vendidas`, `ingreso_total`, `porcentaje_ingresos`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_top_productos AS
SELECT pr.nombre AS producto,
       ca.nombre_categoria AS categoria,
       SUM(pe.cantidad) AS unidades_vendidas,
       SUM(pe.total) AS ingreso_total,
       ROUND(SUM(pe.total) * 100 / (SELECT SUM(total) FROM pedidos), 2) AS porcentaje_ingresos
FROM pedidos pe
JOIN productos pr ON pe.id_producto = pr.id_producto
JOIN categorias ca ON pr.id_categoria = ca.id_categoria
GROUP BY pr.nombre, ca.nombre_categoria;

SELECT * FROM v_top_productos ORDER BY ingreso_total DESC;
```

Ingreso total global = 1220 + 350 + 136.50 + 75 + 81 + 99.95 + 179 = 2141.45

Resultado:
| producto           | categoria   | unidades_vendidas | ingreso_total | porcentaje_ingresos |
|--------------------|-------------|-------------------|---------------|---------------------|
| Laptop Pro         | Electrónica | 1                 | 1220.00       | 56.97               |
| Monitor 4K         | Electrónica | 1                 | 350.00        | 16.34               |
| Zapatillas Running | Ropa        | 2                 | 179.00        | 8.36                |
| Ratón Inalámbrico  | Electrónica | 3                 | 136.50        | 6.37                |
| Camiseta Básica    | Ropa        | 5                 | 99.95         | 4.67                |
| Lámpara LED        | Hogar       | 2                 | 81.00         | 3.78                |
| Teclado Mecánico   | Electrónica | 1                 | 75.00         | 3.50                |

</details>

### Tarea 1.2 — Vista: Estado de Inventario Crítico

Crea una vista `v_inventario_critico` que muestre los productos cuyo stock es menor o igual a 25 unidades, incluyendo un campo `alerta` que diga `'SIN STOCK'` si stock = 0, `'CRÍTICO'` si stock <= 10, o `'BAJO'` si stock <= 25. Hazla de solo lectura.

Muestra: `producto`, `precio`, `stock`, `alerta`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_inventario_critico AS
SELECT nombre AS producto, precio, stock,
       CASE
           WHEN stock = 0 THEN 'SIN STOCK'
           WHEN stock <= 10 THEN 'CRÍTICO'
           ELSE 'BAJO'
       END AS alerta
FROM productos
WHERE stock <= 25
WITH READ ONLY;

SELECT * FROM v_inventario_critico ORDER BY stock;
```

Resultado:
| producto   | precio  | stock | alerta    |
|------------|---------|-------|-----------|
| Sofá de Cuero | 405.00 | 0   | SIN STOCK |
| Laptop Pro    | 1220.00 | 15  | BAJO      |
| Monitor 4K    | 350.00  | 25  | BAJO      |

</details>

### Tarea 1.3 — Vista + Secuencia: Registro de Alertas

Crea una secuencia `seq_alertas` que empiece en 1. Luego escribe un INSERT que use la vista `v_inventario_critico` y la secuencia para registrar alertas en una hipotética tabla `log_alertas(id_alerta, producto, alerta, fecha_registro)`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE SEQUENCE seq_alertas START WITH 1 INCREMENT BY 1 NOCACHE;

-- Tabla de log (si no existe):
-- CREATE TABLE log_alertas (
--     id_alerta NUMBER PRIMARY KEY,
--     producto VARCHAR2(100),
--     alerta VARCHAR2(20),
--     fecha_registro DATE DEFAULT SYSDATE
-- );

-- INSERT masivo desde la vista usando la secuencia:
INSERT INTO log_alertas (id_alerta, producto, alerta, fecha_registro)
SELECT seq_alertas.NEXTVAL, producto, alerta, SYSDATE
FROM v_inventario_critico;

-- Resultado: 3 filas insertadas (Sofá, Laptop Pro, Monitor 4K)
-- con id_alerta = 1, 2, 3
```

> 💡 Este patrón (vista + secuencia + INSERT...SELECT) es muy común en sistemas de monitoreo y alertas automatizadas.

</details>

---

## 🎯 Misión 2 — Hospital: Dashboard Médico

La dirección médica necesita vistas para monitorizar la actividad del hospital.

### Tarea 2.1 — Vista: Rendimiento Médico

Crea una vista `v_rendimiento_medico` que muestre por cada médico: su nombre, especialidad, total de citas, citas completadas, citas pendientes, citas canceladas y una **tasa de completitud** (completadas / total * 100). Hazla de solo lectura.

Muestra: `medico`, `especialidad`, `total_citas`, `completadas`, `pendientes`, `canceladas`, `tasa_completitud`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_rendimiento_medico AS
SELECT m.nombre_completo AS medico,
       e.nombre_especialidad AS especialidad,
       COUNT(ci.id_cita) AS total_citas,
       SUM(CASE WHEN ci.estado = 'C' THEN 1 ELSE 0 END) AS completadas,
       SUM(CASE WHEN ci.estado = 'P' THEN 1 ELSE 0 END) AS pendientes,
       SUM(CASE WHEN ci.estado = 'X' THEN 1 ELSE 0 END) AS canceladas,
       ROUND(SUM(CASE WHEN ci.estado = 'C' THEN 1 ELSE 0 END) * 100
             / COUNT(ci.id_cita), 1) AS tasa_completitud
FROM medicos m
JOIN especialidades e ON m.id_especialidad = e.id_especialidad
LEFT JOIN citas ci ON m.id_medico = ci.id_medico
GROUP BY m.nombre_completo, e.nombre_especialidad
WITH READ ONLY;

SELECT * FROM v_rendimiento_medico ORDER BY tasa_completitud DESC;
```

Resultado:
| medico             | especialidad     | total_citas | completadas | pendientes | canceladas | tasa_completitud |
|--------------------|------------------|-------------|-------------|------------|------------|------------------|
| Dra. Elena Ruiz    | Traumatología    | 1           | 1           | 0          | 0          | 100.0            |
| Dr. Carlos Vega    | Traumatología    | 1           | 1           | 0          | 0          | 100.0            |
| Dra. Marta López   | Neurología       | 2           | 1           | 1          | 0          | 50.0             |
| Dra. Sarah Adams   | Neurología       | 2           | 1           | 0          | 1          | 50.0             |
| Dr. Luis Moreno    | Medicina General | 1           | 0           | 1          | 0          | 0.0              |

</details>

### Tarea 2.2 — Vista: Pacientes con Historial Completo

Crea una vista `v_historial_pacientes` que muestre cada paciente con su número de citas, última fecha de cita y un campo `necesita_seguimiento` que sea `'SÍ'` si tiene alguna cita pendiente y `'NO'` en caso contrario.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_historial_pacientes AS
SELECT pa.nombre_completo AS paciente,
       pa.telefono,
       COUNT(ci.id_cita) AS num_citas,
       MAX(ci.fecha_hora_cita) AS ultima_cita,
       CASE
           WHEN SUM(CASE WHEN ci.estado = 'P' THEN 1 ELSE 0 END) > 0
           THEN 'SÍ'
           ELSE 'NO'
       END AS necesita_seguimiento
FROM pacientes pa
LEFT JOIN citas ci ON pa.id_paciente = ci.id_paciente
GROUP BY pa.nombre_completo, pa.telefono;

SELECT * FROM v_historial_pacientes ORDER BY num_citas DESC;
```

Resultado:
| paciente        | telefono    | num_citas | ultima_cita | necesita_seguimiento |
|-----------------|-------------|-----------|-------------|----------------------|
| Laura Martinez  | 611...      | 2         | ...         | NO                   |
| Carlos Gomez    | 622...      | 2         | ...         | NO                   |
| Pedro Sánchez   | 633...      | 1         | ...         | SÍ                   |
| Ana Ruiz        | NULL        | 1         | ...         | NO                   |
| David Torres    | 655...      | 1         | ...         | SÍ                   |

</details>

---

## 🎯 Misión 3 — Aerolínea: Dashboard de Operaciones

El director de operaciones necesita visibilidad completa de la flota y las rutas.

### Tarea 3.1 — Vista: Mapa de Rutas Activas

Crea una vista `v_mapa_rutas` que muestre cada ruta con: origen, destino, distancia, número de vuelos programados, total de capacidad de pasajeros y un campo `popularidad` (`'ALTA'` si tiene más de 1 vuelo, `'NORMAL'` si tiene 1, `'SIN SERVICIO'` si no tiene vuelos).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_mapa_rutas AS
SELECT r.aeropuerto_origen AS origen,
       r.aeropuerto_destino AS destino,
       r.distancia_km,
       COUNT(v.id_vuelo) AS num_vuelos,
       COALESCE(SUM(a.capacidad_pasajeros), 0) AS capacidad_total,
       CASE
           WHEN COUNT(v.id_vuelo) > 1 THEN 'ALTA'
           WHEN COUNT(v.id_vuelo) = 1 THEN 'NORMAL'
           ELSE 'SIN SERVICIO'
       END AS popularidad
FROM rutas r
LEFT JOIN vuelos v ON r.id_ruta = v.id_ruta
LEFT JOIN aviones a ON v.id_avion = a.id_avion
GROUP BY r.aeropuerto_origen, r.aeropuerto_destino, r.distancia_km;

SELECT * FROM v_mapa_rutas ORDER BY num_vuelos DESC;
```

Resultado:
| origen | destino | distancia_km | num_vuelos | capacidad_total | popularidad |
|--------|---------|--------------|------------|-----------------|-------------|
| MAD    | LHR     | 1350         | 2          | 650             | ALTA        |
| JFK    | LAX     | 4000         | 1          | 350             | NORMAL      |
| BCN    | CDG     | 830          | 1          | 120             | NORMAL      |
| MAD    | FRA     | 1420         | 1          | 300             | NORMAL      |
| BCN    | FCO     | 850          | 1          | 120             | NORMAL      |

> 💡 Todas las rutas tienen al menos un vuelo. La ruta MAD→LHR es la única con popularidad ALTA.

</details>

### Tarea 3.2 — Vista + Sinónimo: Acceso Simplificado

Crea un sinónimo `rutas_activas` para `v_mapa_rutas` y otro `flota` para `v_estadisticas_flota` (del ejercicio 3). Luego escribe una consulta que use solo sinónimos para encontrar la ruta de alta popularidad y el avión con más distancia volada.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE SYNONYM rutas_activas FOR v_mapa_rutas;
CREATE OR REPLACE SYNONYM flota FOR v_estadisticas_flota;

-- Ruta más popular
SELECT origen || ' → ' || destino AS ruta, capacidad_total
FROM rutas_activas
WHERE popularidad = 'ALTA';
-- Resultado: MAD → LHR, 650

-- Avión con más distancia
SELECT modelo, distancia_total_km
FROM flota
WHERE distancia_total_km = (SELECT MAX(distancia_total_km) FROM flota);
-- Resultado: Boeing 777, 5350
```

> 💡 Los sinónimos hacen que las consultas sean más legibles y los dashboards más fáciles de mantener.

</details>

---

<div align="center">

⬅️ [**📝 Ejercicios del Tema 11**](../ejercicios/ejercicios_vistas.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 12: Indexación →**](../../12-indexacion/README.md)

</div>
