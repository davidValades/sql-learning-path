# 🏆 Proyecto — Tema 15: Reporte Analítico Evolutivo

> 📋 **Contexto:** ya tienes una base de datos operativa construida durante los temas anteriores. Tu misión es añadir una capa analítica sin romper el modelo transaccional y dejando artefactos reutilizables para PL/SQL (Tema 16).

---

## 🎯 Objetivo

Construir un mini sistema de analítica con:
1. Segmentación de clientes.
2. KPIs mensuales persistidos.
3. Alertas de negocio.

Todo sobre el **estado acumulado** de tablas (`clientes`, `productos`, `pedidos`, etc.).

---

## Entregable 1 — Tabla de KPIs mensuales

```sql
CREATE TABLE kpi_ventas_mensuales (
  mes               DATE PRIMARY KEY,
  total_pedidos     NUMBER NOT NULL,
  facturacion_total NUMBER(12,2) NOT NULL,
  ticket_medio      NUMBER(12,2) NOT NULL,
  clientes_activos  NUMBER NOT NULL,
  actualizado_en    DATE DEFAULT SYSDATE NOT NULL
);

MERGE INTO kpi_ventas_mensuales k
USING (
  SELECT TRUNC(fecha_pedido, 'MM') AS mes,
         COUNT(*) AS total_pedidos,
         SUM(total) AS facturacion_total,
         ROUND(AVG(total), 2) AS ticket_medio,
         COUNT(DISTINCT id_cliente) AS clientes_activos
  FROM pedidos
  GROUP BY TRUNC(fecha_pedido, 'MM')
) s
ON (k.mes = s.mes)
WHEN MATCHED THEN UPDATE SET
  k.total_pedidos = s.total_pedidos,
  k.facturacion_total = s.facturacion_total,
  k.ticket_medio = s.ticket_medio,
  k.clientes_activos = s.clientes_activos,
  k.actualizado_en = SYSDATE
WHEN NOT MATCHED THEN INSERT (
  mes, total_pedidos, facturacion_total, ticket_medio, clientes_activos
) VALUES (
  s.mes, s.total_pedidos, s.facturacion_total, s.ticket_medio, s.clientes_activos
);
```

---

## Entregable 2 — Alertas de negocio (tabla incremental)

```sql
CREATE TABLE alertas_negocio (
  id_alerta      NUMBER PRIMARY KEY,
  tipo_alerta    VARCHAR2(50) NOT NULL,
  referencia     VARCHAR2(100) NOT NULL,
  detalle        VARCHAR2(400),
  fecha_alerta   DATE DEFAULT SYSDATE NOT NULL,
  estado         VARCHAR2(20) DEFAULT 'ABIERTA' CHECK (estado IN ('ABIERTA','RESUELTA'))
);
```

Casos mínimos a cargar:
- producto sin ventas en los últimos 90 días,
- cliente de alto valor con caída de gasto > 40% mes a mes.

---

## Entregable 3 — Reglas de continuidad

1. `clientes.segmento_cliente` pasa a ser columna oficial del modelo.
2. `kpi_ventas_mensuales` y `alertas_negocio` se reutilizarán en Tema 16 (procedimientos automáticos).
3. Cualquier nuevo ejercicio debe respetar estas estructuras ya creadas.

---

<div align="center">

⬅️ [**🏋️ Ejercicios del Tema 15**](../ejercicios/ejercicios_sql_avanzado.md) · 🏠 [**Tema 15**](../README.md) · [**Tema 16: PL/SQL →**](../../16-plsql/README.md)

</div>
