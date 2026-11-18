# Guía para preparar el reporte del proyecto Ames

## Propósito del reporte

El reporte debe explicar cómo el equipo diseñó, implementó y validó el proceso ETL en Azure Data Factory para integrar las fuentes del proyecto Ames y producir `salida.csv`. Su función es permitir que otra persona comprenda las decisiones tomadas, encuentre la evidencia que las respalda y determine si la salida cumple el enunciado.

El reporte no debe limitarse a enumerar actividades ni a reunir capturas de pantalla. Cada decisión importante debe responder tres preguntas:

1. ¿Qué se hizo?
2. ¿Por qué se hizo de esa manera?
3. ¿Qué evidencia demuestra que funcionó?

No invente resultados ni presente como verificados los conteos de referencia del enunciado. Informe los valores observados durante la ejecución del equipo.

## Relación con la rúbrica

La evaluación del reporte se distribuye de esta forma:

| Criterio | Porcentaje |
|---|---:|
| Objetivo del proyecto | 15 % |
| Extracción de las fuentes | 25 % |
| Transformaciones realizadas | 30 % |
| Tecnologías utilizadas | 10 % |
| Conclusiones y lecciones aprendidas | 20 % |

La descripción de la extracción y las transformaciones representa más de la mitad de la evaluación. Dedique espacio suficiente a explicar decisiones, cardinalidades, reglas y verificaciones; no concentre el reporte en la introducción o en imágenes generales de ADF.

## Principios de documentación

### Diferenciar referencia y observación

Los tamaños indicados en el enunciado son cifras de referencia. El reporte debe distinguirlos de los valores medidos por el equipo.

Ejemplo de forma de documentar:

| Objeto | Tamaño de referencia | Tamaño observado | Fecha o ejecución | Evidencia |
|---|---:|---:|---|---|
| `[nombre del objeto]` | `[valor del enunciado]` | `[valor medido]` | `[fecha o ejecución]` | `[consulta, captura o registro]` |

No complete la tabla con valores supuestos. Si una medición no fue posible, indíquelo y explique la limitación.

### Relacionar afirmaciones y evidencias

Una captura aislada no demuestra una conclusión. Presente primero la afirmación, después la evidencia y finalmente su interpretación.

Una evidencia útil debe tener:

- título o pie descriptivo;
- origen y momento de captura;
- datos legibles;
- relación explícita con la decisión que respalda;
- ausencia de credenciales, tokens y cadenas de conexión.

### Justificar decisiones

Evite frases como “se hizo un join” o “se limpiaron los datos” sin explicar criterios. Indique el tipo de unión, su cardinalidad esperada, los campos involucrados, la regla de faltantes y la comprobación posterior.

### Mantener trazabilidad

Use los mismos nombres para tablas, colecciones, columnas, actividades y Data Flows en el texto, las figuras y el repositorio. Numere tablas y figuras cuando el reporte tenga varias evidencias y haga referencia a ellas desde el texto.

A continuación se propone una estructura que cubre los criterios del proyecto. El equipo puede adaptar los títulos, pero debe conservar el contenido técnico y las evidencias solicitadas.

## 1. Identificación del proyecto

Incluya:

- nombre del curso y unidad;
- nombre del proyecto;
- integrantes del equipo;
- fecha de entrega;
- enlace al repositorio o identificador de la entrega, si corresponde.

Verifique que los nombres y roles coincidan con el repositorio. No incluya correos, usuarios, contraseñas ni datos de conexión.

## 2. Introducción y objetivo

Explique el problema en un párrafo breve: qué fuentes deben integrarse, qué tecnología se usa y cuál es el producto esperado.

El objetivo debe indicar:

- la construcción del ETL en Azure Data Factory;
- la integración del CSV, PostgreSQL y MongoDB;
- la generación de una fila por propiedad;
- la producción de los 81 campos de `salida.csv`.

Preguntas de control:

- ¿El objetivo describe un resultado verificable?
- ¿Distingue la integración de fuentes de una simple copia de datos?
- ¿Evita prometer resultados que el reporte no demuestra?

No copie el objetivo del enunciado sin contextualizarlo para el trabajo realizado por el equipo.

## 3. Fuentes de datos y exploración

Presente el inventario de fuentes antes de describir el pipeline.

### 3.1 Archivo Ames Property

Documente:

- ubicación utilizada por el pipeline;
- separador, codificación y encabezado observados;
- número de filas y columnas medido;
- tipo, unicidad y faltantes de `PID`;
- campos que aporta a la salida;
- problemas de calidad relevantes.

