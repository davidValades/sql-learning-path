## 🏆 Proyecto Maestro DQL (Nivel Medio): Reportes Gerenciales Complejos

### 📘 El Concepto

Las extracciones básicas ya están dominadas. Sin embargo, en el mundo real, los gerentes hacen preguntas retorcidas que requieren mezclar múltiples reglas de negocio en una sola consulta.

En este nivel, deberás demostrar tu dominio de los **paréntesis lógicos** (mezclar `AND` y `OR` sin romper la consulta), las **columnas calculadas** (matemáticas en vuelo) y la **concatenación** de texto.

### 🏗️ Las Misiones de Nivel Medio

**🏥 Misión 1: La Trampa Lógica (Hospital Central)**
La junta directiva quiere premiar a los especialistas de alto nivel. Necesitan un listado de los médicos, pero con unas reglas muy estrictas:

- **Tu tarea:** Extrae el `nombre_completo` y el `salario_base` de los médicos que pertenezcan a "Traumatología" (ID: 100) **O** a "Neurología" (ID: 200)... pero de esos, **SOLO** los que tengan un salario estrictamente superior a `3000.00`.
- _Pista del DBA:_ Si mezclas `AND` y `OR` sin usar paréntesis `( )` para agrupar las condiciones, el motor de base de datos se confundirá y te dará resultados falsos.

**🛒 Misión 2: Proyección de Impuestos (E-commerce)**
El departamento de contabilidad necesita ver cómo quedarían los precios de la categoría "Hogar" (ID: 2) si se les aplica un impuesto del 21% (IVA).

- **Tu tarea:** Extrae el `nombre` del producto y crea una columna calculada llamada `"Precio con IVA"`. Esta columna debe ser el resultado de multiplicar el `precio` por `1.21`. Filtra para que solo salgan los productos de la categoría `2`.

**✈️ Misión 3: El Panel de Salidas (Aerolínea)**
Los operarios de las pantallas del aeropuerto no entienden de bases de datos, necesitan leer frases completas en los monitores.

- **Tu tarea:** Utilizando la tabla `rutas`, concatena las columnas para generar una única columna llamada `"Panel Informativo"` que muestre frases con este formato exacto:
  `La ruta MADLHR tiene una distancia de 1200 km.`
  Ordena este panel para que los vuelos más largos aparezcan primero.

---

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tus 3 consultas escritas para auditar tu código</b></summary>

<b>Soluciones Exactas:</b>

```sql
-- MISIÓN 1: HOSPITAL (Uso vital de paréntesis lógicos)
SELECT nombre_completo, salario_base
FROM medicos
WHERE (id_especialidad = 100 OR id_especialidad = 200) AND salario_base > 3000.00;

-- MISIÓN 2: E-COMMERCE (Columnas calculadas)
SELECT nombre, (precio * 1.21) AS "Precio con IVA"
FROM productos
WHERE id_categoria = 2;

-- MISIÓN 3: AEROLÍNEA (Concatenación y Ordenamiento)
SELECT 'La ruta ' || id_ruta || ' tiene una distancia de ' || distancia_km || ' km.' AS "Panel Informativo"
FROM rutas
ORDER BY distancia_km DESC;
```

</details>
---

<div align="center">

### ✅ ¡Has completado los proyectos del Tema 05!

⬅️ [**Proyecto Anterior**](./proyecto_dql_basico.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 06: Operadores Lógicos y Filtros →**](../../06-operadores)

</div>
