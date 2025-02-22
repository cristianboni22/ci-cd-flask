# ci-cd-flask

En este tutorial, vamos a implementar un flujo de Integración Continua (CI) y Entrega/Despliegue Continuo (CD) para una aplicación web desarrollada con Flask. El objetivo es automatizar pruebas, la creación de imágenes Docker y el despliegue en AWS.

---

## Objetivos


- **Intregración Continua (CI)**. Vamos a automatizar la ejecución de tests unitarios cada vez que se haga un _push_ a la rama `main`.

- **Entrega Continua (CD)**. Vamos a automatizar la creación y publicación de una imagen Docker en Docker Hub cuando se pasen los test unitarios.
- **Despliegue Continuo (CD)**. Vamos a automatizar el despliegue de la imagen Docker en AWS.

---

## Requisitos Previos


- **Tener instalado Python 3.8 o superior**

- **Cuenta en Docker Hub**

- **AWS CLI configurado con credenciales válidas**

- **Git instalado y repositorio configurado en GitHub**

---

## Preparación del Entorno

**Crear y activar un entorno virtual**

```bash
python3 -m venv venv
source venv/bin/activate
```

**Instalar las dependencias**

```bash
pip install -r requirements.txt
```

**Desactivar el entorno virtual**

```bash
deactivate
```

---

## Ejecución de Pruebas Unitarias

Usaremos unittest, el módulo integrado de Python.

### Comando para ejecutar las pruebas

```bash
python3 -m unittest tests/*.py
```

---

## Configuración de CI con GitHub Actions

El archivo .github/workflows/ci-cd.yml podría tener el siguiente contenido:

```bash
name: CI/CD Preproduction

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  # github.repository = <account>/<repo>. Example: josejuansanchez/ci-cd-python
  #IMAGE_NAME: ${{ github.repository }}
  IMAGE_NAME: ci-cd-python

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.13'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Run tests
        run: |
          python -m unittest discover tests

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_HUB_TOKEN }}" | docker login -u "${{ secrets.DOCKER_HUB_USERNAME }}" --password-stdin
      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:latest .
      - name: Push Docker image
        run: docker push ${{ secrets.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:latest
```

---

## Creación y Publicación de la Imagen Docker

### Comandos para construir y publicar la imagen

```bash
# Iniciar sesión en Docker Hub
docker login

# Construir la imagen Docker
docker build -t usuario/ci-cd-flask .

# Enviar la imagen al repositorio
docker push usuario/ci-cd-flask
```
---

## Ejecución Completa con un Solo Comando

```bash
git push origin main
```

Con este comando se disparará todo el flujo CI/CD: pruebas automáticas, creación de la imagen Docker y despliegue en AWS
