# OBJETOS DE LA BASE DE DATOS

Además de las tablas, un esquema de Oracle puede contener vistas, secuencias, sinónimos, triggers y procedimientos.

#### VISTAS

Una vista es una consulta guardada con un nombre. No almacena datos: los lee de las tablas cada vez que se consulta.

  - Crear una vista.
```sql
  CREATE VIEW nombrevista AS
  SELECT columna1, columna2 FROM nombretabla WHERE condición;
```
  - Crearla o reemplazarla si ya existe.
```sql
  CREATE OR REPLACE VIEW v_empleados_ventas AS
  SELECT e.empleado_id, e.nombre, e.salario, d.nombre AS departamento
  FROM empleados e
  JOIN departamentos d ON e.departamento_id = d.departamento_id
  WHERE d.nombre = 'Ventas';
```
  - Impedir que se inserten o actualicen filas que quedarían fuera de la vista.
```sql
  CREATE OR REPLACE VIEW v_madrid AS
  SELECT * FROM departamentos WHERE localidad = 'Madrid'
  WITH CHECK OPTION;
```
  - Vista de solo lectura.
```sql
  CREATE OR REPLACE VIEW v_empleados AS
  SELECT nombre, email FROM empleados
  WITH READ ONLY;
```
  - Borrar una vista. Las tablas no se tocan.
```sql
  DROP VIEW nombrevista;
```
  - Consultarla igual que una tabla.
```sql
  SELECT * FROM v_empleados_ventas WHERE salario > 40000;
```

>[!NOTE]
>Se puede hacer `INSERT`, `UPDATE` y `DELETE` sobre una vista **simple** (una sola tabla, sin `GROUP BY`, sin `DISTINCT`, sin funciones de grupo). Si la vista tiene un `JOIN` o agrupa, es de solo lectura.

##### Vistas materializadas

A diferencia de una vista normal, **sí guarda el resultado en disco**. Se usa para consultas pesadas que no necesitan datos al segundo.

```sql
  CREATE MATERIALIZED VIEW mv_ventas_mes
  REFRESH COMPLETE ON DEMAND
  AS SELECT TRUNC(fecha,'MM') AS mes, SUM(importe) AS total
     FROM ventas GROUP BY TRUNC(fecha,'MM');
```
  - Actualizarla a mano.
```sql
  EXEC DBMS_MVIEW.REFRESH('MV_VENTAS_MES');
```

#### SECUENCIAS

Generan números correlativos. Es la forma clásica de rellenar una clave primaria.

  - Crear una secuencia.
```sql
  CREATE SEQUENCE nombresecuencia
    START WITH 1
    INCREMENT BY 1
    NOCACHE
    NOCYCLE;
```
  - Obtener el siguiente valor. Cada llamada avanza el contador.
```sql
  SELECT nombresecuencia.NEXTVAL FROM DUAL;
```
  - Ver el último valor obtenido en esta sesión, sin avanzar.
```sql
  SELECT nombresecuencia.CURRVAL FROM DUAL;
```
  - Usarla en un `INSERT`.
```sql
  INSERT INTO empleados (empleado_id, nombre)
  VALUES (seq_empleados.NEXTVAL, 'Ana Ruiz');
```
  - Modificar el incremento (no se puede cambiar el `START WITH`).
```sql
  ALTER SEQUENCE nombresecuencia INCREMENT BY 10;
```
  - Borrar la secuencia.
```sql
  DROP SEQUENCE nombresecuencia;
```

>[!CAUTION]
>Los números de una secuencia **no son consecutivos garantizados**. Un `ROLLBACK` no devuelve el número consumido, y con `CACHE` se pierden huecos al reiniciar la base de datos. Una secuencia garantiza unicidad, no una serie sin saltos.

##### Columnas de identidad (Oracle 12c en adelante)

Es la forma moderna: Oracle crea y gestiona la secuencia por ti.

```sql
  CREATE TABLE empleados (
    empleado_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre      VARCHAR2(50) NOT NULL
  );

  INSERT INTO empleados (nombre) VALUES ('Ana Ruiz');   -- el id se rellena solo
```

#### SINÓNIMOS

Un sinónimo es un alias para un objeto, normalmente de otro esquema. Evita escribir `esquema.tabla` en todas partes.

  - Sinónimo privado, solo lo ve quien lo crea.
```sql
  CREATE SYNONYM emp FOR hr.empleados;
```
  - Sinónimo público, lo ven todos los usuarios de la base de datos.
```sql
  CREATE PUBLIC SYNONYM emp FOR hr.empleados;
```
  - Borrarlo.
```sql
  DROP SYNONYM emp;
  DROP PUBLIC SYNONYM emp;
```

#### TRIGGERS (DISPARADORES)

Un bloque de código que Oracle ejecuta **automáticamente** cuando ocurre un evento sobre una tabla.

  - Estructura general.
