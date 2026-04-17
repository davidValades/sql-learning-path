## 🏆 Mini-Proyecto DDL 3: Gestión de Aerolíneas

### 📘 El Concepto

Las bases de datos de aviación son el mejor ejemplo de relaciones críticas. Un pequeño error aquí puede significar billetes duplicados o vuelos sin avión asignado. Este esquema nos preparará para cuando estudiemos subconsultas (ej. _"¿Qué rutas operan aviones con más de 200 asientos?"_).

### 🏗️ La Arquitectura (El Reto)

Eres el DBA principal del aeropuerto. Diseña el script SQL (Oracle) para construir esta estructura:

**1. Tabla `aviones`**:

- `id_avion`: Número entero de hasta 4 cifras. Clave Primaria.
- `modelo`: Texto de hasta 50 caracteres. No nulo.
- `capacidad_pasajeros`: Número entero de hasta 3 cifras. Debe ser mayor a 0 y menor o igual a 853 (el máximo del Airbus A380). ¡Usa un `CHECK`!

**2. Tabla `rutas`**:

- `id_ruta`: Texto fijo de exactamente 6 caracteres (ej. 'MADLHR'). Clave Primaria.
- `aeropuerto_origen`: Texto de exactamente 3 caracteres (código IATA, ej. 'MAD'). No nulo.
- `aeropuerto_destino`: Texto de exactamente 3 caracteres. No nulo.
- `distancia_km`: Número de hasta 5 cifras (sin decimales). Mayor a 0.

**3. Tabla `vuelos` (La tabla de operaciones)**:

- `id_vuelo`: Número de hasta 6 cifras. Clave Primaria.
- `id_ruta`: Clave Foránea referenciando a `rutas`.
- `id_avion`: Clave Foránea referenciando a `aviones`.
- `fecha_salida`: Debe registrar fecha y hora exacta con precisión de milisegundos.
- `puerta_embarque`: Texto de hasta 5 caracteres (ej. 'T4-55').

**4. El Ajuste (ALTER)**:

- Por nuevas normativas, necesitamos saber el año de fabricación del avión. Añade una columna `anio_fabricacion` (número de 4 cifras) a la tabla `aviones`.

**5. El Vaciado Express (TRUNCATE)**:

- Tienes una tabla llamada `vuelos_prueba_sistema` que utilizaste ayer para probar la carga de datos. Quieres vaciarla de golpe para las pruebas de hoy, pero sin destruir la tabla. Escribe el comando exacto.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
CREATE TABLE aviones(
    id_avion NUMBER(4) PRIMARY KEY,
    modelo VARCHAR2(50) NOT NULL,
    capacidad_pasajeros NUMBER(3) CHECK (capacidad_pasajeros > 0 AND capacidad_pasajeros <= 853)
);

CREATE TABLE rutas(
    id_ruta CHAR(6) PRIMARY KEY,
    aeropuerto_origen CHAR(3) NOT NULL,
    aeropuerto_destino CHAR(3) NOT NULL,
    distancia_km NUMBER(5) CHECK (distancia_km > 0)
);

CREATE TABLE vuelos(
    id_vuelo NUMBER(6) PRIMARY KEY,
    id_ruta CHAR(6) REFERENCES rutas(id_ruta),
    id_avion NUMBER(4) REFERENCES aviones(id_avion),
    fecha_salida TIMESTAMP,
    puerta_embarque VARCHAR2(5)
);

ALTER TABLE aviones ADD (
    anio_fabricacion NUMBER(4)
);

TRUNCATE TABLE vuelos_prueba_sistema;
```

</details>
