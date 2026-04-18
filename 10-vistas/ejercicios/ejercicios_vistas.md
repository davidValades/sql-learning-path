# 🏋️ Ejercicios — Tema 10: Vistas y Objetos de BD

> 💡 **Instrucciones:** Resuelve cada ejercicio escribiendo la sentencia SQL completa. Todos los ejercicios usan el estado acumulado de la base de datos, incluyendo las tablas y datos creados en los temas 01-09.

---

## Ejercicio 1 — E-commerce · Crear una Vista con JOIN

**Enunciado:** Crea una vista llamada `v_pedidos_detalle` que muestre: `id_pedido`, `nombre` del cliente, `email` del cliente, `nombre` del producto, `cantidad`, `total` y `fecha_pedido`. Luego consulta los pedidos de `Ana López`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_pedidos_detalle AS
SELECT pe.id_pedido, cl.nombre AS cliente, cl.email,
       pr.nombre AS producto, pe.cantidad, pe.total, pe.fecha_pedido
FROM pedidos pe
JOIN clientes cl ON pe.id_cliente = cl.id_cliente
JOIN productos pr ON pe.id_producto = pr.id_producto;

SELECT producto, cantidad, total
FROM v_pedidos_detalle
WHERE cliente = 'Ana López';
```

Resultado:
| producto         | cantidad | total   |
|------------------|----------|---------|
| Laptop Pro       | 1        | 1220.00 |
| Lámpara LED      | 2        | 81.00   |
| Teclado Mecánico | 1        | 75.00   |

</details>

---

## Ejercicio 2 — Hospital · Vista de Solo Lectura

**Enunciado:** Crea una vista `v_salarios_especialidad` que muestre el salario promedio y el número de médicos por especialidad. Hazla de **solo lectura** con `WITH READ ONLY`. Luego consulta la especialidad con el salario medio más alto.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_salarios_especialidad AS
SELECT e.nombre_especialidad AS especialidad,
       COUNT(m.id_medico) AS num_medicos,
       ROUND(AVG(m.salario_base), 2) AS salario_medio
FROM especialidades e
JOIN medicos m ON e.id_especialidad = m.id_especialidad
GROUP BY e.nombre_especialidad
WITH READ ONLY;

SELECT * FROM v_salarios_especialidad
ORDER BY salario_medio DESC;
```

Resultado:
| especialidad     | num_medicos | salario_medio |
|------------------|-------------|---------------|
| Neurología       | 2           | 4250.00       |
| Traumatología    | 2           | 3000.00       |
| Medicina General | 1           | 2600.00       |

> 💡 `WITH READ ONLY` impide hacer INSERT/UPDATE/DELETE sobre esta vista.

</details>

---

## Ejercicio 3 — Aerolínea · Vista con Datos Calculados

**Enunciado:** Crea una vista `v_estadisticas_flota` que muestre por cada avión: `modelo`, `capacidad_pasajeros`, número de vuelos y distancia total volada en km. Incluye los aviones sin vuelos (con 0 vuelos y 0 distancia).

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_estadisticas_flota AS
SELECT a.modelo,
       a.capacidad_pasajeros,
       COUNT(v.id_vuelo) AS num_vuelos,
       COALESCE(SUM(r.distancia_km), 0) AS distancia_total_km
FROM aviones a
LEFT JOIN vuelos v ON a.id_avion = v.id_avion
LEFT JOIN rutas r ON v.id_ruta = r.id_ruta
GROUP BY a.modelo, a.capacidad_pasajeros;

SELECT * FROM v_estadisticas_flota ORDER BY distancia_total_km DESC;
```

Resultado:
| modelo       | capacidad_pasajeros | num_vuelos | distancia_total_km |
|--------------|---------------------|------------|--------------------|
| Boeing 777   | 350                 | 2          | 5350               |
| Airbus A350  | 300                 | 2          | 2770               |
| Embraer E195 | 120                 | 2          | 1680               |
| Boeing 737   | 180                 | 0          | 0                  |

</details>

---

## Ejercicio 4 — E-commerce · Vista Actualizable + CHECK OPTION

**Enunciado:** Crea una vista actualizable `v_productos_con_stock` que muestre solo los productos con `stock > 0`. Usa `WITH CHECK OPTION`. Luego intenta actualizar el stock de la Lámpara LED a 0. ¿Qué ocurre?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_productos_con_stock AS
SELECT id_producto, nombre, precio, stock
FROM productos
WHERE stock > 0
WITH CHECK OPTION;

-- Ver productos con stock
SELECT * FROM v_productos_con_stock;
-- Muestra 7 productos (todos menos Sofá de Cuero con stock = 0)

-- Intentar poner stock a 0:
UPDATE v_productos_con_stock
SET stock = 0
WHERE id_producto = 15;
-- ORA-01402: view WITH CHECK OPTION where-clause violation
```

> 💡 La actualización falla porque si `stock = 0`, la fila dejaría de cumplir la condición `stock > 0` de la vista. `CHECK OPTION` impide esa "desaparición".

</details>

