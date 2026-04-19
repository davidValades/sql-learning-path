# 🏋️ Ejercicios — Tema 16: PL/SQL

> 💡 **Instrucciones:** Todos los ejercicios usan el estado acumulado de la base de datos del curso. Los objetos creados (procedimientos, funciones, triggers) se asumen vigentes para el proyecto del Tema 16 y temas siguientes.

---

## Ejercicio 1 — Bloque anónimo con variables

**Enunciado:** Declara una variable `v_total_pedidos` de tipo `NUMBER`, cuenta los pedidos existentes en la tabla `pedidos` y muestra el resultado con `DBMS_OUTPUT` en formato: `'Total pedidos en el sistema: X'`.

<details>
<summary>👉 Ver solución</summary>

```sql
DECLARE
  v_total_pedidos NUMBER;
BEGIN
  SELECT COUNT(*) INTO v_total_pedidos FROM pedidos;
  DBMS_OUTPUT.PUT_LINE('Total pedidos en el sistema: ' || v_total_pedidos);
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Total pedidos en el sistema: 7
```

> 💡 El bloque anónimo no se guarda en la base de datos. Se ejecuta una vez y desaparece.

</details>

---

## Ejercicio 2 — Manejo de excepciones con múltiples capturas

**Enunciado:** Escribe un bloque PL/SQL que intente buscar el nombre de un cliente con `id_cliente = 99999`. Captura las excepciones `NO_DATA_FOUND` (mostrando `'Cliente no encontrado'`) y `OTHERS` (mostrando el código y mensaje del error).

<details>
<summary>👉 Ver solución</summary>

```sql
DECLARE
  v_nombre clientes.nombre%TYPE;
BEGIN
  SELECT nombre INTO v_nombre
  FROM clientes
  WHERE id_cliente = 99999;

  DBMS_OUTPUT.PUT_LINE('Cliente: ' || v_nombre);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('Cliente no encontrado');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error [' || SQLCODE || ']: ' || SQLERRM);
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Cliente no encontrado
```

> 💡 Como no existe ningún cliente con id 99999, Oracle lanza `NO_DATA_FOUND` y el bloque `EXCEPTION` captura el error mostrando el mensaje personalizado. El `WHEN OTHERS` actúa como red de seguridad por si se produce un error diferente.

</details>

---

## Ejercicio 3 — Cursor con FOR LOOP y lógica condicional

**Enunciado:** Recorre todos los productos usando un cursor explícito con `FOR LOOP`. Para cada producto muestra su nombre y precio, y si el stock es 0 añade la etiqueta `[AGOTADO]` al final de la línea.

<details>
<summary>👉 Ver solución</summary>

```sql
DECLARE
  CURSOR c_productos IS
    SELECT nombre, precio, stock
    FROM productos
    ORDER BY id_producto;
BEGIN
  FOR r IN c_productos LOOP
    IF r.stock = 0 THEN
      DBMS_OUTPUT.PUT_LINE(r.nombre || ' - ' || r.precio || '€ [AGOTADO]');
    ELSE
      DBMS_OUTPUT.PUT_LINE(r.nombre || ' - ' || r.precio || '€ (Stock: ' || r.stock || ')');
    END IF;
  END LOOP;
END;
/
```

Salida esperada por consola (DBMS_OUTPUT):
```
Laptop Pro - 1220€ (Stock: 50)
Ratón Inalámbrico - 45.5€ (Stock: 200)
Sofá de Cuero - 405€ [AGOTADO]
Lámpara LED - 40.5€ (Stock: 30)
Camiseta Básica - 19.99€ (Stock: 100)
Zapatillas Running - 89.5€ (Stock: 45)
Monitor 4K - 350€ (Stock: 25)
Teclado Mecánico - 75€ (Stock: 80)
```

> 💡 El cursor recorre los 8 productos del catálogo. El Sofá de Cuero (stock 0) muestra la etiqueta `[AGOTADO]`. Esto combina cursores con estructuras de control `IF`.

</details>

---

## Ejercicio 4 — Procedimiento almacenado con parámetro IN

**Enunciado:** Crea un procedimiento `sp_productos_categoria` que reciba un `id_categoria` como parámetro `IN` y muestre por consola todos los productos de esa categoría con formato: `'- <nombre> (Precio: <precio>€)'`. Si no hay productos, muestra `'No hay productos en esta categoría'`.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE OR REPLACE PROCEDURE sp_productos_categoria (
  p_id_categoria IN productos.id_categoria%TYPE
) AS
  v_encontrado BOOLEAN := FALSE;
  CURSOR c_productos IS
    SELECT nombre, precio
    FROM productos
    WHERE id_categoria = p_id_categoria
    ORDER BY precio DESC;
BEGIN
  FOR r IN c_productos LOOP
    DBMS_OUTPUT.PUT_LINE('- ' || r.nombre || ' (Precio: ' || r.precio || '€)');
    v_encontrado := TRUE;
  END LOOP;

  IF NOT v_encontrado THEN
    DBMS_OUTPUT.PUT_LINE('No hay productos en esta categoría');
  END IF;
