# 🏦 Predicción de Fuga de Clientes (Customer Churn) en Productos Financieros

Nuestro Trabajo Final de Máster — Máster Universitario en Análisis y Visualización de Datos Masivos (UNIR)

Desarrollamos un modelo de machine learning y un dashboard interactivo para anticipar la fuga de clientes bancarios y priorizar los esfuerzos de retención con criterios basados en datos.

**Autoras:** Diana Andrea Carballo Sarabia y Stephany Rosario Pineda Sauceda

---

## 📑 Tabla de contenidos

- [Contexto del problema](#-contexto-del-problema)
- [Dataset](#-dataset)
- [Metodología](#-metodología)
- [Arquitectura del prototipo](#-arquitectura-del-prototipo)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Instalación y ejecución](#-instalación-y-ejecución)
- [Resultados](#-resultados)
- [Dashboard interactivo](#-dashboard-interactivo)
- [Hallazgos clave](#-hallazgos-clave)
- [Limitaciones y consideraciones éticas](#-limitaciones-y-consideraciones-éticas)
- [Próximos pasos](#-próximos-pasos)
- [Documentación completa](#-documentación-completa)
- [Autoras](#-autoras)

---

## 🎯 Contexto del problema

La fuga de clientes (*churn*) tiene un impacto directo y medible en la rentabilidad de una entidad financiera: retener a un cliente existente es sistemáticamente más barato que adquirir uno nuevo. Partimos de un coste de adquisición de cliente (CAC) de referencia de **100 USD** y de un escenario de negocio en el que reducir la tasa de churn del **20 % al 15 %** representaría un ahorro estimado de **50.000 USD**.

Nos planteamos un objetivo doble:

1. **Predictivo:** construir un modelo capaz de identificar, con antelación, qué clientes tienen mayor probabilidad de abandonar la entidad.
2. **Prescriptivo:** traducir esas predicciones en una herramienta operativa (dashboard) que el área de negocio pueda usar para priorizar campañas de retención.

## 📊 Dataset

| | |
|---|---|
| **Fuente** | [Bank Customer Churn Prediction](https://www.kaggle.com/datasets/shantanudhakadd/bank-customer-churn-prediction) (Kaggle) |
| **Registros** | 10.000 |
| **Variables** | 14 |
| **Variable objetivo** | `Exited` (1 = el cliente abandonó el banco) |
| **Balance de clases** | ≈ 80 % / 20 % (desbalanceado hacia clientes que permanecen) |

Variables principales: `CreditScore`, `Geography`, `Gender`, `Age`, `Tenure`, `Balance`, `NumOfProducts`, `HasCrCard`, `IsActiveMember`, `EstimatedSalary`.

## 🧪 Metodología

Seguimos el marco **CRISP-DM**. Así resumimos el tratamiento de datos y el modelado que aplicamos (el detalle completo está en el informe, Sección 3):

1. **Partición estratificada** 80 % entrenamiento / 20 % prueba.
2. **Tratamiento de outliers:** winsorización de la variable `Age`.
3. **Feature engineering:** creamos `Balance_Salary_Ratio`, `Products_Balance_Interaction`, `Has_Balance`, y agrupamos por rangos etarios (`Age_Group`).
4. **Codificación:** one-hot encoding de `Geography` y `Gender`.
5. **Normalización:** estandarización Z-score de las variables numéricas.
6. **Balanceo de clases:** aplicamos SMOTE **únicamente** sobre el conjunto de entrenamiento, para no contaminar la evaluación.
7. **Validación:** validación cruzada de 5 pliegues (k=5).
8. **Modelos comparados:** Regresión Logística (interpretable) vs. Random Forest (mejor desempeño).

## 🏗️ Arquitectura del prototipo

```
 Churn_Modelling.csv
        │
        ▼
 pipeline_modelado.py  ──────►  churn_pipeline.sqlite
 (Python: pandas,               (train_preprocessed,
  scikit-learn,                  test_predictions)
  imbalanced-learn)                     │
        │                               ▼
        │                    Dashboard_Churn_TFM_Seminario.pbix
        │                    (Power BI — 3 vistas interactivas)
        ▼
 Flujo equivalente en KNIME Analytics Platform
 (ruta alternativa sin código, ver docs/)
```

Elegimos Python como pipeline principal. El flujo de KNIME (documentado en `docs/`) es la ruta de referencia equivalente que dejamos documentada, útil para auditar o reproducir el proceso sin necesidad de un entorno de programación.

## 📁 Estructura del repositorio

```
.
├── README.md
├── requirements.txt
├── data/                      # Churn_Modelling.csv (o instrucciones de descarga)
├── src/
│   └── pipeline_modelado.py   # Pipeline completo: limpieza → feature engineering → SMOTE → modelado → evaluación
├── db/
│   └── churn_pipeline.sqlite  # Base de datos generada por el pipeline
├── dashboard/
│   └── Dashboard_Churn_TFM_Seminario.pbix   # Dashboard en Power BI
└── docs/
    └── knime_flow.pdf         # Flujo equivalente en KNIME Analytics Platform
```

## ⚙️ Instalación y ejecución

### Requisitos
- Google Colab (nuestro entorno de desarrollo) o Python 3.9+ en local
- Power BI Desktop, para abrir el dashboard
- Opcional: KNIME Analytics Platform, para la ruta alternativa sin Python

### 1. Clonar el repositorio e instalar dependencias

**En Google Colab:**
```python
!git clone https://github.com/stephanyIS/tfm-prediccion-fuga-ml-bancario.git
%cd tfm-prediccion-fuga-ml-bancario
!pip install -r requirements.txt
```

**En local:**
```bash
git clone https://github.com/stephanyIS/tfm-prediccion-fuga-ml-bancario.git
cd tfm-prediccion-fuga-ml-bancario
pip install -r requirements.txt
```

### 2. Ejecutar el pipeline

Coloca `Churn_Modelling.csv` en `data/` y ejecuta:

```python
!python src/pipeline_modelado.py          # en Colab
python src/pipeline_modelado.py           # en local
```

Esto reproduce de forma determinista (`random_state = 42`) todo el proceso que diseñamos —limpieza, feature engineering, SMOTE, validación cruzada, entrenamiento y evaluación— y regenera `db/churn_pipeline.sqlite`.

### 3. Explorar el dashboard

Abre `dashboard/Dashboard_Churn_TFM_Seminario.pbix` en Power BI Desktop apuntando a las tablas regeneradas en el paso anterior.

### 4. Ruta alternativa sin Python

El flujo equivalente en KNIME está documentado en `docs/`, para quien prefiera ejecutar o auditar el proceso de forma visual.

## 📈 Resultados

| Métrica (holdout, 2.000 registros) | Regresión Logística | Random Forest |
|---|---|---|
| AUC-ROC | 0,802 | **0,859** |
| Accuracy | 0,741 | **0,824** |
| Precision | 0,420 | **0,553** |
| Recall | **0,720** | 0,703 |
| F1-score | 0,530 | **0,619** |

Seleccionamos **Random Forest** como modelo principal por su mejor desempeño global (AUC-ROC, Accuracy, Precision y F1). Conservamos la **Regresión Logística** como modelo secundario interpretable, relevante en un contexto bancario donde la explicabilidad frente a auditorías regulatorias importa tanto como el desempeño puro.

El umbral de decisión es ajustable (rango 0,10–0,90 en el dashboard); en el rango que recomendamos, de **0,3 a 0,4**, el Recall de Random Forest oscila entre **70,3 % y 92,9 %**, según el compromiso Recall–Precisión que el área de negocio decida priorizar.

### Variables más predictivas
Encontramos que `NumOfProducts`, `Age` e `IsActiveMember` son, de forma consistente en ambos modelos, los predictores más fuertes de churn.

## 📊 Dashboard interactivo

Construimos el dashboard en Power BI (`Dashboard_Churn_TFM_Seminario.pbix`) con medidas DAX dinámicas ligadas a un parámetro de umbral, de modo que recalcula sus indicadores en vivo sin necesidad de reentrenar el modelo. Tiene tres vistas:

| Vista | Contenido |
|---|---|
| **Resumen Ejecutivo** | 4 tarjetas KPI (`Ahorro_Estimado_USD`, `Tasa_Churn_Global`, `Recall_Dinamico`, `Clientes_En_Riesgo`) + 2 controles deslizantes (umbral de decisión y edad) |
| **Análisis de Riesgo** | Dispersión Age–Balance coloreada por predicción, y barras de probabilidad media de churn por número de productos |
| **Priorización Operativa** | Tabla de los 100 clientes con mayor riesgo, ordenada por probabilidad de churn |

## 🔍 Hallazgos clave

Esto es lo que encontramos al analizar los resultados:

- La relación entre `NumOfProducts` y el churn **no es lineal**: los clientes con 3-4 productos abandonan más que los que tienen 2 —un patrón en forma de "U" que Random Forest captura mejor que la Regresión Logística.
- Los segmentos de mayor riesgo combinan: edad entre 45 y 65 años, 3-4 productos contratados, inactividad, y residencia en Alemania.
- El umbral de decisión es una palanca de negocio, no solo un parámetro técnico: moverlo entre 0,3 y 0,4 permite a negocio elegir explícitamente entre capturar más clientes en riesgo (mayor Recall) o reducir falsos positivos (mayor Precision).

## ⚠️ Limitaciones y consideraciones éticas

Reconocemos las siguientes limitaciones en nuestro trabajo:

- El dataset es una fotografía estática y anonimizada; nuestro modelo capta **asociación, no causalidad**, y no incorpora datos longitudinales del comportamiento del cliente.
- `Geography` y `Gender` mejoran el desempeño predictivo, pero su uso comercial directo requiere una **auditoría de equidad (fairness)** previa, para evitar un impacto dispar sobre grupos protegidos.
- Los resultados deben validarse frente a datos productivos reales antes de cualquier implementación operativa; el rendimiento que obtuvimos sobre el dataset de Kaggle no garantiza el mismo desempeño en producción.

## 🚀 Próximos pasos

Estos son los próximos pasos que nos planteamos como equipo:

- Incorporar segmentadores de `Tenure` y `Geography` en el dashboard (los campos ya existen en nuestro modelo de datos).
- Evaluar algoritmos de *gradient boosting* (XGBoost, LightGBM) como extensión de la comparación actual.
- Incorporar SHAP para explicabilidad local de las predicciones.
- Explorar modelado de supervivencia con datos longitudinales.
- Validar el impacto real del modelo mediante un piloto controlado de intervenciones de retención.

## 📚 Documentación completa

En este repositorio dejamos el **prototipo funcional** del proyecto (código, modelo de datos y dashboard). El informe completo que elaboramos —con la justificación de negocio, el análisis exploratorio de datos, la metodología detallada, la evaluación de resultados, las conclusiones, las recomendaciones y el plan de implementación— lo entregamos como documento independiente (Secciones 1 a 9 del TFM).

## 👥 Autoras

- **Diana Andrea Carballo Sarabia**
- **Stephany Rosario Pineda Sauceda**

Máster Universitario en Análisis y Visualización de Datos Masivos — Universidad Internacional de La Rioja (UNIR), 2026.

---

*Este es nuestro Trabajo Final de Máster; no está abierto a contribuciones externas.*
