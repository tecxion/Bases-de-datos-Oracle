![Base_Datos_Logo](./Media_BD/COMANDOS.gif)

# Aquí publicaré los comandos más importantes que se usan en ORACLE SQL así como ejemplos y otras cosas como la tarea realizada en la clase de BD.

>[!CAUTION]
>## *En el pdf Comandos básicos puedes ver unas capturas de pantalla con la creación de varias tablas y los comandos utilizados, así mismo en este Readme iré dejando los comandos más utilizados y el resumen de SQL ORACLE.*
  [Comandos básicos Tarea2.pdf](https://github.com/tecxion/Bases-de-datos-Oracle/blob/main/Comandos%20b%C3%A1sicos%20Tarea2.pdf)


  ### INSTALACIÓN E INICIO:
  - Para iniciar sesión en sql utilizaremos el comando: sqlplus en cmd.
  - Introduciremos el usuario y contraseña generados en la instalación de ORACLE SQL.

  ### UNA VEZ INICIADO PODEMOS USAR LOS SIGUIENTES COMANDOS:

#### USUARIOS
  - Crear Usuario y dar contraseña
```
  CREATE USER nombreusuario IDENTIFIED BY contraseña;
```
  - También podemos añadir el espacio que se le asigna a un usuario y el perfil.
```
  CCREATE USER nombreusuario IDENTIFIED BY contraseña
  DEFAULT TABLESPACE users
  QUOTA tamañodelespacio ON users
  PROFILE nombreperfil;
```
  - Ver todos los usuarios.
```
   DESC SYS.ALL_USERS;
```
  - Borrar un usuario
```
  DROP USER nombreusuario;
```
##### Privilegios de los Usuarios
 - GRANT: concedemos  (añadimos los privilegios) // TO al usuario que se le conceden  // *ALL puede ser todo los privilegios // PUBLIC privilegios a todos los usuarios. Ej:
```
 GRANT SELECT, UPDATE, INSERT TO nombreusuario;
```
  - REVOKE: elimina privilegios
```
  REVOKE nombreprivilegio TO nombreusuario;
```

 >[!IMPORTANT]
>En el siguiente enlace dejo todos los privilegios que se le pueden dar a un usuario. [Leer Privilegios](Privilegios.md)

#### BASES DE DATOS.

  - Creación de una Base de Datos.<br>
```
CREATE DATABASE Nombrebd;
```
  - Creación de una tabla: <br>
```
CREATE TABLE nombretabla (
 *A continuación escribiriamos los Atributos con el tipo y las restricciones que se explica más adelante*;
```
  -Eliminar una tabla, aunque sirve para usuarios y columnas es el comando DROP, Ej:
```
  DROP TABLE Nombretabla;
```
#### Modificar Tablas:
  - Cambiar el nombre a una tabla:
```
  RENAME nombreviejo TO nombrenuevo;
```
  - Añadir Columna:
```
  ALTER TABLE nombretabla ADD nombrecolumna tipo [propiedades];
```
  - Eliminar Columna:
```
  ALTER TABLE nombretabla DROP COLUMN nombrecolumna;
```
  - Modificar Columna:
```
  ALTER TABLE nombretabla MODIFY nombrecolumna;
```
  - Renombrar Columna:
```
  ALTER TABLE nombretabla RENAME COLUMN nombreviejocolumna TO nombrenuevocolumna;
```
  - Borrar Restricciones:
```
  ALTER TABLE nombretabla DROP CONSTRAINT nombrerestriccion;
```
  - Modificar Restricción:
```
  ALTER TABLE nombretabla RENAME CONSTRAINT nombreviejo TO nombrenuevo;
```
  - Activar o Desactivar restricciones:
```
  ALTER TABLE nombretabla ENABLE CONSTRAINT Nombrerestriccion;
  ALTER TABLE nombretabla DISABLE CONSTRAINT Nombrerestriccion;
```

#### RESTRICCIONES
  - NOT NULL: Prohibe valores nulos. 
  - UNIQUE: No se pueden repetir valores.
  - PRIMARY KEY: Clave Primaria, será not null y unique.
  - FOREIGN KEY: Claves externas que relacionan tablas.
  - DEFAULT: Valor por defecto.

#### CREACIÓN Y MODIFICACIÓN DE INDICES.
  - Crear Indice
```
  CREATE INDEX nombreindice ON nombretabla (columna1 [tipo1], columna2 [tipo2]...);
```
  - Borrar Indice
```
  DROP INDEX nombreindice;
```

#### CLAUSULAS
 - FROM: Especificar Tabla.
 - WHERE: especificar condiciones que filas que se van a seleccionar.
 - GROUP BY: Separar filas en grupos especificos.
 - HAVING: expresar condición de cada grupo.
 - ORDER BY: Ordenar filas por orden específico.
Ejemplo:
```
SELECT departamento_id, AVG(salario) AS salario_promedio
FROM empleados
WHERE salario > 3000
GROUP BY departamento_id
HAVING AVG(salario) > 5000
ORDER BY salario_promedio DESC;
```
#### OPERADORES
 - AND: evalua 2 condiciones, devuelve TRUE si ambos son TRUE.
 - OR: evalua 2 condiciones, devuelve TRUE si una de las dos es TRUE.
 - NOT: devuelve el valor contrario.

#### OPERADORES COMPARACIÓN
 - <: Menor que.                              - <=: Menor o igual que.
 -> : Mayor que.                              - >=: Mayor o igual que.
 -<>: Distinto de                             - = : Igual.
 - BETWEEN: Intervalo entre 2 valores.
 - LIKE: Comparar.
 - IN: especificar filas.

  


