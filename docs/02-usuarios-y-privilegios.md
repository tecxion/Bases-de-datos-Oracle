# USUARIOS, PRIVILEGIOS Y ROLES

#### USUARIOS
  - Crear Usuario y dar contraseña.
```sql
  CREATE USER nombreusuario IDENTIFIED BY contraseña;
```
  - También podemos añadir el espacio que se le asigna a un usuario y el perfil.
```sql
  CREATE USER nombreusuario IDENTIFIED BY contraseña
  DEFAULT TABLESPACE users
  QUOTA tamañodelespacio ON users
  PROFILE nombreperfil;
```
  - Ver todos los usuarios.
```sql
  SELECT username, created FROM ALL_USERS ORDER BY username;
```
  - Ver la estructura de la vista de usuarios (qué columnas tiene).
```sql
  DESC SYS.ALL_USERS;
```
  - Cambiar la contraseña de un usuario.
```sql
  ALTER USER nombreusuario IDENTIFIED BY contraseñanueva;
```
  - Bloquear y desbloquear una cuenta.
```sql
  ALTER USER nombreusuario ACCOUNT LOCK;
  ALTER USER nombreusuario ACCOUNT UNLOCK;
```
  - Forzar a que el usuario cambie la contraseña en su próximo inicio de sesión.
```sql
  ALTER USER nombreusuario PASSWORD EXPIRE;
```
  - Ampliar la cuota de espacio de un usuario. `UNLIMITED` la deja sin límite.
```sql
  ALTER USER nombreusuario QUOTA UNLIMITED ON users;
```
  - Borrar un usuario.
```sql
  DROP USER nombreusuario;
```
  - Borrar un usuario **y todos los objetos de su esquema** (tablas, vistas...). Es obligatorio si el usuario tiene objetos creados.
```sql
  DROP USER nombreusuario CASCADE;
```

>[!CAUTION]
>`DROP USER ... CASCADE` elimina de forma irreversible todas las tablas y datos del usuario. No hay confirmación.

#### PRIVILEGIOS DE LOS USUARIOS

Un usuario recién creado no puede ni conectarse. Hay que concederle privilegios.

  - **GRANT**: concedemos (añadimos los privilegios) // `TO` al usuario que se le conceden // `ALL` puede ser todos los privilegios // `PUBLIC` privilegios a todos los usuarios.
  - Privilegio de sistema (permite hacer algo en la base de datos).
```sql
  GRANT CREATE SESSION, CREATE TABLE TO nombreusuario;
```
  - Privilegio de objeto (permite hacer algo sobre una tabla concreta). Fíjate en el `ON`.
```sql
  GRANT SELECT, UPDATE, INSERT ON nombretabla TO nombreusuario;
```
  - Conceder todos los privilegios sobre una tabla.
```sql
  GRANT ALL ON nombretabla TO nombreusuario;
```
  - Conceder un privilegio a todos los usuarios de la base de datos.
```sql
  GRANT SELECT ON nombretabla TO PUBLIC;
```
  - Permitir que el usuario, a su vez, conceda ese privilegio a otros.
```sql
  GRANT SELECT ON nombretabla TO nombreusuario WITH GRANT OPTION;
```
  - **REVOKE**: elimina privilegios. Se usa `FROM`, no `TO`.
```sql
  REVOKE nombreprivilegio FROM nombreusuario;
```
  - Retirar un privilegio de objeto.
```sql
  REVOKE UPDATE ON nombretabla FROM nombreusuario;
```

>[!IMPORTANT]
>En el siguiente enlace dejo todos los privilegios que se le pueden dar a un usuario. [Leer Privilegios](privilegios.md)

#### ROLES

Un rol es un conjunto de privilegios con nombre. En vez de conceder 20 privilegios a 10 usuarios, se conceden al rol y el rol a los usuarios.

  - Crear un rol.
```sql
  CREATE ROLE nombrerol;
```
  - Añadir privilegios al rol.
```sql
  GRANT CREATE SESSION, SELECT ANY TABLE TO nombrerol;
```
  - Asignar el rol a un usuario.
```sql
  GRANT nombrerol TO nombreusuario;
```
  - Quitar el rol a un usuario.
```sql
  REVOKE nombrerol FROM nombreusuario;
```
  - Borrar el rol (se retira automáticamente de todos los usuarios que lo tuvieran).
```sql
  DROP ROLE nombrerol;
```
  - Roles predefinidos que trae Oracle: `CONNECT` (conectarse), `RESOURCE` (crear objetos) y `DBA` (casi todo).
```sql
  GRANT CONNECT, RESOURCE TO nombreusuario;
```

#### PERFILES

Un perfil limita los recursos y la política de contraseñas de un usuario.

  - Crear un perfil.
```sql
  CREATE PROFILE nombreperfil LIMIT
    SESSIONS_PER_USER 3
    FAILED_LOGIN_ATTEMPTS 5
    PASSWORD_LIFE_TIME 90
    IDLE_TIME 30;
```
  - Asignar el perfil a un usuario.
```sql
  ALTER USER nombreusuario PROFILE nombreperfil;
```
  - Borrar un perfil y devolver a sus usuarios al perfil `DEFAULT`.
```sql
  DROP PROFILE nombreperfil CASCADE;
```

#### CONSULTAR QUÉ PERMISOS TENGO
  - Privilegios de sistema concedidos al usuario actual.
```sql
  SELECT * FROM USER_SYS_PRIVS;
```
  - Privilegios sobre objetos de otros esquemas concedidos al usuario actual.
```sql
  SELECT * FROM USER_TAB_PRIVS;
```
  - Roles que tiene el usuario actual.
```sql
  SELECT * FROM USER_ROLE_PRIVS;
```
  - Privilegios que tiene un rol concreto.
```sql
  SELECT privilege FROM ROLE_SYS_PRIVS WHERE role = 'NOMBREROL';
```
