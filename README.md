# Kubernetes + Minikube Project Notes

## Prerequisites

For this project, we need basic knowledge of:

- Docker
- Kubernetes
- Minikube
- kubectl

Make sure Docker Desktop is running in the background before starting Minikube.

---

## Starting Minikube

Start the local Kubernetes cluster:

```bash
minikube start
```

Minikube creates a local Kubernetes cluster and uses Docker images.

---

## Creating a Pod

Create a YAML file (`resnet.yaml`) inside the project folder.

Create the pod:

```bash
kubectl apply -f resnet.yaml
```

Check running pods:

```bash
kubectl get pods
```

View pod logs:

```bash
kubectl logs -f resnet-pod
```

This helps us monitor the pod status and check whether the container is running correctly.

Forward the container port to the local machine:

```bash
kubectl port-forward pod/resnet-pod 8081:8080
```

Now the application can be accessed through:

```text
http://localhost:8081
```

### What is a Pod?

A Pod is the smallest unit in Kubernetes.

A Pod acts as a box that contains one or more containers running together.

---

## ReplicaSet

A ReplicaSet ensures that a desired number of pods are always running.

Create the ReplicaSet:

```bash
kubectl apply -f resnet_replicaset.yaml
```

We configured:

```yaml
replicas: 3
```

Therefore Kubernetes creates 3 pods.

Check the pods:

```bash
kubectl get pods
```

Delete one pod:

```bash
kubectl delete pod <pod-name>
```

Kubernetes automatically creates a new pod to maintain the desired replica count.

Delete the ReplicaSet:

```bash
kubectl delete rs resnet-rs
```

---

## Deployment

A Deployment is used to manage ReplicaSets and Pods.

Create a namespace:

```bash
kubectl create namespace cloudproject
```

Deploy the application:

```bash
kubectl -n cloudproject apply -f deployment.yaml
```

Check deployment resources:

```bash
kubectl -n cloudproject get deploy
kubectl -n cloudproject get rs
kubectl -n cloudproject get po
```

---

## Updating the Application

If a new Docker image version is available:

```bash
kubectl set image deployment/resnet-deployment resnet-container=<image_name>
```

General syntax:

```bash
kubectl set image deployment/<deployment-name> <container-name>=<image-name>
```

---

## Rollback

If the update causes issues, revert to the previous version:

```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

---

## Scaling

Increase or decrease the number of replicas manually:

```bash
kubectl scale deployment <deployment-name> --replicas=<number> -n <namespace>
```

Example:

```bash
kubectl scale deployment resnet-deployment --replicas=5 -n cloudproject
```

---

# Kubernetes Services

## ClusterIP
Internal communication only. Accessible only inside the Kubernetes cluster.

## NodePort
Exposes the application outside the cluster using NodeIP:NodePort.

## LoadBalancer
Creates an external load balancer and provides a public entry point to the application.

## Ingress
Smart traffic router that routes requests to different services using domains or paths.

---

## NodePort Service

Check services:

```bash
kubectl -n cloudproject get svc
```

NodePort exposes the application externally but depends on the node IP and assigned port.

---

## LoadBalancer Service

Initially, the external IP may remain in a pending state.

To expose the LoadBalancer locally in Minikube:

```bash
minikube tunnel
```

After the tunnel starts, the application becomes accessible through:

```text
http://127.0.0.1:8080
```

---

## Project Architecture

```text
Deployment
     ↓
ReplicaSet
     ↓
Pods
     ↓
Service (NodePort / LoadBalancer)
     ↓
Users
```

### Autoscaling Architecture

```text
Deployment
     ↓
Pods
     ↓
Service
     ↓
HPA (Horizontal Pod Autoscaler)
```
