# Guía para preparar la presentación del proyecto Ames

## Propósito de la presentación

La presentación debe explicar de forma clara cómo el equipo construyó y validó el proceso ETL del proyecto Ames. La audiencia debe comprender el objetivo, el flujo del pipeline, las decisiones técnicas, los resultados observados y las lecciones aprendidas.

La exposición no es una lectura del reporte ni una secuencia de capturas de Azure Data Factory. Debe seleccionar la evidencia más importante y construir una narrativa que conecte:

1. el problema que debía resolverse;
2. las decisiones tomadas;
3. la implementación realizada;
4. las comprobaciones ejecutadas;
5. los aprendizajes del equipo.

No presente conteos de referencia como resultados propios ni invente métricas para completar las diapositivas.

## Relación con la rúbrica

La rúbrica evalúa seis dimensiones de la presentación:

| Dimensión | Aspectos observados |
|---|---|
| Introducción | Objetivo, integrantes, agenda y contexto claro |
| Construcción del pipeline | Etapas, decisiones técnicas, flujo general y contribuciones |
| Resultados | Claridad, interpretación, visualizaciones y relación con el objetivo |
| Lecciones aprendidas | Desafíos, estrategias, aprendizajes y mejoras futuras |
| Manejo del tiempo | Cumplimiento del tiempo, equilibrio y selección de detalles |
| Calidad de las diapositivas | Legibilidad, organización, visualizaciones y narrativa |

La presentación debe cubrir todas las dimensiones. Una explicación técnicamente correcta puede perder claridad si omite el objetivo, no interpreta los resultados o concentra toda la participación en una sola persona.

## Mensaje central

Antes de crear diapositivas, escriban en una frase qué debe recordar la audiencia al terminar. Esa frase debe relacionar el problema con el resultado verificado, sin adelantar datos que el equipo no haya medido.

Utilicen la frase como filtro:

- si una diapositiva no ayuda a comprender el objetivo, una decisión, un resultado o una lección, probablemente no es necesaria;
- si una evidencia es importante pero demasiado detallada, ubíquenla en una diapositiva de respaldo;
- si una afirmación no puede demostrarse, reformúlenla o elimínenla.

## Preparación del contenido

La [guía del reporte](GUIA_PREPARACION_REPORTE.md) contiene la documentación completa. Para la presentación, seleccionen solo la información que la audiencia necesita para seguir el razonamiento.

Antes de diseñar las diapositivas, reúnan:

- objetivo final aprobado por el equipo;
- diagrama del flujo de integración;
- inventario resumido de las fuentes;
- decisiones de cardinalidad y unión;
- transformaciones obligatorias;
- evidencia legible del pipeline;
- resultados de la validación;
- errores encontrados y correcciones realizadas;
- desafíos y aprendizajes concretos.

Comprueben que las cifras de las diapositivas coincidan con la última ejecución descrita en el reporte.

## Secuencia recomendada

La cantidad de diapositivas depende del tiempo asignado. Cada etapa puede ocupar una o más diapositivas, pero debe conservarse la secuencia narrativa.

## 1. Apertura

Incluya:

- nombre del proyecto;
- curso y unidad;
- integrantes del equipo;
- una descripción breve del producto construido.

La apertura debe permitir que la audiencia identifique al equipo y el propósito antes de entrar en detalles técnicos.

Evite:

- una portada con información administrativa excesiva;
- leer todos los datos de la diapositiva;
- comenzar mostrando ADF sin explicar el problema.

## 2. Agenda y objetivo

Presente una agenda breve que anticipe el recorrido de la exposición. El objetivo debe indicar la integración de las tres tecnologías, el uso de Azure Data Factory y la generación de `salida.csv` con una fila por propiedad y 81 campos. Al describir el esquema, indique la convención elegida por el equipo —camel case, snake case o kebab case—, la ausencia de espacios y la conservación en mayúsculas de `PID`, `MS` y `QC`.

Preguntas de control:

- ¿El objetivo es verificable?
- ¿La agenda coincide con lo que realmente se presentará?
- ¿La introducción permite entender por qué la integración es necesaria?

## 3. Fuentes y situación inicial

Muestre una visión compacta del CSV, PostgreSQL y MongoDB Atlas.

Una tabla o diagrama puede resumir:

