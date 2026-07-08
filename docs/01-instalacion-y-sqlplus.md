# INSTALACIÓN E INICIO

>[!NOTE]
>SQL*Plus es el cliente de línea de comandos que trae Oracle. Todo lo que se ve aquí se escribe en la consola de `sqlplus`, no dentro de una sentencia SQL.

### CONECTARSE A LA BASE DE DATOS
  - Para iniciar sesión en sql utilizaremos el comando `sqlplus` en cmd. Introduciremos el usuario y contraseña generados en la instalación de ORACLE SQL.
```sql
  sqlplus
```
  - Conectarse directamente indicando usuario y contraseña.
```sql
  sqlplus nombreusuario/contraseña
```
  - Conectarse indicando además la instancia (cadena de conexión).
```sql
  sqlplus nombreusuario/contraseña@localhost:1521/XEPDB1
```
  - Conectarse como administrador (DBA).
```sql
  sqlplus / AS SYSDBA
```
  - Cambiar de usuario sin salir de la sesión.
```sql
  CONNECT nombreusuario/contraseña;
```
  - Ver con qué usuario estamos conectados.
```sql
  SHOW USER;
```
  - Salir de SQL*Plus.
```sql
  EXIT;
```

>[!IMPORTANT]
>En Oracle las sentencias SQL terminan siempre en `;`. Los comandos propios de SQL*Plus (`SHOW`, `SET`, `SPOOL`, `DESCRIBE`...) no lo necesitan, aunque tampoco molesta.

### CONTENEDORES Y ESQUEMAS (CDB / PDB)

Desde Oracle 12c la base de datos se divide en un contenedor raíz (**CDB**) y bases de datos conectables (**PDB**). Casi siempre trabajaremos dentro de una PDB.

  - Ver el contenedor actual.
```sql
  SHOW CON_NAME;
```
  - Listar las PDB disponibles (hay que estar conectado como DBA).
```sql
  SELECT name, open_mode FROM v$pdbs;
```
  - Cambiar a una PDB.
```sql
  ALTER SESSION SET CONTAINER = XEPDB1;
```

>[!CAUTION]
>Si al crear un usuario te da el error `ORA-65096: nombre de usuario o rol común no válido`, es que estás conectado al contenedor raíz (`CDB$ROOT`) en vez de a una PDB. Cambia de contenedor con `ALTER SESSION SET CONTAINER` o crea el usuario con el prefijo `C##`.

### DESCRIBIR OBJETOS
  - Ver la estructura de una tabla o vista: columnas, si admiten nulos y su tipo.
```sql
  DESCRIBE nombretabla;
```
  - Se puede abreviar como `DESC`.
```sql
  DESC SYS.ALL_USERS;
```

### FORMATEAR LA SALIDA

La salida por defecto de SQL*Plus queda descuadrada. Estos comandos la hacen legible.

  - Ancho de línea en caracteres, antes de partir la fila.
```sql
  SET LINESIZE 200;
```
  - Número de filas antes de repetir la cabecera. Con `0` no se repite nunca.
```sql
  SET PAGESIZE 50;
```
  - Ancho de una columna concreta.
```sql
  COLUMN nombrecolumna FORMAT A30;
```
  - Formato de una columna numérica.
```sql
  COLUMN salario FORMAT 999G999D99;
```
  - Mostrar la sentencia SQL que ejecuta un script.
```sql
  SET ECHO ON;
```
  - Ocultar el mensaje de filas devueltas ("30 filas seleccionadas").
```sql
  SET FEEDBACK OFF;
```
  - Guardar los ajustes de formato de una sesión.
```sql
  STORE SET mi_formato.sql;
```

### SCRIPTS Y FICHEROS
  - Ejecutar un script `.sql` guardado en disco.
```sql
  @ruta/del/script.sql
```
  - Ejecutar un script en la sesión actual (no abre una nueva).
```sql
  @@script.sql
```
  - Volcar a un fichero todo lo que salga por pantalla.
```sql
  SPOOL salida.txt
  SELECT * FROM empleados;
  SPOOL OFF
```
  - Guardar la última sentencia ejecutada en un fichero.
```sql
  SAVE consulta.sql;
```
  - Editar la última sentencia en el editor de texto del sistema.
```sql
  EDIT;
```
  - Volver a ejecutar la última sentencia del búfer.
```sql
  /
```

### VARIABLES DE SUSTITUCIÓN

Sirven para parametrizar un script y que pida los valores al ejecutarlo.

  - Con `&` SQL*Plus pregunta el valor cada vez que aparece.
```sql
  SELECT * FROM empleados WHERE departamento_id = &depto;
```
  - Con `&&` lo pregunta una sola vez y lo reutiliza en toda la sesión.
```sql
  SELECT * FROM empleados WHERE departamento_id = &&depto;
```
  - Definir una variable sin que la pregunte.
```sql
  DEFINE depto = 10;
```
  - Borrar una variable definida.
```sql
  UNDEFINE depto;
```

### AYUDA Y DIAGNÓSTICO
  - Ver el mensaje completo de un error de Oracle desde la terminal del sistema.
```sql
  oerr ora 00942
```
  - Mostrar los errores de compilación del último objeto PL/SQL creado.
```sql
  SHOW ERRORS;
```
  - Activar la salida de `DBMS_OUTPUT.PUT_LINE` (por defecto está apagada).
```sql
  SET SERVEROUTPUT ON;
```
