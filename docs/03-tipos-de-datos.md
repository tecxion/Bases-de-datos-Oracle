# TIPOS DE DATOS

Cada columna de una tabla tiene que declarar de qué tipo son los valores que va a guardar.

#### TEXTO
 - `VARCHAR2(n)`: Cadena de longitud variable, hasta `n` caracteres. **Es el tipo de texto que se usa en Oracle.**
 - `CHAR(n)`: Cadena de longitud fija. Si el valor es más corto, Oracle rellena con espacios.
 - `NVARCHAR2(n)` / `NCHAR(n)`: Igual que los anteriores pero en el juego de caracteres nacional (Unicode).
 - `CLOB`: Texto muy largo, hasta 4 GB. Para descripciones, documentos, etc.

>[!NOTE]
>`VARCHAR` existe pero Oracle recomienda no usarlo: está reservado para cambios futuros. Usa siempre `VARCHAR2`.

```sql
  nombre    VARCHAR2(50),
  provincia CHAR(2),
  biografia CLOB
```

#### NÚMEROS
 - `NUMBER(p, s)`: Numérico. `p` es la precisión (total de dígitos) y `s` la escala (dígitos decimales).
 - `NUMBER`: Sin precisión, admite cualquier número.
 - `NUMBER(p)`: Entero de `p` dígitos, equivale a `NUMBER(p, 0)`.
 - `FLOAT`, `BINARY_FLOAT`, `BINARY_DOUBLE`: Coma flotante. Más rápidos pero con errores de redondeo.

```sql
  id       NUMBER(6),          -- entero hasta 999999
  salario  NUMBER(8,2),        -- hasta 999999.99
  ratio    NUMBER              -- cualquier número
```

>[!CAUTION]
>Para dinero usa siempre `NUMBER(p,s)`, nunca `BINARY_DOUBLE`. La coma flotante pierde céntimos.

#### FECHAS Y HORAS
 - `DATE`: Fecha **y hora** (siglo, año, mes, día, hora, minuto, segundo). No guarda fracciones de segundo.
 - `TIMESTAMP(n)`: Como `DATE` pero con `n` decimales de segundo.
 - `TIMESTAMP WITH TIME ZONE`: Añade la zona horaria.
 - `TIMESTAMP WITH LOCAL TIME ZONE`: Convierte a la zona horaria de quien consulta.
 - `INTERVAL YEAR TO MONTH` / `INTERVAL DAY TO SECOND`: Guardan una duración, no un instante.

```sql
  fecha_alta    DATE DEFAULT SYSDATE,
  registrado_en TIMESTAMP(6),
  antiguedad    INTERVAL YEAR TO MONTH
```

>[!IMPORTANT]
>`DATE` **siempre** lleva hora, aunque no la veas. Si comparas `fecha = DATE '2024-01-15'` y la fila se guardó a las 10:30, no la encontrarás. Usa `TRUNC(fecha) = DATE '2024-01-15'`.

#### BINARIOS
 - `BLOB`: Datos binarios hasta 4 GB (imágenes, PDF, vídeo).
 - `RAW(n)`: Binario corto, hasta 2000 bytes.
 - `BFILE`: Puntero a un fichero binario guardado fuera de la base de datos.

#### OTROS
 - `ROWID`: Dirección física única de una fila. Oracle la asigna sola.
 - `XMLTYPE`: Documento XML, con consultas propias.
 - `JSON`: Documento JSON nativo (desde Oracle 21c). En versiones anteriores se usa `VARCHAR2` o `CLOB` con `IS JSON`.

#### CONVERSIÓN ENTRE TIPOS
  - Número o fecha a texto.
```sql
  TO_CHAR(salario, '999G999D99')
  TO_CHAR(fecha_alta, 'DD/MM/YYYY')
```
  - Texto a número.
```sql
  TO_NUMBER('1234.56')
```
  - Texto a fecha. **Hay que indicar siempre la máscara**, no confíes en el formato por defecto de la sesión.
```sql
  TO_DATE('15/01/2024', 'DD/MM/YYYY')
```
  - Literal de fecha ANSI, no necesita máscara. El formato es siempre `YYYY-MM-DD`.
```sql
  DATE '2024-01-15'
```

#### MÁSCARAS DE FORMATO MÁS USADAS

| Máscara | Significado | Ejemplo |
|---|---|---|
| `DD` | Día del mes (01-31) | `15` |
| `MM` | Mes en número (01-12) | `01` |
| `MON` | Mes abreviado | `ENE` |
| `MONTH` | Mes completo | `ENERO` |
| `YYYY` | Año con 4 dígitos | `2024` |
| `YY` | Año con 2 dígitos | `24` |
| `HH24` | Hora (00-23) | `14` |
| `HH` | Hora (01-12) | `02` |
| `MI` | Minutos | `30` |
| `SS` | Segundos | `45` |
| `DAY` | Día de la semana | `LUNES` |
| `9` | Dígito, se omite si es cero a la izquierda | `999` |
| `0` | Dígito, se muestra aunque sea cero | `0999` |
| `D` | Separador decimal | `999D99` |
| `G` | Separador de miles | `999G999` |
| `L` | Símbolo de moneda local | `L999D99` |

  - Ver y cambiar el formato de fecha por defecto de la sesión.
```sql
  SELECT * FROM NLS_SESSION_PARAMETERS WHERE parameter = 'NLS_DATE_FORMAT';
  ALTER SESSION SET NLS_DATE_FORMAT = 'DD/MM/YYYY HH24:MI:SS';
```

#### VALORES NULOS

`NULL` no es cero ni cadena vacía: significa "valor desconocido". Cualquier operación aritmética con `NULL` devuelve `NULL`.

  - No se compara con `=`, se usa `IS NULL` / `IS NOT NULL`.
```sql
  SELECT * FROM empleados WHERE comision IS NULL;
```
  - Sustituir el nulo por un valor.
```sql
  SELECT nombre, NVL(comision, 0) AS comision FROM empleados;
```

>[!CAUTION]
>En Oracle la cadena vacía `''` **es** `NULL`. Es la única base de datos que hace esto, y sorprende al venir de MySQL o PostgreSQL.