| Fuente | Objetos utilizados | Papel en la salida | Riesgo de integración |
|---|---|---|---|
| `[fuente]` | `[objetos]` | `[campos aportados]` | `[cardinalidad, faltantes o tipos]` |

La explicación debe destacar los hallazgos que influyeron en el diseño, por ejemplo:

- presencia y tipo de `PID`;
- objetos con varias filas por propiedad;
- colecciones sin cobertura completa;
- diferencias entre ausencia, `NA`, cero y valor válido;
- códigos que requieren tablas de referencia.

No dedique tiempo a leer todas las columnas de cada fuente. Muestre únicamente las que ayudan a entender el pipeline y las transformaciones.

## 4. Diseño de la integración

Presente el flujo general antes de mostrar detalles de ADF.

El diagrama debe permitir identificar:

- fuente que define el universo de propiedades;
- preparación de cada origen;
- agregaciones anteriores a las uniones;
- secuencia de integración mediante `PID`;
- tipo de unión utilizado;
- aplicación de reglas de faltantes;
- generación de la salida.

Explique las decisiones más importantes:

- cómo se evitó perder propiedades sin registro en `pool` u otra colección;
- cómo se evitó multiplicar propiedades al integrar `FloorDetail`;
- cómo se comprobó la cardinalidad antes y después de cada unión;
- cómo se conservaron los 81 campos en el orden requerido.

No presente el tipo de unión como una decisión obvia. Relacione su elección con la cobertura y cardinalidad observadas.

## 5. Construcción del pipeline en ADF

Muestre el pipeline y los Data Flows con suficiente resolución para leer nombres y relaciones.

La explicación debe cubrir:

- actividades principales y su secuencia;
- datasets y linked services utilizados, sin mostrar parámetros sensibles;
- transformaciones dentro de los Data Flows;
- dependencias entre actividades;
- configuración relevante del sink;
- forma de ejecución y monitoreo.

Utilice una captura general para mostrar la arquitectura y acercamientos separados para decisiones específicas. No intente explicar todas las propiedades visibles en la interfaz.

Para cada captura, indique:

1. qué componente se muestra;
2. qué función cumple;
3. qué decisión o regla permite verificar.

## 6. Transformaciones principales

Seleccione las transformaciones que determinan la corrección de la salida:

- cálculo del campo de área habitable (`GrLivArea` o su equivalente en la convención elegida);
- agregación de habitaciones y baños de `FloorDetail`;
- obtención de mes y año de venta;
- sustitución de `Year Remod/Add`;
- conversión de variables de calidad;
- aplicación de las reglas del enunciado para representar ausencias de MongoDB, usando únicamente `NA` o `0` según corresponda;
- transformaciones opcionales realizadas por el equipo.

Una tabla compacta puede comunicar mejor que varias capturas:

| Campo o grupo | Fuente | Regla | Implementación | Validación |
|---|---|---|---|---|
| `[campo]` | `[origen]` | `[regla]` | `[actividad o expresión]` | `[prueba]` |

Incluya una captura de la expresión o actividad cuando sea necesaria para demostrar la implementación. Evite mostrar bloques largos de código o configuraciones que no puedan leerse durante la exposición.

## 7. Resultados y validación

Presente resultados medidos en la ejecución del equipo. Conecte cada resultado con el objetivo y con una prueba.

Incluya como mínimo:

- dimensiones observadas de `salida.csv`;
- unicidad y cobertura de `PID`;
- convención de encabezados utilizada, sin espacios y con `PID`, `MS` y `QC` en mayúsculas;
- revisión de faltantes y uso de los marcadores permitidos `NA` y `0`;
- validación de las transformaciones obligatorias;
- comprobaciones contra las fuentes;
- estado de la ejecución final del pipeline.

Puede utilizar el resumen del [notebook de autovalidación](../notebooks/validar_salida_ames.ipynb), pero debe interpretarlo. No muestre una tabla extensa sin explicar cuáles pruebas fueron determinantes y qué correcciones se realizaron.

Una estructura útil es:

| Verificación | Resultado observado | Estado | Interpretación |
|---|---|---|---|
| `[prueba]` | `[medición]` | `[estado]` | `[qué demuestra]` |

Si una prueba quedó como `NO EVALUADO`, indíquelo y explique la causa. No la presente como aprobada.

## 8. Desafíos y decisiones correctivas

Seleccione pocos desafíos que hayan exigido análisis. Para cada uno explique:

