# 🏆 Proyecto — Tema 16: Automatización Operativa con PL/SQL

> 📋 **Contexto:** Ya tienes una base de datos operativa con tablas analíticas del Tema 15 (`kpi_ventas_mensuales`, `alertas_negocio`). Tu misión como DBA es automatizar las operaciones críticas usando PL/SQL: procedimientos que refrescan KPIs, generan alertas y registran errores. Todo lo que hasta ahora hacías "a mano" con consultas SQL, ahora lo empaquetas en objetos reutilizables.

---

## 🎯 Objetivo

Construir un sistema de automatización operativa completo con:
1. Procedimiento para actualizar KPIs mensuales.
2. Procedimiento para generar alertas de negocio.
3. Sistema de logging de errores.
4. Función de utilidad para formatear reportes.

Todo sobre el **estado acumulado** de tablas (`clientes`, `productos`, `pedidos`, `kpi_ventas_mensuales`, `alertas_negocio`, etc.).

---

## Entregable 1 — Tabla de log y procedimiento de actualización de KPIs

### Paso 1: Tabla de logging

Crea una tabla `log_procesos_plsql` para registrar la ejecución de todos los procedimientos automatizados:

```
Columnas:
- id_log: NUMBER autogenerado (PK)
- nombre_proceso: VARCHAR2(100) NOT NULL
- estado: VARCHAR2(20) CHECK ('OK', 'ERROR') 
- mensaje: VARCHAR2(500)
- filas_afectadas: NUMBER
- fecha_ejecucion: DATE DEFAULT SYSDATE
```

### Paso 2: Procedimiento `sp_refrescar_kpis_mensuales`

Crea un procedimiento que:
1. Recalcule los KPIs por mes usando `MERGE` sobre `kpi_ventas_mensuales`.
2. Registre la ejecución exitosa en `log_procesos_plsql`.
3. Haga `COMMIT` al finalizar.
4. En caso de error, haga `ROLLBACK` y registre el error en el log.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu código completo para auditarlo</b></summary>

<b>Solución:</b>

```sql
-- Paso 1: Tabla de logging
CREATE TABLE log_procesos_plsql (
  id_log           NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  nombre_proceso   VARCHAR2(100) NOT NULL,
  estado           VARCHAR2(20) DEFAULT 'OK' CHECK (estado IN ('OK', 'ERROR')),
  mensaje          VARCHAR2(500),
  filas_afectadas  NUMBER,
  fecha_ejecucion  DATE DEFAULT SYSDATE
);

-- Paso 2: Procedimiento
CREATE OR REPLACE PROCEDURE sp_refrescar_kpis_mensuales AS
  v_filas NUMBER;
BEGIN
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
    k.total_pedidos     = s.total_pedidos,
    k.facturacion_total = s.facturacion_total,
    k.ticket_medio      = s.ticket_medio,
    k.clientes_activos  = s.clientes_activos,
    k.actualizado_en    = SYSDATE
  WHEN NOT MATCHED THEN INSERT (
    mes, total_pedidos, facturacion_total, ticket_medio, clientes_activos
  ) VALUES (
    s.mes, s.total_pedidos, s.facturacion_total, s.ticket_medio, s.clientes_activos
  );

  v_filas := SQL%ROWCOUNT;

  INSERT INTO log_procesos_plsql (nombre_proceso, estado, mensaje, filas_afectadas)
  VALUES ('sp_refrescar_kpis_mensuales', 'OK', 'KPIs mensuales actualizados correctamente', v_filas);

  COMMIT;
  DBMS_OUTPUT.PUT_LINE('KPIs actualizados: ' || v_filas || ' meses procesados');
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    INSERT INTO log_procesos_plsql (nombre_proceso, estado, mensaje)
    VALUES ('sp_refrescar_kpis_mensuales', 'ERROR', SQLERRM);
    COMMIT;  -- Commit del log de error
    DBMS_OUTPUT.PUT_LINE('Error al refrescar KPIs: ' || SQLERRM);
END sp_refrescar_kpis_mensuales;
/

-- Ejecución:
EXECUTE sp_refrescar_kpis_mensuales;
```

Salida esperada:
```
KPIs actualizados: 2 meses procesados
```

Verificación:
```sql
SELECT * FROM kpi_ventas_mensuales ORDER BY mes;
```
| mes | total_pedidos | facturacion_total | ticket_medio | clientes_activos |
|-----|---------------|-------------------|--------------|------------------|
| 2025-01-01 | 4 | 1537.45 | 384.36 | 3 |
| 2025-02-01 | 3 | 604.00 | 201.33 | 2 |

```sql
SELECT nombre_proceso, estado, mensaje, filas_afectadas FROM log_procesos_plsql;
```
| nombre_proceso | estado | mensaje | filas_afectadas |
|---------------|--------|---------|-----------------|
| sp_refrescar_kpis_mensuales | OK | KPIs mensuales actualizados correctamente | 2 |

</details>

---

## Entregable 2 — Procedimiento de alertas de negocio

