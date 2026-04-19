# 🖼️ Tema 11: Vistas y Objetos de BD

> **"Si una consulta compleja la usas todos los días, no la reescribas: conviértela en una vista."** Las vistas, secuencias y sinónimos son objetos de base de datos que simplifican tu trabajo, protegen los datos y mantienen la arquitectura ordenada. Son el paso de escribir SQL a diseñar bases de datos profesionales.

## 📋 Índice

- [11.1 ¿Qué es una Vista?](#111-qué-es-una-vista)
- [11.2 Vistas Actualizables vs. Solo Lectura](#112-vistas-actualizables-vs-solo-lectura)
- [11.3 Vistas Materializadas](#113-vistas-materializadas)
- [11.4 Secuencias](#114-secuencias)
- [11.5 Sinónimos](#115-sinónimos)

---

---

## 11.1 ¿Qué es una Vista?

### 📘 El Concepto

Una **vista** (`VIEW`) es una **consulta almacenada con nombre**. No guarda datos físicamente — cada vez que la consultas, Oracle ejecuta la sentencia `SELECT` subyacente y te devuelve los resultados actualizados.

```
CREATE [OR REPLACE] VIEW nombre_vista AS
SELECT columnas
FROM tablas
WHERE condiciones;
```

| Aspecto | Vista | Tabla |
|---------|-------|-------|
| Almacena datos | ❌ No (solo la definición SQL) | ✅ Sí |
| Ocupa espacio en disco | Mínimo (solo metadatos) | Sí (datos reales) |
| Se actualiza sola | ✅ Siempre muestra datos actuales | — |
| Se puede hacer SELECT | ✅ Sí | ✅ Sí |
| Se puede hacer INSERT/UPDATE | ⚠️ Depende (ver 11.2) | ✅ Sí |

> 📌 `OR REPLACE` permite modificar una vista existente sin tener que hacer `DROP` primero.

### 🏠 La Analogía

Imagina una **ventana** en tu oficina. No almacena el paisaje — simplemente te muestra lo que hay fuera en ese momento. Si plantan un árbol nuevo, lo verás al instante a través de la ventana. Una vista SQL funciona igual: es una ventana a los datos que siempre muestra la realidad actual sin duplicar la información.

### 💻 El Código

```sql
-- E-commerce: catálogo completo con nombre de categoría
CREATE OR REPLACE VIEW v_catalogo_completo AS
SELECT p.id_producto, p.nombre, p.precio, c.nombre_categoria, p.stock
FROM productos p
JOIN categorias c ON p.id_categoria = c.id_categoria;

-- Consultar la vista es idéntico a consultar una tabla
SELECT * FROM v_catalogo_completo;
-- Resultado:
-- | id_producto | nombre             | precio  | nombre_categoria | stock |
-- |-------------|--------------------|---------|------------------|-------|
-- | 10          | Laptop Pro         | 1220.00 | Electrónica      | 15    |
-- | 11          | Ratón Inalámbrico  | 45.50   | Electrónica      | 100   |
-- | 12          | Monitor 4K         | 350.00  | Electrónica      | 25    |
-- | 13          | Teclado Mecánico   | 75.00   | Electrónica      | 50    |
-- | 14          | Sofá de Cuero      | 405.00  | Hogar            | 0     |
-- | 15          | Lámpara LED        | 40.50   | Hogar            | 200   |
-- | 16          | Camiseta Básica    | 19.99   | Ropa             | 500   |
-- | 17          | Zapatillas Running | 89.50   | Ropa             | 75    |

-- Puedes filtrar sobre la vista como si fuera una tabla
SELECT nombre, precio FROM v_catalogo_completo
WHERE nombre_categoria = 'Electrónica' AND precio < 100;
-- Resultado: Ratón Inalámbrico (45.50), Teclado Mecánico (75.00)

-- Hospital: agenda médica completa
CREATE OR REPLACE VIEW v_agenda_medica AS
SELECT ci.id_cita, pa.nombre AS paciente, m.nombre_completo AS medico,
       e.nombre_especialidad AS especialidad, ci.fecha_hora_cita, ci.estado
FROM citas ci
JOIN pacientes pa ON ci.id_paciente = pa.id_paciente
JOIN medicos m ON ci.id_medico = m.id_medico
JOIN especialidades e ON m.id_especialidad = e.id_especialidad;

SELECT paciente, medico, especialidad, estado
FROM v_agenda_medica
WHERE estado = 'P';
-- Resultado:
-- | paciente       | medico           | especialidad     | estado |
-- |----------------|------------------|------------------|--------|
-- | Pedro Sánchez  | Dra. Marta López | Neurología       | P      |
-- | David Torres   | Dr. Luis Moreno  | Medicina General | P      |

-- Aerolínea: operaciones de vuelo
CREATE OR REPLACE VIEW v_operaciones_vuelo AS
SELECT v.id_vuelo, r.aeropuerto_origen, r.aeropuerto_destino,
       a.modelo, a.capacidad_pasajeros, v.fecha_salida, v.puerta_embarque
FROM vuelos v
JOIN rutas r ON v.id_ruta = r.id_ruta
JOIN aviones a ON v.id_avion = a.id_avion;

SELECT id_vuelo, aeropuerto_origen, aeropuerto_destino, modelo
FROM v_operaciones_vuelo
WHERE aeropuerto_origen = 'MAD';
-- Resultado:
-- | id_vuelo | aeropuerto_origen | aeropuerto_destino | modelo      |
-- |----------|-------------------|--------------------|-------------|
-- | 1001     | MAD               | LHR                | Airbus A350 |
-- | 1004     | MAD               | FRA                | Airbus A350 |
-- | 1005     | MAD               | LHR                | Boeing 777  |
```

> 📌 **Para eliminar una vista:** `DROP VIEW nombre_vista;`  
> No se eliminan los datos de las tablas subyacentes — solo desaparece la "ventana".

### 🧠 El Reto

Crea una vista llamada `v_resumen_pedidos` que muestre: `id_pedido`, `nombre` del cliente, `nombre` del producto, `cantidad`, `total` y `fecha_pedido`. Luego consulta la vista para ver solo los pedidos con total mayor a 200.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_resumen_pedidos AS
SELECT pe.id_pedido, cl.nombre AS cliente, pr.nombre AS producto,
       pe.cantidad, pe.total, pe.fecha_pedido
FROM pedidos pe
JOIN clientes cl ON pe.id_cliente = cl.id_cliente
JOIN productos pr ON pe.id_producto = pr.id_producto;

SELECT cliente, producto, total
FROM v_resumen_pedidos
WHERE total > 200;
```

Resultado:
| cliente   | producto   | total   |
|-----------|------------|---------|
| Ana López | Laptop Pro | 1220.00 |
| Pedro Ruiz | Monitor 4K | 350.00 |

</details>

---

---

## 11.2 Vistas Actualizables vs. Solo Lectura

### 📘 El Concepto

Una vista es **actualizable** si Oracle puede determinar sin ambigüedad a qué fila de la tabla base corresponde cada fila de la vista. Puedes hacer `INSERT`, `UPDATE` y `DELETE` a través de ella.

**Condiciones para que una vista sea actualizable:**
- Se basa en **una sola tabla**.
- No usa `GROUP BY`, `DISTINCT`, funciones de agregación ni `UNION`.
- Incluye la **clave primaria** de la tabla base (para UPDATE/DELETE).

**Vista de solo lectura:** se fuerza con `WITH READ ONLY`.

```
-- Vista actualizable (una sola tabla, sin agregaciones)
CREATE OR REPLACE VIEW v_productos_electronicos AS
SELECT id_producto, nombre, precio, stock
FROM productos
WHERE id_categoria = 1;

-- Vista de solo lectura (proteger contra modificaciones)
CREATE OR REPLACE VIEW v_estadisticas_ventas AS
SELECT id_cliente, COUNT(*) AS num_pedidos, SUM(total) AS gasto_total
FROM pedidos
GROUP BY id_cliente
WITH READ ONLY;
```

### 🏠 La Analogía

Piensa en dos tipos de ventana:
- **Ventana abatible** (actualizable): puedes ver el paisaje **y** además abrirla para pasar objetos al otro lado. Puedes modificar lo que hay fuera.
- **Ventana fija con cristal blindado** (`WITH READ ONLY`): solo puedes mirar, pero no tocar. Es más segura porque nadie puede alterar los datos a través de ella.

### 💻 El Código

```sql
-- Vista actualizable: productos de Hogar
CREATE OR REPLACE VIEW v_productos_hogar AS
SELECT id_producto, nombre, precio, stock
FROM productos
WHERE id_categoria = 2;

-- Podemos actualizar a través de la vista
UPDATE v_productos_hogar
SET precio = 42.00
WHERE id_producto = 15;
-- Actualiza el precio de Lámpara LED en la tabla productos

-- Revertimos para mantener consistencia
UPDATE v_productos_hogar
SET precio = 40.50
WHERE id_producto = 15;

-- Vista de solo lectura: proteger estadísticas médicas
CREATE OR REPLACE VIEW v_citas_por_medico AS
SELECT m.nombre_completo AS medico, COUNT(ci.id_cita) AS total_citas
FROM medicos m
LEFT JOIN citas ci ON m.id_medico = ci.id_medico
GROUP BY m.nombre_completo
WITH READ ONLY;

SELECT * FROM v_citas_por_medico;
-- | medico             | total_citas |
-- |--------------------|-------------|
-- | Dra. Sarah Adams   | 2           |
-- | Dra. Elena Ruiz    | 1           |
-- | Dr. Carlos Vega    | 1           |
-- | Dra. Marta López   | 2           |
-- | Dr. Luis Moreno    | 1           |

-- Si intentamos modificar:
-- DELETE FROM v_citas_por_medico WHERE total_citas = 1;
-- ORA-42399: cannot perform a DML operation on a read-only view

-- Aerolínea: vista de solo lectura para panel de control
CREATE OR REPLACE VIEW v_panel_vuelos AS
SELECT r.aeropuerto_origen || ' → ' || r.aeropuerto_destino AS ruta,
       COUNT(v.id_vuelo) AS vuelos_programados,
       SUM(a.capacidad_pasajeros) AS capacidad_total
FROM vuelos v
JOIN rutas r ON v.id_ruta = r.id_ruta
JOIN aviones a ON v.id_avion = a.id_avion
GROUP BY r.aeropuerto_origen, r.aeropuerto_destino
WITH READ ONLY;
```

> ⚠️ **CHECK OPTION:** `WITH CHECK OPTION` evita que insertes/actualices filas que luego "desaparezcan" de la vista:
> ```sql
> CREATE OR REPLACE VIEW v_productos_baratos AS
> SELECT * FROM productos WHERE precio < 100
> WITH CHECK OPTION;
>
> -- Esto falla: el precio 500 no cumple la condición de la vista
> UPDATE v_productos_baratos SET precio = 500 WHERE id_producto = 16;
> -- ORA-01402: view WITH CHECK OPTION where-clause violation
> ```

### 🧠 El Reto

Crea una vista actualizable `v_pacientes_con_telefono` que muestre solo los pacientes que **tienen teléfono** (no NULL). Luego intenta insertar un paciente sin teléfono usando `WITH CHECK OPTION`. ¿Qué ocurre?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE VIEW v_pacientes_con_telefono AS
SELECT id_paciente, nombre, telefono
FROM pacientes
WHERE telefono IS NOT NULL
WITH CHECK OPTION;

-- Consultar: muestra los 4 pacientes con teléfono (excluye Ana Ruiz)
SELECT * FROM v_pacientes_con_telefono;

-- Intentar insertar sin teléfono:
-- INSERT INTO v_pacientes_con_telefono (id_paciente, nombre, telefono)
-- VALUES (6, 'Test Sin Tel', NULL);
-- ORA-01402: view WITH CHECK OPTION where-clause violation
```

> 💡 El `CHECK OPTION` protege la integridad: no permite insertar filas que no cumplan el `WHERE` de la vista.

</details>

---

---

## 11.3 Vistas Materializadas

### 📘 El Concepto

Una **vista materializada** (`MATERIALIZED VIEW`) sí almacena físicamente los datos del resultado de la consulta. A diferencia de una vista normal, **no recalcula cada vez** que la consultas — usa una copia almacenada.

```
CREATE MATERIALIZED VIEW nombre_mv
BUILD IMMEDIATE          -- Carga datos al crear
REFRESH ON DEMAND        -- Se refresca manualmente
AS
SELECT ...;

-- Refrescar manualmente:
BEGIN
    DBMS_MVIEW.REFRESH('nombre_mv');
END;
/
```

| Característica | Vista Normal | Vista Materializada |
|---------------|-------------|---------------------|
| Almacena datos | ❌ No | ✅ Sí |
| Rendimiento en lectura | Recalcula siempre | ⚡ Muy rápido (datos precalculados) |
| Datos actualizados | Siempre en tiempo real | Depende del modo de refresco |
| Usa espacio en disco | Mínimo | Sí (como una tabla) |
| Caso de uso ideal | Consultas ligeras | Informes pesados, dashboards |

**Modos de refresco:**
- `REFRESH ON DEMAND` — Se refresca manualmente cuando tú lo decides.
- `REFRESH ON COMMIT` — Se refresca automáticamente tras cada `COMMIT` en las tablas base.
- `REFRESH COMPLETE` — Reconstruye todos los datos desde cero.
- `REFRESH FAST` — Solo actualiza los cambios incrementales (requiere logs de materialización).

### 🏠 La Analogía

Si una vista normal es una **ventana** (ves el paisaje en vivo), una vista materializada es una **fotografía enmarcada** del paisaje. Es instantánea de ver (no necesitas mirar por la ventana), pero muestra el paisaje tal como estaba cuando se tomó la foto. Necesitas **tomar una nueva foto** (refrescar) para ver los cambios recientes.

### 💻 El Código

```sql
-- E-commerce: resumen de ventas por categoría (vista materializada)
CREATE MATERIALIZED VIEW mv_ventas_categoria
BUILD IMMEDIATE
REFRESH ON DEMAND
AS
SELECT c.nombre_categoria,
       COUNT(pe.id_pedido) AS total_pedidos,
       SUM(pe.total) AS ingresos_totales,
       ROUND(AVG(pe.total), 2) AS ticket_medio
FROM pedidos pe
JOIN productos p ON pe.id_producto = p.id_producto
JOIN categorias c ON p.id_categoria = c.id_categoria
GROUP BY c.nombre_categoria;

-- Consultar es instantáneo (datos ya precalculados)
SELECT * FROM mv_ventas_categoria;
-- | nombre_categoria | total_pedidos | ingresos_totales | ticket_medio |
-- |------------------|---------------|------------------|--------------|
-- | Electrónica      | 4             | 1781.50          | 445.38       |
-- | Hogar            | 1             | 81.00            | 81.00        |
-- | Ropa             | 2             | 278.95           | 139.48       |

-- Hospital: estadísticas de actividad por especialidad
CREATE MATERIALIZED VIEW mv_actividad_especialidades
BUILD IMMEDIATE
REFRESH ON DEMAND
AS
SELECT e.nombre_especialidad AS especialidad,
       COUNT(ci.id_cita) AS total_citas,
       SUM(CASE WHEN ci.estado = 'C' THEN 1 ELSE 0 END) AS completadas,
       SUM(CASE WHEN ci.estado = 'P' THEN 1 ELSE 0 END) AS pendientes,
       SUM(CASE WHEN ci.estado = 'X' THEN 1 ELSE 0 END) AS canceladas
FROM especialidades e
LEFT JOIN medicos m ON e.id_especialidad = m.id_especialidad
LEFT JOIN citas ci ON m.id_medico = ci.id_medico
GROUP BY e.nombre_especialidad;

-- Refrescar cuando los datos cambien:
-- BEGIN
--     DBMS_MVIEW.REFRESH('mv_actividad_especialidades');
-- END;
-- /

-- Eliminar vista materializada:
-- DROP MATERIALIZED VIEW mv_ventas_categoria;
```

> ⚠️ **Nota:** Crear vistas materializadas requiere privilegios adicionales (`CREATE MATERIALIZED VIEW`). En entornos de aprendizaje, es importante conocer el concepto aunque no siempre puedas ejecutarlo.

### 🧠 El Reto

¿En cuál de estos casos usarías una vista materializada en vez de una vista normal? Justifica tu respuesta.

1. Una consulta que une `vuelos`, `rutas` y `aviones` para mostrar 6 filas.
2. Un informe mensual que cruza millones de filas de ventas con inventario y calcula KPIs.
3. Un formulario que edita datos de pacientes.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

**Respuesta: Caso 2.**

1. ❌ Solo 6 filas — una vista normal es suficiente y siempre tendrás datos en tiempo real.
2. ✅ **Vista materializada** — millones de filas + cálculos complejos = lento. Precalcular y refrescar diariamente es ideal para dashboards e informes.
3. ❌ El formulario necesita datos en tiempo real y debe poder hacer INSERT/UPDATE — una vista materializada no es actualizable.

> 💡 Regla general: usa vistas materializadas para **informes pesados que no necesitan datos al segundo**, nunca para operaciones transaccionales.

</details>

---

---

## 11.4 Secuencias

### 📘 El Concepto

Una **secuencia** (`SEQUENCE`) es un objeto que genera **números únicos y secuenciales**. Es la forma estándar en Oracle de crear IDs autoincrementales antes de Oracle 12c (que introdujo columnas `IDENTITY`).

```
CREATE SEQUENCE nombre_secuencia
START WITH valor_inicial
INCREMENT BY incremento
[MINVALUE n] [MAXVALUE n]
[NOCACHE | CACHE n]
[NOCYCLE | CYCLE];
```

| Propiedad | Descripción | Ejemplo |
|-----------|-------------|---------|
| `START WITH` | Primer valor generado | `START WITH 100` |
| `INCREMENT BY` | Paso entre valores | `INCREMENT BY 1` |
| `MAXVALUE` | Valor máximo permitido | `MAXVALUE 99999` |
| `NOCACHE` / `CACHE n` | Pregenerar n valores en memoria (rápido) | `CACHE 20` |
| `NOCYCLE` / `CYCLE` | ¿Reiniciar al llegar al máximo? | `NOCYCLE` |

**Funciones clave:**
- `nombre_secuencia.NEXTVAL` — Genera y devuelve el **siguiente** valor.
- `nombre_secuencia.CURRVAL` — Devuelve el **último valor generado** en la sesión actual.

> ⚠️ **CURRVAL solo funciona después de haber llamado a NEXTVAL** al menos una vez en la sesión.

### 🏠 La Analogía

Piensa en una **máquina expendedora de turnos** en un banco. Cada persona que llega arranca un número (NEXTVAL) y recibe un número único que nunca se repite. Si quieres recordar qué número te tocó, miras tu ticket (CURRVAL). La máquina nunca retrocede — siempre avanza al siguiente número.

### 💻 El Código

```sql
-- E-commerce: secuencia para IDs de pedidos futuros
CREATE SEQUENCE seq_pedidos
START WITH 100
INCREMENT BY 1
NOCACHE;

-- Generar el siguiente valor
SELECT seq_pedidos.NEXTVAL FROM DUAL;
-- Resultado: 100

SELECT seq_pedidos.NEXTVAL FROM DUAL;
-- Resultado: 101

-- Usar en un INSERT
INSERT INTO pedidos (id_pedido, id_cliente, id_producto, cantidad, fecha_pedido, total)
VALUES (seq_pedidos.NEXTVAL, 1, 11, 3, SYSDATE, 136.50);
-- Inserta con id_pedido = 102

-- Ver el último valor generado en esta sesión
SELECT seq_pedidos.CURRVAL FROM DUAL;
-- Resultado: 102

-- Hospital: secuencia para citas
CREATE SEQUENCE seq_citas
START WITH 50
INCREMENT BY 1
CACHE 20;

-- Aerolínea: secuencia para vuelos
CREATE SEQUENCE seq_vuelos
START WITH 2000
INCREMENT BY 1
NOCACHE;

-- Información de la secuencia
SELECT sequence_name, last_number, increment_by, cache_size
FROM user_sequences
WHERE sequence_name = 'SEQ_PEDIDOS';

-- Eliminar secuencia
-- DROP SEQUENCE seq_pedidos;
```

> 📌 **Importante:** Los valores de secuencia **nunca se reutilizan**, incluso si haces `ROLLBACK` de la transacción. Si usaste NEXTVAL y luego hiciste ROLLBACK, ese número se pierde. Esto garantiza unicidad pero puede dejar "huecos" en la numeración.

### 🧠 El Reto

Crea una secuencia `seq_productos` que empiece en 100, incremente de 10 en 10, y tenga un valor máximo de 990. Luego genera 3 valores. ¿Cuáles son?

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE SEQUENCE seq_productos
START WITH 100
INCREMENT BY 10
MAXVALUE 990
NOCACHE;

SELECT seq_productos.NEXTVAL FROM DUAL; -- 100
SELECT seq_productos.NEXTVAL FROM DUAL; -- 110
SELECT seq_productos.NEXTVAL FROM DUAL; -- 120
```

Los tres valores son: **100, 110, 120**.

> 💡 Con `INCREMENT BY 10`, los códigos de producto serían 100, 110, 120, 130... hasta 990. Esto deja espacio para insertar valores intermedios manualmente si fuera necesario.

</details>

---

---

## 11.5 Sinónimos

### 📘 El Concepto

Un **sinónimo** (`SYNONYM`) es un **alias permanente** para un objeto de base de datos (tabla, vista, secuencia, etc.). Sirve para simplificar nombres largos o proporcionar acceso transparente a objetos de otros esquemas.

```
-- Sinónimo privado (solo para tu esquema)
CREATE [OR REPLACE] SYNONYM nombre_corto FOR esquema.nombre_largo;

-- Sinónimo público (accesible por todos los usuarios)
CREATE [OR REPLACE] PUBLIC SYNONYM nombre_corto FOR esquema.nombre_largo;
```

**Casos de uso:**
- Simplificar nombres de objetos en otros esquemas: `hr.employees` → `emp`.
- Abstraer la ubicación: si una tabla se mueve de esquema, solo cambias el sinónimo.
- Compatibilidad: mantener nombres antiguos cuando renombras objetos.

### 🏠 La Analogía

Un sinónimo es como un **contacto favorito en tu teléfono**. En vez de marcar el número completo (+34-91-555-1234), guardas "Mamá" como nombre corto. Si "Mamá" cambia de número, solo actualizas el contacto — no necesitas recordar el número nuevo. Igualmente, si una tabla se mueve de esquema, solo actualizas el sinónimo.

### 💻 El Código

```sql
-- Simplificar acceso a vistas con nombres largos
CREATE OR REPLACE SYNONYM catalogo FOR v_catalogo_completo;
CREATE OR REPLACE SYNONYM agenda FOR v_agenda_medica;
CREATE OR REPLACE SYNONYM vuelos_ops FOR v_operaciones_vuelo;

-- Ahora puedes usar el sinónimo como si fuera la tabla/vista original
SELECT nombre, precio FROM catalogo WHERE stock > 50;
-- Equivale a: SELECT nombre, precio FROM v_catalogo_completo WHERE stock > 50;
-- Resultado:
-- | nombre             | precio | 
-- |--------------------|--------|
-- | Ratón Inalámbrico  | 45.50  |
-- | Lámpara LED        | 40.50  |
-- | Camiseta Básica    | 19.99  |
-- | Zapatillas Running | 89.50  |

SELECT paciente, medico, estado FROM agenda WHERE estado = 'C';
-- Muestra las 4 citas completadas

-- Sinónimo para una secuencia
CREATE OR REPLACE SYNONYM siguiente_pedido FOR seq_pedidos;
SELECT siguiente_pedido.NEXTVAL FROM DUAL;

-- Ver sinónimos existentes
SELECT synonym_name, table_name FROM user_synonyms;

-- Eliminar sinónimo
-- DROP SYNONYM catalogo;
```

### 🧠 El Reto

Imagina que el departamento de TI ha reestructurado el hospital y la tabla de `pacientes` ahora está en el esquema `hospital_admin`. Crea un sinónimo para que los médicos puedan seguir escribiendo `SELECT * FROM pacientes` sin cambiar sus consultas.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE OR REPLACE SYNONYM pacientes FOR hospital_admin.pacientes;

-- Los médicos siguen usando:
SELECT * FROM pacientes;
-- Oracle internamente redirige a hospital_admin.pacientes
```

> 💡 Sin el sinónimo, cada consulta tendría que usar `hospital_admin.pacientes`. Con el sinónimo, el cambio es **transparente** para los usuarios finales.

</details>

---

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 11

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Practicar ejercicios | 📝 [Ejercicios de Vistas](ejercicios/ejercicios_vistas.md) |
| 3️⃣ | Completar el proyecto | 🏆 [Proyecto: El Panel de Control Empresarial](proyectos/proyecto_vistas.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 12: Indexación](../12-indexacion) |

---

<div align="center">

⬅️ [**🏆 Proyecto del Tema 10**](../10-subconsultas/proyectos/proyecto_subconsultas.md) · 🏠 [**Índice del Curso**](../README.md) · [**📝 Ejercicios del Tema 11 →**](ejercicios/ejercicios_vistas.md)

</div>