1. situación o síntoma observado;
2. causa identificada;
3. decisión aplicada;
4. evidencia de que la corrección funcionó.

Ejemplos de categorías posibles, sin asumir que ocurrieron en todos los equipos:

- problemas de conexión o tipos de datos;
- multiplicación o pérdida de filas;
- diferencias entre nulos, vacíos y ceros;
- códigos sin correspondencia;
- configuración del delimitador o encabezado;
- discrepancias detectadas por el notebook.

No convierta esta sección en una lista de errores sin solución ni oculte los problemas encontrados.

## 9. Lecciones aprendidas y mejoras

Presente aprendizajes técnicos y de trabajo en equipo relacionados con situaciones concretas.

Incluya:

- decisión que mejoró la confiabilidad del ETL;
- práctica que habría evitado un problema encontrado;
- aprendizaje sobre integración de fuentes heterogéneas;
- aprendizaje sobre pruebas y observabilidad;
- cambio que realizarían en una siguiente versión.

Evite conclusiones genéricas como “aprendimos a usar Azure”. Explique qué aprendieron, a partir de qué situación y cómo cambiaría una implementación futura.

## 10. Cierre

Regrese al objetivo y resuma qué quedó demostrado. El cierre debe responder:

- ¿Se produjo el archivo solicitado?
- ¿Qué verificaciones respaldan su corrección?
- ¿Qué limitación relevante permanece?

No introduzca información técnica nueva en la última diapositiva. Termine con una frase clara y deje visible una diapositiva adecuada para preguntas.

## Participación del equipo

Todos los integrantes deben conocer el flujo completo, aunque cada persona explique una sección diferente.

Distribuya la exposición según bloques coherentes, por ejemplo:

- objetivo y fuentes;
- diseño de integración;
- implementación y transformaciones;
- validación y resultados;
- desafíos y lecciones.

La distribución debe ajustarse al número de integrantes y al tiempo disponible. Evite dividir una misma explicación entre varias personas sin una transición clara.

Cada transición debe indicar:

- qué se acaba de demostrar;
- qué sección continúa;
- cómo se relaciona con el objetivo.

No afirme que una persona fue responsable exclusiva de un componente si la evidencia del repositorio o la organización del equipo no lo respalda.

## Manejo del tiempo

Ensayen la presentación completa con el mismo orden, archivos y dispositivo que utilizarán en la exposición.

Durante el ensayo:

- registren la duración total;
- midan el tiempo de cada bloque;
- comprueben que todos los integrantes participan;
- identifiquen explicaciones repetidas;
- reduzcan detalles antes de acelerar el ritmo;
- reserven un cierre claro y tiempo para preguntas, si la dinámica lo contempla.

La mayor parte del tiempo debe dedicarse a la construcción del pipeline, las decisiones técnicas y los resultados. La agenda, la descripción de herramientas y la portada deben ser breves.

Si una sección excede su tiempo, decidan qué idea es esencial y qué evidencia puede trasladarse a diapositivas de respaldo.

## Diseño de las diapositivas

### Legibilidad

- Use tipografía y tamaños que puedan leerse a distancia.
- Mantenga alto contraste entre texto y fondo.
- Evite párrafos completos y tablas con demasiadas filas.
- Recorte las capturas para mostrar la evidencia necesaria sin perder contexto.
- Compruebe la legibilidad de nombres, expresiones y cifras en la pantalla de presentación.

### Organización visual

- Dedique cada diapositiva a una idea principal.
- Utilice títulos que expresen el contenido, no títulos genéricos como “Resultados 2”.
- Mantenga posiciones, colores y estilos consistentes.
- Use diagramas para flujos, tablas para comparaciones y gráficos cuando exista una relación cuantitativa que interpretar.
- Destaque solo los elementos que se explicarán oralmente.

### Uso de evidencia

- Identifique la fuente o ejecución de cada cifra relevante.
- Diferencie conteos de referencia y conteos observados.
- Incluya pies breves en capturas y diagramas.
- No muestre credenciales, cadenas de conexión, nombres de usuario, tokens ni paneles con información sensible.

### Cantidad de texto

Las diapositivas apoyan la explicación; no contienen el guion completo. Use frases breves, etiquetas y cifras. Coloque explicaciones adicionales en notas del presentador o en el reporte.

## Demostraciones y contingencias

Una demostración en vivo solo debe incluirse si aporta evidencia que no puede comunicarse de forma más confiable con una captura o un diagrama.

