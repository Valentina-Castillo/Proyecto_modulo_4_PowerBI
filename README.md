# 🛵 Food Delivery Market Analysis

> **Power BI · Python · Data Analytics**  
> Análisis integral del mercado de *food delivery* combinando datos de encuesta y datos operativos reales para entender el comportamiento del cliente y la eficiencia de entrega.

**Autoras:** María Granero · Valentina Castillo  
![Linkedin](https://www.linkedin.com/in/mar%C3%ADa-granero-l%C3%B3pez/)
![Linkedin](https://www.linkedin.com/in/valentina-castillo-escobar-191863202/)
**Período analizado:** 2024  
**Herramientas:** Python · Power BI · Kaggle · DAX

---

## 📌 Tabla de contenidos

1. [Contexto y motivación](#contexto-y-motivación)
2. [Fuentes de datos](#fuentes-de-datos)
3. [Pipeline de datos: Python](#pipeline-de-datos-python)
4. [Transformaciones en Power BI](#transformaciones-en-power-bi)
5. [Ingeniería de features](#ingeniería-de-features)
6. [Estructura del informe](#estructura-del-informe)
7. [Hallazgos clave](#hallazgos-clave)
8. [Dificultades técnicas y soluciones](#dificultades-técnicas-y-soluciones)
9. [Diseño y experiencia de usuario](#diseño-y-experiencia-de-usuario)
10. [Estructura del repositorio](#estructura-del-repositorio)

---

## Contexto y motivación

El sector del *food delivery* ha experimentado un crecimiento exponencial en los últimos años, acelerado especialmente tras la pandemia. Sin embargo, existe una brecha significativa entre la **percepción del cliente** y la **realidad operativa** de las plataformas: los usuarios sobreestiman los tiempos de espera, infravaloran el volumen real de pedidos y mantienen tolerancias altas al retraso que las empresas no están aprovechando estratégicamente.

Este proyecto nació con el objetivo de cruzar dos perspectivas complementarias —la voz del cliente a través de encuestas y los datos transaccionales reales— para construir un análisis que vaya más allá de los KPIs superficiales y permita tomar decisiones basadas en evidencia.

---

## Fuentes de datos

Ambos datasets fueron obtenidos de **[Kaggle](https://www.kaggle.com)**, seleccionados tras una búsqueda y evaluación de su calidad, completitud y relevancia para el dominio.

| Dataset | Registros | Variables | Descripción |
|---|---|---|---|
| **Dataset 1** · Datos sobre clientes | 488 | 22 | Perfil demográfico, hábitos de consumo, preferencias y percepciones declaradas |
| **Dataset 2** · Datos sobre el servicio a domicilio | 20.000 | 35 | Transacciones reales: tiempos, distancias, precios, métodos de entrega y satisfacción |

### Criterios de selección
- Cobertura temporal completa (2024)
- Variables suficientes para cruzar dimensiones de cliente y operación
- Datos con variabilidad real (evitar datasets artificialmente perfectos)
- Licencia compatible con uso académico y portfolio

---

## Pipeline de datos: Python

Todo el preprocesamiento inicial se realizó en **Python** antes de cargar los datos en Power BI, garantizando reproducibilidad y trazabilidad de cada transformación.

### 1. Carga e inspección inicial

```python
# ============================================================================
# CONFIGURACIÓN DEL ENTORNO
# Importación de librerías para la manipulación, análisis y visualización
# de datos durante la fase exploratoria (EDA).
# ============================================================================

# Manipulación y análisis de datos
import pandas as pd
import numpy as np

# Visualización de datos
import seaborn as sns
import matplotlib.pyplot as plt


# ============================================================================
# CONFIGURACIÓN DE VISUALIZACIÓN EN PANDAS
# Ajuste de opciones para mejorar la legibilidad durante la inspección
# exploratoria, mostrando todas las columnas y un número acotado de filas.
# ============================================================================

pd.set_option("display.max_columns", None)  # mostrar todas las columnas
pd.set_option("display.max_rows", 100)      # limitar la visualización de filas
pd.set_option("display.width", None)        # ajustar automáticamente el ancho



df1 = pd.read_csv("../Data/sin procesar/Customer online delivery dataset - Customer_data.csv", index_col=0)
df2 = pd.read_csv("../Data/sin procesar/food_delivery_dataset.csv",index_col=0)


# ============================================================================
# INSPECCIÓN INICIAL
# Visualización de las primeras filas para verificar la correcta carga,
# estructura general y nombres de columnas.
# ============================================================================

display(df1.head())
display(df2.head())
```

- Revisión de tipos de datos, rangos y distribuciones
- Detección de columnas con alta cardinalidad o varianza casi nula
- Identificación de valores nulos y patrones de missing data

### 2. Limpieza y estandarización

- **Homogenización del nombrado de columnas:** En esta sección se estandarizan los nombres de las columnas para garantizar consistencia y facilitar su uso en las fases posteriores del análisis. Primero, se normalizan los nombres al formato snake_case, eliminando espacios y caracteres especiales.
Posteriormente, se renombran aquellas variables con nomenclatura ambigua para mejorar su claridad semántica.
Este proceso mejora la legibilidad del dataset y facilita su manipulación en el flujo de limpieza y análisis.
- **Limpieza de variables categóricas:** En esta sección se estandariza el formato de las variables categóricas.
Se eliminan espacios al inicio y al final de los valores y se aplica un formato homogéneo de capitalización.
Este proceso garantiza la consistencia de las categorías y evita la duplicación de valores equivalentes con distinta representación.
- **Tratamiento de los valores nulos:** En esta sección se gestionan los valores nulos identificados durante el EDA con el objetivo de garantizar la consistencia y calidad del dataset.
Se eliminan los registros que presentan valores nulos en variables categóricas o numéricas.
Este enfoque es apropiado dado el volumen de nulos a gestionar, hay en varias variables pero su % es muy pequeño lo que no afectará de ninguna forma nuestro análisis.
- **Exploración del dataset limpio:** En esta sección se exportan los datasets tras aplicar los procesos de limpieza, tratamiento de nulos y transformación de variables.
Los datasets se guardan en formato .csv dentro del directorio data/procesada/, siguiendo una estructura organizada del proyecto.
Esta exportación garantiza la reutilización de los mismos sin necesidad de repetir las fases de preparación.

### 3. Exportación

Los DataFrames limpios se exportaron como `.csv` con codificación UTF-8 para su ingesta en Power BI sin pérdida de caracteres especiales.

---

## Transformaciones en Power BI

Una vez cargados los datos, se aplicaron transformaciones adicionales en **Power Query** para ajustes específicos del modelo relacional.

- Cambio de tipos en columnas que Power Query infirió incorrectamente
- Cambio del nombre de columnas al castellano
- Eliminación de columnas que no aportaban información relevante para el análisis

---

## Ingeniería de features

### El problema de la linealidad

Uno de los mayores retos técnicos del proyecto fue la **escasa variabilidad** en algunas variables clave. Por ejemplo, el rating de satisfacción se concentraba casi exclusivamente entre 2.5 y 3.5, lo que hacía inútil cualquier visualización directa: todos los segmentos parecían idénticos.

**Solución adoptada:** en lugar de mostrar el valor absoluto, se crearon **medidas de desviación respecto a la media global**, lo que amplificó las diferencias reales entre grupos y permitió comparaciones significativas.

### Medidas DAX destacadas

```dax
--Categoría_demora =
IF(food_delivery_dataset[retraso_delivery] < 0, "Adelantado",
IF(food_delivery_dataset[retraso_delivery] <= 5, "A tiempo",
IF(food_delivery_dataset[retraso_delivery] <= 15, "Leve retraso",
"Tarde")))

--Demora_simple =
IF(food_delivery_dataset[retraso_delivery] <= 0, "A tiempo",
"Con retraso")

-- % Pct_Pedidos_Tardios =
    DIVIDE(
        CALCULATE(COUNTROWS(orders), orders[is_late] = 1),
        COUNTROWS(orders),
        0
    ) * 100

-- Dificultad_externa =
IF(food_delivery_dataset[impacto_clima] + food_delivery_dataset[puntaje_trafico] <= 3, "Fácil",
IF(food_delivery_dataset[impacto_clima] + food_delivery_dataset[puntaje_trafico] <= 4, "Moderada",
"Difícil"))

-- Etiqueta_satisfaccion =
IF(food_delivery_dataset[satisfaccion _cliente] = 1, "Muy insatisfecho",
IF(food_delivery_dataset[satisfaccion _cliente] = 2, "Insatisfecho",
IF(food_delivery_dataset[satisfaccion _cliente] = 3, "Neutral",
IF(food_delivery_dataset[satisfaccion _cliente] = 4, "Satisfecho",
"Muy satisfecho"))))

--puntaje_calidad =
(food_delivery_dataset[frescura_comida] + food_delivery_dataset[calidad_paquete]) / 2

--Rango_precio =
IF(food_delivery_dataset[valor_pedido] < 20, "Económico",
IF(food_delivery_dataset[valor_pedido] < 35, "Rango medio",
IF(food_delivery_dataset[valor_pedido] < 50, "Premium",
"Lujo")))

--Segmento Edad =
IF(food_delivery_dataset[edad] <= 25, "Joven (18-25)",
IF(food_delivery_dataset[edad] <= 35, "Adulto (26-35)",
IF(food_delivery_dataset[edad] <= 50, "Maduro (36-50)", "Senior (+50)")))

```

### Columnas calculadas nuevas

- `Dificultad_externa`
- `Etiqueta_satisfaccion`
- `Impacto_clima`
- `puntaje_calidad`
- `Rango_precio`
- `Segmento_edad`

---

## Estructura del informe

El informe Power BI está compuesto por **6 páginas** navegables mediante botones personalizados:

### Página 1 — Portada
Pantalla de bienvenida con identidad visual del proyecto. Incluye botones de navegación directa a cada sección del informe.

### Página 2 — Introducción
Contextualización del proyecto: descripción de los datasets, período analizado y objetivos analíticos. Diseño orientado a audiencias no técnicas.

### Página 3 — Perfil del Cliente
**¿Quién genera el negocio?**

| KPI | Valor |
|---|---|
| Total clientes | 488 |
| Total pedidos declarados | 68.000 |
| Ticket medio | 27,34 € |
| Preferencia dominante | No Veggie |

Visualizaciones: pedidos por ocupación (barras), distribución por género (donut), pedidos por número de familiares (barras), estado civil (barras apiladas). Filtros interactivos: género, ocupación, número de familia, estado civil.

### Página 4 — Factores de Decisión
**¿Qué hace que los clientes pidan más?**

| KPI | Valor |
|---|---|
| Rating medio entrega | 2,96 |
| Tiempo medio entrega | 38 min |
| Tiempo máximo espera | 39 min |
| % influenciados por rating | 76% |
| % sensibles a descuentos | 59% |

Visualizaciones: pedidos por rating (línea), impacto de entregas tardías (barras agrupadas), impacto de descuentos (barras horizontales), perfil de conciencia saludable (circular). Filtros: género, ocupación, tamaño familia, influencia del rating.

### Página 5 — Experiencia de Entrega
**Del pedido al cliente: calidad, precio y puntualidad**

| KPI | Valor |
|---|---|
| Total pedidos registrados | 20.000 |
| Ingresos totales | 547.000 € |
| KM distancia media | 8 |
| % pedidos tardíos | 67% |

Visualizaciones: representación a tiempo vs con retraso (donut), top 5 productos más pedidos, evolución mensual de pedidos (línea), pedidos por dificultad de entrega y condiciones (barras), pedidos por edad y rango de precio (barras horizontales apiladas). Filtros: demora, rango de precio, método de entrega, rango de edad.

### Página 6 — Visión Ejecutiva
**Clientes + operaciones: ¿Qué nos dicen los datos?**

Comparativa directa entre datos percibidos (encuesta) y datos reales (operativos):

| Métrica | Percibido | Real |
|---|---|---|
| Ingresos | 135.000 € | 547.000 € |
| Tiempo de espera | 38 min | 5 min |
| Volumen de pedidos | 68.000 | 20.000 |
| Satisfacción / Rating | 2,96 | 2,98 |

---

## Hallazgos clave

### 🕐 Brecha de percepción temporal
Los clientes perciben **38 minutos** de espera cuando el retraso real es de solo **5 minutos** — una brecha del 60% que sugiere que la comunicación proactiva sobre tiempos podría mejorar drásticamente la satisfacción sin cambiar operaciones.

### 💰 Infraestimación del volumen de negocio
Los ingresos estimados desde la encuesta (135K€) representan apenas el **25% del ingreso real operativo (547K€)**. La base de clientes que responde encuestas no es representativa del volumen total del negocio.

### ⭐ Alta tolerancia al retraso
A pesar de que el 67% de los pedidos llega tarde, tanto el rating declarado (2,96) como la satisfacción real (2,98) se mantienen estables — este mercado tiene **alta tolerancia al retraso** pero infravalora el volumen real de negocio.

### 👥 Perfil dominante del cliente
- **Género:** 56% masculino
- **Ocupación top:** estudiantes y empleados
- **Grupo familiar de mayor pedido:** familias de 3 miembros
- **Estado civil:** solteros lideran en volumen absoluto
### 🎯 Sensibilidad a descuentos
El 59% de los usuarios declara ser sensible a descuentos, y los pedidos del grupo *Agree* (22K pedidos) prácticamente duplican al grupo *Disagree* — hay margen claro para estrategias de fidelización basadas en pricing dinámico.

---

## Dificultades técnicas y soluciones

| Problema | Impacto | Solución |
|---|---|---|
| Variables con distribución casi lineal (rating, satisfacción) | Los gráficos no mostraban diferencias entre segmentos | Medidas de desviación respecto a media global en DAX |
| Desconexión entre datasets (encuesta vs operativo) | No había ID común para cruzar ambas fuentes | Cruce indirecto por variables demográficas compartidas (edad, ocupación) + análisis paralelo |

---

## Diseño y experiencia de usuario

El diseño del informe siguió principios de **data storytelling** orientados a presentaciones ejecutivas:

- **Paleta cromática consistente:** azul marino (fondo institucional), naranja y verde (KPIs positivos/negativos), morado (elementos de decisión)
- **Jerarquía visual:** KPIs en tarjetas prominentes en la parte superior; visualizaciones de detalle debajo
- **Navegación:** botones con iconos en cada página permiten saltar directamente a cualquier sección sin usar las pestañas nativas de Power BI
- **Filtros contextuales:** cada página tiene su propio panel de segmentación con las dimensiones más relevantes para esa vista; el botón *"Borrar todas las segmentaciones"* resetea todos los filtros de la página activa
- **Responsividad:** diseñadas en resolución 16:9 optimizada para presentación en pantalla y exportación a PDF
- **Iconografía:** ilustraciones de delivery personalizadas usadas como elementos visuales de identidad en todas las páginas

---


## Tecnologías utilizadas

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-2.0-blue?logo=pandas)
![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-yellow?logo=powerbi)
![DAX](https://img.shields.io/badge/DAX-Measures-orange)
![Kaggle](https://img.shields.io/badge/Data-Kaggle-20BEFF?logo=kaggle)

---

*Proyecto desarrollado como análisis de portfolio. Los datos utilizados son de acceso público en Kaggle bajo licencia abierta.*
