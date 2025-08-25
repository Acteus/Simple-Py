# Simple-Py

This repository contains a tiny Flask app and Kubernetes manifests to test Docker and Kubernetes across multiple devices on a local network.

Use case: run the app in a container locally, deploy to a local Kubernetes cluster (or cluster on your LAN), and access it from other machines using the NodePort service.

## Contents

- `app.py` - minimal Flask app listening on port 5000
- `Dockerfile` - builds the container image
- `requirements.txt` - Python dependencies
- `deployment.yaml` - Kubernetes Deployment (image: `acteus/flask-hello:1.0`)
- `service.yaml` - Kubernetes Service (NodePort 30080 -> container 5000)

## Quick start (PowerShell)

Build the Docker image:

```powershell
docker build -t acteus/flask-hello:1.0 .
```

Run locally (maps host 5000 -> container 5000):

```powershell
docker run --rm -p 5000:5000 acteus/flask-hello:1.0
```

Verify from the same machine:

```powershell
Invoke-WebRequest -UseBasicParsing http://localhost:5000
# or: curl http://localhost:5000
```

Deploy to Kubernetes:

```powershell
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Access from other devices on your LAN

1. Find the node IP (the host running the Kubernetes node). Example using PowerShell:

```powershell
ipconfig
# look for the IPv4 address of the node on your LAN, e.g. 192.168.1.42
```

2. Open a browser on another device and visit:

```
http://<NODE_IP>:30080
```

Notes

- If your cluster cannot pull the `acteus/flask-hello:1.0` image, either push the image to a registry accessible by the cluster or update `deployment.yaml` to use an image you host.
- If you're using a single-machine Kubernetes (minikube/kind), follow that tool's image-loading instructions or point `deployment.yaml` at a registry image.
- Ensure any host firewall allows inbound traffic to the NodePort (30080) if you want to reach it from other devices on the LAN.

Troubleshooting

- If you receive a 404 or connection refused, check `kubectl get pods`, `kubectl describe pod <pod>` and `kubectl logs <pod>`.
- If the Service shows `Pending` or `0/1` endpoints, verify the Deployment pods are running and labeled with `app: flask-hello` (the selector in `service.yaml`).

License / Attribution

This is a small demo repository for local testing. Modify freely for your environment.