```sql
  CREATE OR REPLACE TRIGGER nombretrigger
  BEFORE | AFTER   INSERT | UPDATE | DELETE   ON nombretabla
  [FOR EACH ROW]
  [WHEN (condición)]
  BEGIN
    -- código PL/SQL
  END;
  /
```
  - Ejemplo: rellenar la clave primaria con una secuencia antes de insertar.
```sql
  CREATE OR REPLACE TRIGGER trg_empleados_pk
  BEFORE INSERT ON empleados
  FOR EACH ROW
  BEGIN
    :NEW.empleado_id := seq_empleados.NEXTVAL;
  END;
  /
```
  - Ejemplo: auditar los cambios de salario.
```sql
  CREATE OR REPLACE TRIGGER trg_auditar_salario
  AFTER UPDATE OF salario ON empleados
  FOR EACH ROW
  WHEN (OLD.salario <> NEW.salario)
  BEGIN
    INSERT INTO auditoria_salarios (empleado_id, salario_viejo, salario_nuevo, fecha)
    VALUES (:OLD.empleado_id, :OLD.salario, :NEW.salario, SYSDATE);
  END;
  /
```
  - Ejemplo: impedir modificaciones fuera del horario laboral.
```sql
  CREATE OR REPLACE TRIGGER trg_horario
  BEFORE INSERT OR UPDATE OR DELETE ON empleados
  BEGIN
    IF TO_CHAR(SYSDATE,'HH24') NOT BETWEEN '09' AND '18' THEN
      RAISE_APPLICATION_ERROR(-20001, 'Solo se puede modificar en horario laboral');
    END IF;
  END;
  /
```
  - Activar, desactivar y borrar.
```sql
  ALTER TRIGGER nombretrigger DISABLE;
  ALTER TRIGGER nombretrigger ENABLE;
  DROP TRIGGER nombretrigger;
```

>[!IMPORTANT]
>Dentro de un trigger `FOR EACH ROW`, `:OLD` es la fila antes del cambio y `:NEW` la fila después. En un `INSERT` solo existe `:NEW`; en un `DELETE`, solo `:OLD`. Solo se puede **asignar** a `:NEW` en un trigger `BEFORE`.

>[!NOTE]
>La barra `/` en una línea suelta al final le dice a SQL*Plus "ejecuta el bloque". Es obligatoria en todo bloque PL/SQL (triggers, procedimientos, funciones).

#### PROCEDIMIENTOS Y FUNCIONES

Bloques de PL/SQL guardados en la base de datos. Un **procedimiento** ejecuta acciones; una **función** devuelve un valor y se puede usar dentro de un `SELECT`.

  - Procedimiento. Los parámetros pueden ser `IN` (entrada, por defecto), `OUT` (salida) o `IN OUT`.
```sql
  CREATE OR REPLACE PROCEDURE subir_salario (
    p_empleado_id IN NUMBER,
    p_porcentaje  IN NUMBER
  ) AS
  BEGIN
    UPDATE empleados
    SET salario = salario * (1 + p_porcentaje / 100)
    WHERE empleado_id = p_empleado_id;
    COMMIT;
  END;
  /
```
  - Ejecutarlo.
```sql
  EXEC subir_salario(101, 5);
```
  - Función. Tiene que llevar `RETURN` en la cabecera y en el cuerpo.
```sql
  CREATE OR REPLACE FUNCTION antiguedad (p_empleado_id IN NUMBER)
  RETURN NUMBER AS
    v_anios NUMBER;
  BEGIN
    SELECT TRUNC(MONTHS_BETWEEN(SYSDATE, fecha_alta) / 12)
    INTO v_anios
    FROM empleados
    WHERE empleado_id = p_empleado_id;

    RETURN v_anios;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN RETURN NULL;
  END;
  /
```
  - Usarla en una consulta.
```sql
  SELECT nombre, antiguedad(empleado_id) AS anios FROM empleados;
```
  - Ver los errores de compilación si el objeto se crea "con errores de compilación".
```sql
  SHOW ERRORS;
```
  - Borrarlos.
```sql
  DROP PROCEDURE subir_salario;
  DROP FUNCTION antiguedad;
```

>[!CAUTION]
>Una función que se usa dentro de un `SELECT` no debería contener `COMMIT` ni modificar tablas. Oracle puede lanzar `ORA-14551: no se puede realizar una operación DML dentro de una consulta`.

#### BLOQUES PL/SQL ANÓNIMOS

Código PL/SQL que se ejecuta una vez, sin guardarse.

```sql
  SET SERVEROUTPUT ON;

  DECLARE
    v_total NUMBER;
  BEGIN
    SELECT COUNT(*) INTO v_total FROM empleados;
    DBMS_OUTPUT.PUT_LINE('Hay ' || v_total || ' empleados');
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
  END;
  /
```

>[!NOTE]
>Sin `SET SERVEROUTPUT ON` los `DBMS_OUTPUT.PUT_LINE` no muestran nada y parece que el bloque no hace nada.
