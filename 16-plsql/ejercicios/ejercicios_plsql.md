# 🏋️ Ejercicios — Tema 16: PL/SQL

---

## Ejercicio 1 — Bloque anónimo con variables

**Enunciado:** Declara una variable `v_total_pedidos`, cuenta pedidos y muestra el resultado con `DBMS_OUTPUT`.

<details>
<summary>👉 Ver solución</summary>

```sql
DECLARE
  v_total_pedidos NUMBER;
BEGIN
  SELECT COUNT(*) INTO v_total_pedidos FROM pedidos;
  DBMS_OUTPUT.PUT_LINE('Total pedidos: ' || v_total_pedidos);
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Total pedidos: 7
```

</details>

---

## Ejercicio 2 — Manejo de excepciones

**Enunciado:** Busca cliente por ID y captura `NO_DATA_FOUND`.

<details>
<summary>👉 Ver solución</summary>

```sql
DECLARE
  v_nombre clientes.nombre%TYPE;
BEGIN
  SELECT nombre INTO v_nombre
  FROM clientes
  WHERE id_cliente = 99999;

  DBMS_OUTPUT.PUT_LINE(v_nombre);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('Cliente no encontrado');
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Cliente no encontrado
```

> 💡 Como no existe ningún cliente con id 99999, Oracle lanza `NO_DATA_FOUND` y el bloque `EXCEPTION` captura el error mostrando el mensaje personalizado.

</details>

---

## Ejercicio 3 — Cursor explícito

**Enunciado:** Recorre productos de categoría 1 e imprime nombre y precio.

<details>
<summary>👉 Ver solución</summary>

```sql
DECLARE
  CURSOR c_productos IS
    SELECT nombre, precio
    FROM productos
    WHERE id_categoria = 1;
BEGIN
  FOR r IN c_productos LOOP
    DBMS_OUTPUT.PUT_LINE(r.nombre || ' - ' || r.precio);
  END LOOP;
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Laptop Pro - 1220
Ratón Inalámbrico - 45.5
Monitor 4K - 350
Teclado Mecánico - 75
```

> 💡 Se muestran los 4 productos de categoría 1 (Electrónica). El cursor recorre cada fila e imprime nombre y precio.

</details>

---

<div align="center">

⬅️ [**Tema 16: PL/SQL**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 16 →**](../proyectos/proyecto_plsql.md)

</div>