Crea un procedimiento `sp_generar_alertas_negocio` que:
1. Detecte productos con stock 0 y genere alerta tipo `'STOCK_AGOTADO'`.
2. Detecte clientes de segmento `'ALTO_VALOR'` cuyo último pedido sea de hace más de 60 días y genere alerta tipo `'CLIENTE_INACTIVO'`.
3. Registre la ejecución en `log_procesos_plsql`.
4. Evite alertas duplicadas: no insertar si ya existe una alerta `ABIERTA` para la misma referencia.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu código completo para auditarlo</b></summary>

<b>Solución:</b>

```sql
CREATE OR REPLACE PROCEDURE sp_generar_alertas_negocio AS
  v_siguiente_id  NUMBER;
  v_alertas_nuevas NUMBER := 0;
  v_existe         NUMBER;
BEGIN
  -- Obtener el siguiente ID disponible
  SELECT NVL(MAX(id_alerta), 0) INTO v_siguiente_id FROM alertas_negocio;

  -- 1. Alertas de STOCK_AGOTADO
  FOR r IN (SELECT id_producto, nombre FROM productos WHERE stock = 0) LOOP
    -- Verificar si ya existe una alerta abierta para este producto
    SELECT COUNT(*) INTO v_existe
    FROM alertas_negocio
    WHERE tipo_alerta = 'STOCK_AGOTADO'
      AND referencia = 'Producto ID: ' || r.id_producto
      AND estado = 'ABIERTA';

    IF v_existe = 0 THEN
      v_siguiente_id := v_siguiente_id + 1;
      INSERT INTO alertas_negocio (id_alerta, tipo_alerta, referencia, detalle)
      VALUES (
        v_siguiente_id,
        'STOCK_AGOTADO',
        'Producto ID: ' || r.id_producto,
        'El producto "' || r.nombre || '" tiene stock 0'
      );
      v_alertas_nuevas := v_alertas_nuevas + 1;
    END IF;
  END LOOP;

  -- 2. Alertas de CLIENTE_INACTIVO (alto valor sin pedidos en 60 días)
  FOR r IN (
    SELECT c.id_cliente, c.nombre
    FROM clientes c
    WHERE c.segmento_cliente = 'ALTO_VALOR'
      AND NOT EXISTS (
        SELECT 1 FROM pedidos p
        WHERE p.id_cliente = c.id_cliente
          AND p.fecha_pedido > SYSDATE - 60
      )
  ) LOOP
    SELECT COUNT(*) INTO v_existe
    FROM alertas_negocio
    WHERE tipo_alerta = 'CLIENTE_INACTIVO'
      AND referencia = 'Cliente ID: ' || r.id_cliente
      AND estado = 'ABIERTA';

    IF v_existe = 0 THEN
      v_siguiente_id := v_siguiente_id + 1;
      INSERT INTO alertas_negocio (id_alerta, tipo_alerta, referencia, detalle)
      VALUES (
        v_siguiente_id,
        'CLIENTE_INACTIVO',
        'Cliente ID: ' || r.id_cliente,
        'Cliente "' || r.nombre || '" (ALTO_VALOR) sin pedidos en los últimos 60 días'
      );
      v_alertas_nuevas := v_alertas_nuevas + 1;
    END IF;
  END LOOP;

  -- Log de ejecución
  INSERT INTO log_procesos_plsql (nombre_proceso, estado, mensaje, filas_afectadas)
  VALUES ('sp_generar_alertas_negocio', 'OK', 'Alertas generadas correctamente', v_alertas_nuevas);

  COMMIT;
  DBMS_OUTPUT.PUT_LINE('Alertas generadas: ' || v_alertas_nuevas);
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    INSERT INTO log_procesos_plsql (nombre_proceso, estado, mensaje)
    VALUES ('sp_generar_alertas_negocio', 'ERROR', SQLERRM);
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Error al generar alertas: ' || SQLERRM);
END sp_generar_alertas_negocio;
/

-- Ejecución:
EXECUTE sp_generar_alertas_negocio;
```

Salida esperada:
```
Alertas generadas: 2
```

Verificación:
```sql
SELECT id_alerta, tipo_alerta, referencia, estado FROM alertas_negocio ORDER BY id_alerta;
```
| id_alerta | tipo_alerta | referencia | estado |
|-----------|------------|------------|--------|
| 1 | STOCK_AGOTADO | Producto ID: 12 | ABIERTA |
| 2 | CLIENTE_INACTIVO | Cliente ID: 1 | ABIERTA |

> 💡 Si ejecutas el procedimiento una segunda vez, no generará alertas duplicadas gracias a la verificación de existencia.

</details>

---

## Entregable 3 — Función de resumen operativo

Crea una función `fn_resumen_operativo` que no reciba parámetros y devuelva un `VARCHAR2` con un resumen en una línea del estado del sistema: `'Productos: X | Agotados: Y | Pedidos: Z | Alertas abiertas: W'`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu código completo para auditarlo</b></summary>

<b>Solución:</b>

