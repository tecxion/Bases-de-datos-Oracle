# CONSULTAS: SELECT

#### ESTRUCTURA DE UNA CONSULTA

```sql
  SELECT   columnas          -- qué quiero ver
  FROM     tabla             -- de dónde
  WHERE    condición         -- qué filas
  GROUP BY columnas          -- cómo las agrupo
  HAVING   condición         -- qué grupos me quedo
  ORDER BY columnas;         -- en qué orden lo muestro
```

>[!IMPORTANT]
>Se escribe en ese orden, pero Oracle lo **ejecuta** en otro: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`. Por eso no se puede usar un alias definido en el `SELECT` dentro del `WHERE`, pero sí dentro del `ORDER BY`.

#### CLÁUSULAS
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

>[!NOTE]
>La diferencia entre `WHERE` y `HAVING`: `WHERE` filtra **filas** antes de agrupar, `HAVING` filtra **grupos** después de agrupar. En `WHERE` no se pueden usar funciones de grupo como `AVG()`.

#### SELECCIONAR COLUMNAS
  - Todas las columnas.
```sql
  SELECT * FROM empleados;
```
  - Columnas concretas, con alias para la cabecera.
```sql
  SELECT nombre AS empleado, salario * 14 AS "Salario anual" FROM empleados;
```
  - Eliminar filas repetidas del resultado.
```sql
  SELECT DISTINCT departamento_id FROM empleados;
```
  - Concatenar textos con `||`.
```sql
  SELECT nombre || ' (' || email || ')' AS contacto FROM empleados;
```

>[!NOTE]
>Los alias con espacios o mayúsculas van entre comillas dobles: `AS "Salario anual"`. Sin comillas, Oracle lo pasa a mayúsculas.

#### OPERADORES LÓGICOS
 - **AND**: evalúa 2 condiciones, devuelve TRUE si ambos son TRUE.
 - **OR**: evalúa 2 condiciones, devuelve TRUE si una de las dos es TRUE.
 - **NOT**: devuelve el valor contrario.

`AND` tiene más prioridad que `OR`. Usa paréntesis si mezclas los dos.
```sql
  SELECT * FROM empleados
  WHERE departamento_id = 10
    AND (salario > 50000 OR comision IS NOT NULL);
```

#### OPERADORES DE COMPARACIÓN
 - `<` : Menor que
 - `>` : Mayor que
 - `<>` : Distinto de (también vale `!=`)
 - `<=` : Menor o igual
 - `>=` : Mayor o igual
 - `=` : Igual
 - **BETWEEN**: Intervalo entre 2 valores. Incluye los extremos.
 - **LIKE**: Para comparar dos valores usando comodines.
 - **IN**: Especificar las filas dentro de una lista de valores.
 - **IS NULL** / **IS NOT NULL**: Comprobar si un valor es nulo.

```sql
  SELECT * FROM empleados WHERE salario BETWEEN 30000 AND 50000;
  SELECT * FROM empleados WHERE departamento_id IN (10, 20, 30);
  SELECT * FROM empleados WHERE comision IS NULL;
```

##### Comodines de LIKE
 - `%` : Cualquier cantidad de caracteres, incluso ninguno.
 - `_` : Exactamente un carácter.

```sql
  SELECT * FROM empleados WHERE nombre LIKE 'A%';      -- empieza por A
  SELECT * FROM empleados WHERE email  LIKE '%@%.es';  -- correo español
  SELECT * FROM empleados WHERE nombre LIKE '_ana%';   -- 2ª letra en adelante: ana
```
  - Para buscar un `%` o un `_` literales, hay que escaparlos.
```sql
  SELECT * FROM productos WHERE codigo LIKE '50\%%' ESCAPE '\';
```

>[!CAUTION]
>`LIKE` distingue mayúsculas de minúsculas. Para buscar sin distinguir: `WHERE UPPER(nombre) LIKE 'A%'`.

#### ORDENAR RESULTADOS
  - Ascendente (`ASC`) es el valor por defecto; descendente es `DESC`.
```sql
  SELECT nombre, salario FROM empleados ORDER BY salario DESC, nombre ASC;
```
  - Ordenar por la posición de la columna en el `SELECT`.
```sql
  SELECT nombre, salario FROM empleados ORDER BY 2 DESC;
```
  - Colocar los nulos al principio o al final.
```sql
  SELECT nombre, comision FROM empleados ORDER BY comision DESC NULLS LAST;
```

>[!NOTE]
>Por defecto, en un orden ascendente los `NULL` van al **final**; en descendente, al **principio**.

#### FUNCIONES DE GRUPO (AGREGADAS)
 - **AVG**: Calcula el promedio entre valores de 1 campo.
 - **COUNT**: Nº Filas de la selección.
 - **SUM**: Suma todos los valores de un atributo determinado.
 - **MAX**: Valor más alto.
 - **MIN**: Valor más bajo.

```sql
  SELECT COUNT(*)       AS total_empleados,
         AVG(salario)   AS salario_medio,
         MAX(salario)   AS salario_maximo,
         MIN(fecha_alta) AS mas_antiguo,
         SUM(salario)   AS coste_total
  FROM empleados;
```

>[!IMPORTANT]
>Las funciones de grupo **ignoran los nulos**, salvo `COUNT(*)`. Si de 10 empleados 3 no tienen comisión, `AVG(comision)` divide entre 7, no entre 10. Y `COUNT(comision)` devuelve 7 mientras que `COUNT(*)` devuelve 10.

  - Contar valores distintos.
```sql
  SELECT COUNT(DISTINCT departamento_id) FROM empleados;
```

#### AGRUPAR

Regla de oro: **toda columna del `SELECT` que no esté dentro de una función de grupo tiene que aparecer en el `GROUP BY`**.

  - Agrupar por una columna.
```sql
  SELECT departamento_id, COUNT(*) AS empleados
  FROM empleados
  GROUP BY departamento_id;
```
  - Agrupar por varias.
```sql
  SELECT departamento_id, EXTRACT(YEAR FROM fecha_alta) AS anio, COUNT(*)
  FROM empleados
  GROUP BY departamento_id, EXTRACT(YEAR FROM fecha_alta)
  ORDER BY departamento_id, anio;
```
  - Filtrar grupos con `HAVING`.
```sql
  SELECT departamento_id, COUNT(*) AS empleados
  FROM empleados
  GROUP BY departamento_id
  HAVING COUNT(*) >= 5;
```

#### LIMITAR EL NÚMERO DE FILAS
  - Sintaxis moderna (Oracle 12c en adelante).
```sql
  SELECT nombre, salario FROM empleados
  ORDER BY salario DESC
  FETCH FIRST 10 ROWS ONLY;
```
  - Incluyendo los empates en la última posición.
```sql
  SELECT nombre, salario FROM empleados
  ORDER BY salario DESC
  FETCH FIRST 10 ROWS WITH TIES;
```
  - Saltarse las primeras filas (paginación).
```sql
  SELECT nombre FROM empleados
  ORDER BY nombre
  OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```
  - Sintaxis antigua con `ROWNUM`, aún muy común.
```sql
  SELECT * FROM (SELECT nombre, salario FROM empleados ORDER BY salario DESC)
  WHERE ROWNUM <= 10;
```

>[!CAUTION]
>`ROWNUM` se asigna **antes** del `ORDER BY`. `SELECT * FROM empleados WHERE ROWNUM <= 10 ORDER BY salario DESC` no devuelve los 10 mejor pagados, sino 10 filas cualesquiera ordenadas. Por eso hay que envolver la consulta en una subconsulta.
