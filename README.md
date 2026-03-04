# ⚓ HiQode — Helm Charts Demo Guide

> **Kubernetes Ingress Demo** | 3 Microservices → AWS ALB | Login · Order · Payment

---

## 📁 Final Folder Structure

```
hiqode-ingress-demo/
├── login-app/
│   ├── Dockerfile
│   └── index.html
├── order-app/
│   ├── Dockerfile
│   └── index.html
├── payment-app/
│   ├── Dockerfile
│   └── index.html
├── hiqode-chart/                  ← Helm Chart (replaces k8s/ folder)
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── namespace.yaml
│       ├── login-deployment.yaml
│       ├── login-service.yaml
│       ├── order-deployment.yaml
│       ├── order-service.yaml
│       ├── payment-deployment.yaml
│       ├── payment-service.yaml
│       └── ingress.yaml
├── values-dev.yaml                ← Optional: env-specific overrides
├── values-prod.yaml
└── COMMANDS.sh
```

---

## 🔧 Step 1 — Install Helm

```bash
# Option A: Script (recommended)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Option B: macOS
brew install helm

# Option C: Windows
choco install kubernetes-helm

# Verify
helm version
```

---

## 📦 Step 2 — Create the Chart Scaffold

```bash
helm create hiqode-chart

# Remove default boilerplate templates
rm -rf hiqode-chart/templates/*
```

---

## 📄 Step 3 — Chart.yaml

```yaml
# hiqode-chart/Chart.yaml
apiVersion: v2
name: hiqode-chart
description: HiQode 3-app Kubernetes Ingress Demo
type: application
version: 1.0.0
appVersion: "1.0.0"
```

---

## ⚙️ Step 4 — values.yaml

```yaml
# hiqode-chart/values.yaml

namespace: hiqode

# AWS ECR Registry
registry: <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com

# Login App
login:
  image: login-app
  tag: v1
  replicas: 2
  port: 80
  resources:
    requests:
      cpu: "50m"
      memory: "64Mi"
    limits:
      cpu: "100m"
      memory: "128Mi"

# Order App
order:
  image: order-app
  tag: v1
  replicas: 2
  port: 80
  resources:
    requests:
      cpu: "50m"
      memory: "64Mi"
    limits:
      cpu: "100m"
      memory: "128Mi"

# Payment App
payment:
  image: payment-app
  tag: v1
  replicas: 2
  port: 80
  resources:
    requests:
      cpu: "50m"
      memory: "64Mi"
    limits:
      cpu: "100m"
      memory: "128Mi"

# Ingress / ALB
ingress:
  scheme: internet-facing
  targetType: ip
  healthcheckPath: /
```

---

## 📝 Step 5 — Templates

### `templates/namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Values.namespace }}
  labels:
    app: hiqode-demo
    managed-by: {{ .Release.Service }}
```

---

### `templates/login-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: login-deployment
  namespace: {{ .Values.namespace }}
  labels:
    app: login-app
spec:
  replicas: {{ .Values.login.replicas }}
  selector:
    matchLabels:
      app: login-app
  template:
    metadata:
      labels:
        app: login-app
    spec:
      containers:
        - name: login-app
          image: {{ .Values.registry }}/{{ .Values.login.image }}:{{ .Values.login.tag }}
          ports:
            - containerPort: {{ .Values.login.port }}
          resources:
            requests:
              cpu: {{ .Values.login.resources.requests.cpu }}
              memory: {{ .Values.login.resources.requests.memory }}
            limits:
              cpu: {{ .Values.login.resources.limits.cpu }}
              memory: {{ .Values.login.resources.limits.memory }}
```

---

### `templates/login-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: login-service
  namespace: {{ .Values.namespace }}
spec:
  selector:
    app: login-app
  ports:
    - protocol: TCP
      port: {{ .Values.login.port }}
      targetPort: {{ .Values.login.port }}
  type: ClusterIP
```

### `templates/order-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-deployment
  namespace: {{ .Values.namespace }}
  labels:
    app: order-app
spec:
  replicas: {{ .Values.order.replicas }}
  selector:
    matchLabels:
      app: order-app
  template:
    metadata:
      labels:
        app: order-app
    spec:
      containers:
        - name: order-app
          image: {{ .Values.registry }}/{{ .Values.order.image }}:{{ .Values.order.tag }}
          ports:
            - containerPort: {{ .Values.order.port }}
          resources:
            requests:
              cpu: {{ .Values.order.resources.requests.cpu }}
              memory: {{ .Values.order.resources.requests.memory }}
            limits:
              cpu: {{ .Values.order.resources.limits.cpu }}
              memory: {{ .Values.order.resources.limits.memory }}
```

---

### `templates/order-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: {{ .Values.namespace }}
spec:
  selector:
    app: order-app
  ports:
    - protocol: TCP
      port: {{ .Values.order.port }}
      targetPort: {{ .Values.order.port }}
  type: ClusterIP
```

---

### `templates/payment-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-deployment
  namespace: {{ .Values.namespace }}
  labels:
    app: payment-app