Evidencia mínima:

- vista de los encabezados o esquema;
- conteo de filas;
- comprobación de duplicados y faltantes de `PID`.

### 3.2 PostgreSQL en Neon

Describa el propósito de cada tabla utilizada, no solo su nombre. Incluya las tablas de datos, los catálogos y las relaciones necesarias para la salida.

Para cada objeto utilizado, documente:

- campos relevantes;
- llave declarada o candidata;
- cardinalidad respecto a `PID`;
- número de filas observado;
- faltantes, duplicados o códigos sin correspondencia;
- campos que aporta a `salida.csv`.

Dedique atención especial a `FloorDetail`, porque puede tener varias filas por propiedad.

Evidencia mínima:

- consulta de estructura o esquema;
- consulta de conteo;
- comprobación de duplicados según la llave correspondiente;
- ejemplo de la relación entre `PID` y el detalle por piso.

### 3.3 MongoDB Atlas

Documente por separado `garage`, `pool`, `bsmt` y `misc`.

Para cada colección incluya:

- campos encontrados y tipos observados;
- cantidad de documentos medida;
- presencia y tipo de `PID`;
- cantidad de PID duplicados o documentos sin PID;
- campos opcionales o ausentes;
- diferencia entre documento ausente, campo ausente, valor nulo y cero válido;
- campos que aporta a la salida.

Evidencia mínima:

- muestra legible de la estructura, sin datos sensibles;
- conteo de documentos;
- agregación o consulta de PID duplicados;
- evidencia de cobertura respecto al conjunto de propiedades.

### 3.4 Síntesis de la exploración

Consolide los hallazgos en una tabla. No repita todas las capturas en esta sección.

| Fuente y objeto | Llave | Cardinalidad respecto a PID | Tamaño observado | Problema de calidad | Aporte a la salida |
|---|---|---|---:|---|---|
| `[objeto]` | `[llave]` | `[1:1, 1:N u otra]` | `[medición]` | `[hallazgo]` | `[campos]` |

Explique cuáles hallazgos modificaron el diseño del ETL.

## 4. Diseño de la integración

Explique cómo se definió el universo de propiedades y cómo se integraron las fuentes.

El reporte debe responder:

- ¿Qué fuente determina la lista base de `PID`?
- ¿Qué cardinalidad se encontró antes de cada unión?
- ¿Qué objetos debieron agregarse antes de unirlos?
- ¿Qué tipo de unión se utilizó con cada fuente y por qué?
- ¿Cómo se conservaron las propiedades sin registro en `pool`, `garage`, `bsmt` o `misc`?
- ¿Cómo se comprobó que ninguna unión multiplicara propiedades?
- ¿Cómo se comprobó que ninguna unión eliminara propiedades esperadas?

Incluya un diagrama del flujo o una tabla de decisiones.

| Paso | Entrada izquierda | Entrada derecha | Preparación previa | Tipo de unión | Llave | Filas antes y después | Justificación |
|---|---|---|---|---|---|---|---|
| `[paso]` | `[objeto]` | `[objeto]` | `[agregación o limpieza]` | `[tipo]` | `PID` | `[medición]` | `[razón]` |

No afirme que una unión es correcta únicamente porque el pipeline terminó sin error.

## 5. Transformaciones

Organice esta sección por regla de negocio. Para cada transformación indique origen, operación, tratamiento de faltantes, resultado esperado y validación.

### 5.1 Área habitable

Documente el cálculo de `Gr Liv Area` a partir de las tres áreas de piso indicadas en el enunciado.

Incluya:

- nombres de los campos de entrada;
- expresión utilizada en ADF;
- tratamiento de valores no numéricos o ausentes;
- comprobación por fila, no solo mediante un total o promedio.

### 5.2 Habitaciones y baños

Explique la agregación de `FloorDetail` por `PID`.

Incluya:

- campos agregados;
- función empleada;
- momento en que se ejecuta la agregación;
- cardinalidad antes y después;
- comparación de una muestra de PID con la fuente.

### 5.3 Fecha de venta

Explique cómo se obtuvieron mes y año a partir de `Sale Date`.

Incluya el tipo de dato original, la función utilizada y una comprobación contra registros de la fuente.

### 5.4 Fecha de remodelación

Explique cómo se detectó la ausencia de `Year Remod/Add` y cómo se sustituyó por `Year Built`.

La evidencia debe distinguir un dato originalmente presente de uno sustituido; no basta con demostrar que la columna final no tiene vacíos.

