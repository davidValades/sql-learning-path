# 🏆 Proyecto — Tema 14: Hardening de Seguridad SQL

---

## 🎯 Objetivo

Implementar un esquema realista de usuarios, roles y privilegios aplicando **mínimo privilegio** y trazabilidad.

---

## Entregable 1 — Matriz de acceso

Define perfiles:
- `dba_junior`
- `analista_bi`
- `soporte_operativo`

Para cada perfil, detalla qué operaciones puede ejecutar sobre tablas clave (`clientes`, `pedidos`, `citas`).

---

## Entregable 2 — Implementación DCL

Crear:
- usuarios,
- roles por perfil,
- grants de sistema y objeto,
- revokes de permisos excesivos.

---

## Entregable 3 — Auditoría de permisos

Genera consultas sobre vistas de diccionario (`DBA_SYS_PRIVS`, `DBA_TAB_PRIVS`, `DBA_ROLE_PRIVS`) para validar:
1. permisos concedidos,
2. herencia por roles,
3. cumplimiento de mínimo privilegio.

---

<div align="center">

⬅️ [**🏋️ Ejercicios del Tema 14**](../ejercicios/ejercicios_dcl_seguridad.md) · 🏠 [**Tema 14**](../README.md) · [**Tema 15: SQL Avanzado →**](../../15-sql-avanzado)

</div>
