# DICCIONARIO DE DATOS

El diccionario de datos son las tablas y vistas donde Oracle guarda información **sobre la propia base de datos**: qué tablas existen, qué columnas tienen, quién puede verlas. Se consultan con `SELECT` como cualquier otra tabla.

#### LOS TRES PREFIJOS

Casi todas las vistas del diccionario existen tres veces, cambiando solo el prefijo:

 - `USER_*` : objetos **de mi propio esquema**. No tiene columna `owner`.
 - `ALL_*` : objetos **a los que tengo acceso**, sean míos o de otros.
 - `DBA_*` : **todos** los objetos de la base de datos. Requiere privilegios de administrador.

```sql
  SELECT table_name FROM USER_TABLES;                       -- mis tablas
  SELECT owner, table_name FROM ALL_TABLES;                 -- las que puedo consultar
  SELECT owner, table_name FROM DBA_TABLES;                 -- todas (necesita DBA)
```

>[!IMPORTANT]
>El diccionario guarda los nombres **en mayúsculas**. `WHERE table_name = 'empleados'` no devuelve nada; hay que escribir `WHERE table_name = 'EMPLEADOS'` o usar `UPPER()`.

#### VISTAS MÁS USADAS

| Vista | Qué contiene |
|---|---|
| `USER_TABLES` | Mis tablas |
| `USER_TAB_COLUMNS` | Columnas de mis tablas, con tipo y longitud |
| `USER_CONSTRAINTS` | Restricciones (PK, FK, unique, check) |
| `USER_CONS_COLUMNS` | Qué columnas participan en cada restricción |
| `USER_INDEXES` | Índices |
| `USER_IND_COLUMNS` | Columnas de cada índice |
| `USER_VIEWS` | Vistas y su consulta original |
| `USER_SEQUENCES` | Secuencias y su valor actual |
| `USER_TRIGGERS` | Triggers y su código |
| `USER_OBJECTS` | Todos mis objetos, de cualquier tipo |
| `USER_SOURCE` | Código fuente de procedimientos, funciones y paquetes |
| `USER_SYS_PRIVS` | Privilegios de sistema que tengo |
| `USER_TAB_PRIVS` | Privilegios sobre objetos que me han concedido |
| `USER_ROLE_PRIVS` | Roles que tengo asignados |
| `ALL_USERS` | Todos los usuarios de la base de datos |
| `DICT` | Índice de todas las vistas del diccionario |

#### CONSULTAS ÚTILES

  - Ver todas mis tablas y cuántas filas tienen (aproximado, según las últimas estadísticas).
```sql
  SELECT table_name, num_rows FROM USER_TABLES ORDER BY table_name;
```
  - Ver las columnas de una tabla con su tipo, tamaño y si admiten nulos.
```sql
  SELECT column_name, data_type, data_length, nullable, data_default
  FROM USER_TAB_COLUMNS
  WHERE table_name = 'EMPLEADOS'
  ORDER BY column_id;
```
  - Ver las restricciones de una tabla.
```sql
  SELECT constraint_name, constraint_type, status, search_condition
  FROM USER_CONSTRAINTS
  WHERE table_name = 'EMPLEADOS';
```

  `constraint_type`: `P` primary key · `R` foreign key · `U` unique · `C` check y not null.

  - Ver las claves foráneas y a qué tabla apuntan.
```sql
  SELECT c.constraint_name, c.table_name, r.table_name AS tabla_referenciada
  FROM USER_CONSTRAINTS c
  JOIN USER_CONSTRAINTS r ON c.r_constraint_name = r.constraint_name
  WHERE c.constraint_type = 'R';
```
  - Buscar en qué tablas aparece una columna con un nombre determinado.
```sql
  SELECT table_name, column_name
  FROM USER_TAB_COLUMNS
  WHERE column_name LIKE '%EMAIL%';
```
  - Ver la consulta con la que se creó una vista.
```sql
  SELECT text FROM USER_VIEWS WHERE view_name = 'V_EMPLEADOS_VENTAS';
```
  - Ver el código de un procedimiento o función.
```sql
  SELECT line, text FROM USER_SOURCE
  WHERE name = 'SUBIR_SALARIO' ORDER BY line;
```
  - Ver los objetos que están inválidos (no compilan).
```sql
  SELECT object_name, object_type FROM USER_OBJECTS WHERE status = 'INVALID';
```
  - Recuperar la sentencia `CREATE` completa de cualquier objeto.
```sql
  SELECT DBMS_METADATA.GET_DDL('TABLE', 'EMPLEADOS') FROM DUAL;
```
  - Ver qué tablas de la papelera se pueden recuperar.
```sql
  SELECT original_name, object_name, droptime FROM RECYCLEBIN;
```
  - Vaciar la papelera.
```sql
  PURGE RECYCLEBIN;
```

#### INFORMACIÓN DE LA INSTANCIA
  - Versión de Oracle.
```sql
  SELECT * FROM V$VERSION;
```
  - Nombre de la base de datos y del contenedor.
```sql
  SELECT name FROM V$DATABASE;
  SHOW CON_NAME;
```
  - Sesiones abiertas en este momento.
```sql
  SELECT sid, serial#, username, status, program FROM V$SESSION WHERE username IS NOT NULL;
```
  - Parámetros de idioma y formato de la sesión.
```sql
  SELECT * FROM NLS_SESSION_PARAMETERS;
```

>[!NOTE]
>Las vistas que empiezan por `V$` son *vistas dinámicas de rendimiento*: no leen de disco, muestran el estado de la instancia en memoria en este instante. Suelen necesitar privilegios de DBA.