### 5.5 Variables de calidad y catálogos

Explique cómo se utilizaron las tablas de referencia para convertir identificadores o códigos cuando corresponda.

Indique:

- tabla de referencia;
- campos de unión;
- cardinalidad;
- tratamiento de códigos sin correspondencia;
- comprobación por `PID`.

### 5.6 Datos ausentes de MongoDB

Explique la aplicación de:

- `NA` para variables cualitativas ausentes;
- `0` para variables numéricas ausentes.

Muestre cómo la regla se aplica cuando falta el documento y cuando falta un campo. Aclare cómo se conserva un cero que ya era un valor válido.

### 5.7 Transformaciones opcionales

Para `Neighborhood`, `Condition` y `Lot Shape`, indique si el equipo conservó los valores originales o aplicó la conversión a códigos.

Si se aplicó la transformación, documente la tabla de correspondencias y demuestre que se aplicó de manera completa y consistente. Si no se aplicó, indíquelo sin presentar la ausencia de transformación como un error.

### Matriz de transformaciones

| Campo de salida | Fuente y campo original | Regla | Implementación en ADF | Faltantes | Evidencia de validación |
|---|---|---|---|---|---|
| `[campo]` | `[origen]` | `[regla]` | `[actividad o expresión]` | `[tratamiento]` | `[prueba]` |

## 6. Implementación en Azure Data Factory

Describa la arquitectura implementada y la secuencia de ejecución.

Incluya:

- linked services utilizados, sin parámetros sensibles;
- datasets de entrada y salida;
- pipelines y Data Flows;
- dependencias entre actividades;
- transformaciones principales;
- configuración del sink de `salida.csv`;
- forma de ejecución y monitoreo.

Para cada figura de ADF, explique qué parte del proceso representa y qué decisión permite verificar. Una captura completa del canvas puede mostrar el panorama general, pero no reemplaza las evidencias legibles de las expresiones, agregaciones y uniones relevantes.

Si el equipo reintentó actividades, cambió particiones, ajustó tipos o resolvió errores del conector, documente el problema y la decisión que permitió continuar.

## 7. Validación de salida.csv

Explique la estrategia de validación y presente los resultados obtenidos por el equipo. Puede utilizar el [notebook de autovalidación](../notebooks/validar_salida_ames.ipynb), pero el reporte debe interpretar sus resultados; no basta con adjuntar una captura del resumen.

Incluya como mínimo:

- cantidad de filas y columnas observada;
- convención de nombres elegida;
- verificación del orden de los 81 campos;
- unicidad y cobertura de `PID`;
- faltantes y marcadores `NA` o `0`;
- resultados de las transformaciones obligatorias;
- validaciones contra fuentes disponibles;
- errores encontrados y correcciones realizadas;
- resultado de la última ejecución del pipeline.

Organice los resultados en una tabla:

| Verificación | Método | Resultado observado | Estado | Evidencia o corrección |
|---|---|---|---|---|
| `[prueba]` | `[consulta o notebook]` | `[medición]` | `[aprobado, error o no evaluado]` | `[referencia]` |

No copie como resultado propio los valores de otro equipo o de un archivo de ejemplo. No elimine pruebas fallidas del reporte: explique el hallazgo, la corrección y la nueva ejecución.

## 8. Tecnologías utilizadas

Liste únicamente tecnologías que el equipo utilizó y explique su función concreta.

Ejemplos de categorías que pueden corresponder al proyecto:

- orquestación e integración;
- almacenamiento de archivos;
- base de datos relacional;
- base de datos documental;
- exploración y consulta;
- validación;
- control de versiones y documentación.

Evite una lista genérica de productos. Relacione cada tecnología con una actividad descrita previamente.

## 9. Resultados, limitaciones y reproducibilidad

Resuma qué produjo la ejecución y bajo qué condiciones.

Documente:

- versión o fecha de las fuentes utilizadas;
- parámetros no sensibles necesarios para repetir la ejecución;
- ubicación de los artefactos del pipeline;
- ubicación de `salida.csv` y del notebook;
- limitaciones conocidas;
- verificaciones que no pudieron ejecutarse y su causa.

No incluya credenciales ni dependa de rutas personales de un integrante.

## 10. Conclusiones y lecciones aprendidas

Las conclusiones deben derivarse de la experiencia del equipo, no repetir el objetivo.

Incluya:

