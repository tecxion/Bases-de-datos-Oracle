# TABLAS, RESTRICCIONES E ÍNDICES

#### BASES DE DATOS

  - Creación de una Base de Datos.
```sql
  CREATE DATABASE Nombrebd;
```

>[!CAUTION]
>En Oracle **no se crea una base de datos por proyecto**, como en MySQL. Una instancia de Oracle *es* la base de datos, y dentro de ella cada usuario tiene su propio **esquema** con sus tablas. `CREATE DATABASE` es una operación de administrador que se ejecuta una sola vez al instalar. Lo que en MySQL sería "crear una base de datos" aquí es `CREATE USER`.

#### CREACIÓN DE TABLAS

  - Creación de una tabla: a continuación escribiríamos los atributos con el tipo y las restricciones.
```sql
  CREATE TABLE nombretabla (
    columna1 tipo [restricción],
    columna2 tipo [restricción],
    ...
  );
```
  - Ejemplo completo con restricciones **en línea** (declaradas junto a la columna).
```sql
  CREATE TABLE departamentos (
    departamento_id   NUMBER(4)    PRIMARY KEY,
    nombre            VARCHAR2(30) NOT NULL UNIQUE,
    localidad         VARCHAR2(50) DEFAULT 'Madrid'
  );
```
  - Ejemplo con restricciones **de tabla** (al final, con nombre propio). Es la forma recomendada: los errores dicen qué restricción ha fallado.
```sql
  CREATE TABLE empleados (
    empleado_id     NUMBER(6),
    nombre          VARCHAR2(50) NOT NULL,
    email           VARCHAR2(80),
    salario         NUMBER(8,2),
    fecha_alta      DATE DEFAULT SYSDATE,
    departamento_id NUMBER(4),
    CONSTRAINT pk_empleados      PRIMARY KEY (empleado_id),
    CONSTRAINT uq_emp_email      UNIQUE (email),
    CONSTRAINT ck_emp_salario    CHECK (salario > 0),
    CONSTRAINT fk_emp_depto      FOREIGN KEY (departamento_id)
                                 REFERENCES departamentos (departamento_id)
  );
```
  - Crear una tabla copiando la estructura y los datos de otra.
```sql
  CREATE TABLE copia_empleados AS SELECT * FROM empleados;
```
  - Crear solo la estructura, sin datos: basta con una condición que nunca se cumpla.
```sql
  CREATE TABLE copia_empleados AS SELECT * FROM empleados WHERE 1 = 0;
```
  - Eliminar una tabla. El comando `DROP` también sirve para usuarios, vistas, índices...
```sql
  DROP TABLE Nombretabla;
```
  - Eliminar una tabla que tiene claves foráneas apuntándola.
```sql
  DROP TABLE nombretabla CASCADE CONSTRAINTS;
```
  - Eliminar una tabla sin que pase por la papelera de reciclaje.
```sql
  DROP TABLE nombretabla PURGE;
```

>[!NOTE]
>Sin `PURGE`, la tabla va a la papelera (`RECYCLEBIN`) y se puede recuperar con `FLASHBACK TABLE nombretabla TO BEFORE DROP;`. Ver la papelera con `SELECT * FROM RECYCLEBIN;`.

#### MODIFICAR TABLAS

  - Cambiar el nombre a una tabla.
```sql
  RENAME nombreviejo TO nombrenuevo;
```
  - Lo mismo, con la sintaxis de `ALTER TABLE`.
```sql
  ALTER TABLE nombreviejo RENAME TO nombrenuevo;
```
  - Añadir Columna.
```sql
  ALTER TABLE nombretabla ADD nombrecolumna tipo [propiedades];
```
  - Añadir varias columnas de una vez.
```sql
  ALTER TABLE empleados ADD (telefono VARCHAR2(15), activo CHAR(1) DEFAULT 'S');
```
  - Eliminar Columna.
```sql
  ALTER TABLE nombretabla DROP COLUMN nombrecolumna;
```
  - Modificar Columna: cambiar su tipo, tamaño o valor por defecto.
```sql
  ALTER TABLE nombretabla MODIFY nombrecolumna tipo [propiedades];
```
  - Ejemplo: ampliar el tamaño y hacerla obligatoria.
```sql
  ALTER TABLE empleados MODIFY nombre VARCHAR2(100) NOT NULL;
```
  - Renombrar Columna.
```sql
  ALTER TABLE nombretabla RENAME COLUMN nombreviejocolumna TO nombrenuevocolumna;
```
  - Marcar una columna como no usada. Es instantáneo aunque la tabla sea enorme; el espacio se libera después.
```sql
  ALTER TABLE nombretabla SET UNUSED COLUMN nombrecolumna;
  ALTER TABLE nombretabla DROP UNUSED COLUMNS;
```

>[!IMPORTANT]
>No se puede reducir el tamaño de una columna (`VARCHAR2(100)` → `VARCHAR2(50)`) ni cambiar su tipo si la tabla tiene datos. Hay que vaciarla o crear una columna nueva, copiar y borrar la vieja.

