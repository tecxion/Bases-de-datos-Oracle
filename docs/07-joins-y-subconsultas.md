# JOINS, SUBCONSULTAS Y CONJUNTOS

#### JOINS

Un `JOIN` combina filas de dos o más tablas relacionadas por una columna común, normalmente una clave foránea.

  - **INNER JOIN**: solo las filas que tienen correspondencia en **ambas** tablas.
```sql
  SELECT e.nombre, d.nombre AS departamento
  FROM empleados e
  INNER JOIN departamentos d ON e.departamento_id = d.departamento_id;
```
  - **LEFT [OUTER] JOIN**: todas las filas de la izquierda, aunque no tengan pareja a la derecha (rellena con `NULL`).
```sql
  SELECT e.nombre, d.nombre AS departamento
  FROM empleados e
  LEFT JOIN departamentos d ON e.departamento_id = d.departamento_id;
```
  - **RIGHT [OUTER] JOIN**: todas las de la derecha. Sirve para ver los departamentos sin ningún empleado.
```sql
  SELECT e.nombre, d.nombre AS departamento
  FROM empleados e
  RIGHT JOIN departamentos d ON e.departamento_id = d.departamento_id;
```
  - **FULL [OUTER] JOIN**: todas las filas de las dos tablas, emparejadas cuando se puede.
```sql
  SELECT e.nombre, d.nombre AS departamento
  FROM empleados e
  FULL JOIN departamentos d ON e.departamento_id = d.departamento_id;
```
  - **CROSS JOIN**: producto cartesiano, cada fila de una con cada fila de la otra. 100 × 50 = 5000 filas.
```sql
  SELECT e.nombre, d.nombre FROM empleados e CROSS JOIN departamentos d;
```
  - **SELF JOIN**: una tabla consigo misma. El caso típico es empleado ↔ jefe.
```sql
  SELECT e.nombre AS empleado, j.nombre AS jefe
  FROM empleados e
  LEFT JOIN empleados j ON e.jefe_id = j.empleado_id;
```

##### Atajos de sintaxis
  - `USING`: cuando la columna se llama igual en las dos tablas. Ojo, no se le pone alias de tabla.
```sql
  SELECT nombre, departamento_id
  FROM empleados JOIN departamentos USING (departamento_id);
```
  - `NATURAL JOIN`: une por **todas** las columnas que se llamen igual. Cómodo y peligroso.
```sql
  SELECT * FROM empleados NATURAL JOIN departamentos;
```

>[!CAUTION]
>Evita `NATURAL JOIN` en código real. Si mañana alguien añade una columna `nombre` a las dos tablas, la consulta cambia de significado sin avisar y empieza a devolver menos filas.

##### Sintaxis antigua de Oracle

Antes del estándar ANSI, Oracle unía tablas en el `WHERE` y marcaba el lado externo con `(+)`. La vas a encontrar en código heredado.

```sql
  -- Equivale a un INNER JOIN
  SELECT e.nombre, d.nombre
  FROM empleados e, departamentos d
  WHERE e.departamento_id = d.departamento_id;

  -- Equivale a un LEFT JOIN: el (+) va en el lado que puede faltar
  SELECT e.nombre, d.nombre
  FROM empleados e, departamentos d
  WHERE e.departamento_id = d.departamento_id (+);
```

>[!IMPORTANT]
>Usa siempre la sintaxis `JOIN ... ON`. Con la antigua, olvidar una condición en el `WHERE` produce un producto cartesiano silencioso: millones de filas y ningún error.

#### SUBCONSULTAS

Una subconsulta es un `SELECT` dentro de otra sentencia.

  - **Escalar**: devuelve un único valor. Se compara con `=`, `>`, `<`...
```sql
  SELECT nombre, salario FROM empleados
  WHERE salario > (SELECT AVG(salario) FROM empleados);
```
  - **De varias filas**: se compara con `IN`, `ANY` o `ALL`.
```sql
  SELECT nombre FROM empleados
  WHERE departamento_id IN (SELECT departamento_id FROM departamentos WHERE localidad = 'Madrid');
```
  - `ANY`: se cumple si es cierto para **alguno** de los valores.
```sql
  SELECT nombre FROM empleados
  WHERE salario > ANY (SELECT salario FROM empleados WHERE departamento_id = 10);
```
  - `ALL`: se cumple si es cierto para **todos** los valores.
```sql
  SELECT nombre FROM empleados
  WHERE salario > ALL (SELECT salario FROM empleados WHERE departamento_id = 10);
```
  - **Correlacionada**: la subconsulta usa una columna de la consulta externa, así que se evalúa una vez por cada fila.
```sql
  SELECT e.nombre, e.salario FROM empleados e
  WHERE e.salario > (SELECT AVG(s.salario) FROM empleados s
                     WHERE s.departamento_id = e.departamento_id);
```
  - `EXISTS`: comprueba si la subconsulta devuelve al menos una fila. No importa qué devuelva, por eso se pone `1`.
```sql
  SELECT d.nombre FROM departamentos d
  WHERE EXISTS (SELECT 1 FROM empleados e WHERE e.departamento_id = d.departamento_id);
```
  - `NOT EXISTS`: departamentos que no tienen ningún empleado.
```sql
  SELECT d.nombre FROM departamentos d
  WHERE NOT EXISTS (SELECT 1 FROM empleados e WHERE e.departamento_id = d.departamento_id);
```
  - **En el FROM** (*vista en línea*): la subconsulta actúa como una tabla temporal.
```sql
  SELECT departamento_id, salario_medio
  FROM (SELECT departamento_id, AVG(salario) AS salario_medio
        FROM empleados GROUP BY departamento_id)
  WHERE salario_medio > 40000;
```

>[!CAUTION]
>`NOT IN` con una subconsulta que devuelve algún `NULL` **no devuelve ninguna fila**, nunca. Es un clásico. Usa `NOT EXISTS`, que sí funciona bien con nulos.

#### CLÁUSULA WITH (SUBCONSULTAS CON NOMBRE)

Permite dar nombre a una subconsulta y reutilizarla. Hace las consultas largas mucho más legibles.

```sql
  WITH medias AS (
    SELECT departamento_id, AVG(salario) AS salario_medio
    FROM empleados
    GROUP BY departamento_id
  )
  SELECT d.nombre, m.salario_medio
  FROM medias m
  JOIN departamentos d ON d.departamento_id = m.departamento_id
  WHERE m.salario_medio > 40000
  ORDER BY m.salario_medio DESC;
```

#### OPERADORES DE CONJUNTO

Combinan el resultado de dos consultas. Las dos tienen que devolver **el mismo número de columnas y de tipos compatibles**.

 - **UNION**: une los resultados y elimina las filas repetidas.
 - **UNION ALL**: une los resultados sin eliminar repetidas. Es más rápido.
 - **INTERSECT**: solo las filas que están en las dos consultas.
 - **MINUS**: filas de la primera consulta que no están en la segunda.

```sql
  SELECT nombre FROM empleados_madrid
  UNION
  SELECT nombre FROM empleados_sevilla;

  SELECT departamento_id FROM departamentos
  MINUS
  SELECT DISTINCT departamento_id FROM empleados;   -- departamentos vacíos
```

>[!NOTE]
>`MINUS` es específico de Oracle. En el estándar SQL y en PostgreSQL se llama `EXCEPT`.

>[!IMPORTANT]
>El `ORDER BY` va **una sola vez, al final** de todo el conjunto, y se refiere a las columnas de la primera consulta.
