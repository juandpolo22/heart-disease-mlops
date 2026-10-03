# Conclusiones

1. **Datos.** El dataset no tiene valores `NaN`, pero sí 172 colesteroles y una presión en reposo registrados como 0, que son faltantes ocultos. Se imputan dentro del pipeline con la mediana del entrenamiento, de modo que el mismo tratamiento se aplica a los datos que llegan a la API.

2. **Fuga de datos.** Con una variable que contiene la respuesta, el AUC llega a 1.000 incluso usando `Pipeline`: el pipeline evita la fuga de preprocesamiento, pero no la fuga de objetivo {cite:p}`kaufman2012leakage`. Sin esa variable y con todo el preprocesamiento dentro del pipeline, el AUC honesto del SVC es de 0.918.

3. **Modelos.** Los cuatro clasificadores comparados (regresión logística, KNN, Random Forest y Gradient Boosting {cite:p}`friedman2001gbm`) alcanzan AUC de validación cruzada entre 0.917 y 0.932. Las diferencias entre ellos son menores que la variación entre pliegues {cite:p}`kohavi1995cv`, así que su desempeño es prácticamente equivalente. Se eligió el Random Forest por tener el mayor AUC de validación.

4. **Desempeño final.** En el conjunto de prueba el modelo obtiene un AUC de 0.925 {cite:p}`fawcett2006roc` y una accuracy de 0.853, con sensibilidad de 0.882 y especificidad de 0.817. En un tamizaje clínico los falsos negativos son el error más costoso, por lo que el umbral de decisión podría ajustarse para aumentar la sensibilidad.

5. **Sobreajuste.** La prueba de permutación da un AUC de 0.507 con la respuesta permutada frente a 0.929 con la original, lo que confirma que el modelo aprende una relación real y no ruido.

6. **MLOps.** El mismo pipeline se sirve con FastAPI, se empaqueta en Docker con versiones fijas, se despliega en Kubernetes local, se valida en cada `push` con GitHub Actions y se monitorea con Evidently. La experiencia mostró que la versión de scikit-learn con la que se entrena el modelo debe coincidir con la del contenedor: una diferencia de versión produjo un error 500 en la API.

El código completo está en [github.com/juandpolo22/heart-disease-mlops](https://github.com/juandpolo22/heart-disease-mlops).
