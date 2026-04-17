## 🏆 Proyecto DML Parte 2: Día 30 en Producción (El Caos Cotidiano)

### 📘 El Concepto

Las bases de datos ya llevan un mes funcionando y el negocio está vivo. Las cosas cambian, hay errores humanos que corregir y nuevas reglas de negocio que aplicar.

Como Administrador de Base de Datos, te van a llegar "tickets" de soporte de los diferentes departamentos. Tu trabajo es ejecutar los comandos DML (`UPDATE`, `DELETE`, `INSERT`) y DDL (`ALTER`) necesarios para mantener la integridad y actualización de la información, sin romper nada.

¡Recuerda siempre la regla de oro del DBA! **Un `UPDATE` o `DELETE` sin `WHERE` es un billete de ida a la cola del paro.**

### 🏗️ Los Tickets de Soporte

Escribe un único script SQL que resuelva los siguientes problemas en orden:

**🛒 E-commerce (Campaña Black Friday y Errores)**

1. **Ticket 101 (UPDATE):** El departamento de marketing ha decidido hacer una rebaja flash. Necesitan que **descuentes un 10%** al precio de TODOS los productos que pertenezcan a la categoría `2` (Hogar).
2. **Ticket 102 (UPDATE):** ¡Alerta de logística! Se ha inundado parte del almacén. El stock del 'Sofá de Cuero' (ID 12) ha quedado inservible. Actualiza su stock a `0`.
3. **Ticket 103 (ALTER):** Negocio quiere empezar a clasificar los productos por un "Código de Barras" alfanumérico. Añade una nueva columna a la tabla `productos` llamada `codigo_barras` que soporte hasta 20 caracteres de texto.

**🏥 Hospital Central (Actualizaciones de Personal y Pacientes)** 4. **Ticket 201 (UPDATE):** La paciente 'Laura Martinez' (DNI '11111111A') ha llamado para actualizar su número de contacto. Su nuevo teléfono es `'555-9999'`. 5. **Ticket 202 (DELETE):** El 'Dr. John Smith' (ID 1) se ha jubilado. Elimina su registro de la base de datos. 6. **Ticket 203 (INSERT):** Contratamos a un nuevo médico para sustituirlo.

- ID: `4`, Nombre: `'Dra. Elena Ruiz'`, Especialidad: `100` (Traumatología), Salario: `3200.00`.

**✈️ Aerolínea (Mantenimiento de Flota)** 7. **Ticket 301 (DELETE):** El avión más antiguo, el 'Airbus A320' (ID 10) de 2015, ha sido vendido a otra compañía. Bórralo de nuestra flota. 8. **Ticket 302 (UPDATE):** Por una nueva regulación del espacio aéreo, la ruta 'MADLHR' (Madrid-Londres) ahora tiene una desviación obligatoria. Súmale `150` km a la distancia original de esa ruta. 9. **TRANSACTION:** No olvides confirmar los cambios para que se guarden en disco.

<details>
<summary>👉 <b>Haz clic aquí SOLO cuando tengas tu script completo para auditar tu código</b></summary>

<b>Solución Exacta:</b>

```sql
-- 🛒 E-COMMERCE
-- Ticket 101: Descuento del 10% (Multiplicar por 0.90 es restar el 10%)
UPDATE productos
SET precio = precio * 0.90
WHERE id_categoria = 2;

-- Ticket 102: Actualizar stock a cero
UPDATE productos
SET stock = 0
WHERE id_producto = 12;

-- Ticket 103: Añadir nueva columna
ALTER TABLE productos
ADD (codigo_barras VARCHAR2(20));

-- 🏥 HOSPITAL CENTRAL
-- Ticket 201: Actualizar teléfono
UPDATE pacientes
SET telefono = '555-9999'
WHERE dni = '11111111A';

-- Ticket 202: Borrar médico jubilado
DELETE FROM medicos
WHERE id_medico = 1;

-- Ticket 203: Insertar nuevo médico
INSERT INTO medicos (id_medico, nombre_completo, id_especialidad, salario_base)
VALUES (4, 'Dra. Elena Ruiz', 100, 3200.00);

-- ✈️ AEROLÍNEA
-- Ticket 301: Borrar avión vendido
DELETE FROM aviones
WHERE id_avion = 10;

-- Ticket 302: Modificar distancia de la ruta
UPDATE rutas
SET distancia_km = distancia_km + 150
WHERE id_ruta = 'MADLHR';

-- Guardar los cambios
COMMIT;
```

</details>
