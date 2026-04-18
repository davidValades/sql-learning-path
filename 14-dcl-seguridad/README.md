# 🔒 Tema 14: DCL y Seguridad (Data Control Language)

> **"Una base de datos segura no confía en las personas, confía en los permisos."** En este tema vas a controlar quién puede ver, insertar, modificar o administrar objetos en la base de datos usando DCL y buenas prácticas de seguridad.

## 📋 Índice

- [14.0 Estado heredado del modelo](#140-estado-heredado-del-modelo)
- [14.1 Principio de mínimo privilegio](#141-principio-de-mínimo-privilegio)
- [14.2 Usuarios, roles y perfiles](#142-usuarios-roles-y-perfiles)
- [14.3 GRANT y REVOKE aplicados al caso real](#143-grant-y-revoke-aplicados-al-caso-real)
- [14.4 Seguridad por capas con vistas](#144-seguridad-por-capas-con-vistas)
- [14.5 Auditoría de privilegios](#145-auditoría-de-privilegios)

---

## 14.0 Estado heredado del modelo

Llegamos desde el Tema 13 con tres dominios operativos activos:

- **E-commerce:** `clientes`, `productos`, `pedidos`.
- **Hospital:** `pacientes`, `medicos`, `citas`, `especialidades`.
- **Aerolínea:** `aviones`, `rutas`, `vuelos`.

A partir de ahora no solo importa modelar y consultar, también importa **quién puede hacer cada acción**.

---

## 14.1 Principio de mínimo privilegio

### 📘 El Concepto

Cada usuario debe tener solo los permisos estrictamente necesarios para su trabajo.

- Un analista: `SELECT`.
- Un operador: `INSERT/UPDATE` en tablas concretas.
- Un admin: gestión técnica controlada.

### 💻 El Código

```sql
-- Rol de solo lectura para reporting
CREATE ROLE rol_reporting;
GRANT SELECT ON clientes TO rol_reporting;
GRANT SELECT ON pedidos TO rol_reporting;
GRANT SELECT ON productos TO rol_reporting;

-- Rol operativo de ventas
CREATE ROLE rol_operaciones_ventas;
GRANT SELECT, INSERT, UPDATE ON pedidos TO rol_operaciones_ventas;
GRANT SELECT, UPDATE ON productos TO rol_operaciones_ventas;
```

---

## 14.2 Usuarios, roles y perfiles

### 📘 El Concepto

En Oracle, separas identidad y privilegios:

- **Usuario:** la cuenta (`CREATE USER`).
- **Rol:** paquete reusable de privilegios (`CREATE ROLE`).
- **Perfil:** límites de sesión/contraseña (`CREATE PROFILE`).

### 💻 El Código

```sql
CREATE PROFILE perfil_app LIMIT
  SESSIONS_PER_USER 3
  FAILED_LOGIN_ATTEMPTS 5
  PASSWORD_LIFE_TIME 90;

CREATE USER usr_analista IDENTIFIED BY "Analista#2026" PROFILE perfil_app;
CREATE USER usr_operador IDENTIFIED BY "Operador#2026" PROFILE perfil_app;

GRANT CREATE SESSION TO usr_analista, usr_operador;
GRANT rol_reporting TO usr_analista;
GRANT rol_operaciones_ventas TO usr_operador;
```

---

## 14.3 GRANT y REVOKE aplicados al caso real

### 📘 El Concepto

DCL no es estático: cambian equipos, responsabilidades y riesgos.

### 💻 El Código

```sql
-- Permiso temporal para una campaña de ajustes de stock
GRANT UPDATE (stock) ON productos TO usr_operador;

-- Se termina la campaña y se retira acceso sensible
REVOKE UPDATE (stock) ON productos FROM usr_operador;
```

---

## 14.4 Seguridad por capas con vistas

### 📘 El Concepto

No des acceso directo a tablas sensibles si puedes exponer solo lo necesario con vistas.

### 💻 El Código

```sql
CREATE OR REPLACE VIEW v_pedidos_reporting AS
SELECT p.id_pedido,
       c.nombre AS cliente,
       pr.nombre AS producto,
       p.total,
       p.fecha_pedido
FROM pedidos p
JOIN clientes c ON p.id_cliente = c.id_cliente
JOIN productos pr ON p.id_producto = pr.id_producto;

GRANT SELECT ON v_pedidos_reporting TO rol_reporting;
```

---

## 14.5 Auditoría de privilegios

### 📘 El Concepto

Para operar en entorno profesional necesitas trazabilidad de permisos.

### 💻 El Código

```sql
-- Privilegios de sistema
SELECT grantee, privilege
FROM dba_sys_privs
WHERE grantee IN ('USR_ANALISTA', 'USR_OPERADOR', 'ROL_REPORTING');

-- Privilegios de objeto
SELECT grantee, owner, table_name, privilege
FROM dba_tab_privs
WHERE grantee IN ('USR_ANALISTA', 'USR_OPERADOR', 'ROL_REPORTING');
```

### 🧠 El Reto

Diseña dos roles nuevos para continuidad operativa:

1. `rol_soporte_nocturno`: puede consultar todo, pero solo actualizar `estado` de `citas`.
2. `rol_auditoria`: solo lectura sobre vistas de negocio y vistas de permisos.

---

<div align="center">

### 🗺️ Ruta de Aprendizaje — Tema 14

</div>

| # | Paso | Recurso |
|:-:|------|---------|
| 1️⃣ | Estudiar el temario | 📖 _Estás aquí_ |
| 2️⃣ | Completar ejercicios | 🏋️ [Ir a Ejercicios →](./ejercicios/ejercicios_dcl_seguridad.md) |
| 3️⃣ | Resolver proyecto | 🏆 [Ir al Proyecto →](./proyectos/proyecto_dcl_seguridad.md) |
| 4️⃣ | Avanzar al siguiente tema | ➡️ [Tema 15: SQL Avanzado](../15-sql-avanzado) |

---

<div align="center">

⬅️ [**Tema 13: Transacciones y Propiedades ACID**](../13-transacciones-acid) · 🏠 [**Índice del Curso**](../README.md) · [**🏋️ Ejercicios →**](./ejercicios/ejercicios_dcl_seguridad.md)

</div>
