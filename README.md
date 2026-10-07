# Kubernetes Ingress Application Deployment

A hands-on Kubernetes project deploying a Flask Docker application using **Deployment, Service, and Ingress** on Minikube.

## Architecture

```text
                         Minikube
                            │
Client / Browser ─────→ Ingress
                            │
                            ↓
                  level2-app-service
                         :80
                            │
                    targetPort: 5000
                            │
                 ┌──────────┴──────────┐
                 ↓                     ↓
             Pod :5000             Pod :5000
                 │                     │
                 └──── Flask App ──────┘
```

The request path is:

```text
Client
  ↓
Ingress Controller
  ↓
Ingress
  ↓
Service
  ↓
Pods
  ↓
Application
```

The **Deployment is not part of the request path**. It creates and manages the Pods.

---

## 1. Docker Application

Docker image:

```text
nikhilb619/level2-app:latest
```

The Flask application listens on:

```text
0.0.0.0:5000
```

---

## 2. Deployment

The Deployment was configured with **2 replicas**.

### Labels and Selectors

The Pods are given a label:

```text
app: level2-app
```

The Deployment uses the same label in its selector:

```text
selector:
  matchLabels:
    app: level2-app
```

This creates the relationship:

```text
Deployment selector
        ↓
finds Pods with
app: level2-app
```


---

## 3. Container Port

For the container we used:

```yaml
ports:
  - containerPort: 5000
```

`containerPort` indicates the application port

In this project:

```text
Flask application → port 5000
containerPort     → 5000
```

### Important note

`containerPort` is mainly **documentation/metadata**. It is not what connects the Service to the container.

The Service uses **`targetPort`** to send traffic to the application.

---

## 4. Service

The Service was created as a `ClusterIP` Service.

Its important ports are:

```yaml
port: 80
targetPort: 5000
```

This means:

```text
Service port 80
      ↓
Pod port 5000
```

The Service selects the Pods using their label:

```text
Service selector
      ↓
app: level2-app
      ↓
Pods with matching label
```

So the complete connection is:

```text
Service
  │
  │ selector: app: level2-app
  ↓
Matching Pods
  │
  │ targetPort: 5000
  ↓
Flask application
```

### Port distinction

```text
containerPort → describes the application's container port
targetPort    → tells the Service where to send traffic
port          → port exposed by the Service
```

---

## 5. Ingress

The Ingress provides an HTTP/HTTPS routing rule that sends requests for `/` to the Service.

Conceptually:

```text
/
 ↓
level2-app-service:80
```

### Path

We used:

```text
path: /
```

`/` represents the root path.

For example:

```text
http://example.com/
```

An Ingress can also have different paths:

```text
/api
/admin
/
```

which can be routed to different Services.

### `pathType: Prefix`

We used:

```text
pathType: Prefix
```

This means the rule matches the specified path and paths below it.

For example, with:

```text
path: /
pathType: Prefix
```

requests such as these can match:

```text
/
/login
/api
/products
```

For a path such as:

```text
/api
```

the Prefix rule can match:

```text
/api
/api/users
/api/products
```

---

## 6. Ingress Controller

An Ingress resource defines the routing rules, but an **Ingress Controller** actually processes those rules.

Minikube's NGINX Ingress Controller was enabled:

```bash
minikube addons enable ingress
```

The controller was verified with:

```bash
kubectl get pods -n ingress-nginx
```

The controller was running successfully.

recent:///14e6639fb97c8943533a607d6ac5d990
---

## Final Traffic Flow

```text
                    NGINX
Client ───────→ Ingress Controller
                       │
                       ↓
                 Ingress Rule
                   path: /
                       │
                       ↓
             level2-app-service
                    port: 80
                       │
                 targetPort: 5000
                       │
                ┌──────┴──────┐
                ↓             ↓
             Pod :5000     Pod :5000
                │             │
                └── Flask ────┘
```

The relationships to remember:

```text
Deployment
    ↓
creates/manages Pods

Labels + Selectors
    ↓
identify/select the correct Pods

Service
    ↓
selects Pods using labels
    ↓
port 80 → targetPort 5000

Ingress
    ↓
routes HTTP traffic using paths
    ↓
Service :80

Ingress Controller
    ↓
actually processes the Ingress rules
```

## Verification

The application was tested through the Ingress address:

```bash
curl http://192.168.49.2/
```

Response:

```text
Level 3 Achieved! Hello,Live-Reloading is Working!
```
recent:///f1b72fce42053ed1228ea1db6ac5db1b

## Key Takeaways

- **Deployment** manages application Pods.
- **Labels and selectors** connect Kubernetes resources to the correct Pods.
- **Service** provides stable networking to Pods.
- **`port`** is the Service port.
- **`targetPort`** is the application port the Service sends traffic to.
- **`containerPort`** documents the intended container port; it does not create the Service connection.
- **Ingress** defines HTTP/HTTPS routing rules.
- **`/`** is the root path.
- **`Prefix`** matches the path and its subpaths.
- **Ingress Controller** implements the Ingress rules.
