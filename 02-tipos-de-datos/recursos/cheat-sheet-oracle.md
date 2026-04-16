# 📑 Cheat Sheet: Tipos de Datos en Oracle SQL

Esta guía rápida resume los tipos de datos más utilizados en Oracle, los cuales usaremos a lo largo de este bloque.

## 1. Numéricos

- **`NUMBER(p, s)`**: El tipo universal.
  - `p` (precisión): total de dígitos.
  - `s` (escala): dígitos decimales.
- **`INTEGER`**: Alias de `NUMBER(38)`. Se usa para números enteros sin decimales.

## 2. Cadenas de Texto

- **`VARCHAR2(n)`**: Cadena de longitud variable. **Es el estándar en Oracle**. Ocupa solo el espacio de lo que escribes.
- **`CHAR(n)`**: Longitud fija. Si defines `CHAR(10)` y escribes "HOLA", Oracle rellena con 6 espacios en blanco. Útil para códigos de longitud fija (ej: ISO de países 'ES', 'MX').

## 3. Fecha y Hora

- **`DATE`**: Guarda fecha y hora (segundos incluidos).
- **`TIMESTAMP`**: Más preciso que DATE (incluye fracciones de segundo).

## 4. Gran Tamaño (LOBs)

- **`CLOB`**: Para textos larguísimos (como un post de un blog o un JSON).
- **`BLOB`**: Para datos binarios (imágenes, PDFs).
