# FUNCIONES

Las **funciones de fila** devuelven un resultado por cada fila. Las **funciones de grupo** (`AVG`, `COUNT`, `SUM`, `MAX`, `MIN`) devuelven un resultado por cada grupo, y están en [Consultas: SELECT](06-consultas-select.md).

>[!NOTE]
>Para probar cualquier función sin necesidad de una tabla, usa `DUAL`: `SELECT UPPER('hola') FROM DUAL;`

#### FUNCIONES DE CADENA

| Función | Descripción | Ejemplo | Resultado |
|---|---|---|---|
| `UPPER(s)` | Pasa a mayúsculas | `UPPER('ana')` | `ANA` |
| `LOWER(s)` | Pasa a minúsculas | `LOWER('ANA')` | `ana` |
| `INITCAP(s)` | Primera letra de cada palabra en mayúscula | `INITCAP('ana ruiz')` | `Ana Ruiz` |
| `LENGTH(s)` | Número de caracteres | `LENGTH('Oracle')` | `6` |
| `SUBSTR(s,p,n)` | Extrae `n` caracteres desde la posición `p` | `SUBSTR('Oracle',1,3)` | `Ora` |
| `INSTR(s,b)` | Posición donde aparece `b` (0 si no está) | `INSTR('Oracle','a')` | `3` |
| `TRIM(s)` | Quita espacios de los dos lados | `TRIM('  hi  ')` | `hi` |
| `LTRIM(s)` / `RTRIM(s)` | Quita espacios por la izquierda / derecha | `RTRIM('hi  ')` | `hi` |
| `LPAD(s,n,c)` | Rellena por la izquierda hasta `n` caracteres | `LPAD('7',3,'0')` | `007` |
| `RPAD(s,n,c)` | Rellena por la derecha | `RPAD('7',3,'*')` | `7**` |
| `REPLACE(s,a,b)` | Sustituye `a` por `b` | `REPLACE('2024-01','-','/')` | `2024/01` |
| `CONCAT(a,b)` | Une dos cadenas (solo dos) | `CONCAT('a','b')` | `ab` |
| `REVERSE(s)` | Invierte la cadena | `REVERSE('abc')` | `cba` |

  - El operador `||` concatena y admite cualquier número de valores. Es más práctico que `CONCAT`.
```sql
  SELECT nombre || ' ' || apellido AS completo FROM empleados;
```
  - `SUBSTR` con posición negativa cuenta desde el final.
```sql
  SELECT SUBSTR('987654321', -4) FROM DUAL;   -- '4321'
```

#### FUNCIONES NUMÉRICAS

| Función | Descripción | Ejemplo | Resultado |
|---|---|---|---|
| `ROUND(n,d)` | Redondea a `d` decimales | `ROUND(45.678, 2)` | `45.68` |
| `TRUNC(n,d)` | Trunca (corta) a `d` decimales, sin redondear | `TRUNC(45.678, 2)` | `45.67` |
| `MOD(a,b)` | Resto de la división | `MOD(10,3)` | `1` |
| `CEIL(n)` | Redondea hacia arriba | `CEIL(45.1)` | `46` |
| `FLOOR(n)` | Redondea hacia abajo | `FLOOR(45.9)` | `45` |
| `ABS(n)` | Valor absoluto | `ABS(-15)` | `15` |
| `POWER(a,b)` | `a` elevado a `b` | `POWER(2,10)` | `1024` |
| `SQRT(n)` | Raíz cuadrada | `SQRT(16)` | `4` |
| `SIGN(n)` | -1 si negativo, 0 si cero, 1 si positivo | `SIGN(-7)` | `-1` |

  - Con `d` negativo se redondea a decenas, centenas...
```sql
  SELECT ROUND(1234, -2) FROM DUAL;   -- 1200
```

#### FUNCIONES DE FECHA

| Función | Descripción |
|---|---|
| `SYSDATE` | Fecha y hora actual del **servidor** |
| `CURRENT_DATE` | Fecha y hora actual en la zona horaria de la **sesión** |
| `SYSTIMESTAMP` | Igual que `SYSDATE` pero con fracciones de segundo y zona horaria |
| `ADD_MONTHS(f,n)` | Suma `n` meses a la fecha `f` |
| `MONTHS_BETWEEN(a,b)` | Meses entre dos fechas (puede tener decimales) |
| `NEXT_DAY(f,'LUNES')` | Siguiente día de la semana indicado |
| `LAST_DAY(f)` | Último día del mes de `f` |
| `TRUNC(f)` | Quita la hora, deja la fecha a las 00:00 |
| `TRUNC(f,'MM')` | Primer día del mes |
| `EXTRACT(YEAR FROM f)` | Extrae año, mes o día como número |

  - A una fecha se le suman y restan **días** directamente.
```sql
  SELECT SYSDATE + 30 AS dentro_de_un_mes, SYSDATE - 1 AS ayer FROM DUAL;
```
  - Restar dos fechas devuelve el número de días entre ellas.
