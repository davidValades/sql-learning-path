# 🏆 Proyecto — Motor de Pedidos Transaccional (PL/SQL)

## Contexto

La operación de ventas necesita encapsular toda la lógica en procedimientos para evitar errores manuales en SQL ad-hoc.

## Objetivo

Implementar un mini-módulo PL/SQL que procese pedidos con validaciones, auditoría y rollback seguro.

---

## Requisitos

1. Crear procedimiento `sp_procesar_pedido` con validación de stock y cantidad.
2. Registrar eventos en tabla de auditoría (`auditoria_stock` o equivalente).
3. Crear función de segmentación de cliente por gasto.
4. Crear paquete `pkg_pedidos` con al menos 2 procedimientos públicos.
5. Incluir bloque de pruebas y bloque de rollback de datos de prueba.

---

## Criterios de aceptación

- Si no hay stock, el pedido no se inserta.
- Si hay error, no quedan cambios parciales.
- Quedan trazas de cambios de stock.
- El paquete compila sin errores (`USER_ERRORS` vacío).

---

<div align="center">

⬅️ [**Ejercicios PL/SQL**](../ejercicios/ejercicios_plsql.md) · 🏠 [**Volver al Tema 16**](../README.md) · [**Tema 17 →**](../../17-optimizacion)

</div>