---

## Ejercicio 5 — Hospital · Consultar Vistas como Tablas

**Enunciado:** Usando la vista `v_agenda_medica` (creada en la teoría), escribe una consulta que muestre cuántas citas tiene cada médico agrupadas por estado (`C`, `P`, `X`). Ordena por medico y estado.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
SELECT medico, estado, COUNT(*) AS num_citas
FROM v_agenda_medica
GROUP BY medico, estado
ORDER BY medico, estado;
```

Resultado:
| medico             | estado | num_citas |
|--------------------|--------|-----------|
| Dr. Carlos Vega    | C      | 1         |
| Dr. Luis Moreno    | P      | 1         |
| Dra. Elena Ruiz    | C      | 1         |
| Dra. Marta López   | C      | 1         |
| Dra. Marta López   | P      | 1         |
| Dra. Sarah Adams   | C      | 1         |
| Dra. Sarah Adams   | X      | 1         |

> 💡 Puedes usar `GROUP BY` sobre una vista exactamente igual que sobre una tabla.

</details>

---

## Ejercicio 6 — Secuencias · Crear y Usar

**Enunciado:** Crea una secuencia `seq_citas_nuevas` que empiece en 100 e incremente de 1 en 1. Luego escribe un `INSERT` (sin ejecutarlo) que use esta secuencia para insertar una nueva cita para el paciente Laura Martinez (id 1) con el Dr. Luis Moreno (id 7), fecha `2024-03-15 09:00`, estado `'P'`.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE SEQUENCE seq_citas_nuevas
START WITH 100
INCREMENT BY 1
NOCACHE;

-- INSERT usando la secuencia:
INSERT INTO citas (id_cita, id_paciente, id_medico, fecha_hora_cita, estado)
VALUES (seq_citas_nuevas.NEXTVAL, 1, 7, 
        TO_DATE('2024-03-15 09:00', 'YYYY-MM-DD HH24:MI'), 'P');

-- Verificar el valor asignado:
SELECT seq_citas_nuevas.CURRVAL FROM DUAL;
-- Resultado: 100
```

> 💡 Cada vez que se inserte una nueva cita, `seq_citas_nuevas.NEXTVAL` genera un ID único automáticamente.

</details>

---

## Ejercicio 7 — Sinónimos · Simplificar Acceso

**Enunciado:** Crea sinónimos para las tres vistas principales creadas en la teoría:
- `catalogo` → `v_catalogo_completo`
- `agenda` → `v_agenda_medica`
- `vuelos_ops` → `v_operaciones_vuelo`

Luego escribe una consulta usando el sinónimo `catalogo` para obtener los productos de Ropa con precio menor a 50.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE SYNONYM catalogo FOR v_catalogo_completo;
CREATE OR REPLACE SYNONYM agenda FOR v_agenda_medica;
CREATE OR REPLACE SYNONYM vuelos_ops FOR v_operaciones_vuelo;

SELECT nombre, precio
FROM catalogo
WHERE nombre_categoria = 'Ropa' AND precio < 50;
```

Resultado:
| nombre          | precio |
|-----------------|--------|
| Camiseta Básica | 19.99  |

> 💡 El sinónimo `catalogo` es más corto e intuitivo que `v_catalogo_completo`, y funciona exactamente igual.

</details>

---

## Ejercicio 8 — Combinado · Vista + Secuencia + Consulta

**Enunciado:** Crea una vista `v_clientes_vip` que muestre los clientes con gasto total superior a 400€ (usando la tabla `pedidos`). Luego crea una secuencia `seq_vip` que empiece en 1. Finalmente, escribe una consulta que combine la vista con la secuencia para asignar un "número VIP" a cada cliente.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
-- Vista de clientes VIP
CREATE OR REPLACE VIEW v_clientes_vip AS
SELECT cl.id_cliente, cl.nombre, cl.email,
       SUM(pe.total) AS gasto_total
FROM clientes cl
JOIN pedidos pe ON cl.id_cliente = pe.id_cliente
GROUP BY cl.id_cliente, cl.nombre, cl.email
HAVING SUM(pe.total) > 400;

-- Secuencia VIP
CREATE SEQUENCE seq_vip
START WITH 1
INCREMENT BY 1
NOCACHE;

-- Asignar número VIP (conceptual — cada NEXTVAL genera un valor único)
SELECT seq_vip.NEXTVAL AS num_vip, nombre, email, gasto_total
FROM v_clientes_vip
ORDER BY gasto_total DESC;
```

Resultado:
| num_vip | nombre     | email           | gasto_total |
|---------|------------|-----------------|-------------|
| 1       | Ana López  | ana@email.com   | 1376.00     |
| 2       | Pedro Ruiz | pedro@email.com | 486.50      |

> 💡 `NEXTVAL` se incrementa por cada fila devuelta, asignando un número VIP único a cada cliente que supera los 400€.

</details>

---

<div align="center">

⬅️ [**Volver al Tema 10**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Proyecto Vistas →**](../proyectos/proyecto_vistas.md)

</div>
