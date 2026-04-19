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

Salida esperada:
```
User created.
Grant succeeded.
```

Verificación:
```sql
SELECT username, default_tablespace, account_status
FROM dba_users WHERE username = 'ANALISTA_REPORTES';
```
| username | default_tablespace | account_status |
|----------|-------------------|----------------|
| ANALISTA_REPORTES | USERS | OPEN |

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

Salida esperada:
```
Grant succeeded.
Revoke succeeded.
```

> 💡 Tras el REVOKE, si `analista_reportes` intenta `SELECT * FROM ventas_mensuales`, obtendrá: `ORA-00942: table or view does not exist`.

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

Salida esperada:
```
Role created.
Grant succeeded.
Grant succeeded.
Grant succeeded.
```

Verificación:
```sql
SELECT * FROM dba_role_privs WHERE grantee = 'USUARIO_SOPORTE';
```
| grantee | granted_role | admin_option |
|---------|-------------|-------------|
| USUARIO_SOPORTE | ROL_SOPORTE_LECTURA | NO |

> 💡 Ahora `usuario_soporte` puede hacer SELECT sobre `clientes` y `pedidos`, pero no INSERT, UPDATE ni DELETE.

</details>

---

<div align="center">

⬅️ [**Tema 14**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 14 →**](../proyectos/proyecto_dcl_seguridad.md)

</div>
