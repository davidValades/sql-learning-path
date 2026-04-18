# 🏋️ Ejercicios — Tema 14: DCL y Seguridad

---

## Ejercicio 1 — Crear usuario y permisos mínimos

**Enunciado:** Crea un usuario `analista_reportes` con cuota de 50M y permiso solo de conexión.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE USER analista_reportes IDENTIFIED BY "Segura#2026"
DEFAULT TABLESPACE users
TEMPORARY TABLESPACE temp
QUOTA 50M ON users;

GRANT CREATE SESSION TO analista_reportes;
```

</details>

---

## Ejercicio 2 — Privilegios de objeto

**Enunciado:** Otorga `SELECT` sobre `ventas_mensuales` al usuario anterior y después revócalo.

<details>
<summary>👉 Ver solución</summary>

```sql
GRANT SELECT ON ventas_mensuales TO analista_reportes;
REVOKE SELECT ON ventas_mensuales FROM analista_reportes;
```

</details>

---

## Ejercicio 3 — Rol personalizado

**Enunciado:** Crea rol `rol_soporte_lectura` con permisos de lectura sobre `clientes` y `pedidos`, y asígnalo a `usuario_soporte`.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE ROLE rol_soporte_lectura;
GRANT SELECT ON clientes TO rol_soporte_lectura;
GRANT SELECT ON pedidos TO rol_soporte_lectura;
GRANT rol_soporte_lectura TO usuario_soporte;
```

</details>

---

<div align="center">

⬅️ [**Tema 14**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 14 →**](../proyectos/proyecto_dcl_seguridad.md)

</div>
