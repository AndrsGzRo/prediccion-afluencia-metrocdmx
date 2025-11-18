# Predicción de Afluencia del Metro de la Ciudad de México (2010-2025)
Este proyecto desarrolla un Modelo de Machine Learning capaz de predecir la **afluencia diaria del Metro de la Ciudad de México** por línea, utilizando datos históricos publicados en la plataforma de [Portal de Datos Abiertos](https://datos.cdmx.gob.mx/) del Gobierno de la Ciudad de México.

Incluye análisis exploratorio, ingeniería de características, prueba de varios modelos y una comparación del desempeño final para el año de 2025.

## 1. Estructura del Repositorio

|Archivo|Descripción|
|---|---|
|```visualizations```|Carpeta con las visualizaciones exportadas (png)|
|```Afluencia_Diaria_MetroCDMX.zip```| Datos originales sin limpiar|
|```Prediccion_Metro_CDMX.ipynb```| Notebook en donde se elaboró el análisis, modelo y visualizaciones|
|```README.md```|Descripción del Proyecto|

## 2. Dataset 
- **Fuente:** Portal de Datos Abiertos de la Ciudad de México
- **Periodo analizado:** 2010-2025
- **Tamaño del dataset**: 55.6 MB

**Variables principales**
|Variable|Descripción|
|---|---|
|```fecha```|Fecha del Registro|
|```anio```,```mes```| Componentes Temporales|
|```linea```| Línea del Metro|
|```estacion```|Estación del Metro|
|```afluencia```|Total de usuarios por día|

## 3. Objetivo del Proyecto
- Predecir la **afluencia diarioa por línea del Metro de la Ciudad de México**.
- Comparar resultados reales vs predichos para 2025.
- Identificar líneas con mayor/menor precisión.
- Evaluar el desempeño global del modelo.

## 4. Limpieza y Preprocesamiento
- Convertir ```fecha``` a ```datetime```.
- Correción de nombres de líneas y estaciones.
- Cálculo de componentes temporales.
- Detección de días feriados.
- Eliminación de duplicados y registros corruptos.

## 5. Análisis Exploratorio (EDA)
Este análisis incluye:
- Tendencia de afluencia por año
- Comparación entre líneas
- Distribución por mes y día
- Estaciones o líneas más transitadas
- Heatmap de afluencia por mes
- Top 10 de Estaciones con Mayor Afluencia Total
- Top 10 de Estaciones con Mayor Afluencia Promedio

  
![Evolucion Afluencia](visualizations/EDA_Evolucion_Afluencia.png)
![Afluencia Promedio](visualizations/EDA_Afluencia_Promedio.png)
![Afluencia Promedio por Línea y Mes](visualizations/EDA_Afluencia_Promedio_Mes.png)
![Promedio por mes](visualizations/EDA_Afluencia_Promedio_porMes.png)
![Top 10 Estaciones](visualizations/EDA_Top10EstacionesProm.png)

## 6. Ingeniería de Características 

### Variables Temporales 
- Año, mes, día
- Número de día del año
- Día de la semana
- Indicador de fin de semana
- Indicador de feriado oficial

### Variables cíclicas
- Transformaciones seno/coseno para capturar periodicidad:
- ```mes_sin```,```mes_cos```
- ```dia_semana_sin```,```dia_semana__cos```

### Lag Features
- ```lag1```
- ```lag7```
- ```lag14```
- ```lag28```

### Rolling Windoes
- ```rolling7```
- ```rolling14```
- ```rolling30```
### One-hot Encoding
Para cada línea del Metro se uso One-hot Enconding.

## 7. Modelos 
Para la elaboración de los modelos, es **importante** mencionar que se utilizaron los datos de **2010 a 2024** como **entrenamiento**, mientras que los de **2025** se usaron como **prueba**, esto para evaluar el desempeño del modelo al momento de hacer predicciones. 

### Modelo Base (sin Lag Features)
Este primer modelo se elaboró sin ingeniería temporal avanzada. Para obtener predicciones se uso **XGBRegressor**. 
Las variables predictoras que se utilizaron fueron:
|**Variables Predictoras**| **Descripción**
|---|---|
| ```Anio```,```mes```,```dia```,```dia_semana```,```es_fin_de_semana```| Componentes temporales de la fecha del registro|
|```Línea 1```,```Línea 2```,...,```Línea 12```| One-Hot Encoding de las líneas del Sistema del Metro de la Ciudad de México|

La variable objetivo fue ```afluencia```.

#### Resultados
**Métricas globales** para predicciones de afluencia para 2025
- **MAE**: 34612.39
- **RMSE**: 2901454080.00
- **R²** : 0.885

![R2 por línea](visualizations/I_R2_porLinea.png)

Como se puede observar en la imagen anterior, el modelo tiene un gran falla al tratar de predecir la afluencia para la Línea 1. Por lo que, es necesario tomar en cuenta más variables. 

### Modelo Final: Modelo Final XGB

Se volvió a usar **XGBRegressor**, en este caso, se usaron las variables cíclicas, lag features y rolling windows. 

Las variables predictoras fueron: 

|**Variables Predictoras**| **Descripción**
|---|---|
| ```Anio```,```mes```,```dia```,```dia_semana```,```es_fin_de_semana```| Componentes temporales de la fecha del registro|
|```Línea 1```,```Línea 2```,...,```Línea 12```| One-Hot Encoding de las líneas del Sistema del Metro de la Ciudad de México|
| ```Anio```,```mes```,```dia```,```dia_semana```,```es_fin_de_semana```| Componentes temporales de la fecha del registro|
|```mes_sin```,```mes_cos```,```dia_semana_sin```,```dia_semana_cos```| Son variables que se repiten en ciclos, como mes, día de la semana o día del año.|
|```lag1```,```lag7```,```lag14```,```lag28```|Afluencia del día anterior, Afluencia de una semana antes, Afluencia de dos semanas antes, Afluencia de cuatro semanas antes, respectivamente|
|```rolling7```,```rolling14```,```rolling30```| Promedio de los últimos 7 días, Promedio de los últimos 14 días, Promedio de los últimos 30 días, respectivamente.|

**Hiperparámetros principales**:
- ```n_estimators```= 500
- ```max_depth```= 6
- ```learning_rate```= 0.05
- ```subsample```=0.8
- ```colsample_bytree```=0.8
- ```objective```='reg:squarederror'
- ```random_state```=42

Las **predicciones** se hicieron en escala ```log1p```. 

#### Feature Importance
![Feature Importance](visualizations/II_FI.png)
El modelo muestra que su capacidad predictiva depende **fuertemente** de variables que capturan **tendencias recientes**, **patrones temporales** y **dinámicas específicas por línea**.  A continuación se interpretan los resultados. 

**1. Rolling Windows** (factores predominantes) 
- ```rolling7``` (0.56) Es la variable más importante del modelo. Esta captura el promedio de afluencia de la última semana y refleja la tendencia inmediata.
- ```rolling14``` (0.13) Aporta información de tendencias quincenales, complementando la tendencia semanal.
- **2. Lag Features**
- ```lag1```(0.07) Influye de manera significativa, la afluencia del día anterior tiene un efecto directo.
- ```lag7``` (0.049) Refleja la estacionalidad semanal.
- ```lag14``` y ```lag28``` Capturan ciclos semanales extendidos, tienen menor contribución pero siguen siendo útiles.

**3. Efecto por Línea**: 
Algunas líneas tienen patrones particulares que el modelo aprende:
- ```linea_Línea 4``` (0.0976)
- ```linea_Línea 12``` (0.0288)

**4. Variables de calendario**: 
- ```es_feriado``` (0.0044) Hay cambios significativos de afluencia en días festivos.

Debido a las variables anteriores, el modelo es impulsado pro variables que capturan **tendencias recientes de afluencia**, como ```rolling7``` y ```rolling14```, que explican más del 70% de la importancia total. Los rezagos (```lag1```,```lag7```,```lag14```,```lag28```) confirman que se presenta un **estacionalidad semanal**. 


#### Resultados 
**Métricas globales** para predicciones de afluencia para 2025
- **MAE**: 11309.11
- **RMSE**: 375679584.00
- **R²** : 0.985

![R2 por línea](visualizations/II_R2_Linea.png "R² por Línea")

![MAE por línea](visualizations/II_MAE_Linea.png "MAE por Línea")

![Error Relativo por Línea](visualizations/II_ErrorRel_Linea.png "Error Relativo por Línea")

![Real vs Predicho ](visualizations/II_RealvsPredicho.png "Real vs Predicciones")

## 8. Predicciones de afluencia para los meses restantes de 2025

Una vez generado el modelo, se generaron predicciones para los meses restantes del año 2025: octubre, noviembre y diciembre.
Para ello, se construyó un conjunto de datos futuro que incluye:
- Fechas faltantes del año.
- Todas las líneas del conjunto original.}
- Las mismas variables predictores utilizadas durante el entrenamiento:
    - Variables temporales.
    - Variables cíclicas.
    - Lag features.
    - Rolling Windows.
    - Dummies de línea.
