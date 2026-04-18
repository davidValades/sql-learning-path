## 🏆 Proyecto Maestro DQL: Extracción de Inteligencia (Día 1 de Operaciones)

### 📘 El Concepto

Las tres empresas han superado el "Día Cero" (inserción de datos) y ya están operando. Hoy es el "Día 1". Los gerentes de cada sector están en sus oficinas tomando decisiones críticas y necesitan que tú, como Analista de Datos Principal, les extraigas información muy específica de las bases de datos.

Tendrás que combinar `SELECT`, `WHERE`, `AND`/`OR` y `ORDER BY` en un solo script para entregar los reportes exactos.

### 🏗️ Las Misiones de Extracción

Escribe las consultas SQL para resolver las siguientes tres peticiones:

**🛒 Misión 1: El Reporte de Inventario Crítico (E-commerce)**
El gerente de almacén necesita saber qué productos de la categoría "Electrónica" (ID: 1) representan un mayor valor para la empresa, pero que a la vez tengan un precio accesible para una campaña rápida.

- **Tu tarea:** Extrae el `nombre`, el `precio` y el `stock` de los productos que pertenezcan a la categoría `1` **Y** que tengan un precio estrictamente menor a `1000.00`. Ordena el resultado para que el producto con **mayor stock** aparezca el primero en la lista.

**🏥 Misión 2: Auditoría de Nóminas y Guardias (Hospital Central)**
Recursos Humanos está revisando los presupuestos de este mes. Necesitan un listado de médicos que cumplan con al menos una de dos condiciones de riesgo financiero o de saturación.

- **Tu tarea:** Extrae el `nombre_completo` y el `salario_base` de los médicos que tengan un salario mayor o igual a `3500.00` **O** que pertenezcan a la especialidad de "Medicina General" (ID: 300). Ordena la lista por salario de forma **descendente** (los que más cobran arriba).

**✈️ Misión 3: Renovación de Flota (Aerolínea)**
El equipo de ingeniería de vuelo está evaluando qué aviones de tamaño medio necesitan mantenimiento preventivo basado en su antigüedad.

- **Tu tarea:** Extrae el `modelo`, la `capacidad_pasajeros` y el `anio_fabricacion` de los aviones que tengan una capacidad de **180 pasajeros o más**. El reporte debe estar ordenado mostrando **los aviones más antiguos primero** (orden ascendente por año).

---

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tus 3 consultas escritas para auditar tu código</b></summary>

<b>Soluciones Exactas:</b>

```sql
-- MISIÓN 1: E-COMMERCE (Filtro AND + Orden Descendente)
SELECT nombre, precio, stock
FROM productos
WHERE id_categoria = 1 AND precio < 1000.00
ORDER BY stock DESC;

-- MISIÓN 2: HOSPITAL (Filtro OR + Orden Descendente)
SELECT nombre_completo, salario_base
FROM medicos
WHERE salario_base >= 3500.00 OR id_especialidad = 300
ORDER BY salario_base DESC;

-- MISIÓN 3: AEROLÍNEA (Filtro Simple + Orden Ascendente)
SELECT modelo, capacidad_pasajeros, anio_fabricacion
FROM aviones
WHERE capacidad_pasajeros >= 180
ORDER BY anio_fabricacion ASC;
-- Nota: Poner ASC es opcional porque es el valor por defecto, pero es una excelente práctica de lectura.
```

</details>
---

<div align="center">

⬅️ [**Volver al Temario**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Siguiente Proyecto →**](./proyecto_dql_medio.md)

</div>
