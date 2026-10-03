# Etapa 0 · Estructura y ejecución

El proyecto sigue la estructura propuesta en el enunciado. Cada carpeta corresponde a una etapa del ciclo de vida del modelo. Código completo en el [repositorio de GitHub](https://github.com/juandpolo22/heart-disease-mlops).

```
heart-disease-mlops/
├── app/
│   └── api.py                        # Etapa 3: API FastAPI
├── docker/
│   ├── Dockerfile                    # Etapa 3: contenedor
│   └── requirements.txt
├── k8s/
│   ├── deployment.yaml               # Etapa 4: Kubernetes
│   └── service.yaml
├── notebooks/
│   ├── 1_model_leakage_demo.ipynb    # Etapa 1
│   ├── 2_model_pipeline_cv.ipynb     # Etapa 2
│   └── heart.csv                     # Dataset
├── .github/
│   └── workflows/
│       └── ci.yml                    # Etapa 5: integración continua
├── drift_report.html                 # Etapa 6: reporte de Evidently
├── model.joblib                      # Modelo exportado en la Etapa 2
└── README.md
```

El notebook 2 exporta el modelo a `app/model.joblib` (la ruta que lee `api.py` y que copia el Dockerfile) y a `model.joblib` en la raíz.

## Cómo ejecutarlo

Requiere Python 3.9 o superior, Docker Desktop, Minikube y kubectl. El proyecto está probado con Python 3.9 (equipo local) y 3.10 (contenedor y CI), con `scikit-learn==1.6.1`, la versión que funciona en ambos.

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    |    Linux/Mac: source .venv/bin/activate
pip install -r docker/requirements.txt
pip install jupyter matplotlib seaborn evidently==0.4.40
```

Las etapas 1, 2 y 6 se ejecutan en orden con `notebooks/1_model_leakage_demo.ipynb` y `notebooks/2_model_pipeline_cv.ipynb` (*Run All*). El segundo genera `app/model.joblib`, `model.joblib` y `drift_report.html`. Las etapas 3, 4 y 5 se describen en sus capítulos.

```{note}
Funciones reutilizables. Para mantener la estructura exacta del enunciado, las funciones de la Etapa 1 (`load_data`, `split_data`, `build_preprocessor`, `train_pipeline`, `compare_models`, `get_models`, `make_cv`) están en la sección 0 de los notebooks, en lugar de un módulo aparte.
```

## Dónde ver el código

Todo el código del proyecto se muestra dentro de este libro y también está en el [repositorio](https://github.com/juandpolo22/heart-disease-mlops).

| Código | Dónde verlo en el libro | Archivo en GitHub |
|---|---|---|
| Funciones reutilizables, demostración de fuga de datos, `Pipeline` + `GridSearchCV` y ranking de modelos | {doc}`notebooks/1_model_leakage_demo` (todas las celdas de código, con sus resultados) | [`notebooks/1_model_leakage_demo.ipynb`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/notebooks/1_model_leakage_demo.ipynb) |
| Partición, `GridSearchCV`, matriz de confusión, curva ROC, AUC y pruebas de sobreajuste | {doc}`notebooks/2_model_pipeline_cv` (todas las celdas de código, con sus resultados) | [`notebooks/2_model_pipeline_cv.ipynb`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/notebooks/2_model_pipeline_cv.ipynb) |
| Exportación del modelo, API y contenedor | {doc}`3_Despliegue_FastAPI_Docker` | [`app/api.py`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/app/api.py), [`docker/`](https://github.com/juandpolo22/heart-disease-mlops/tree/main/docker) |
| Manifiestos de Kubernetes | {doc}`4_Kubernetes` | [`k8s/`](https://github.com/juandpolo22/heart-disease-mlops/tree/main/k8s) |
| Flujo de integración continua | {doc}`5_Integracion_continua` | [`.github/workflows/ci.yml`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/.github/workflows/ci.yml) |
| Monitoreo de deriva con Evidently | {doc}`6_Monitoreo_Evidently` (y la sección 7 del notebook 2) | `drift_report.html` |