```sql
CREATE OR REPLACE FUNCTION fn_resumen_operativo RETURN VARCHAR2 AS
  v_productos     NUMBER;
  v_agotados      NUMBER;
  v_pedidos       NUMBER;
  v_alertas       NUMBER;
BEGIN
  SELECT COUNT(*) INTO v_productos FROM productos;
  SELECT COUNT(*) INTO v_agotados FROM productos WHERE stock = 0;
  SELECT COUNT(*) INTO v_pedidos FROM pedidos;
  SELECT COUNT(*) INTO v_alertas FROM alertas_negocio WHERE estado = 'ABIERTA';

  RETURN 'Productos: ' || v_productos ||
         ' | Agotados: ' || v_agotados ||
         ' | Pedidos: ' || v_pedidos ||
         ' | Alertas abiertas: ' || v_alertas;
EXCEPTION
  WHEN OTHERS THEN
    RETURN 'Error al generar resumen: ' || SQLERRM;
END fn_resumen_operativo;
/

-- Uso en SELECT:
SELECT fn_resumen_operativo AS dashboard FROM DUAL;
```

Resultado:
| dashboard |
|-----------|
| Productos: 8 \| Agotados: 1 \| Pedidos: 7 \| Alertas abiertas: 2 |

</details>

---

## Entregable 4 — Procedimiento maestro `sp_ciclo_operativo`

Crea un procedimiento `sp_ciclo_operativo` que orqueste la ejecución completa del ciclo operativo:
1. Llama a `sp_refrescar_kpis_mensuales`.
2. Llama a `sp_generar_alertas_negocio`.
3. Muestra el resumen usando `fn_resumen_operativo`.
4. Registra la ejecución completa en `log_procesos_plsql`.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu código completo para auditarlo</b></summary>

<b>Solución:</b>

```sql
CREATE OR REPLACE PROCEDURE sp_ciclo_operativo AS
  v_resumen VARCHAR2(500);
BEGIN
  DBMS_OUTPUT.PUT_LINE('========================================');
  DBMS_OUTPUT.PUT_LINE('  CICLO OPERATIVO — Inicio');
  DBMS_OUTPUT.PUT_LINE('========================================');

  -- Paso 1: Refrescar KPIs
  DBMS_OUTPUT.PUT_LINE('[1/3] Refrescando KPIs mensuales...');
  sp_refrescar_kpis_mensuales;

  -- Paso 2: Generar alertas
  DBMS_OUTPUT.PUT_LINE('[2/3] Generando alertas de negocio...');
  sp_generar_alertas_negocio;

  -- Paso 3: Mostrar resumen
  DBMS_OUTPUT.PUT_LINE('[3/3] Generando resumen...');
  v_resumen := fn_resumen_operativo;
  DBMS_OUTPUT.PUT_LINE('Dashboard: ' || v_resumen);

  -- Log del ciclo completo
  INSERT INTO log_procesos_plsql (nombre_proceso, estado, mensaje)
  VALUES ('sp_ciclo_operativo', 'OK', 'Ciclo completo ejecutado. ' || v_resumen);
  COMMIT;

  DBMS_OUTPUT.PUT_LINE('========================================');
  DBMS_OUTPUT.PUT_LINE('  CICLO OPERATIVO — Fin');
  DBMS_OUTPUT.PUT_LINE('========================================');
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    INSERT INTO log_procesos_plsql (nombre_proceso, estado, mensaje)
    VALUES ('sp_ciclo_operativo', 'ERROR', SQLERRM);
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('ERROR CRÍTICO en ciclo operativo: ' || SQLERRM);
END sp_ciclo_operativo;
/

-- Ejecución:
EXECUTE sp_ciclo_operativo;
```

Salida esperada:
```
========================================
  CICLO OPERATIVO — Inicio
========================================
[1/3] Refrescando KPIs mensuales...
KPIs actualizados: 2 meses procesados
[2/3] Generando alertas de negocio...
Alertas generadas: 0
[3/3] Generando resumen...
Dashboard: Productos: 8 | Agotados: 1 | Pedidos: 7 | Alertas abiertas: 2
========================================
  CICLO OPERATIVO — Fin
========================================
```

> 💡 En la segunda ejecución, las alertas generadas serán 0 porque ya existen alertas abiertas para los mismos productos/clientes. Esto demuestra la protección contra duplicados del Entregable 2.

Verificación final del log:
```sql
SELECT nombre_proceso, estado, fecha_ejecucion FROM log_procesos_plsql ORDER BY fecha_ejecucion;
```
| nombre_proceso | estado | fecha_ejecucion |
|---------------|--------|-----------------|
| sp_refrescar_kpis_mensuales | OK | 2025-XX-XX |
| sp_generar_alertas_negocio | OK | 2025-XX-XX |
| sp_refrescar_kpis_mensuales | OK | 2025-XX-XX |
| sp_generar_alertas_negocio | OK | 2025-XX-XX |
| sp_ciclo_operativo | OK | 2025-XX-XX |

</details>

---

<div align="center">

⬅️ [**📝 Ejercicios del Tema 16**](../ejercicios/ejercicios_plsql.md) · 🏠 [**Índice del Curso**](../../README.md) · [**Tema 17: Optimización de Consultas →**](../../17-optimizacion/README.md)

</div>
