# Guía de uso del notebook de validación de salida Ames

## Propósito

`validar_salida_ames.ipynb` permite revisar `salida.csv` antes de la entrega del proyecto. El notebook no modifica el archivo, no corrige encabezados y no calcula una nota. Presenta hallazgos para que el equipo corrija su ETL y vuelva a generar la salida.

Las pruebas siguen los siete criterios de la rúbrica:

1. Dimensiones correctas.
2. Ausencia de valores nulos.
3. Nombres y orden de las columnas.
4. Transformaciones obligatorias.
5. Transformaciones opcionales.
6. Variables de calidad.
7. Extracción de MongoDB.

## Requisitos

- Python 3 con Jupyter y pandas.
- El repositorio del proyecto con la carpeta `sources/` completa.
- El archivo producido por el pipeline, llamado `salida.csv`.
- No se necesitan credenciales de Neon, MongoDB Atlas ni Azure Data Factory.

## Preparación

La ubicación predeterminada de la salida es:

```text
out/salida.csv
```

Puede copiar el archivo a esa ubicación o cambiar `SALIDA_PATH` en la primera celda de configuración. No edite las fuentes originales para hacer las pruebas.

El notebook busca automáticamente la raíz del repositorio y espera encontrar:

```text
sources/AmesProperty.csv
sources/scriptDatos/
sources/mongoCollections/
```

Si el equipo solo dispone de `salida.csv`, puede establecer `RUN_SOURCE_CHECKS = False`. En ese caso, las pruebas que necesitan orígenes aparecerán como `NO EVALUADO`, no como aprobadas.

## Selección de la convención de nombres

Cada equipo debe elegir una sola convención en `NAMING_STYLE`:

```python
NAMING_STYLE = "camelCase"
```

```python
NAMING_STYLE = "snake_case"
```

```python
NAMING_STYLE = "kebab-case"
```

Las siglas `PID`, `MS` y `QC` permanecen en mayúsculas. Los demás componentes siguen la convención seleccionada.

| Campo | camelCase | snake_case | kebab-case |
|---|---|---|---|
| PID | `PID` | `PID` | `PID` |
| MS SubClass | `MSSubClass` | `MS_sub_class` | `MS-sub-class` |
| Heating QC | `heatingQC` | `heating_QC` | `heating-QC` |
| Year Remod Add | `yearRemodAdd` | `year_remod_add` | `year-remod-add` |

El notebook muestra al comienzo la lista completa de los 81 encabezados esperados. La comparación es exacta y respeta el orden. No se eliminan espacios, no se cambia el uso de mayúsculas y no se convierten los encabezados recibidos.

## Separador y codificación

Con `DELIMITER = None`, el notebook intenta detectar `;`, `,`, tabulador o `|`. Si la detección no funciona, establezca explícitamente el separador, por ejemplo:

```python
DELIMITER = ";"
```

La codificación predeterminada es `utf-8-sig`, que permite leer archivos UTF-8 con o sin marca BOM.

## Reglas para valores faltantes

- Campo cualitativo faltante: `NA`, exactamente en mayúsculas.
- Campo numérico faltante: `0`.
- No se aceptan celdas vacías ni marcadores como `null`, `NaN`, `N/A`, `na` o `<NA>`.
- `None` puede ser un valor categórico real en Ames y no se interpreta automáticamente como nulo.
- Las reglas específicas tienen prioridad. Por ejemplo, si `Year Remod/Add` falta en la fuente, la salida debe contener `Year Built`, no `0`.

Un cero puede ser un dato real o el marcador de ausencia exigido. La prueba con solo `salida.csv` verifica que su representación sea válida; las pruebas con fuentes determinan si fue aplicado al `PID` correcto.

## Ejecución

1. Abra `notebooks/validar_salida_ames.ipynb` en Jupyter o VS Code.
2. Ajuste las rutas y `NAMING_STYLE` en la primera celda de código.
3. Ejecute todas las celdas desde el inicio.
4. Revise primero el resumen por criterio y después el detalle de las pruebas.
5. Corrija el pipeline o las transformaciones que producen el error.
6. Genere nuevamente `salida.csv` y vuelva a ejecutar todo el notebook.

También puede iniciar Jupyter desde la raíz del repositorio:

```bash
jupyter notebook notebooks/validar_salida_ames.ipynb
```

## Interpretación de estados

| Estado | Significado |
|---|---|
| `APROBADO` | La prueba no encontró diferencias. |
| `ERROR` | Se encontró al menos un incumplimiento que debe corregirse. |
| `ADVERTENCIA` | Se observó un riesgo, por ejemplo espacios al inicio o al final de valores. |
| `NO EVALUADO` | Faltó el archivo, una columna o una fuente necesaria. No equivale a aprobado. |

Los ejemplos de `PID` se limitan para mantener legible la salida. El conteo del problema incluye todas las filas revisadas.

## Qué comprueba cada grupo

### Dimensiones y PID

- 2.930 filas y 81 columnas.
- `PID` presente, numérico, no vacío y único.
- Cobertura frente a `Ames Property.csv` cuando la fuente está disponible.

### Encabezados

- Convención declarada aplicada a los 81 campos.
- Orden exacto.
- Ausencia de encabezados repetidos.
- Ausencia de espacios.

### Transformaciones obligatorias

- `Gr Liv Area` calculada por fila.
- Sustitución de `Year Remod/Add` faltante por `Year Built`.
- Suma por `PID` de habitaciones, baños completos y medios baños de `FloorDetail`.
- Mes y año derivados de `Sale Date`.

### Transformaciones opcionales

Para `Neighborhood`, `Condition 1`, `Condition 2` y `Lot Shape` se acepta una de dos decisiones completas:

- conservar los valores originales, o
- transformar toda la columna a los códigos definidos.

Una transformación parcial o una mezcla inconsistente se reporta como error.

### Variables de calidad

Se comparan por `PID` `Exter Cond`, `Exter Qual`, `Heating QC` y `Kitchen Qual` con los códigos de `TypeQuality`.

### MongoDB

- Se revisa la cardinalidad de `garage`, `pool`, `bsmt` y `misc` antes de comparar.
- Se rechazan PID duplicados o documentos sin PID en las fuentes locales.
- Los campos se comparan por `PID`.
- Cuando no hay documento o campo, se esperan `NA` y `0` según el tipo.

## Problemas frecuentes

- Elegir `snake_case` en la configuración, pero producir encabezados camelCase.
- Dejar espacios o mezclar guiones con guiones bajos.
- Cambiar el orden de los 81 campos.
- Usar `NA` en una columna numérica o `0` en una cualitativa.
- Hacer un `INNER JOIN` con `pool` y perder propiedades.
- Unir `FloorDetail` sin agregar primero por `PID` y multiplicar filas.
- Validar una transformación mediante un total global: dos errores pueden compensarse. El notebook compara por fila o por `PID`.

## Alcance y seguridad

El notebook es una herramienta de autovalidación. Un resultado aprobado no reemplaza la revisión del pipeline, el reporte, la presentación ni la evaluación docente. No agregue contraseñas, cadenas de conexión, tokens ni archivos de configuración con secretos.
