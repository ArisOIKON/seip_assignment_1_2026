# Echo API Kubernetes Deployment

This repository contains a Node.js Echo API packaged with Docker and deployed to a local Kubernetes cluster using Minikube.

The application is containerized, published to the GitHub Container Registry (GHCR), and deployed using declarative Kubernetes manifests stored in the `k8s/` directory.

## Repository Structure

```text
.
├── Dockerfile
├── package.json
├── server.js
├── README.md
├── .github/
│   └── workflows/
│       └── ci-cd.yaml
└── k8s/
    ├── configmap.yaml
    ├── secret.yaml
    ├── deployment.yaml
    └── service.yaml
```

## Prerequisites

Before running the application, make sure the following tools are installed:

- Git
- Docker Desktop
- Minikube
- kubectl
- A GitHub account with access to GHCR

Docker Desktop must be running before starting Minikube with the Docker driver.

## Clone the Repository

Clone the repository to your local machine:

```bash
git clone <your-repository-url>
cd <your-repository-name>
```

Replace `<your-repository-url>` and `<your-repository-name>` with your actual GitHub repository URL and folder name.

## Start Minikube

Start a local Kubernetes cluster using the Docker driver:

```bash
minikube start --driver=docker
```

Verify that Minikube is running:

```bash
minikube status
```

Verify that `kubectl` can communicate with the Minikube cluster:

```bash
kubectl get nodes
```

You should see the `minikube` node with status `Ready`.

## Kubernetes Manifests

All Kubernetes manifests are stored inside the `k8s/` directory.

The manifests include:

- `configmap.yaml` — stores non-sensitive configuration such as `WELCOME_MESSAGE` and `NODE_ENV`.
- `secret.yaml` — stores the Base64-encoded `API_SECRET_KEY`.
- `deployment.yaml` — defines the application workload, replicas, resources, probes, and environment variables.
- `service.yaml` — exposes the application internally using a `ClusterIP` service.

## Apply the Manifests Sequentially

Although the whole directory can be applied at once, the manifests can also be applied sequentially for clarity.

From the root of the repository, run:

```bash
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Alternatively, apply everything with a single command:

```bash
kubectl apply -f k8s/
```

## Verify the Deployment

Check that the ConfigMap was created:

```bash
kubectl get configmaps
```

Check that the Secret was created:

```bash
kubectl get secrets
```

Check the Deployment:

```bash
kubectl get deployments
```

Check the Pods:

```bash
kubectl get pods
```

The deployment should create exactly 3 replicas. You should eventually see all pods in the `Running` state.

Example:

```text
echo-api-deployment-xxxxxxxxxx-xxxxx   1/1   Running   0   1m
echo-api-deployment-xxxxxxxxxx-xxxxx   1/1   Running   0   1m
echo-api-deployment-xxxxxxxxxx-xxxxx   1/1   Running   0   1m
```

Check the Service:

```bash
kubectl get services
```

The service should be of type `ClusterIP` and expose port `80`, forwarding traffic to the container port `3000`.

## Port Forwarding

The Kubernetes Service is internal to the cluster because it uses the `ClusterIP` type. To access it from your local machine, use `kubectl port-forward`.

Run:

```bash
kubectl port-forward service/echo-api-service 8080:80
```

This maps:

- Local machine: `localhost:8080`
- Kubernetes Service: `echo-api-service` on port `80`
- Container: port `3000`

Keep this terminal window open while testing the application.

## Interact with the Application

Open the application in your browser:

```text
http://localhost:8080
```

Or use `curl`:

```bash
curl http://localhost:8080
```

Test the health endpoint:

```bash
curl http://localhost:8080/health
```

The `/health` endpoint is used by the Kubernetes liveness and readiness probes.


## Notes

- The Docker image is pulled from GHCR using the lowercase image name:

```text
ghcr.io/arisoikon/echo-api:latest
```
- Docker image names must be lowercase.
- If the image cannot be pulled, confirm that the GitHub Actions workflow succeeded and that the GHCR package is public or accessible to Minikube.

