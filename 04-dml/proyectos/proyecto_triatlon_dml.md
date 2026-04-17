## 🏆 Proyecto Final Nivel 3: El Triatlón de Datos (DML)

### 📘 El Concepto

Has sido contratado por una firma de consultoría de élite como un "Data Fixer" (Solucionador de Datos). Esta mañana, las tres empresas con las que trabajamos (el E-commerce, el Hospital Central y la Aerolínea) han sufrido crisis simultáneas. Necesitan que alguien manipule sus datos de urgencia, de forma quirúrgica y sin cometer errores.

### 🏗️ Las Crisis (El Reto)

Abre tu consola SQL. Tienes 3 misiones. Necesito que escribas las sentencias exactas para resolver cada problema.

**🚨 Misión 1: El E-commerce (Operación INSERT)**
El departamento de logística acaba de recibir un palet sorpresa de un producto muy esperado y el CEO lo quiere a la venta en la web YA.

- **Tabla:** `productos` (Recuerda que las columnas son: `id_producto`, `nombre`, `precio`, `id_categoria`, `stock`).
- **Los datos:** ID `999`, Nombre `'Gafas de Realidad Virtual'`, Precio `499.99`, ID de Categoría `1` (asumiremos que es Electrónica), Stock `50`.
- **Tu tarea:** Escribe el comando para insertar este producto.

**🚨 Misión 2: El Hospital (Operación UPDATE)**
El departamento de Recursos Humanos ha cometido un error. Hay un médico estrella en la plantilla, el `Dr. House` (cuyo `id_medico` es el número `1`), que ha amenazado con irse si no le suben el sueldo. El director ha cedido.

- **Tabla:** `medicos`
- **Tu tarea:** Escribe el comando para actualizar el `salario_base` a `8500.00` **SOLO** al médico con el ID 1. ¡Salva al hospital sin arruinarlo subiendo el sueldo a todos!

**🚨 Misión 3: La Aerolínea (Operación DELETE y Transacciones)**
El departamento de aviación civil ha prohibido temporalmente volar a un modelo de avión por fallos en el motor. Necesitan que lo borres del sistema para que no se le asigne ningún vuelo.

- **Tabla:** `aviones`
- **Tu tarea:** Escribe el comando para eliminar **solo** el avión cuyo `id_avion` sea el número `2`.
- **Bonus:** Una vez hayas hecho el borrado, escribe el comando mágico para guardar todas estas operaciones (el Insert, el Update y el Delete) definitivamente en la base de datos.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu respuesta para comprobarla</b></summary>

<b>Respuesta del Profesor:</b>

```sql
-- Misión 1: E-commerce
INSERT INTO productos (id_producto, nombre, precio, id_categoria, stock)
VALUES (999, 'Gafas de Realidad Virtual', 499.99, 1, 50);

-- Misión 2: Hospital
UPDATE medicos
SET salario_base = 8500.00
WHERE id_medico = 1;

-- Misión 3: Aerolínea
DELETE FROM aviones
WHERE id_avion = 2;

-- ¡Guardado definitivo!
COMMIT;
```

</details>

---
