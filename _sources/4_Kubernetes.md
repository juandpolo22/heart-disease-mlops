# Etapa 4 · Despliegue en Kubernetes local

Kubernetes {cite:p}`burns2016borg` administra el ciclo de vida del contenedor: lo reinicia si falla, mantiene el número de réplicas pedido y lo expone con un servicio de red. El despliegue se hace en un clúster local con Minikube.

Archivos de esta etapa en GitHub: [`k8s/deployment.yaml`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/k8s/deployment.yaml) y [`k8s/service.yaml`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/k8s/service.yaml).

## Manifiestos

```{literalinclude} k8s/deployment.yaml
:language: yaml
:caption: k8s/deployment.yaml
```

El manifiesto sigue el del enunciado con un cambio: en lugar de `<TU_USUARIO_DOCKER>/heart-api` (una imagen de Docker Hub) usa la imagen local `heart-api`, con `imagePullPolicy: Never` para que Kubernetes no intente descargarla. Por eso la imagen se carga antes en Minikube con `minikube image load`.

```{literalinclude} k8s/service.yaml
:language: yaml
:caption: k8s/service.yaml
```

El servicio recibe tráfico en el puerto 80 y lo envía al puerto 8000 del contenedor.

## Despliegue

```bash
minikube start --memory=2048 --cpus=2
minikube image load heart-api
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods
kubectl get svc
```

Estado obtenido en el clúster local:

```text
$ kubectl apply -f k8s/deployment.yaml
deployment.apps/heart-model unchanged
$ kubectl apply -f k8s/service.yaml
service/heart-service unchanged

$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
heart-model-5d85d976d8-8n66j   1/1     Running   0          12m

$ kubectl get svc
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
heart-service   LoadBalancer   10.96.224.153   <pending>     80:30969/TCP   11m
kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP        12m
```

```{figure} images/etapa4_k8s_terminal.png
:name: fig-k8s-terminal
:width: 100%

Terminal con el pod `heart-model` en `Running` (1/1, 0 reinicios) y el reenvío de puerto hacia el servicio activo (`8080 -> 8000`).
```

El pod `heart-model` queda en `Running` (1/1) y el servicio `heart-service` expone el puerto 80 hacia el 8000 del contenedor. Que `EXTERNAL-IP` aparezca como `<pending>` es normal en Minikube, que no tiene un balanceador de carga externo.

```{admonition} Si el pod aparece en ErrImageNeverPull
:class: note
Con `imagePullPolicy: Never`, Kubernetes solo usa imágenes que ya estén dentro de Minikube. Si el pod muestra `ErrImageNeverPull`, la imagen todavía no se cargó: se resuelve ejecutando `minikube image load heart-api` y esperando a que el pod pase a `Running`.
```

## Acceso al servicio desde el navegador

Con Minikube sobre el driver Docker en Windows, la IP del nodo (`minikube ip`, por ejemplo `192.168.49.2`) no es accesible desde el navegador del equipo, por lo que `http://192.168.49.2:30969` no responde aunque el pod esté sano. Hay dos formas de llegar al servicio:

```bash
# Opción A: Minikube abre un túnel y muestra la URL (dejar la terminal abierta)
minikube service heart-service --url

# Opción B: reenvío de puerto hacia el servicio (dejar la terminal abierta)
kubectl port-forward service/heart-service 8080:80
```

Se usó la opción B. Con el reenvío activo, la API queda en `http://localhost:8080/docs`: la misma interfaz de la Etapa 3, pero servida por el pod de Kubernetes y no por el contenedor Docker (puerto 8000).

```{figure} images/etapa4_k8s_docs.png
:name: fig-k8s-docs
:width: 100%

Documentación interactiva de la API servida desde Kubernetes (`localhost:8080`, mediante `kubectl port-forward`), con el endpoint `POST /predict`.
```
