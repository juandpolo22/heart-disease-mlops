# Heart Disease MLOps: predicción de falla cardíaca (MLOps local)

Proyecto integrador de Aprendizaje Automático (sección 10.20). Modelo de clasificación binaria que predice si un paciente está en riesgo de falla cardíaca (`HeartDisease = 1`) o no (`HeartDisease = 0`), con un flujo local de MLOps: FastAPI, Docker, Kubernetes (Minikube), GitHub Actions y Evidently.

**Dataset:** [Heart Failure Prediction (Kaggle)](https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction), 918 pacientes y 11 predictores. El archivo `heart.csv` ya está incluido en `notebooks/`.

## Estructura

```
heart-disease-mlops/
├── app/
│   └── api.py
├── docker/
│   ├── Dockerfile
│   └── requirements.txt
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── notebooks/
│   ├── 1_model_leakage_demo.ipynb
│   ├── 2_model_pipeline_cv.ipynb
│   └── heart.csv
├── .github/
│   └── workflows/
│       └── ci.yml
├── drift_report.html
├── model.joblib
└── README.md
```

El notebook 2 exporta el modelo a `app/model.joblib` (la ruta que lee `api.py` y que copia el Dockerfile) y a `model.joblib` en la raíz.

## Resultados

Se comparan cuatro clasificadores (LogisticRegression, RandomForestClassifier, KNeighborsClassifier y GradientBoostingClassifier), cada uno en un `Pipeline` con `GridSearchCV`. Partición 80/20 estratificada (734 entrenamiento, 184 prueba) y `GridSearchCV` con `StratifiedKFold` de 5 pliegues. El modelo se elige por AUC de validación cruzada.

| # | Modelo | AUC CV | Accuracy CV |
|---|---|---|---|
| 1 | **RandomForest** | **0.9316 ± 0.0264** | 0.8529 ± 0.0431 |
| 2 | GradientBoosting | 0.9269 ± 0.0305 | 0.8570 ± 0.0285 |
| 3 | LogisticRegression | 0.9227 ± 0.0350 | 0.8515 ± 0.0311 |
| 4 | KNN | 0.9169 ± 0.0393 | 0.8529 ± 0.0293 |

Modelo final (RandomForest, `max_depth=8`, `min_samples_leaf=5`, `n_estimators=200`) en el conjunto de prueba: AUC 0.9247, accuracy 0.8533, sensibilidad 0.882 y especificidad 0.817.

Demostración de fuga de datos (SVC, como en el enunciado): con `leaky_feature` el AUC es 1.000; con el flujo correcto, 0.918.

## Ejecución

Requiere Python 3.9 o superior, Docker Desktop, Minikube y kubectl. El proyecto está probado con Python 3.9 (local) y 3.10 (contenedor y CI), con `scikit-learn==1.6.1`, la versión que funciona en ambos.

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    |    Linux/Mac: source .venv/bin/activate
pip install -r docker/requirements.txt
pip install jupyter matplotlib seaborn evidently==0.4.40
```

**Etapas 1, 2 y 6.** Ejecutar en orden `notebooks/1_model_leakage_demo.ipynb` y `notebooks/2_model_pipeline_cv.ipynb` (Run All). El segundo genera `app/model.joblib`, `model.joblib` y `drift_report.html`.

**Etapa 3: API y Docker**

```bash
docker build -t heart-api -f docker/Dockerfile .
docker run -p 8000:8000 heart-api
```

Documentación interactiva en `http://localhost:8000/docs`. La API recibe los valores en el orden `Age, Sex, ChestPainType, RestingBP, Cholesterol, FastingBS, RestingECG, MaxHR, ExerciseAngina, Oldpeak, ST_Slope`:

```bash
curl -X POST http://localhost:8000/predict -H "Content-Type: application/json" -d '{"features": [58, "M", "ASY", 140, 289, 0, "Normal", 110, "Y", 2.0, "Flat"]}'
```

```json
{"heart_disease_probability": 0.9596, "prediction": 1}
```

**Etapa 4: Kubernetes**

```bash
minikube start
minikube image load heart-api
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get svc
minikube service heart-service --url
```

**Etapa 5: CI.** `.github/workflows/ci.yml` se ejecuta en cada push: instala las dependencias, revisa el estilo con `flake8 app/` y ejecuta una prueba de la API con el modelo cargado.

**Etapa 6: Evidently.** El reporte `drift_report.html` se genera al final del notebook 2: 1 de 11 variables con deriva entre entrenamiento y prueba, sin deriva global.