- qué decisiones fueron determinantes para conservar una fila por propiedad;
- qué dificultades surgieron por las diferencias entre CSV, PostgreSQL y MongoDB;
- qué errores se detectaron durante la validación;
- cómo se resolvieron;
- qué haría diferente el equipo en una siguiente versión;
- aprendizajes técnicos y de coordinación.

Cada lección debe relacionarse con una situación concreta. Evite afirmaciones genéricas como “aprendimos mucho sobre ADF” sin explicar el problema, la acción y el resultado.

## 11. Referencias y anexos

Incluya referencias cuando se utilicen documentos, manuales o recursos externos. Use un formato consistente, como IEEE o ACM.

Los anexos pueden contener:

- consultas completas;
- matrices de caracterización;
- tablas de correspondencias;
- detalles adicionales de validación;
- capturas secundarias.

El cuerpo del reporte debe contener la explicación necesaria para comprender el trabajo sin depender por completo de los anexos.

## Evidencias mínimas sugeridas

| Evidencia | Qué debe demostrar |
|---|---|
| Inventario de fuentes | Objetos realmente disponibles y su propósito |
| Conteos observados | Volumen medido, no asumido |
| Revisión de PID | Tipo, unicidad, cobertura y cardinalidad |
| Agregación de FloorDetail | Reducción de varias filas a una fila por PID |
| Decisiones de unión | Conservación de propiedades y ausencia de multiplicación |
| Expresiones de transformación | Implementación de cada regla obligatoria |
| Tratamiento de MongoDB | Uso correcto de `NA` y `0` |
| Flujo general de ADF | Secuencia y dependencias del pipeline |
| Ejecución monitoreada | Estado y momento de una ejecución reproducible |
| Validación final | Esquema, PID, faltantes y reglas por fila o por PID |

La cantidad de capturas no determina la calidad del reporte. Seleccione evidencias legibles y explique su significado.

## Lista de verificación antes de entregar

### Objetivo y alcance

- [ ] El objetivo describe la integración, ADF y el archivo esperado.
- [ ] Se diferencia la exploración inicial de la implementación final.
- [ ] No se presentan resultados no verificados.

### Fuentes

- [ ] Se documentan CSV, PostgreSQL y las cuatro colecciones MongoDB.
- [ ] Los conteos de referencia se distinguen de los observados.
- [ ] Se justifican llaves y cardinalidades.
- [ ] Se documentan faltantes, duplicados y diferencias de tipos.

### Integración y transformaciones

- [ ] Se explica qué fuente define el universo de `PID`.
- [ ] Se justifica cada tipo de unión.
- [ ] `FloorDetail` se agrega antes de integrarlo.
- [ ] Se documentan las cuatro reglas de transformación obligatorias.
- [ ] Se explica el tratamiento de ausencias en MongoDB.
- [ ] Se documenta la decisión sobre las transformaciones opcionales.

### Implementación

- [ ] Se explica la arquitectura de ADF y la secuencia de ejecución.
- [ ] Las capturas son legibles y tienen título o pie.
- [ ] Las expresiones y configuraciones importantes están respaldadas por evidencia.
- [ ] No se incluyen secretos ni cadenas de conexión.

### Validación

- [ ] Se verifican 81 campos en el orden requerido.
- [ ] Se declara y comprueba la convención de encabezados.
- [ ] Se comprueba unicidad y cobertura de `PID`.
- [ ] Se revisan marcadores `NA` y `0`.
- [ ] Las transformaciones se validan por fila o por `PID`.
- [ ] Se documentan errores, correcciones y nueva ejecución.

### Cierre

- [ ] Las tecnologías se relacionan con su uso real.
- [ ] Las conclusiones responden al objetivo.
- [ ] Las lecciones aprendidas se apoyan en situaciones concretas.
- [ ] Las referencias siguen un formato consistente.
- [ ] El reporte puede entenderse sin acceder a credenciales o cuentas personales.

## Errores que reducen la calidad del reporte

- Copiar el enunciado como introducción sin explicar el trabajo realizado.
- Presentar conteos de referencia como si hubieran sido medidos.
- Describir herramientas sin relacionarlas con decisiones del ETL.
- Insertar capturas sin título, interpretación o texto legible.
- Mostrar únicamente que el pipeline terminó, sin validar la salida.
- Reportar sumas globales que pueden ocultar errores por propiedad.
- Omitir los problemas encontrados y mostrar solo el resultado final.
- Confundir un documento ausente en MongoDB con un cero observado.
- Incluir credenciales, cadenas de conexión o rutas personales.
- Redactar conclusiones genéricas que no se desprenden de la experiencia del equipo.
