# 🏎️ Tema 17: Optimización de Consultas

> **"Optimizar no es adivinar: es medir, diagnosticar y corregir."** En este tema trabajas como harías en producción: identificas cuellos de botella, ajustas índices y validas impacto.

## 📋 Índice

- [17.0 Estado heredado del modelo](#170-estado-heredado-del-modelo)
- [17.1 EXPLAIN PLAN y lectura de ejecución](#171-explain-plan-y-lectura-de-ejecución)
- [17.2 Índices estratégicos sobre carga real](#172-índices-estratégicos-sobre-carga-real)
- [17.3 Reescritura de consultas costosas](#173-reescritura-de-consultas-costosas)
- [17.4 Estadísticas del optimizador](#174-estadísticas-del-optimizador)
- [17.5 Checklist de tuning en trabajo real](#175-checklist-de-tuning-en-trabajo-real)

---

## 17.0 Estado heredado del modelo

Además de tablas de negocio, llegamos con:

- Vistas (Tema 11).
- Índices base (Tema 12).
- Auditoría PL/SQL en `auditoria_stock` (Tema 16).

La optimización se enfoca en consultas operativas y analíticas sobre ese estado acumulado.

---

## 17.1 EXPLAIN PLAN y lectura de ejecución

```sql
EXPLAIN PLAN FOR
SELECT p.id_pedido, c.nombre, pr.nombre, p.total
FROM pedidos p
JOIN clientes c ON c.id_cliente = p.id_cliente
JOIN productos pr ON pr.id_producto = p.id_producto
WHERE p.fecha_pedido >= ADD_MONTHS(TRUNC(SYSDATE), -3)
  AND p.total >= 100;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

Busca principalmente:

- `TABLE ACCESS FULL` en tablas grandes.
- `HASH JOIN` vs `NESTED LOOPS` según cardinalidad.
- Filtros aplicados tarde.

---

## 17.2 Índices estratégicos sobre carga real

```sql
-- Reportes por cliente y fecha
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos(id_cliente, fecha_pedido);

-- Auditoría por producto y fecha
CREATE INDEX idx_aud_stock_producto_fecha ON auditoria_stock(id_producto, fecha_cambio);
```

Regla práctica: indexar patrones repetidos de `WHERE`, `JOIN`, `ORDER BY`, no columnas “por si acaso”.

---

## 17.3 Reescritura de consultas costosas

```sql
-- Versión poco eficiente: función sobre columna indexada
SELECT *
FROM pedidos
WHERE TRUNC(fecha_pedido) = TRUNC(SYSDATE);

-- Versión optimizada: rango de fechas
SELECT *
FROM pedidos
WHERE fecha_pedido >= TRUNC(SYSDATE)
  AND fecha_pedido < TRUNC(SYSDATE) + 1;
```

---

## 17.4 Estadísticas del optimizador

```sql
BEGIN
  DBMS_STATS.GATHER_TABLE_STATS(USER, 'PEDIDOS');
  DBMS_STATS.GATHER_TABLE_STATS(USER, 'AUDITORIA_STOCK');
END;
/
```

Sin estadísticas actualizadas, el plan elegido puede ser incorrecto aunque el SQL esté bien escrito.

---

## 17.5 Checklist de tuning en trabajo real

1. Medir consulta lenta (tiempo + plan).
2. Confirmar cardinalidad real.
3. Revisar selectividad de filtros.
4. Aplicar índice o reescritura mínima.
5. Volver a medir y documentar impacto.

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 17

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Completar ejercicios | 🏋️ [Ir a Ejercicios →](./ejercicios/ejercicios_optimizacion.md) |
| 3️⃣ | Resolver proyecto | 🏆 [Ir al Proyecto →](./proyectos/proyecto_optimizacion.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 18: Ecosistema SQL](../18-ecosistema-sql) |

---

<div align="center">

⬅️ [**Tema 16: PL/SQL**](../16-plsql) · 🏠 [**Índice del Curso**](../README.md) · [**🏋️ Ejercicios →**](./ejercicios/ejercicios_optimizacion.md)

</div>