END sp_productos_categoria;
/

-- Prueba con categoría 1 (Electrónica):
EXECUTE sp_productos_categoria(1);

-- Prueba con categoría inexistente:
EXECUTE sp_productos_categoria(999);
```

Salida esperada de la primera ejecución:
```
- Laptop Pro (Precio: 1220€)
- Monitor 4K (Precio: 350€)
- Teclado Mecánico (Precio: 75€)
- Ratón Inalámbrico (Precio: 45.5€)
```

Salida esperada de la segunda ejecución:
```
No hay productos en esta categoría
```

</details>

---

## Ejercicio 5 — Función para usar en SELECT

**Enunciado:** Crea una función `fn_estado_stock` que reciba la cantidad de stock de un producto y devuelva un `VARCHAR2` con la clasificación: `'AGOTADO'` si es 0, `'BAJO'` si es menor a 30, `'NORMAL'` si es menor a 100, y `'ALTO'` si es 100 o más. Luego úsala en un `SELECT` sobre la tabla `productos`.

<details>
<summary>👉 Ver solución</summary>

```sql
CREATE OR REPLACE FUNCTION fn_estado_stock (
  p_stock IN NUMBER
) RETURN VARCHAR2 AS
BEGIN
  IF p_stock = 0 THEN
    RETURN 'AGOTADO';
  ELSIF p_stock < 30 THEN
    RETURN 'BAJO';
  ELSIF p_stock < 100 THEN
    RETURN 'NORMAL';
  ELSE
    RETURN 'ALTO';
  END IF;
END fn_estado_stock;
/

-- Consulta usando la función:
SELECT nombre, stock, fn_estado_stock(stock) AS estado
FROM productos
ORDER BY stock;
```

Resultado:
| nombre | stock | estado |
|--------|-------|--------|
| Sofá de Cuero | 0 | AGOTADO |
| Monitor 4K | 25 | BAJO |
| Lámpara LED | 30 | NORMAL |
| Zapatillas Running | 45 | NORMAL |
| Laptop Pro | 50 | NORMAL |
| Teclado Mecánico | 80 | NORMAL |
| Camiseta Básica | 100 | ALTO |
| Ratón Inalámbrico | 200 | ALTO |

> 💡 La función se integra directamente en el `SELECT` como si fuera una columna calculada. Esto es una ventaja clave de las funciones sobre los procedimientos.

✅ **Esta función queda incorporada al estado acumulado.**

</details>

---

## Ejercicio 6 — Trigger de auditoría

**Enunciado:** Crea una tabla `log_cambios_stock` con columnas: `id_log` (autogenerado), `id_producto`, `stock_anterior`, `stock_nuevo`, `fecha_cambio` y `usuario`. Luego crea un trigger `trg_auditoria_stock` que se dispare `AFTER UPDATE OF stock ON productos` y registre cada cambio en esa tabla.

<details>
<summary>👉 Ver solución</summary>

```sql
-- Tabla de auditoría:
CREATE TABLE log_cambios_stock (
  id_log         NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  id_producto    NUMBER NOT NULL,
  stock_anterior NUMBER,
  stock_nuevo    NUMBER,
  fecha_cambio   DATE DEFAULT SYSDATE,
  usuario        VARCHAR2(50) DEFAULT USER
);

-- Trigger:
CREATE OR REPLACE TRIGGER trg_auditoria_stock
AFTER UPDATE OF stock ON productos
FOR EACH ROW
WHEN (OLD.stock != NEW.stock)
BEGIN
  INSERT INTO log_cambios_stock (id_producto, stock_anterior, stock_nuevo)
  VALUES (:OLD.id_producto, :OLD.stock, :NEW.stock);
END trg_auditoria_stock;
/

-- Prueba: modificamos el stock de un producto
UPDATE productos SET stock = stock - 5 WHERE id_producto = 10;

-- Verificamos el log:
SELECT * FROM log_cambios_stock;
```

Resultado:
| id_log | id_producto | stock_anterior | stock_nuevo | fecha_cambio | usuario |
|--------|------------|----------------|-------------|-------------|---------|
| 1 | 10 | 50 | 45 | 2025-XX-XX | SYSTEM |

```sql
ROLLBACK;  -- Revertimos para no alterar el estado del curso
```

> 💡 El trigger se ejecuta automáticamente cada vez que se modifica el stock de un producto. Ni el desarrollador que hace el `UPDATE` necesita saber que el trigger existe — la auditoría ocurre de forma transparente.

✅ **La tabla `log_cambios_stock` y el trigger quedan incorporados al estado acumulado.**

</details>

---

<div align="center">

⬅️ [**Tema 16: PL/SQL**](../README.md) · 🏠 [**Índice del Curso**](../../README.md) · [**🏆 Proyecto del Tema 16 →**](../proyectos/proyecto_plsql.md)

</div>
