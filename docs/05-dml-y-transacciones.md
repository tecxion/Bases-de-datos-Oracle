# DML Y TRANSACCIONES

**DML** (*Data Manipulation Language*) son las sentencias que manipulan los **datos** de las tablas: `INSERT`, `UPDATE`, `DELETE` y `MERGE`. Frente al **DDL** (`CREATE`, `ALTER`, `DROP`), que manipula la **estructura**.

#### INSERT
  - Insertar una fila indicando las columnas. Es la forma recomendada: si mañana se añade una columna, la sentencia sigue funcionando.
```sql
  INSERT INTO empleados (empleado_id, nombre, salario)
  VALUES (101, 'Ana Ruiz', 32000);
```
  - Insertar sin nombrar las columnas: hay que dar un valor para **todas**, en el orden exacto de la tabla.
```sql
  INSERT INTO empleados VALUES (101, 'Ana Ruiz', 'ana@empresa.com', 32000, SYSDATE, 10);
```
  - Insertar el resultado de una consulta (copia masiva).
```sql
  INSERT INTO empleados_historico (empleado_id, nombre)
  SELECT empleado_id, nombre FROM empleados WHERE fecha_baja IS NOT NULL;
```
  - Insertar varias filas de golpe con `INSERT ALL`.
```sql
  INSERT ALL
    INTO departamentos VALUES (10, 'Ventas',      'Madrid')
    INTO departamentos VALUES (20, 'Informática', 'Sevilla')
    INTO departamentos VALUES (30, 'Contabilidad','Bilbao')
  SELECT * FROM DUAL;
```

>[!NOTE]
>`DUAL` es una tabla del sistema con una única fila y una única columna. Sirve para ejecutar expresiones que no leen de ninguna tabla real: `SELECT SYSDATE FROM DUAL;`.

#### UPDATE
  - Actualizar filas que cumplen una condición.
```sql
  UPDATE empleados
  SET salario = salario * 1.05
  WHERE departamento_id = 20;
```
  - Actualizar varias columnas a la vez.
```sql
  UPDATE empleados
  SET salario = 40000, departamento_id = 30
  WHERE empleado_id = 101;
```
  - Actualizar con el valor de una subconsulta.
```sql
  UPDATE empleados e
  SET salario = (SELECT AVG(salario) FROM empleados WHERE departamento_id = e.departamento_id)
  WHERE e.salario IS NULL;
```

>[!CAUTION]
>Un `UPDATE` o un `DELETE` **sin `WHERE` afecta a todas las filas de la tabla**. Antes de ejecutarlo, prueba la condición con un `SELECT` y comprueba cuántas filas devuelve.

#### DELETE
  - Borrar las filas que cumplen una condición.
```sql
  DELETE FROM empleados WHERE fecha_baja < DATE '2020-01-01';
```
  - Borrar todas las filas (se puede deshacer con `ROLLBACK`).
```sql
  DELETE FROM empleados;
```

#### MERGE

Combina `INSERT` y `UPDATE` en una sola sentencia: si la fila existe la actualiza, si no la inserta. Se le llama *upsert*.

```sql
  MERGE INTO empleados e
  USING empleados_nuevos n
     ON (e.empleado_id = n.empleado_id)
  WHEN MATCHED THEN
    UPDATE SET e.salario = n.salario, e.departamento_id = n.departamento_id
  WHEN NOT MATCHED THEN
    INSERT (empleado_id, nombre, salario)
    VALUES (n.empleado_id, n.nombre, n.salario);
```

#### TRANSACCIONES

Una transacción es un conjunto de sentencias DML que se confirman o se descartan **como un bloque**. Empieza sola con la primera sentencia DML y termina con `COMMIT` o `ROLLBACK`.

  - Confirmar los cambios: se hacen permanentes y visibles para los demás usuarios.
```sql
  COMMIT;
```
  - Deshacer todos los cambios desde el último `COMMIT`.
```sql
  ROLLBACK;
```
  - Marcar un punto intermedio al que poder volver.
```sql
  SAVEPOINT antes_de_borrar;
```
  - Deshacer solo hasta ese punto, conservando lo anterior.
```sql
  ROLLBACK TO SAVEPOINT antes_de_borrar;
```
  - Ejemplo completo.
```sql
  INSERT INTO departamentos VALUES (40, 'Marketing', 'Valencia');
  SAVEPOINT depto_creado;

  DELETE FROM empleados WHERE departamento_id = 10;
  ROLLBACK TO SAVEPOINT depto_creado;   -- deshace el DELETE, mantiene el INSERT

  COMMIT;                                -- confirma solo el INSERT
```

>[!IMPORTANT]
>El **DDL hace `COMMIT` implícito**. Si ejecutas un `CREATE TABLE` o un `ALTER TABLE` en medio de una transacción, todo lo que hubieras insertado o borrado antes queda confirmado y ya no puedes hacer `ROLLBACK`.

>[!CAUTION]
>Mientras no hagas `COMMIT`, tus cambios solo los ves tú, y las filas modificadas quedan **bloqueadas** para los demás usuarios. Un `UPDATE` sin confirmar puede dejar colgada la sesión de otro compañero.

#### PROPIEDADES ACID

 - **Atomicidad**: la transacción se ejecuta entera o no se ejecuta nada.
 - **Consistencia**: al terminar, la base de datos sigue cumpliendo todas las restricciones.
 - **Aislamiento** (*Isolation*): las transacciones concurrentes no se ven entre sí hasta que confirman.
 - **Durabilidad**: una vez hecho `COMMIT`, los datos sobreviven a un corte de luz.

#### BLOQUEOS
  - Bloquear filas para leerlas y modificarlas sin que nadie las toque mientras tanto.
```sql
  SELECT * FROM empleados WHERE empleado_id = 101 FOR UPDATE;
```
  - Si la fila ya está bloqueada, fallar de inmediato en vez de esperar.
```sql
  SELECT * FROM empleados WHERE empleado_id = 101 FOR UPDATE NOWAIT;
```
  - Esperar como mucho 5 segundos.
```sql
  SELECT * FROM empleados WHERE empleado_id = 101 FOR UPDATE WAIT 5;
```
