# Containerisation-and-Deployment-of-Wisecow-Application-on-Kubernetes
Dockerization: First, we'll need to create a Dockerfile to containerize the Wisecow application. This file is essential for creating a Docker image that can be pushed to a container registry.
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME WisecowApp

# Run app.py when the container launches
CMD ["python", "app.py"]
Kubernetes Deployment: We'll create Kubernetes manifests to deploy the Wisecow application on a Kubernetes cluster. This includes deployment and service files for scaling and exposing the application.4
Kubernetes Deployment (wisecow-deployment.yaml):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
        - name: wisecow-container
          image: <container-registry-url>/wisecow:latest
          ports:
            - containerPort: 80
Kubernetes Service (wisecow-service.yaml):
apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
CI/CD Pipeline (GitHub Actions Workflow): A GitHub Actions workflow is needed to automate the build and deployment of the Docker image to a container registry and Kubernetes cluster.
Actions workflow (.github/workflows/ci-cd.yml):
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        run: |
          docker build -t <container-registry-url>/wisecow:${{ github.sha }} .
          docker push <container-registry-url>/wisecow:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Set up kubectl
        uses: azure/setup-kubectl@v2
        with:
          kubeconfig: ${{ secrets.KUBECONFIG }}

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f wisecow-deployment.yaml
          kubectl apply -f wisecow-service.yaml
TLS Implementation: To implement TLS for secure communication, we will need to generate and configure SSL certificates. You can use tools like Let's Encrypt to generate certificates and configure the application to use HTTPS. In Kubernetes, you can use Ingress controllers to manage TLS.
 Ingress YAML with TLS (wisecow-ingress.yaml):apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  rules:
    - host: wisecow.yourdomain.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: wisecow-service
                port:
                  number: 80
  tls:
    - hosts:
        - wisecow.yourdomain.com
      secretName: wisecow-tls-secret
