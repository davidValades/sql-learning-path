## 🏆 Mini-Proyecto DDL 2: Sistema de Gestión Hospitalaria

### 📘 El Concepto

Para este segundo proyecto vamos a diseñar la arquitectura principal de una clínica médica. Este esquema es fantástico para el futuro porque nos permitirá responder preguntas complejas más adelante, como: _"¿Qué médico ha atendido a más pacientes este mes?"_ o _"¿Cuál es la especialidad con más demanda?"_.

### 🏗️ La Arquitectura (El Reto)

Actúa como el Arquitecto de Datos del "Hospital Central". Tu misión es escribir un script SQL en sintaxis Oracle que cree la siguiente estructura. Lee atentamente las reglas de negocio:

**1. Tabla `especialidades`**:

- `id_especialidad`: Número entero de hasta 3 cifras. Clave Primaria.
- `nombre_especialidad`: Texto de hasta 50 caracteres. No puede estar vacío y debe ser único (no queremos dos especialidades llamadas "Cardiología").

**2. Tabla `medicos`**:

- `id_medico`: Número entero de hasta 4 cifras. Clave Primaria.
- `nombre_completo`: Texto de hasta 100 caracteres. No nulo.
- `id_especialidad`: Número entero. **Clave Foránea** que conecta con la tabla `especialidades`.
- `salario_base`: Número de hasta 7 cifras con 2 decimales. Debe ser mayor a 0 (Restricción `CHECK`).

**3. Tabla `pacientes`**:

- `id_paciente`: Número entero de hasta 6 cifras. Clave Primaria.
- `dni`: Cadena de texto de exactamente 9 caracteres. Único y no nulo.
- `nombre`: Texto de hasta 100 caracteres. No nulo.
- `fecha_nacimiento`: Tipo de dato para fechas.

**4. Tabla `citas` (La tabla intermedia)**:

- `id_cita`: Número entero de hasta 8 cifras. Clave Primaria.
- `id_paciente`: Clave Foránea referenciando a `pacientes`.
- `id_medico`: Clave Foránea referenciando a `medicos`.
- `fecha_hora_cita`: Debe registrar el día y la **hora exacta** con fracciones de segundo (recuerda la lección de Tipos de Datos).
- `estado`: Solo un carácter. Por defecto debe ser 'P' (Pendiente), y solo puede aceptar 'P' (Pendiente), 'C' (Completada) o 'X' (Cancelada).

**5. El Ajuste (ALTER)**:

- El hospital ha decidido que necesita registrar el número de teléfono de los pacientes. Añade una columna `telefono` (texto variable de 15 caracteres) a la tabla `pacientes`.

**6. La Limpieza (DROP)**:

- Borra una tabla obsoleta llamada `registro_visitas_old` que el equipo anterior dejó abandonada en el servidor.