spec:
  replicas: {{ .Values.payment.replicas }}
  selector:
    matchLabels:
      app: payment-app
  template:
    metadata:
      labels:
        app: payment-app
    spec:
      containers:
        - name: payment-app
          image: {{ .Values.registry }}/{{ .Values.payment.image }}:{{ .Values.payment.tag }}
          ports:
            - containerPort: {{ .Values.payment.port }}
          resources:
            requests:
              cpu: {{ .Values.payment.resources.requests.cpu }}
              memory: {{ .Values.payment.resources.requests.memory }}
            limits:
              cpu: {{ .Values.payment.resources.limits.cpu }}
              memory: {{ .Values.payment.resources.limits.memory }}
```

---

### `templates/payment-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-service
  namespace: {{ .Values.namespace }}
spec:
  selector:
    app: payment-app
  ports:
    - protocol: TCP
      port: {{ .Values.payment.port }}
      targetPort: {{ .Values.payment.port }}
  type: ClusterIP
```

---

### `templates/ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hiqode-ingress
  namespace: {{ .Values.namespace }}
  annotations:
    alb.ingress.kubernetes.io/scheme: {{ .Values.ingress.scheme }}
    alb.ingress.kubernetes.io/target-type: {{ .Values.ingress.targetType }}
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
    alb.ingress.kubernetes.io/healthcheck-path: {{ .Values.ingress.healthcheckPath }}
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
          - path: /order
            pathType: Prefix
            backend:
              service:
                name: order-service
                port:
                  number: {{ .Values.order.port }}
          - path: /payment
            pathType: Prefix
            backend:
              service:
                name: payment-service
                port:
                  number: {{ .Values.payment.port }}
          - path: /                        # ← MUST be last
            pathType: Prefix
            backend:
              service:
                name: login-service
                port:
                  number: {{ .Values.login.port }}
```

---

## 🚀 Step 6 — Install, Upgrade & Rollback

### Lint & Dry Run (always do this first)

```bash
helm lint ./hiqode-chart

helm template hiqode ./hiqode-chart          # preview rendered YAML

helm install hiqode ./hiqode-chart \
  --dry-run --debug                          # simulate install
```
What it does
It checks the Helm chart for errors and best practices.
It validates things like:
Chart.yaml structure
YAML syntax
Template formatting
Required fields
Chart naming rules
Example Output

What it does - "--dry-run --debug  "
This simulates the installation without actually creating resources.
It shows:
•	Rendered Kubernetes YAML
•	Release name
•	Values used
•	Debug information
But nothing is deployed to the cluster.
Simple Meaning
It performs a fake installation to verify everything works correctly.



### Install

```bash
helm install hiqode ./hiqode-chart
```

### Verify

```bash
helm list

kubectl get pods    -n hiqode
kubectl get svc     -n hiqode
kubectl get ingress -n hiqode                # wait 2-3 mins for ALB

# Get ALB URL
kubectl get ingress hiqode-ingress -n hiqode \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

### Upgrade (e.g. new image tag)

```bash
helm upgrade hiqode ./hiqode-chart \
  --set login.tag=v2 \
  --set order.tag=v2 \
  --set payment.tag=v2

helm history hiqode                          # see all revisions
```

### Rollback

```bash
helm rollback hiqode 1                       # roll back to revision 1
helm history hiqode
```

### Uninstall

```bash
helm uninstall hiqode
```

---

## 🌍 Multi-Environment Deployments

```bash
# values-dev.yaml  →  1 replica, internal ALB, lower resources
# values-prod.yaml →  3 replicas, internet-facing ALB

helm install hiqode-dev  ./hiqode-chart -f values-dev.yaml
helm install hiqode-prod ./hiqode-chart -f values-prod.yaml

helm list --all-namespaces
```

---

## 🧪 Test in Browser

```
http://<ALB-URL>/           →  Login Page   🔐
http://<ALB-URL>/order      →  Order Page   📦
http://<ALB-URL>/payment    →  Payment Page 💳
```

---

## 📋 Helm Cheat Sheet

| Command | Description |
|---|---|
| `helm create <name>` | Scaffold a new chart |
| `helm lint <chart>` | Check for errors |
| `helm template <release> <chart>` | Render templates (no install) |
| `helm install <release> <chart>` | Install a release |
| `helm install ... --dry-run --debug` | Simulate install |
| `helm install ... --set key=val` | Override a value |
| `helm install ... -f values-x.yaml` | Use custom values file |
| `helm upgrade <release> <chart>` | Upgrade a release |
| `helm rollback <release> <rev>` | Roll back to a revision |
| `helm history <release>` | Show revision history |
| `helm list` | List all releases |
| `helm list --all-namespaces` | All releases in all namespaces |
| `helm get values <release>` | Show values used in a release |
| `helm get manifest <release>` | Show rendered YAML of release |
| `helm uninstall <release>` | Delete release + all resources |
| `helm repo add <name> <url>` | Add a chart repository |
| `helm package <chart-dir>` | Package chart into .tgz |

---

## 💡 Key Concepts

| Term | Meaning |
|---|---|
| **Chart** | Package of Kubernetes templates + default values |
| **Release** | A running instance of a chart in your cluster |
| **Revision** | Each install/upgrade increments the revision number |
| **Values** | Config inputs injected into templates via `{{ .Values.xxx }}` |
| **Repository** | A server hosting packaged Helm charts |

---

*HiQode DevOps Training — Helm Charts Demo*
