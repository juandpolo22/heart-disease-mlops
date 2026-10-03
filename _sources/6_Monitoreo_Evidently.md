# Etapa 6 · Monitoreo con Evidently

El monitoreo compara la distribución de las variables de entrenamiento (datos de referencia) con la de los datos nuevos, para detectar deriva de datos (*data drift*). Se hace con Evidently {cite:p}`evidently`, usando el código del enunciado, en la sección 7 del notebook {doc}`notebooks/2_model_pipeline_cv`:

```python
import os
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset

os.chdir(ROOT)  # drift_report.html queda en la raíz del proyecto

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=X_train, current_data=X_test)
report.save_html("drift_report.html")

res = report.as_dict()["metrics"][0]["result"]
print(f"Columnas con deriva: {res['number_of_drifted_columns']} de {res['number_of_columns']}")
print("¿Deriva del dataset?:", "Sí" if res["dataset_drift"] else "No")
```

Archivo generado en GitHub: [`drift_report.html`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/drift_report.html) (para verlo en el navegador conviene descargarlo).

## Resultados

| Comparación | Columnas con deriva | Deriva del dataset |
|---|---|---|
| Entrenamiento vs. prueba (`drift_report.html`) | 1 de 11 | No |
| Entrenamiento vs. población simulada | 4 de 11 | No (el umbral por defecto es 50 % de las columnas) |

El conjunto de prueba sale de la misma población que el de entrenamiento, así que se espera poca o ninguna deriva: Evidently detecta una variable de 11 y no declara deriva global. Para comprobar que el monitoreo sí detecta cambios, el notebook simula una población distinta (pacientes 10 años mayores, más dolor torácico asintomático, menor frecuencia cardíaca máxima y colesterol 15 % más alto) y el reporte señala las variables alteradas.


Para la comparación con la población simulada, el notebook modifica una copia del conjunto de prueba y vuelve a ejecutar el reporte:

```python
rng = np.random.default_rng(0)
X_drift = X_test.copy()
X_drift["Age"] = (X_drift["Age"] + rng.normal(10, 3, len(X_drift))).round().clip(18, 100)
X_drift["Cholesterol"] = np.where(X_drift["Cholesterol"] > 0, X_drift["Cholesterol"] * 1.15, 0)
X_drift["MaxHR"] = X_drift["MaxHR"] - rng.normal(15, 5, len(X_drift))
X_drift.loc[rng.random(len(X_drift)) < 0.4, "ChestPainType"] = "ASY"

report_sim = Report(metrics=[DataDriftPreset()])
report_sim.run(reference_data=X_train, current_data=X_drift)

res = report_sim.as_dict()["metrics"][0]["result"]
print(f"Columnas con deriva: {res['number_of_drifted_columns']} de {res['number_of_columns']}")
print("¿Deriva del dataset?:", "Sí" if res["dataset_drift"] else "No",
      "(umbral por defecto: 50 % de las columnas)")
```

## Reporte

```{raw} html
<p><a href="drift_report.html" target="_blank">Abrir el reporte de Evidently en una pestaña nueva</a></p>
<iframe src="drift_report.html" width="100%" height="750" style="border:1px solid #ccc;" loading="lazy"></iframe>
```
