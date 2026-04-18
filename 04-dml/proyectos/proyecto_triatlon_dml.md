## 🏆 Proyecto Maestro DML: El Día Cero (Especificaciones Técnicas)

### 📘 El Concepto

Hoy es el "Día Cero". El E-commerce, el Hospital y la Aerolínea salen a producción. Como DBA, tu misión es ejecutar la carga inicial de datos siguiendo estrictamente las especificaciones, pero antes deberás solucionar un problema estructural que ha detectado el equipo de QA.

### 🏗️ Las Misiones de Lanzamiento

Crea un único script SQL que ejecute las siguientes instrucciones en orden exacto:

**🛠️ Misión 0: Ajustes Estructurales Previos (DDL)**

- El equipo de finanzas nos avisa de que vamos a vender productos de más de 999.99€.
- **Tu tarea:** Escribe un `ALTER TABLE` para modificar la columna `precio` de la tabla `productos`. Cámbiala a `NUMBER(7,2)` para soportar precios de hasta decenas de miles de euros.

**🛒 Misión 1: E-commerce**

1. **INSERT** - Tabla `categorias`.
   - ID: `1`, Nombre: `'Electrónica'`
   - ID: `2`, Nombre: `'Hogar'`
2. **INSERT** - Tabla `productos`.
   - ID: `10`, Nombre: `'Laptop Pro'`, Precio: `1200.00`, ID Categoría: `1`, Stock: `50`
   - ID: `11`, Nombre: `'Ratón Inalámbrico'`, Precio: `25.50`, ID Categoría: `1`, Stock: `200`
   - ID: `12`, Nombre: `'Sofá de Cuero'`, Precio: `450.00`, ID Categoría: `2`, Stock: `10`
   - ID: `13`, Nombre: `'Lámpara LED'`, Precio: `45.00`, ID Categoría: `2`, Stock: `30`
3. **UPDATE** - Ajuste de inflación:
   - Súmale **`20`** al precio de TODOS los productos que pertenezcan a la categoría `1` (Electrónica).

**🏥 Misión 2: Hospital Central**

1. **INSERT** - Tabla `especialidades`.
   - ID: `100`, Nombre: `'Traumatología'`
   - ID: `200`, Nombre: `'Neurología'`
   - ID: `300`, Nombre: `'Medicina General'`
2. **INSERT** - Tabla `medicos`.
   - ID: `1`, Nombre: `'Dr. John Smith'`, ID Especialidad: `100`, Salario: `3000.00`
   - ID: `2`, Nombre: `'Dra. Sarah Adams'`, ID Especialidad: `200`, Salario: `4000.00`
   - ID: `3`, Nombre: `'Dr. Frank Abagnale'`, ID Especialidad: `300`, Salario: `2500.00`
3. **INSERT** - Tabla `pacientes`. _(OJO: Usa estrictamente `TO_DATE('fecha', 'YYYY-MM-DD')`)_
   - ID: `1`, DNI: `'11111111A'`, Nombre: `'Laura Martinez'`, Fecha Nac.: `'1990-01-01'`, Teléfono: `'555-0001'`
   - ID: `2`, DNI: `'22222222B'`, Nombre: `'Carlos Gomez'`, Fecha Nac.: `'1985-05-05'`, Teléfono: `'555-0002'`
4. **DELETE** - Despido disciplinario:
   - Borra del sistema al médico con ID `3`.

**✈️ Misión 3: Aerolínea**

1. **INSERT** - Tabla `aviones`.
   - ID: `10`, Modelo: `'Airbus A320'`, Capacidad: `150`, Año Fabricación: `2015`
   - ID: `20`, Modelo: `'Boeing 737'`, Capacidad: `180`, Año Fabricación: `2018`
   - ID: `30`, Modelo: `'Airbus A350'`, Capacidad: `300`, Año Fabricación: `2022`
2. **INSERT** - Tabla `rutas`. _(Verifica el nombre de la columna de distancia en tu tabla)_
   - ID: `'MADLHR'`, Origen: `'MAD'`, Destino: `'LHR'`, Distancia: `1200`
   - ID: `'JFKLAX'`, Origen: `'JFK'`, Destino: `'LAX'`, Distancia: `4000`
3. **TRANSACTION** - Guardado:
   - Escribe el `COMMIT;` final.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu script completo para auditar tu código</b></summary>

<b>Solución Exacta:</b>

```sql
-- MISIÓN 0: PREPARACIÓN
ALTER TABLE productos MODIFY (precio NUMBER(7,2));

-- MISIÓN 1: E-COMMERCE
INSERT INTO categorias (id_categoria, nombre_categoria) VALUES (1, 'Electrónica');
INSERT INTO categorias (id_categoria, nombre_categoria) VALUES (2, 'Hogar');

INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (10, 'Laptop Pro', 1200.00, 1, 50);
INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (11, 'Ratón Inalámbrico', 25.50, 1, 200);
INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (12, 'Sofá de Cuero', 450.00, 2, 10);
INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (13, 'Lámpara LED', 45.00, 2, 30);

UPDATE productos SET precio = precio + 20 WHERE id_categoria = 1;

-- MISIÓN 2: HOSPITAL
INSERT INTO especialidades (id_especialidad, nombre_especialidad) VALUES (100, 'Traumatología');
INSERT INTO especialidades (id_especialidad, nombre_especialidad) VALUES (200, 'Neurología');
INSERT INTO especialidades (id_especialidad, nombre_especialidad) VALUES (300, 'Medicina General');

INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base) VALUES (1, 'Dr. John Smith', 100, 3000.00);
INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base) VALUES (2, 'Dra. Sarah Adams', 200, 4000.00);
INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base) VALUES (3, 'Dr. Frank Abagnale', 300, 2500.00);

INSERT INTO pacientes (id_paciente, dni, nombre, fecha_nacimiento, telefono) VALUES (1, '11111111A', 'Laura Martinez', TO_DATE('1990-01-01', 'YYYY-MM-DD'), '555-0001');
INSERT INTO pacientes (id_paciente, dni, nombre, fecha_nacimiento, telefono) VALUES (2, '22222222B', 'Carlos Gomez', TO_DATE('1985-05-05', 'YYYY-MM-DD'), '555-0002');

DELETE FROM medicos WHERE id_medico = 3;

-- MISIÓN 3: AEROLÍNEA
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros, anio_fabricacion) VALUES (10, 'Airbus A320', 150, 2015);
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros, anio_fabricacion) VALUES (20, 'Boeing 737', 180, 2018);
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros, anio_fabricacion) VALUES (30, 'Airbus A350', 300, 2022);

INSERT INTO rutas (id_ruta, aeropuerto_origen, aeropuerto_destino, distancia_km) VALUES ('MADLHR', 'MAD', 'LHR', 1200);
INSERT INTO rutas (id_ruta, aeropuerto_origen, aeropuerto_destino, distancia_km) VALUES ('JFKLAX', 'JFK', 'LAX', 4000);

COMMIT;
```

</details>
---

<div align="center">

⬅️ [**Volver al Temario**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Siguiente Proyecto →**](./proyecto_DML_parte2.md)

</div>
