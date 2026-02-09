# Documentación de modelado y visualización de datos - Ask A Manager Salary Survey 2021  
**Visualización Y Storytelling - Semana 3**
**Bryan Rodriguez** 
## [Link al Dashboard](https://lookerstudio.google.com/reporting/6ddc76ac-10b2-42f0-b696-01d9041f92e0)
---

## 1. Introducción

Este documento describe el proceso de limpieza, modelado y transformación de la base de datos **Ask A Manager Salary Survey 2021**, con el objetivo de habilitar su análisis y visualización en Looker Studio.

El trabajo está concebido como un insumo interno para un equipo de analítica, priorizando la claridad metodológica, la reproducibilidad del proceso y la utilidad para la exploración de los datos. No está orientado a público general, sino a un contexto de trabajo colaborativo donde es clave entender las decisiones tomadas sobre la base de datos.

---

## 2. Fuente de datos

La base de datos proviene de la encuesta pública **Ask A Manager Salary Survey 2021**, recolectada mediante un formulario en línea y alojada en un Google Sheets público que se actualiza constantemente.

- Formulario original:  
  https://www.askamanager.org/2021/04/how-much-money-do-you-make-4.html

- Base de datos original:  
  https://docs.google.com/spreadsheets/d/1IPS5dBSGtwYVbjsfbaMCYIWnOuRmJcbequohNxCyGVw

---

## 3. Variables de la base de datos original

En esta sección se describen algunas de las variables relevantes de la base original. Los nombres de las variables se presentan **tal como aparecen en la fuente original (en inglés)**, mientras que las descripciones se presentan en español para facilitar su comprensión.

| Variable original | Tipo | Descripción |
|------------------|------|-------------|
| `What country do you work in?` | Texto | País en el que trabaja la persona encuestada. Campo de texto libre con múltiples variaciones de escritura. |
| `What city do you work in?` | Texto | Ciudad en la que trabaja la persona encuestada. Campo de texto libre que puede incluir información adicional. |
| `What is your annual salary?` | Número (texto en origen) | Salario anual reportado por la persona encuestada, sin estandarización de formato. |
| `Additional Monetary Compensation` | Número (texto en origen) | Compensaciones monetarias adicionales anuales, como bonos u horas extra. |
| `Please indicate the currency` | Texto | Moneda en la que está expresado el salario anual. |
| `Industry` | Texto | Industria en la que trabaja la persona encuestada. |
| `How old are you?` | Texto (categórica) | Rango de edad de la persona encuestada. |
| `How many years of professional work experience do you have overall?` | Texto (categórica) | Rango de años de experiencia profesional. |

---

## 4. Limpieza inicial de la base de datos

Antes de realizar el modelado en Looker Studio, se llevó a cabo una **limpieza manual inicial** sobre una copia de la base de datos original. Esta limpieza tuvo como objetivo corregir errores evidentes que impedían el análisis y la conversión monetaria.

Las principales acciones realizadas fueron:

- Eliminación de un número reducido de registros con valores completamente ininteligibles (por ejemplo, textos sin sentido en campos monetarios).
- Corrección de salarios anuales menores a 500 que, por contexto, correspondían a valores expresados en miles (por ejemplo, 42 interpretado como 42.000).  
  Estos valores fueron multiplicados por 1.000 y almacenados en un nuevo campo.
- Reasignación de monedas originalmente marcadas como `Other` a su código correcto cuando la información lo permitía.
- Normalización manual de nombres de países (por ejemplo, distintas variantes de “United States” o “United Kingdom”).
- Creación de campos de país y ciudad con nombres normalizados y consistentes.

Esta limpieza manual se documenta explícitamente como parte del proceso y no se presenta como una transformación automática.

---

## 5. Variables luego del modelado

A partir de la base limpia, se trabajó exclusivamente en Looker Studio para crear los siguientes campos derivados. Todos los nombres de variables se presentan en **español**, de forma consistente.

### Variables geográficas

| Variable | Tipo | Descripción |
|--------|------|-------------|
| `Pais` | Texto | País normalizado en el que trabaja la persona encuestada. |
| `Ciudad` | Texto | Ciudad normalizada en la que trabaja la persona encuestada. |

---

### Variables monetarias base

