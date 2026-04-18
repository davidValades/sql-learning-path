# 🏋️ Ejercicios — Tema 03: Normalización

> 💡 **Instrucciones:** Estos ejercicios arrancan la base de datos evolutiva del curso. Todo cambio de diseño que hagas aquí se arrastra al Tema 04 (DDL) y siguientes.

---

## Ejercicio 1 — Separar una tabla legacy en entidades normalizadas

**Enunciado:** Partiendo de una tabla `pedidos_legacy` (cliente, email, producto, categoria, precio, cantidad, fecha_pedido), diseña el modelo mínimo en 3NF para e-commerce.

Debes definir:
1. Tablas finales.
2. Claves primarias.
3. Claves foráneas.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE TABLE clientes (
  id_cliente NUMBER PRIMARY KEY,
  nombre     VARCHAR2(100) NOT NULL,
  email      VARCHAR2(150) UNIQUE NOT NULL
);

CREATE TABLE categorias (
  id_categoria NUMBER PRIMARY KEY,
  nombre       VARCHAR2(80) NOT NULL UNIQUE
);

CREATE TABLE productos (
  id_producto  NUMBER PRIMARY KEY,
  nombre       VARCHAR2(120) NOT NULL,
  precio       NUMBER(10,2) NOT NULL,
  id_categoria NUMBER NOT NULL,
  CONSTRAINT fk_productos_categoria
    FOREIGN KEY (id_categoria) REFERENCES categorias(id_categoria)
);

CREATE TABLE pedidos (
  id_pedido    NUMBER PRIMARY KEY,
  id_cliente   NUMBER NOT NULL,
  id_producto  NUMBER NOT NULL,
  cantidad     NUMBER NOT NULL,
  fecha_pedido DATE NOT NULL,
  total        NUMBER(10,2) NOT NULL,
  CONSTRAINT fk_pedidos_cliente
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente),
  CONSTRAINT fk_pedidos_producto
    FOREIGN KEY (id_producto) REFERENCES productos(id_producto)
);
```

</details>

---

## Ejercicio 2 — Evolución de esquema con ALTER TABLE

**Enunciado:** Evoluciona el diseño anterior para dejarlo listo para operación real:

1. Añade `fecha_alta` en `clientes` con valor por defecto `SYSDATE`.
2. Añade una restricción para garantizar formato básico de email.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
ALTER TABLE clientes
  ADD fecha_alta DATE DEFAULT SYSDATE NOT NULL;

ALTER TABLE clientes
  ADD CONSTRAINT ck_clientes_email_basico
  CHECK (INSTR(email, '@') > 1);
```

✅ **Este cambio queda vigente para todos los temas posteriores.**

</details>

---

## Ejercicio 3 — Resolver atributo multivalor

**Enunciado:** Cada cliente puede tener varias direcciones de envío. Diseña la solución en 1NF sin romper el modelo actual.

<details>
<summary>👉 Haz clic aquí SOLO cuando tengas tu respuesta</summary>

```sql
CREATE TABLE direcciones_cliente (
  id_direccion NUMBER PRIMARY KEY,
  id_cliente   NUMBER NOT NULL,
  direccion    VARCHAR2(200) NOT NULL,
  ciudad       VARCHAR2(80) NOT NULL,
  cp           VARCHAR2(10),
  principal    CHAR(1) DEFAULT 'N' CHECK (principal IN ('S','N')),
  CONSTRAINT fk_direcciones_cliente
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente)
);
```

</details>

---

## Ejercicio 4 — Preparar el backlog técnico para DDL

**Enunciado:** Define qué objetos deben construirse primero en el Tema 04 para respetar dependencias.

<details>
<summary>�� Haz clic aquí SOLO cuando tengas tu respuesta</summary>

Orden recomendado:
1. `clientes`
2. `categorias`
3. `productos`
4. `pedidos`
5. `direcciones_cliente`

> Si creas `pedidos` antes que `clientes` y `productos`, fallarán las FK.

</details>

---

<div align="center">

⬅️ [**Tema 03: Normalización y Modelado ER**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 03 →**](../proyectos/proyecto_normalizacion.md)

</div>
