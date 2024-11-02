## 1. Privilegios del Sistema<br>
<br>
Estos privilegios permiten a los usuarios realizar ciertas operaciones en la base de datos.<br>
CREATE SESSION: Permite al usuario conectarse a la base de datos.<br>
CREATE TABLE: Permite al usuario crear tablas en su propio esquema.<br>
CREATE VIEW: Permite al usuario crear vistas en su propio esquema.<br>
CREATE PROCEDURE: Permite al usuario crear procedimientos y funciones almacenadas.<br>
CREATE SEQUENCE: Permite al usuario crear secuencias.<br>
CREATE TRIGGER: Permite al usuario crear triggers en su propio esquema.<br>
CREATE SYNONYM: Permite al usuario crear sinónimos para los objetos de base de datos.<br>
CREATE USER: Permite al usuario crear otros usuarios.<br>
ALTER USER: Permite al usuario modificar los atributos de otros usuarios.<br>
DROP USER: Permite al usuario eliminar usuarios de la base de datos.<br>
ALTER SYSTEM: Permite al usuario modificar ciertos aspectos de la base de datos, como parámetros de sistema.<br>
GRANT ANY PRIVILEGE: Permite al usuario conceder cualquier privilegio del sistema a otros usuarios o roles.<br>
GRANT ANY ROLE: Permite al usuario conceder roles a otros usuarios o roles.<br>
SELECT ANY TABLE: Permite al usuario consultar tablas de cualquier esquema.<br>
INSERT ANY TABLE: Permite al usuario insertar registros en tablas de cualquier esquema.<br>
UPDATE ANY TABLE: Permite al usuario actualizar registros en tablas de cualquier esquema.<br>
DELETE ANY TABLE: Permite al usuario eliminar registros en tablas de cualquier esquema.<br>
EXECUTE ANY PROCEDURE: Permite al usuario ejecutar procedimientos y funciones en cualquier esquema.<br>

## 2. Privilegios de Objeto<br>
Estos privilegios permiten a los usuarios realizar acciones en objetos de la base de datos específicos (tablas, vistas, secuencias, etc.).<br>
<br>
SELECT: Permite al usuario consultar datos en una tabla o vista.<br>
INSERT: Permite al usuario insertar datos en una tabla.<br>
UPDATE: Permite al usuario actualizar datos en una tabla.<br>
DELETE: Permite al usuario eliminar datos de una tabla.<br>
ALTER: Permite al usuario modificar la estructura de una tabla, como agregar o eliminar columnas.<br>
INDEX: Permite al usuario crear índices en columnas de una tabla.<br>
REFERENCES: Permite al usuario crear claves foráneas que referencian la tabla.<br>
EXECUTE: Permite al usuario ejecutar procedimientos o funciones almacenadas.<br>
FLASHBACK: Permite al usuario realizar operaciones de recuperación de datos (como FLASHBACK TABLE).<br>

## 3. Privilegios Administrativos<br>
Estos privilegios permiten a los usuarios realizar tareas administrativas.<br>
<br>
DBA: Un rol que otorga casi todos los privilegios de sistema, ideal para administradores de base de datos.<br>
RESOURCE: Permite al usuario crear varios tipos de objetos de base de datos, como tablas y procedimientos.<br>
CONNECT: Otorga permisos mínimos para conectarse a la base de datos.<br>
EXP_FULL_DATABASE / IMP_FULL_DATABASE: Permiten al usuario realizar exportaciones e importaciones de la base de datos completa.<br>
*Resumen de Roles Comunes*<br>
DBA: Otorga casi todos los privilegios necesarios para administrar la base de datos.<br>
RESOURCE: Permite al usuario crear y administrar objetos de base de datos.<br>
CONNECT: Permite al usuario conectarse a la base de datos, generalmente un privilegio básico.<br>