| Variable | Tipo | Descripción |
|--------|------|-------------|
| `Salario Anual Corregido` | Número | Salario anual corregido a partir de la limpieza manual inicial. |
| `Compensaciones Raw` | Número | Compensaciones monetarias adicionales anuales sin conversión de moneda. |
| `Moneda` | Texto | Código de moneda asociado al salario y compensaciones. |

---

### Conversión a pesos colombianos

Para permitir la comparación global de salarios, se decidió utilizar el **peso colombiano (COP)** como moneda base. La conversión se realizó mediante campos calculados en Looker Studio utilizando la tasa de cambio (TRM) del día de realización del ejercicio.

| Variable | Tipo | Descripción |
|--------|------|-------------|
| `Salario Anual COP` | Número | Salario anual convertido a pesos colombianos según la TRM utilizada. |
| `Compensaciones COP` | Número | Compensaciones monetarias adicionales convertidas a pesos colombianos. |
| `Salario Total COP` | Número | Suma del salario anual y las compensaciones en pesos colombianos. |

---

### Variables de control de calidad

Con el fin de evitar que valores inconsistentes afecten los análisis monetarios, se crearon indicadores de calidad de datos.

| Variable | Tipo | Descripción |
|--------|------|-------------|
| `flag_currency_no_convertible` | Número (0/1) | Indica si la moneda no puede convertirse a COP. |
| `flag_salario_cero_o_nulo` | Número (0/1) | Identifica salarios iguales a cero o nulos. |
| `flag_outlier_monetario` | Número (0/1) | Marca valores monetarios extremadamente altos. |
| `flag_excluir_monetario` | Número (0/1) | Indica registros que deben excluirse de análisis monetarios agregados. |

Estos indicadores permiten **filtrar los datos sin eliminar registros**, preservando la trazabilidad de la fuente original.

---

## 6. Dashboard

El dashboard desarrollado en Looker Studio presenta una visión general de los salarios reportados en la encuesta, utilizando los datos previamente limpiados y modelados. Está compuesto por los siguientes módulos:

### Módulo 1: Contador de respuestas
Un contador básico que muestra el **total de encuestas válidas**, permitiendo dimensionar el tamaño de la muestra analizada.

### Módulo 2: Distribución geográfica
Un **mapa por país** que muestra la distribución de las respuestas a nivel global.  
Este módulo permite identificar la concentración geográfica de los registros y contextualizar el alcance internacional de la encuesta.

### Módulo 3: Salarios promedio por industria
Una visualización comparativa que muestra el **salario total promedio** y las **compensaciones promedio** por industria, expresadas en pesos colombianos (COP).  
Este módulo permite identificar diferencias salariales relevantes entre sectores y analizar el peso relativo de las compensaciones adicionales.

Para este módulo se filtran los registros utilizando el indicador:
- `flag_excluir_monetario = 0`

### Módulo 4: Visualizaciones exploratorias
Como módulo libre, se incluyeron visualizaciones adicionales para profundizar en patrones salariales específicos:

- **Salarios promedio por género**, expresados en COP.
- **Salarios promedio por nivel educativo**, expresados en COP.

Estas visualizaciones permiten explorar brechas salariales y tendencias asociadas a variables demográficas y de formación académica.

En todas las visualizaciones que utilizan información monetaria se aplican los filtros de calidad definidos durante el modelado para evitar la distorsión de los resultados por valores atípicos o monedas no convertibles.

---

## 7. Paso a paso para actualizar los datos

1. Acceder a la base de datos original de Ask A Manager.
2. Realizar una limpieza manual inicial para corregir errores evidentes, manteniendo la estructura de columnas.
3. Conectar la base limpia a Looker Studio.
4. Verificar que los nombres de las variables coincidan con los utilizados en el modelado.
5. Actualizar la tasa de cambio (TRM) utilizada para la conversión a pesos colombianos.
6. Revisar los campos calculados de conversión y control de calidad.
7. Refrescar el dashboard y validar las visualizaciones.

Este procedimiento asume que la estructura general de la base de datos original se mantiene en futuras actualizaciones.

---

## 8. Créditos

- Fuente de datos: Ask A Manager – Salary Survey 2021  
- Tasa de cambio utilizada: TRM a pesos colombianos correspondiente al día de realización del ejercicio

- Tasa de cambio (TRM) utilizada: **2026-02-08**

