# Documentación de modelado y visualización de datos - Ask A Manager Salary Survey 2021  
**Visualización Y Storytelling - Semana 3**
**Bryan Rodriguez** 

## 1. Introducción

Este documento describe el proceso de limpieza, modelado y transformación de la base de datos **Ask A Manager Salary Survey 2021**, con el objetivo de habilitar su análisis y visualización en Looker Studio.

El trabajo está pensado como un insumo interno para un equipo de análisis, priorizando la claridad metodológica, la reproducibilidad y la utilidad analítica por encima de una narrativa para público general. Este ejercicio funciona como simulacro de dinámicas reales de trabajo colaborativo en proyectos de analítica.

---

## 2. Fuente de datos

La base de datos proviene de la encuesta pública **Ask A Manager Salary Survey 2021**, recolectada mediante un formulario en línea y alojada en un Google Sheets público que se actualiza constantemente.

- Formulario:  
  https://www.askamanager.org/2021/04/how-much-money-do-you-make-4.html

- Base de datos:  
  https://docs.google.com/spreadsheets/d/1IPS5dBSGtwYVbjsfbaMCYIWnOuRmJcbequohNxCyGVw

---

## 3. Variables de la base de datos original

Los nombres de las variables se presentan en inglés, tal como aparecen en la base de datos original. Las descripciones están escritas en español para facilitar su comprensión.

| Variable | Tipo | Descripción |
|--------|------|-------------|
| `Country` | Texto | País en el que trabaja la persona encuestada. Campo de texto libre con múltiples variaciones de escritura. |
| `City` | Texto | Ciudad en la que trabaja la persona. Campo de texto libre que puede incluir estados, comentarios u otros valores no geográficos. |
| `Annual Salary` | Número (texto en origen) | Salario anual reportado por la persona encuestada. Puede incluir formatos no estandarizados. |
| `Additional Monetary Compensation` | Número (texto en origen) | Compensaciones monetarias adicionales anuales, como bonos u horas extra. |
| `Currency` | Texto | Moneda en la que está expresado el salario anual. |
| `Industry` | Texto | Industria en la que trabaja la persona encuestada. |
| `How old are you?` | Texto (categórica) | Rango de edad de la persona encuestada. |
| `How many years of professional work experience do you have overall?` | Texto (categórica) | Rango de años de experiencia profesional. |

---

## 4. Modelado y limpieza de datos

### 4.1 Limpieza de ubicación (Country y City)

Los campos `Country` y `City` son de texto libre, lo que genera inconsistencias que dificultan el análisis y la visualización geográfica. Para solucionarlo se realizó lo siguiente:

- Normalización de nombres de países (por ejemplo, "US", "USA" y "United States" se unifican como "United States").
- Eliminación de información adicional en el campo `City` (por ejemplo, estados o comentarios).
- Creación de campos de ubicación limpiados para facilitar el uso en mapas.

---

### 4.2 Conversión de salarios y compensaciones a pesos colombianos

Para permitir la comparación global de salarios, se decidió utilizar el **peso colombiano (COP)** como moneda base.

Se crearon campos calculados en Looker Studio utilizando la función condicional `CASE`, multiplicando los valores de salario y compensaciones por la **tasa de cambio (TRM)** vigente al día de realización del ejercicio, según la moneda reportada en el campo `Currency`.

Este procedimiento sigue el enfoque trabajado en los contenidos de la semana y permite homogeneizar los valores monetarios para el análisis.

---

### 4.3 Variables derivadas

A partir de las variables originales se crearon los siguientes campos:

- Salario anual en pesos colombianos.
- Compensaciones adicionales en pesos colombianos.
- Compensación total en pesos colombianos, como la suma del salario anual y las compensaciones.

---

### 4.4 Control de calidad de datos

Adicionalmente, se implementaron validaciones lógicas para identificar posibles inconsistencias en los datos, tales como:

- Personas menores de 18 años con múltiples años de experiencia profesional.
- Casos en los que la experiencia reportada supera la edad posible.

Estos registros fueron marcados mediante un campo de control de calidad y excluidos de los análisis agregados, sin eliminar los datos originales.

---

## 5. Variables luego del modelado

| Variable | Tipo | Descripción |
|--------|------|-------------|
| `salario_anual_cop` | Número | Salario anual convertido a pesos colombianos según la TRM utilizada. |
| `compensaciones_cop` | Número | Compensaciones monetarias adicionales anuales convertidas a COP. |
| `total_compensacion_cop` | Número | Suma del salario anual y las compensaciones en pesos colombianos. |
| `country_clean` | Texto | País normalizado para análisis y visualización. |
| `city_clean` | Texto | Ciudad normalizada para análisis y visualización. |
| `data_quality_issue` | Número (0/1) | Indicador de posibles inconsistencias lógicas en los datos. |

---

## 6. Dashboard

El dashboard desarrollado en Looker Studio contiene los siguientes módulos:

1. **Contador de respuestas:** total de registros válidos utilizados en el análisis.
2. **Mapa geográfico:** distribución de las respuestas por país (y/o ciudad).
3. **Industria vs salario promedio:** comparación del salario promedio por industria utilizando valores en pesos colombianos.
4. **Módulo libre:** visualización exploratoria adicional diseñada para revelar patrones implícitos en los datos.

---

## 7. Paso a paso para actualizar los datos

1. Acceder al Google Sheets original de Ask A Manager.
2. Conectar la hoja actualizada como fuente de datos en Looker Studio.
3. Verificar que la estructura de columnas no haya cambiado.
4. Actualizar manualmente la TRM utilizada en los campos calculados.
5. Revisar las condiciones `CASE` asociadas al campo `Currency`.
6. Validar que los campos derivados se calculen correctamente.
7. Refrescar el dashboard y revisar las visualizaciones.

---

## 8. Créditos

- Fuente de datos: Ask A Manager – Salary Survey 2021  
- Tasa de cambio (TRM) utilizada: **[indicar fecha exacta de la TRM]**