```sql
  SELECT ROUND(SYSDATE - fecha_alta) AS dias_en_la_empresa FROM empleados;
```
  - Sumar horas: hay que dividir entre 24.
```sql
  SELECT SYSDATE + 3/24 AS dentro_de_3_horas FROM DUAL;
```
  - Antigüedad en años.
```sql
  SELECT nombre, TRUNC(MONTHS_BETWEEN(SYSDATE, fecha_alta) / 12) AS anios FROM empleados;
```

>[!IMPORTANT]
>Para comparar solo la parte de fecha, ignorando la hora, usa `TRUNC(fecha) = DATE '2024-01-15'`. Ver [Tipos de datos](03-tipos-de-datos.md).

#### FUNCIONES DE CONVERSIÓN
  - `TO_CHAR(valor, máscara)`: número o fecha → texto.
```sql
  SELECT TO_CHAR(fecha_alta, 'DD "de" MONTH "de" YYYY') FROM empleados;
```
  - `TO_NUMBER(texto)`: texto → número.
  - `TO_DATE(texto, máscara)`: texto → fecha.
  - `CAST(valor AS tipo)`: conversión genérica del estándar SQL.
```sql
  SELECT CAST('123' AS NUMBER) FROM DUAL;
```

Las máscaras de formato están en [Tipos de datos](03-tipos-de-datos.md#máscaras-de-formato-más-usadas).

#### FUNCIONES PARA NULOS

| Función | Descripción |
|---|---|
| `NVL(a, b)` | Si `a` es nulo devuelve `b`, si no devuelve `a` |
| `NVL2(a, b, c)` | Si `a` **no** es nulo devuelve `b`, si es nulo devuelve `c` |
| `COALESCE(a, b, c, ...)` | Devuelve el primer argumento no nulo de la lista |
| `NULLIF(a, b)` | Devuelve `NULL` si `a = b`, si no devuelve `a` |

```sql
  SELECT nombre,
         NVL(comision, 0)                            AS comision,
         NVL2(comision, 'Con comisión', 'Sin comisión') AS tipo,
         COALESCE(movil, fijo, 'Sin teléfono')        AS contacto
  FROM empleados;
```

#### CONDICIONALES

  - `CASE` simple: compara una expresión con valores concretos.
```sql
  SELECT nombre,
         CASE departamento_id
           WHEN 10 THEN 'Ventas'
           WHEN 20 THEN 'Informática'
           ELSE 'Otro'
         END AS departamento
  FROM empleados;
```
  - `CASE` con condiciones: cada rama es una condición completa. Es la forma más flexible.
```sql
  SELECT nombre,
         CASE
           WHEN salario < 25000 THEN 'Bajo'
           WHEN salario < 50000 THEN 'Medio'
           ELSE 'Alto'
         END AS franja
  FROM empleados;
```
  - `DECODE`: la versión antigua y propia de Oracle del `CASE` simple.
```sql
  SELECT nombre, DECODE(departamento_id, 10, 'Ventas', 20, 'Informática', 'Otro') FROM empleados;
```

>[!NOTE]
>`CASE` es estándar SQL y admite rangos y condiciones. `DECODE` solo compara igualdad. Usa `CASE` salvo que mantengas código antiguo.

#### FUNCIONES ANALÍTICAS (VENTANA)

Calculan un valor sobre un conjunto de filas relacionadas, **sin agrupar**: la consulta sigue devolviendo una fila por empleado.

  - Numerar las filas dentro de cada grupo.
```sql
  SELECT nombre, departamento_id, salario,
         ROW_NUMBER() OVER (PARTITION BY departamento_id ORDER BY salario DESC) AS puesto
  FROM empleados;
```
  - `RANK()` deja huecos en los empates (1, 2, 2, 4); `DENSE_RANK()` no los deja (1, 2, 2, 3).
```sql
  SELECT nombre, salario,
         RANK()       OVER (ORDER BY salario DESC) AS rank,
         DENSE_RANK() OVER (ORDER BY salario DESC) AS dense_rank
  FROM empleados;
```
  - Comparar cada fila con el total de su grupo, sin subconsulta.
```sql
  SELECT nombre, salario,
         AVG(salario) OVER (PARTITION BY departamento_id) AS media_depto,
         salario - AVG(salario) OVER (PARTITION BY departamento_id) AS diferencia
  FROM empleados;
```
  - Acceder a la fila anterior o siguiente.
```sql
  SELECT fecha, importe,
         LAG(importe)  OVER (ORDER BY fecha) AS importe_anterior,
         LEAD(importe) OVER (ORDER BY fecha) AS importe_siguiente
  FROM ventas;
```
  - Suma acumulada.
```sql
  SELECT fecha, importe,
         SUM(importe) OVER (ORDER BY fecha) AS acumulado
  FROM ventas;
```

>[!IMPORTANT]
>Las funciones analíticas se calculan **después** del `WHERE` y del `GROUP BY`. Por eso no se pueden filtrar directamente en el `WHERE`: hay que envolver la consulta en una subconsulta o en un `WITH`.
