![Base_Datos_Logo](./Media_BD/COMANDOS.gif)

# comandos más importantes que se usan en ORACLE SQL y algún ejemplo.

>[!CAUTION]
>## *En el pdf Comandos básicos puedes ver unas capturas de pantalla con la creación de varias tablas y los comandos utilizados, así mismo en este repositorio iré dejando los comandos más utilizados y el resumen de SQL ORACLE.*
  [Comandos básicos Tarea2.pdf](https://github.com/tecxion/Bases-de-datos-Oracle/blob/main/Comandos%20b%C3%A1sicos%20Tarea2.pdf)

---

## ÍNDICE

| # | Documento | Contenido |
|---|---|---|
| 1 | [Instalación y SQL*Plus](docs/01-instalacion-y-sqlplus.md) | Conectarse, describir objetos, formatear la salida, scripts y `SPOOL`. |
| 2 | [Usuarios y privilegios](docs/02-usuarios-y-privilegios.md) | `CREATE USER`, `GRANT`, `REVOKE`, roles y perfiles. |
| 3 | [Tipos de datos](docs/03-tipos-de-datos.md) | `VARCHAR2`, `NUMBER`, `DATE`, conversiones, máscaras de formato y `NULL`. |
| 4 | [Tablas y restricciones](docs/04-tablas-y-restricciones.md) | `CREATE TABLE`, `ALTER TABLE`, restricciones e índices. |
| 5 | [DML y transacciones](docs/05-dml-y-transacciones.md) | `INSERT`, `UPDATE`, `DELETE`, `MERGE`, `COMMIT` y `ROLLBACK`. |
| 6 | [Consultas: SELECT](docs/06-consultas-select.md) | Cláusulas, operadores, funciones de grupo y paginación. |
| 7 | [Joins y subconsultas](docs/07-joins-y-subconsultas.md) | `JOIN`, subconsultas, `WITH`, `UNION`, `INTERSECT` y `MINUS`. |
| 8 | [Funciones](docs/08-funciones.md) | Cadena, número, fecha, `NVL`, `CASE` y funciones analíticas. |
| 9 | [Objetos de la base de datos](docs/09-objetos-de-la-bd.md) | Vistas, secuencias, sinónimos, triggers y procedimientos. |
| 10 | [Diccionario de datos](docs/10-diccionario-de-datos.md) | `USER_TABLES`, `ALL_*`, `DBA_*` y vistas `V$`. |
| — | [Privilegios](docs/privilegios.md) | Lista completa de privilegios de sistema, de objeto y administrativos. |

---

## INSTALACIÓN E INICIO
  - Para iniciar sesión en sql utilizaremos el comando: `sqlplus` en cmd.
  - Introduciremos el usuario y contraseña generados en la instalación de ORACLE SQL.

```sql
  sqlplus nombreusuario/contraseña
```

>[!IMPORTANT]
>Toda sentencia SQL termina en `;`. Los bloques PL/SQL (triggers, procedimientos) terminan además con una `/` en una línea suelta.

## CHULETA RÁPIDA

Los comandos que más se repiten. Cada uno enlaza al documento donde está explicado.

#### Usuarios · [ver más](docs/02-usuarios-y-privilegios.md)
```sql
  CREATE USER nombreusuario IDENTIFIED BY contraseña;
  GRANT CREATE SESSION, CREATE TABLE TO nombreusuario;
  GRANT SELECT, UPDATE, INSERT ON nombretabla TO nombreusuario;
  REVOKE nombreprivilegio FROM nombreusuario;
  DROP USER nombreusuario CASCADE;
```

#### Tablas · [ver más](docs/04-tablas-y-restricciones.md)
```sql
  CREATE TABLE nombretabla (columna1 tipo [restricción], columna2 tipo [restricción]);
  ALTER TABLE nombretabla ADD    nombrecolumna tipo;
  ALTER TABLE nombretabla MODIFY nombrecolumna tipo;
  ALTER TABLE nombretabla DROP COLUMN nombrecolumna;
  ALTER TABLE nombretabla RENAME COLUMN nombreviejo TO nombrenuevo;
  RENAME nombreviejo TO nombrenuevo;
  DROP TABLE nombretabla;
```

#### Restricciones · [ver más](docs/04-tablas-y-restricciones.md#restricciones)
 - **NOT NULL**: Prohibe valores nulos.
 - **UNIQUE**: No se pueden repetir valores.
 - **PRIMARY KEY**: Clave Primaria, será not null y unique.
 - **FOREIGN KEY**: Claves externas que relacionan tablas.
 - **CHECK**: El valor tiene que cumplir una condición.
 - **DEFAULT**: Valor por defecto.

```sql
  ALTER TABLE nombretabla ADD  CONSTRAINT nombrerestriccion PRIMARY KEY (columna);
  ALTER TABLE nombretabla DROP CONSTRAINT nombrerestriccion;
  ALTER TABLE nombretabla ENABLE  CONSTRAINT nombrerestriccion;
  ALTER TABLE nombretabla DISABLE CONSTRAINT nombrerestriccion;
```

#### Índices · [ver más](docs/04-tablas-y-restricciones.md#creación-y-modificación-de-índices)
```sql
  CREATE INDEX nombreindice ON nombretabla (columna1 [tipo1], columna2 [tipo2]...);
  DROP INDEX nombreindice;
```

#### Datos · [ver más](docs/05-dml-y-transacciones.md)
```sql
  INSERT INTO nombretabla (columna1, columna2) VALUES (valor1, valor2);
  UPDATE nombretabla SET columna = valor WHERE condición;
  DELETE FROM nombretabla WHERE condición;
  COMMIT;
  ROLLBACK;
```

>[!CAUTION]
>Un `UPDATE` o un `DELETE` sin `WHERE` afecta a **todas** las filas de la tabla.

#### Cláusulas · [ver más](docs/06-consultas-select.md)
 - **FROM**: Especificar Tabla.
 - **WHERE**: especificar condiciones que filas que se van a seleccionar.
 - **GROUP BY**: Separar filas en grupos específicos.
 - **HAVING**: expresar condición de cada grupo.
 - **ORDER BY**: Ordenar filas por orden específico.

Ejemplo:
```sql
  SELECT departamento_id, AVG(salario) AS salario_promedio
  FROM empleados
  WHERE salario > 3000
  GROUP BY departamento_id
  HAVING AVG(salario) > 5000
  ORDER BY salario_promedio DESC;
```

#### Operadores · [ver más](docs/06-consultas-select.md#operadores-lógicos)
 - **AND**: evalúa 2 condiciones, devuelve TRUE si ambos son TRUE.
 - **OR**: evalúa 2 condiciones, devuelve TRUE si una de las dos es TRUE.
 - **NOT**: devuelve el valor contrario.

#### Operadores de comparación · [ver más](docs/06-consultas-select.md#operadores-de-comparación)
 - `<` : Menor que
 - `>` : Mayor que
 - `<>` : Distinto de
 - `<=` : Menor o igual
 - `>=` : Mayor o igual
 - `=` : Igual
 - **BETWEEN**: Intervalo entre 2 valores.
 - **LIKE**: Para comparar dos valores.
 - **IN**: Especificar las filas.

#### Funciones de grupo · [ver más](docs/08-funciones.md)
 - **AVG**: Calcula el promedio entre valores de 1 campo.
 - **COUNT**: Nº Filas de la selección.
 - **SUM**: Suma todos los valores de un atributo determinado.
 - **MAX**: Valor más alto.
 - **MIN**: Valor más bajo.

#### Joins · [ver más](docs/07-joins-y-subconsultas.md)
```sql
  SELECT e.nombre, d.nombre
  FROM empleados e
  INNER JOIN departamentos d ON e.departamento_id = d.departamento_id;
```
 - **INNER JOIN**: solo las filas que casan en las dos tablas.
 - **LEFT JOIN**: todas las de la izquierda, casen o no.
 - **RIGHT JOIN**: todas las de la derecha, casen o no.
 - **FULL JOIN**: todas las de ambas tablas.

#### Diccionario de datos · [ver más](docs/10-diccionario-de-datos.md)
```sql
  DESC nombretabla;
  SELECT table_name FROM USER_TABLES;
  SELECT column_name, data_type FROM USER_TAB_COLUMNS WHERE table_name = 'EMPLEADOS';
```
