# 🏋️ Ejercicios — Tema 01: Fundamentos de Bases de Datos

---

## Ejercicio 1 — Identificar conceptos base

**Enunciado:** Clasifica cada elemento como **BD**, **SGBD** o **SGBDR**:
- Oracle Database
- Un archivo `.csv`
- PostgreSQL
- Un conjunto de tablas de clientes y pedidos

<details>
<summary>👉 Ver solución</summary>

- Oracle Database → **SGBDR**
- Archivo `.csv` aislado → **no es SGBD/SGBDR** (es un archivo de datos)
- PostgreSQL → **SGBDR**
- Conjunto de tablas relacionadas → **BD relacional**

Salida esperada (clasificación):
| Elemento | Tipo |
|----------|------|
| Oracle Database | SGBDR |
| Archivo `.csv` | Archivo de datos (no SGBD) |
| PostgreSQL | SGBDR |
| Conjunto de tablas relacionadas | BD relacional |

</details>

---

## Ejercicio 2 — Entidad, fila y columna

**Enunciado:** Para una tabla `empleados(id_empleado, nombre, salario, fecha_ingreso)`, indica:
1. Cuántas columnas hay.
2. Qué representa una fila.
3. Qué columna elegirías como candidata a clave primaria.

<details>
<summary>👉 Ver solución</summary>

1. Hay **4 columnas**.
2. Una fila representa **un empleado**.
3. `id_empleado` es la mejor candidata a **PRIMARY KEY**.

Salida esperada:
| Pregunta | Respuesta |
|----------|-----------|
| ¿Cuántas columnas? | 4 (id_empleado, nombre, salario, fecha_ingreso) |
| ¿Qué representa una fila? | Un empleado individual |
| ¿Candidata a PK? | `id_empleado` (identificador único numérico) |

</details>

---

## Ejercicio 3 — Claves y restricciones

**Enunciado:** Diseña las claves para un sistema de pedidos con tablas:
- `clientes(id_cliente, nombre, email)`
- `pedidos(id_pedido, id_cliente, total)`

Indica PK y FK.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE TABLE clientes (
  id_cliente NUMBER PRIMARY KEY,
  nombre     VARCHAR2(80) NOT NULL,
  email      VARCHAR2(120) UNIQUE
);

CREATE TABLE pedidos (
  id_pedido  NUMBER PRIMARY KEY,
  id_cliente NUMBER NOT NULL,
  total      NUMBER(10,2) CHECK (total >= 0),
  CONSTRAINT fk_pedidos_clientes
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente)
);
```

Salida esperada al ejecutar en Oracle:
```
Table created.
Table created.
```

Estructura resultante:
| Tabla | Columna | Tipo de clave/restricción |
|-------|---------|--------------------------|
| clientes | id_cliente | PRIMARY KEY |
| clientes | email | UNIQUE |
| pedidos | id_pedido | PRIMARY KEY |
| pedidos | id_cliente | FOREIGN KEY → clientes(id_cliente) |
| pedidos | total | CHECK (total >= 0) |

</details>

---

<div align="center">

⬅️ [**Tema 01: Fundamentos de Bases de Datos**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 01 →**](../proyectos/proyecto_fundamentos_bd.md)

</div>
