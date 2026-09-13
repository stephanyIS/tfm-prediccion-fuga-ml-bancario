# Predicción de Fuga de Clientes (Customer Churn) en Productos Financieros

Trabajo Final de Máster — Máster Universitario en Análisis y Visualización de Datos Masivos (UNIR)

**Autoras:** Diana Andrea Carballo Sarabia y Stephany Rosario Pineda Sauceda

## Objetivo del proyecto

Este proyecto desarrolla un modelo de predicción de fuga de clientes (*customer churn*) para productos financieros, con el fin de identificar de forma anticipada a los clientes con mayor probabilidad de abandono y priorizar los esfuerzos de retención del área de negocio. El prototipo integra un pipeline de modelado en Python, un modelo de datos persistido en SQLite y un dashboard interactivo en Power BI que traduce las predicciones en indicadores de negocio accionables.

## Dataset

- **Fuente:** [Bank Customer Churn Prediction](https://www.kaggle.com/datasets/shantanudhakadd/bank-customer-churn-prediction) (Kaggle)
- **Tamaño:** 10.000 registros, 14 variables
- **Variable objetivo:** `Exited` (desbalance aproximado 80/20)

## Estructura del repositorio

```
.
├── README.md
├── requirements.txt
├── data/                      # Churn_Modelling.csv (o instrucciones de descarga)
├── src/
│   └── pipeline_modelado.py   # Pipeline completo de preprocesamiento y modelado
├── db/
│   └── churn_pipeline.sqlite  # Base de datos generada por el pipeline
├── dashboard/
│   └── Dashboard_Churn_TFM_Seminario.pbix   # Dashboard en Power BI
└── docs/
    └── knime_flow.pdf         # Documentación del flujo equivalente en KNIME Analytics Platform
```

## Cómo ejecutar el prototipo

### 1. Requisitos previos
- Google Colab (entorno usado para desarrollar y ejecutar el pipeline), o Python 3.9+ en local
- Power BI Desktop, para abrir el dashboard
- Opcional: KNIME Analytics Platform, para la ruta alternativa sin Python

### 2. Instalación

**En Google Colab** (entorno recomendado — pandas, numpy y scikit-learn ya vienen preinstalados):

```python
!git clone https://github.com/stephanyIS/tfm-prediccion-fuga-ml-bancario.git
%cd tfm-prediccion-fuga-ml-bancario
!pip install -r requirements.txt
```

El `pip install` de arriba solo instala `imbalanced-learn` (la única librería que Colab no trae por defecto); no reinstala pandas, numpy ni scikit-learn.

**En una máquina local**, los mismos pasos sin el símbolo `!`:

```bash
git clone https://github.com/stephanyIS/tfm-prediccion-fuga-ml-bancario.git
cd tfm-prediccion-fuga-ml-bancario
pip install -r requirements.txt
```

### 3. Ejecutar el pipeline de modelado

Sube o monta `Churn_Modelling.csv` en la carpeta `data/` y ejecuta, en una celda de Colab:

```python
!python src/pipeline_modelado.py
```

(en local, sin el `!`: `python src/pipeline_modelado.py`)

Esto reproduce de forma determinista (`random_state = 42`) las once etapas del pipeline —limpieza, *feature engineering*, partición estratificada 80/20, SMOTE confinado al entrenamiento, validación cruzada k=5, entrenamiento y evaluación de ambos modelos— y regenera `db/churn_pipeline.sqlite` con las tablas `train_preprocessed` y `test_predictions`.

### 4. Explorar el dashboard

Abre `dashboard/Dashboard_Churn_TFM_Seminario.pbix` en Power BI Desktop, apuntando a las tablas regeneradas en el paso anterior. El dashboard tiene tres vistas: **Resumen Ejecutivo**, **Análisis de Riesgo** y **Priorización Operativa**.

### 5. Ruta alternativa sin Python

El flujo equivalente en KNIME Analytics Platform está documentado en `docs/`, para quien prefiera ejecutar o auditar el pipeline de forma visual, sin necesidad de un entorno de programación.

## Resumen de resultados

| Métrica (holdout) | Regresión Logística | Random Forest |
|---|---|---|
| AUC-ROC | 0,802 | **0,859** |
| Accuracy | 0,741 | **0,824** |
| Precision | 0,420 | **0,553** |
| Recall | **0,720** | 0,703 |
| F1 | 0,530 | **0,619** |

**Random Forest** fue seleccionado como modelo principal por su mejor desempeño global, mientras que la **Regresión Logística** se mantiene como modelo secundario interpretable para fines de auditoría regulatoria. El umbral de decisión es ajustable en el dashboard (rango 0,10–0,90); en el rango recomendado de 0,3 a 0,4, el Recall de Random Forest oscila entre 70,3 % y 92,9 %, según el compromiso Recall–Precisión que el área de negocio decida priorizar.

## Documentación completa

Este repositorio contiene el prototipo funcional del proyecto. El informe completo —con la metodología detallada, el análisis exploratorio, la evaluación de resultados, las conclusiones y las recomendaciones— se entrega como documento independiente.

## Autoras

- Diana Andrea Carballo Sarabia
- Stephany Rosario Pineda Sauceda

Máster Universitario en Análisis y Visualización de Datos Masivos — Universidad Internacional de La Rioja (UNIR)