A partir de estas características, el modelo produjo una predicción diaria de **afluencia para cada línea**.

### Predicción de Afluencia por Línea 

La afluencia predicha en los meses restantes es la siguiente: 

![Afluencia en los meses restantes del año](visualizations/PRED_Afluencia_Linea.png)

### Predicción: Afluencia Total
Para facilitar el análisis y comunicación de resultados, se realizó un agrupamiento mensual por línea. 

```df_futuro.groupby(['linea', 'mes_nombre'])['prediccion'].sum()```

Se interpreta de la siguiente manera:
- Indica el **total de pasajeros esperados en el mes completo**, considerando todos los días.
  
![Afluencia total linea](visualizations/PRED_Afluencia_Meses.png) 

Por ejemplo:
Si la Línea 2 acumula **18,025,208 pasajeros en octubre**, esto significa que *a lo largo de noviembre 2025, se espera que la línea 2 transporte cerca de 18 millones de pasajeros*. 

### Predicción: Afluencia Promedio 

```df_futuro.groupby(['linea', 'mes_nombre'])['prediccion'].mean()```

Esta operación obtiene el **promedio de la afluencia diaria estimada** para cada línea durante cada mes. 

Interpretación: 
- Indica cuántas personas se espera que utilicen una línea en **un día típico del mes**.
  
