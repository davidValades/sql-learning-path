# 🌐 Tema 18: Ecosistema SQL

> **"En el trabajo real no existe un único motor."** Vas a preparar el mismo modelo para Oracle, PostgreSQL y SQL Server sin romper la lógica de negocio que arrastras desde temas anteriores.

## 📋 Índice

- [18.0 Estado heredado del modelo](#180-estado-heredado-del-modelo)
- [18.1 Portabilidad de tipos de datos](#181-portabilidad-de-tipos-de-datos)
- [18.2 Diferencias de sintaxis críticas](#182-diferencias-de-sintaxis-críticas)
- [18.3 Estrategia de migración segura](#183-estrategia-de-migración-segura)
- [18.4 Observabilidad y operación multi-motor](#184-observabilidad-y-operación-multi-motor)

---

## 18.0 Estado heredado del modelo

Modelo acumulado:

- Estructura relacional estable (Temas 04–11).
- Índices y transacciones (Temas 12–13).
- Seguridad, PL/SQL y optimización (Temas 14–17).

Ahora se prepara una evolución típica: **convivencia Oracle + PostgreSQL**.

---

## 18.1 Portabilidad de tipos de datos

| Oracle | PostgreSQL | SQL Server | Uso en el proyecto |
|--------|------------|-----------|--------------------|
| `NUMBER(10)` | `NUMERIC(10,0)` | `DECIMAL(10,0)` | IDs y contadores |
| `VARCHAR2(100)` | `VARCHAR(100)` | `VARCHAR(100)` | textos de negocio |
| `DATE` | `DATE` | `DATE` | fechas sin hora |
| `TIMESTAMP` | `TIMESTAMP` | `DATETIME2` | auditoría |

---

## 18.2 Diferencias de sintaxis críticas

```sql
-- Oracle
SELECT * FROM pedidos FETCH FIRST 10 ROWS ONLY;

-- PostgreSQL
SELECT * FROM pedidos LIMIT 10;

-- SQL Server
SELECT TOP 10 * FROM pedidos;
```

```sql
-- Oracle (nulos)
SELECT NVL(total, 0) FROM pedidos;

-- PostgreSQL / SQL Server
SELECT COALESCE(total, 0) FROM pedidos;
```

---

## 18.3 Estrategia de migración segura

Fase recomendada:

1. Congelar versión de esquema Oracle.
2. Generar DDL equivalente por motor.
3. Migrar catálogos maestros.
4. Migrar transaccional por ventanas.
5. Validar checksums y conteos.
6. Habilitar doble escritura temporal si aplica.

---

## 18.4 Observabilidad y operación multi-motor

Métricas mínimas comunes por motor:

- Latencia p95/p99 de consultas.
- Bloqueos y deadlocks.
- Crecimiento de tablas e índices.
- Fallos de conexión/autenticación.
- Éxito de backups y restores.

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 18

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Completar ejercicios | 🏋️ [Ir a Ejercicios →](./ejercicios/ejercicios_ecosistema_sql.md) |
| 3️⃣ | Resolver proyecto | 🏆 [Ir al Proyecto →](./proyectos/proyecto_ecosistema_sql.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 19: Casos Reales y Proyectos Finales](../19-casos-reales) |

---

<div align="center">

⬅️ [**Tema 17: Optimización de Consultas**](../17-optimizacion) · 🏠 [**Índice del Curso**](../README.md) · [**🏋️ Ejercicios →**](./ejercicios/ejercicios_ecosistema_sql.md)

</div>
