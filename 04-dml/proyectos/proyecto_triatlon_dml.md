## 🏆 Proyecto Maestro DML: El Día Cero (Lanzamiento Global)

### 📘 El Concepto

Hoy es el "Día Cero". El E-commerce, el Hospital y la Aerolínea salen a producción mundial mañana por la mañana. Las tablas están vacías. Eres el Director de Datos y tu misión es dejar el sistema completamente operativo, poblado y corregido antes de que lleguen los clientes.

### 🏗️ Las 3 Misiones de Lanzamiento

Crea un único script SQL gigante que ejecute todas estas tareas en orden. Tú inventas los datos (nombres, precios, DNIs...), pero debes cumplir estrictamente estas reglas de negocio:

🛒 Misión 1: E-commerce (Creación y Ajuste de Inflación)

INSERT: Crea 2 Categorías (ej. 'Electrónica' y 'Hogar').

INSERT: Crea 4 Productos (2 para cada categoría). Ponles el precio y stock que quieras.

UPDATE: ¡Crisis de última hora! Los proveedores de tu primera categoría (ej. 'Electrónica') han subido los costes. Haz un UPDATE masivo para sumarle 20€ al precio de TODOS los productos que pertenezcan a esa categoría (Pista: Usa WHERE id_categoria = ..., no el id del producto).

🏥 Misión 2: Hospital Central (Contratación y Despido)

INSERT: Crea 3 Especialidades médicas.

INSERT: Contrata a 3 Médicos (asígnales las especialidades que creaste).

INSERT: Registra a 2 Pacientes nuevos.

DELETE: ¡Escándalo! Has descubierto que uno de tus médicos falsificó su título universitario. Bórralo de la base de datos inmediatamente usando su id_medico.

✈️ Misión 3: Aerolínea (Operativa y Guardado)

INSERT: Compra y registra 3 Aviones nuevos en tu flota.

INSERT: Abre 2 Rutas comerciales (recuerda que el id_ruta son 6 letras, ej. 'MADPAR').

TRANSACTION: Has trabajado muy duro. Escribe el comando final y definitivo que asegure que todos estos Inserts, Updates y Deletes se guarden en el disco duro del servidor para siempre.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu script completo para compararlo con un ejemplo de solución</b></summary>

<b>Respuesta de Ejemplo del Profesor (Tus datos serán diferentes, pero la estructura debe ser esta):</b>

```sql
-- MISIÓN 1: E-COMMERCE
INSERT INTO categorias (id_categoria, nombre_categoria) VALUES (1, 'Electrónica');
INSERT INTO categorias (id_categoria, nombre_categoria) VALUES (2, 'Hogar');

INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (10, 'Laptop', 1000, 1, 50);
INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (11, 'Ratón', 20, 1, 200);
INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (12, 'Sofá', 400, 2, 10);
INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock) VALUES (13, 'Lámpara', 45, 2, 30);

UPDATE productos SET precio = precio + 20 WHERE id_categoria = 1;

-- MISIÓN 2: HOSPITAL
INSERT INTO especialidades (id_especialidad, nombre_especialidad) VALUES (100, 'Traumatología');
INSERT INTO especialidades (id_especialidad, nombre_especialidad) VALUES (200, 'Neurología');
INSERT INTO especialidades (id_especialidad, nombre_especialidad) VALUES (300, 'Dermatología');

INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base) VALUES (1, 'Dr. Smith', 100, 3000);
INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base) VALUES (2, 'Dra. Adams', 200, 4000);
INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base) VALUES (3, 'Dr. Fake', 300, 2500);

INSERT INTO pacientes (id_paciente, dni, nombre, fecha_nacimiento) VALUES (1, '11111111A', 'Laura', TO_DATE('1990-01-01', 'YYYY-MM-DD'));
INSERT INTO pacientes (id_paciente, dni, nombre, fecha_nacimiento) VALUES (2, '22222222B', 'Carlos', TO_DATE('1985-05-05', 'YYYY-MM-DD'));

DELETE FROM medicos WHERE id_medico = 3;

-- MISIÓN 3: AEROLÍNEA
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros) VALUES (10, 'A320', 150);
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros) VALUES (20, 'B737', 180);
INSERT INTO aviones (id_avion, modelo, capacidad_pasajeros) VALUES (30, 'A350', 300);

INSERT INTO rutas (id_ruta, aeropuerto_origen, aeropuerto_destino, distancia_km) VALUES ('MADLHR', 'MAD', 'LHR', 1200);
INSERT INTO rutas (id_ruta, aeropuerto_origen, aeropuerto_destino, distancia_km) VALUES ('JFKLAX', 'JFK', 'LAX', 4000);

COMMIT;
```

</details>

---