![Afluencia promedio linea](visualizations/PRED_PromAfluencia_Meses.png) 

Por ejemplo:
Si la Línea 2 **promedia 600,840 pasajeros** en el mes de noviembre, significa: *En un día promedio de noviembre, la Línea 2 movería alrededor de 600 pasajeros*. 
## 9. Conclusiones
- El modelo logra una **alta precisión** en la mayoría de las líneas.
- Los **lag features y variables cíclicas** fueron claves para alcanzar un desempeño alto.
- Algúnas líneas presentan más variabilidad, como la Línea 1 y 2, las cuales son las que cuentan con más afluencia.
- Los feriados y fines de semana aportan información valiosa

## 10. Limitaciones
- En el modelo no se toman en cuenta los datos por estación, solo por línea.
- No se consideran fallas operativas, cierres o eventos extraordinarios. Solamente en el caso de la Línea 12 se toma en cuenta el tiempo de construcción y el cierro que se tuvo por accidente y trabajos de correcciones.
- Los datos dependen totalmente de reportes oficiales.

## 11. Trabajo Futuro
- Predicción por estación.
- Series de tiempo con LSTM/Transformers.
- Dashboard Interactivo.
- Detección de anomalís en días atípicos.

## 12. Tecnologías Utilizadas
- **Python**:
  - ```pandas```
  - ```matplotlib```
  - ```seaborn```
  - ```numpy```
  - ```sklearn```
  - ```holidays```
- **Jupyter Notebooks** para desarrollo, modelaje, predicción y visualizaciones.

## Créditos:
- **Autor**: [Andrés Guzmán Rodríguez](https://github.com/AndrsGzRo)
- **Fuente**: [Afluencia Diaria del Metro (Desglosada) - Portal de Datos Abiertos del Gobierno de la Ciudad de México](https://datos.cdmx.gob.mx/dataset/afluencia-diaria-del-metro-cdmx/resource/cce544e1-dc6b-42b4-bc27-0d8e6eb3ed72)