#### RESTRICCIONES

Una restricción (*constraint*) es una regla que los datos tienen que cumplir. Oracle rechaza cualquier `INSERT` o `UPDATE` que la incumpla.

 - **NOT NULL**: Prohibe valores nulos.
 - **UNIQUE**: No se pueden repetir valores. Sí admite varios `NULL`.
 - **PRIMARY KEY**: Clave Primaria, será not null y unique. Solo puede haber una por tabla.
 - **FOREIGN KEY**: Claves externas que relacionan tablas. El valor tiene que existir en la tabla referenciada.
 - **CHECK**: El valor tiene que cumplir una condición.
 - **DEFAULT**: Valor por defecto. Técnicamente no es una restricción, sino una propiedad de la columna.

  - Añadir una restricción a una tabla ya creada.
```sql
  ALTER TABLE nombretabla ADD CONSTRAINT nombrerestriccion PRIMARY KEY (columna);
```
  - Añadir una clave foránea.
```sql
  ALTER TABLE empleados ADD CONSTRAINT fk_emp_depto
    FOREIGN KEY (departamento_id) REFERENCES departamentos (departamento_id);
```
  - Añadir un `CHECK`.
```sql
  ALTER TABLE empleados ADD CONSTRAINT ck_emp_salario CHECK (salario BETWEEN 1000 AND 200000);
```
  - Añadir un `NOT NULL`: es la excepción, se hace con `MODIFY`.
```sql
  ALTER TABLE empleados MODIFY nombre NOT NULL;
```
  - Borrar Restricciones.
```sql
  ALTER TABLE nombretabla DROP CONSTRAINT nombrerestriccion;
```
  - Borrar la clave primaria y todas las foráneas que la referencian.
```sql
  ALTER TABLE nombretabla DROP PRIMARY KEY CASCADE;
```
  - Modificar Restricción (solo se puede renombrar; para cambiar la regla hay que borrarla y crearla).
```sql
  ALTER TABLE nombretabla RENAME CONSTRAINT nombreviejo TO nombrenuevo;
```
  - Activar o Desactivar restricciones. Útil al cargar muchos datos de golpe.
```sql
  ALTER TABLE nombretabla ENABLE CONSTRAINT Nombrerestriccion;
  ALTER TABLE nombretabla DISABLE CONSTRAINT Nombrerestriccion;
```

##### Comportamiento del borrado en claves foráneas

Qué pasa al borrar una fila padre que tiene hijos apuntándola:

 - Por defecto: Oracle **rechaza** el borrado (`ORA-02292`).
 - `ON DELETE CASCADE`: borra también las filas hijas.
 - `ON DELETE SET NULL`: pone a `NULL` la columna de las filas hijas.

```sql
  CONSTRAINT fk_emp_depto FOREIGN KEY (departamento_id)
    REFERENCES departamentos (departamento_id) ON DELETE CASCADE
```

>[!NOTE]
>Oracle no tiene `ON UPDATE CASCADE`. Si cambia la clave del padre, hay que actualizar los hijos a mano.

  - Consultar las restricciones de una tabla.
```sql
  SELECT constraint_name, constraint_type, status
  FROM USER_CONSTRAINTS
  WHERE table_name = 'EMPLEADOS';
```

  Los valores de `constraint_type` son: `P` primary key, `R` foreign key (*referential*), `U` unique, `C` check y not null.

#### CREACIÓN Y MODIFICACIÓN DE ÍNDICES

Un índice acelera las búsquedas por una columna, a costa de ralentizar los `INSERT` y `UPDATE`.

  - Crear Índice.
```sql
  CREATE INDEX nombreindice ON nombretabla (columna1 [tipo1], columna2 [tipo2]...);
```
  - Índice único: además de acelerar, impide valores repetidos.
```sql
  CREATE UNIQUE INDEX idx_emp_email ON empleados (email);
```
  - Índice sobre una expresión, no sobre la columna en bruto.
```sql
  CREATE INDEX idx_emp_nombre_upper ON empleados (UPPER(nombre));
```
  - Reconstruir un índice degradado por muchos borrados.
```sql
  ALTER INDEX nombreindice REBUILD;
```
  - Borrar Índice.
```sql
  DROP INDEX nombreindice;
```
  - Ver los índices de una tabla.
```sql
  SELECT index_name, uniqueness FROM USER_INDEXES WHERE table_name = 'EMPLEADOS';
```

>[!IMPORTANT]
>Oracle crea automáticamente un índice único al declarar una `PRIMARY KEY` o un `UNIQUE`. No hace falta crearlo a mano. En cambio **no** indexa las claves foráneas: hazlo tú si consultas mucho por ellas.

#### VACIAR UNA TABLA
  - `DELETE` borra fila a fila, se puede deshacer con `ROLLBACK` y admite `WHERE`.
```sql
  DELETE FROM empleados;
```
  - `TRUNCATE` vacía la tabla de golpe. Es mucho más rápido, pero **no se puede deshacer** y no admite `WHERE`.
```sql
  TRUNCATE TABLE empleados;
```
