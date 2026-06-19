# Kubernetes + Minikube Project Notes

## Prerequisites

For this project, we need basic knowledge of:

- Docker
- Kubernetes
- Minikube
- kubectl

Make sure Docker Desktop is running before starting Minikube.

---

## Start Minikube

Start the local Kubernetes cluster:

```bash
minikube start
```

Minikube creates a local Kubernetes cluster and uses Docker to run Kubernetes nodes.

---

## Creating a Pod

Create a YAML file (`resnet.yaml`) inside the project folder.

Create the pod:

```bash
kubectl apply -f resnet.yaml
```

Check the pod:

```bash
kubectl get pods
```

View pod logs:

```bash
kubectl logs -f resnet-pod
```

This helps us check whether the container is starting correctly.

Forward the container port to localhost:

```bash
kubectl port-forward pod/resnet-pod 8081:8080
```

Access the application:

```text
http://localhost:8081
```

### What is a Pod?

A Pod is the smallest unit in Kubernetes.

A Pod acts like a box that contains one or more containers running together.

---

## ReplicaSet

A ReplicaSet ensures that a desired number of Pods are always running.

Create a ReplicaSet:

```bash
kubectl apply -f resnet_replicaset.yaml
```

We configured:

```yaml
replicas: 3
```

Therefore Kubernetes creates 3 Pods.

Check Pods:

```bash
kubectl get pods
```

Delete one Pod:

```bash
kubectl delete pod resnet-rs-h59vk
```

A new Pod is automatically created because the ReplicaSet maintains the desired number of replicas.

This solves the self-healing requirement.

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

Check resources:

```bash
kubectl -n cloudproject get deploy
kubectl -n cloudproject get rs
kubectl -n cloudproject get po
```

We can now see:

- Deployment
- ReplicaSet
- Pods

managed together.

---

## Updating the Application

If a new image version is available:

```bash
kubectl set image deployment/resnet-deployment resnet-container=<image_name>
```

General syntax:

```bash
kubectl set image deployment/<deployment-name> <container-name>=<image-name>
```

---

## Rollback

Revert to the previous version if an update fails:

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
Internal communication only.

## NodePort
Exposes the application using NodeIP:NodePort.

## LoadBalancer
Provides a public entry point to the application.

## Ingress
Acts as a smart traffic router.

---

## NodePort

Check services:

```bash
kubectl -n cloudproject get svc
```

NodePort works but depends on the node IP and assigned port.

---

## LoadBalancer

Create a LoadBalancer service.

Initially the External IP may stay in a pending state.

Run:

```bash
minikube tunnel
```

After the tunnel starts, the application becomes accessible through:

```text
http://127.0.0.1:8080
```

---

# Ingress

Problem:

If we have multiple services, creating a separate LoadBalancer for every service can become expensive.

Ingress solves this by providing:

```text
One Entry Point → Multiple Destinations
```

Enable Ingress:

```bash
minikube addons enable ingress
```

Start the tunnel:

```bash
minikube tunnel
```

Check Ingress:

```bash
kubectl -n cloudproject get ingress
```

Edit the Windows hosts file:

```powershell
notepad C:\Windows\System32\drivers\etc\hosts
```

Add an entry similar to:

```text
127.0.0.1 resnet.local
```

or

```text
<minikube-ip> resnet.local
```

Now requests to:

```text
http://resnet.local
```

can be routed to the correct Kubernetes service.

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

### Future Autoscaling Architecture

```text
Deployment
     ↓
Pods
     ↓
Service
     ↓
HPA (Horizontal Pod Autoscaler)
```
