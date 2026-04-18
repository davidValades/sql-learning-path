# 🏆 Proyecto — Gobierno de Accesos por Área

## Contexto

La empresa necesita separar accesos para tres equipos:

- BI (lectura transversal).
- Operación (escritura acotada).
- Auditoría (solo lectura de logs y permisos).

## Objetivo

Diseñar e implementar un esquema de roles/usuarios que aplique mínimo privilegio y permita auditoría.

---

## Entregables

1. Script de creación de roles.
2. Script de creación de usuarios y asignaciones.
3. Script de vistas seguras para reporting.
4. Script de verificación de permisos efectivos.
5. Script de cleanup (rollback de seguridad).

---

## Requisitos técnicos

- Sin `GRANT ALL`.
- Permisos por columna cuando aplique.
- Revocación explícita de accesos peligrosos.
- Evidencia SQL de que cada usuario solo puede lo que le corresponde.

---

## Criterio de aprobación

Se considera aprobado cuando puedes demostrar:

- `usr_bi` no puede hacer `INSERT/UPDATE/DELETE`.
- `usr_operacion` puede actualizar `stock` y `estado` en tablas autorizadas.
- `usr_auditoria` puede consultar permisos y trazas, sin modificar negocio.

---

<div align="center">

⬅️ [**Ejercicios DCL**](../ejercicios/ejercicios_dcl_seguridad.md) · 🏠 [**Volver al Tema 14**](../README.md) · [**Tema 15 →**](../../15-sql-avanzado)

</div>
