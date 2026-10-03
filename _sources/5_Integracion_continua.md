# Etapa 5 · Integración continua con GitHub Actions

Cada `push` al repositorio dispara un flujo de trabajo que instala las dependencias, revisa el estilo del código y comprueba que la API predice con el modelo cargado. Si algo falla, GitHub marca el commit en rojo y el error se detecta antes de desplegar.

Archivo de esta etapa en GitHub: [`.github/workflows/ci.yml`](https://github.com/juandpolo22/heart-disease-mlops/blob/main/.github/workflows/ci.yml). Las ejecuciones están en la pestaña [Actions](https://github.com/juandpolo22/heart-disease-mlops/actions).

```{literalinclude} .github/workflows/ci.yml
:language: yaml
:caption: .github/workflows/ci.yml
```

## Qué hace cada paso

| Paso | Qué hace |
|---|---|
| *Install dependencies* | Instala `docker/requirements.txt` (las mismas versiones del contenedor) más `flake8` y `pytest` |
| *Lint* | `flake8 app/` revisa el estilo del código de la API |
| *Tests* | Importa `app/api.py`, carga `app/model.joblib` y verifica que `/predict` devuelve el formato esperado, que un paciente de alto riesgo obtiene predicción 1, uno de bajo riesgo predicción 0, y que la probabilidad del primero es mayor |

El enunciado ejecuta `pytest tests/`. Como la estructura del proyecto no incluye una carpeta `tests/`, la misma verificación se hace con un script de Python dentro del propio flujo.

## Evidencia

El repositorio en GitHub con las carpetas del proyecto y el flujo de trabajo `.github/workflows`:

```{figure} images/etapa5_repositorio.png
:name: fig-repo
:width: 100%

Repositorio `juandpolo22/heart-disease-mlops` con la estructura del enunciado.
```

El primer `push` (commit `5c6d2b7`) disparó la ejecución CI #1 en la rama `main`, que terminó con éxito en 35 segundos:

```{figure} images/etapa5_ejecucion.png
:name: fig-ejecucion
:width: 100%

Ejecución CI #1 en verde en la pestaña Actions.
```

El detalle muestra los pasos del trabajo `build`, todos en verde, incluidos *Lint* y *Tests*:

```{figure} images/etapa5_pasos.png
:name: fig-pasos
:width: 100%

Pasos del trabajo `build`: instalación de dependencias (22 s), *Lint* (1 s) y *Tests* (2 s).
```

```{note}
Las anotaciones que se ven arriba del detalle son avisos informativos de GitHub, no fallos: una advertencia sobre la próxima retirada de Node.js 20 en las acciones `actions/checkout@v3` y `actions/setup-python@v4`, y un aviso de que la etiqueta `ubuntu-latest` pasará a Ubuntu 26 el 19 de octubre de 2026.
```
