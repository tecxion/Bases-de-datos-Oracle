##1. Privilegios del Sistema

Estos privilegios permiten a los usuarios realizar ciertas operaciones en la base de datos.

CREATE SESSION: Permite al usuario conectarse a la base de datos.
CREATE TABLE: Permite al usuario crear tablas en su propio esquema.
CREATE VIEW: Permite al usuario crear vistas en su propio esquema.
CREATE PROCEDURE: Permite al usuario crear procedimientos y funciones almacenadas.
CREATE SEQUENCE: Permite al usuario crear secuencias.
CREATE TRIGGER: Permite al usuario crear triggers en su propio esquema.
CREATE SYNONYM: Permite al usuario crear sinónimos para los objetos de base de datos.
CREATE USER: Permite al usuario crear otros usuarios.
ALTER USER: Permite al usuario modificar los atributos de otros usuarios.
DROP USER: Permite al usuario eliminar usuarios de la base de datos.
ALTER SYSTEM: Permite al usuario modificar ciertos aspectos de la base de datos, como parámetros de sistema.
GRANT ANY PRIVILEGE: Permite al usuario conceder cualquier privilegio del sistema a otros usuarios o roles.
GRANT ANY ROLE: Permite al usuario conceder roles a otros usuarios o roles.
SELECT ANY TABLE: Permite al usuario consultar tablas de cualquier esquema.
INSERT ANY TABLE: Permite al usuario insertar registros en tablas de cualquier esquema.
UPDATE ANY TABLE: Permite al usuario actualizar registros en tablas de cualquier esquema.
DELETE ANY TABLE: Permite al usuario eliminar registros en tablas de cualquier esquema.
EXECUTE ANY PROCEDURE: Permite al usuario ejecutar procedimientos y funciones en cualquier esquema.

##2. Privilegios de Objeto
Estos privilegios permiten a los usuarios realizar acciones en objetos de la base de datos específicos (tablas, vistas, secuencias, etc.).

SELECT: Permite al usuario consultar datos en una tabla o vista.
INSERT: Permite al usuario insertar datos en una tabla.
UPDATE: Permite al usuario actualizar datos en una tabla.
DELETE: Permite al usuario eliminar datos de una tabla.
ALTER: Permite al usuario modificar la estructura de una tabla, como agregar o eliminar columnas.
INDEX: Permite al usuario crear índices en columnas de una tabla.
REFERENCES: Permite al usuario crear claves foráneas que referencian la tabla.
EXECUTE: Permite al usuario ejecutar procedimientos o funciones almacenadas.
FLASHBACK: Permite al usuario realizar operaciones de recuperación de datos (como FLASHBACK TABLE).

##3. Privilegios Administrativos
Estos privilegios permiten a los usuarios realizar tareas administrativas.

DBA: Un rol que otorga casi todos los privilegios de sistema, ideal para administradores de base de datos.
RESOURCE: Permite al usuario crear varios tipos de objetos de base de datos, como tablas y procedimientos.
CONNECT: Otorga permisos mínimos para conectarse a la base de datos.
EXP_FULL_DATABASE / IMP_FULL_DATABASE: Permiten al usuario realizar exportaciones e importaciones de la base de datos completa.
Resumen de Roles Comunes
DBA: Otorga casi todos los privilegios necesarios para administrar la base de datos.
RESOURCE: Permite al usuario crear y administrar objetos de base de datos.
CONNECT: Permite al usuario conectarse a la base de datos, generalmente un privilegio básico.
