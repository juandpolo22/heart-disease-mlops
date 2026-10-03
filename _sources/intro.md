# Predicción de falla cardíaca con MLOps local

**Proyecto Integrador de Aprendizaje Automático (sección 10.20)**

**Autores:** Juan Polo y Gabriela Pallares

```{admonition} Repositorio del proyecto
:class: tip
Todo el código (API, Docker, Kubernetes, CI, notebooks y reporte de Evidently) está en GitHub:

**[github.com/juandpolo22/heart-disease-mlops](https://github.com/juandpolo22/heart-disease-mlops)**

También se puede abrir desde el botón de repositorio de la barra superior de este libro.
```

## Contexto del problema

Las enfermedades cardiovasculares son la principal causa de muerte en el mundo {cite:p}`who2021cvd`, y detectarlas a tiempo es fundamental para evitar complicaciones graves. Con la disponibilidad de datos clínicos estructurados es posible usar aprendizaje automático para estimar el riesgo de falla cardíaca.

El proyecto construye, con scikit-learn {cite:p}`pedregosa2011scikit`, un modelo de clasificación binaria que predice si un paciente está en riesgo de falla cardíaca (`HeartDisease = 1`) o no (`HeartDisease = 0`), y lo lleva a producción local con un flujo de MLOps {cite:p}`sculley2015debt`: FastAPI, Docker, Kubernetes (Minikube), GitHub Actions y Evidently. Todo se ejecuta en un equipo local, sin servicios en la nube.

## Datos

El dataset es *Heart Failure Prediction* {cite:p}`fedesoriano2021heart` (Kaggle). El archivo `heart.csv` está incluido en la carpeta `notebooks/` del repositorio.

| Característica | Valor |
|---|---|
| Registros (pacientes) | 918 |
| Predictores | 11 (5 numéricos, 6 categóricos o binarios) |
| Variable objetivo | `HeartDisease`: 508 con enfermedad (55.3 %) y 410 sin ella (44.7 %) |
| Valores `NaN` explícitos | 0 |
| Faltantes ocultos | 172 registros con `Cholesterol = 0` y 1 con `RestingBP = 0` |
| Partición | 80 % entrenamiento (734) y 20 % prueba (184), estratificada |

## Etapas del proyecto

| Etapa | Descripción | Capítulo |
|---|---|---|
| 0. Estructura | Organización de carpetas y cómo ejecutar el proyecto | {doc}`0_Estructura` |
| 1. Fuga de datos y comparación de modelos | Demostración de *data leakage*, `Pipeline` + `GridSearchCV` y ranking de cuatro clasificadores | {doc}`notebooks/1_model_leakage_demo` |
| 2. Modelado con validación segura | Partición antes del escalado, matriz de confusión, curva ROC, AUC y exportación del modelo | {doc}`notebooks/2_model_pipeline_cv` |
| 3. API y Docker | Servir el modelo con FastAPI y contenerizarlo | {doc}`3_Despliegue_FastAPI_Docker` |
| 4. Kubernetes local | `Deployment` y `Service` en Minikube | {doc}`4_Kubernetes` |
| 5. Integración continua | GitHub Actions: lint y prueba de la API en cada `push` | {doc}`5_Integracion_continua` |
| 6. Monitoreo | Reporte de deriva de datos con Evidently | {doc}`6_Monitoreo_Evidently` |

## Resultados

Se comparan cuatro clasificadores (`LogisticRegression`, `RandomForestClassifier`, `KNeighborsClassifier` y `GradientBoostingClassifier`), cada uno dentro de un `Pipeline` optimizado con `GridSearchCV` y `StratifiedKFold` de 5 pliegues. El modelo se elige por el AUC de validación cruzada, usando solo el conjunto de entrenamiento.

| # | Modelo | AUC CV | Accuracy CV |
|---|---|---|---|
| 1 | RandomForest | 0.9316 ± 0.0264 | 0.8529 ± 0.0431 |
| 2 | GradientBoosting | 0.9269 ± 0.0305 | 0.8570 ± 0.0285 |
| 3 | LogisticRegression | 0.9227 ± 0.0350 | 0.8515 ± 0.0311 |
| 4 | KNN | 0.9169 ± 0.0393 | 0.8529 ± 0.0293 |

El modelo final es un Random Forest {cite:p}`breiman2001rf` (`max_depth=8`, `min_samples_leaf=5`, `n_estimators=200`). En el conjunto de prueba obtiene:

| Métrica | Valor |
|---|---|
| AUC | 0.9247 |
| Accuracy | 0.8533 |
| Sensibilidad | 0.882 |
| Especificidad | 0.817 |

En la demostración de fuga de datos (con un SVC, como pide el enunciado), el AUC es 1.000 cuando se filtra la variable `leaky_feature` y 0.918 con el flujo correcto.