Si se realiza una demostración:

- prepare de antemano la vista exacta que se mostrará;
- evite iniciar sesión o escribir credenciales frente a la audiencia;
- no modifique recursos compartidos durante la exposición;
- tenga capturas de respaldo;
- conozca el resultado esperado y el tiempo de ejecución;
- continúe la presentación si la conexión o el servicio falla.

No dependa de que ADF, Neon o MongoDB Atlas estén disponibles en tiempo real para completar la narrativa.

## Preparación para preguntas

Todos los integrantes deben poder responder preguntas sobre:

- fuente que define el universo de `PID`;
- cardinalidad de cada objeto antes de unirlo;
- tipo de unión seleccionado;
- agregación de `FloorDetail`;
- efecto de una colección con poca cobertura, como `pool`;
- diferencia entre documento ausente, campo ausente y cero válido;
- sustitución de `Year Remod/Add`;
- convención de los 81 encabezados;
- pruebas que detectan pérdida o multiplicación de propiedades;
- correcciones realizadas después de validar la salida.

Preparen respuestas basadas en evidencia propia. Si una comprobación no se ejecutó, indíquenlo con precisión en lugar de improvisar un resultado.

## Diapositivas de respaldo

Las diapositivas de respaldo no forman parte del recorrido principal, pero pueden apoyar preguntas técnicas.

Pueden incluir:

- detalle de consultas de cardinalidad;
- matriz completa de transformaciones;
- campos de las fuentes;
- resultados ampliados del notebook;
- expresiones de ADF;
- evidencia adicional de ejecución;
- referencias externas.

No traslade a respaldo una evidencia indispensable para comprender el resultado principal.

## Lista de verificación antes de presentar

### Introducción

- [ ] La portada identifica proyecto, curso e integrantes.
- [ ] El objetivo es claro y verificable.
- [ ] La agenda coincide con la exposición.
- [ ] El contexto puede entenderse sin conocer previamente el pipeline.

### Construcción del pipeline

- [ ] Se presenta primero el flujo general.
- [ ] Se explican las etapas principales de ADF.
- [ ] Se justifican agregaciones y tipos de unión.
- [ ] Se muestra cómo se evita perder o multiplicar propiedades.
- [ ] Las contribuciones del equipo se describen con claridad.

### Resultados

- [ ] Las cifras provienen de la última ejecución verificada.
- [ ] Se distingue referencia de observación.
- [ ] Los resultados se conectan con el objetivo.
- [ ] Las visualizaciones y capturas son legibles.
- [ ] Las pruebas fallidas y sus correcciones se explican cuando son relevantes.

### Lecciones aprendidas

- [ ] Se presentan desafíos concretos.
- [ ] Se explican las estrategias aplicadas.
- [ ] Se distinguen aprendizajes técnicos y de coordinación.
- [ ] Se proponen mejoras realistas para una siguiente versión.

### Tiempo y participación

- [ ] La presentación completa fue ensayada con cronómetro.
- [ ] Cada integrante tiene una sección y conoce el flujo completo.
- [ ] Las transiciones están preparadas.
- [ ] El cierre y las preguntas tienen espacio suficiente.
- [ ] Los detalles secundarios están en respaldo.

### Calidad de las diapositivas

- [ ] Los títulos comunican la idea de cada diapositiva.
- [ ] El texto y las capturas pueden leerse a distancia.
- [ ] No hay exceso de texto ni tablas densas.
- [ ] Los elementos visuales apoyan la narrativa.
- [ ] El diseño es consistente.
- [ ] No aparecen credenciales ni información sensible.

## Errores que reducen la calidad de la presentación

- Leer las diapositivas en lugar de explicar decisiones.
- Copiar secciones completas del reporte.
- Mostrar un pipeline sin explicar su flujo.
- Presentar capturas ilegibles o sin contexto.
- Enumerar transformaciones sin indicar cómo se validaron.
- Mostrar solo resultados positivos y ocultar correcciones relevantes.
- Usar conteos de referencia como si fueran resultados observados.
- Dedicar demasiado tiempo a herramientas y poco a decisiones y resultados.
- Repetir la misma información entre integrantes.
- Finalizar sin conectar los resultados con el objetivo.
- Depender de una demostración en vivo sin alternativa.
- Mostrar credenciales o datos de conexión.
