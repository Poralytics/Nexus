# 🚀 NEXUS Kubernetes Deployment - Enterprise Scale

## Production-Grade Kubernetes Setup with High Availability

---

## 📋 Prerequisites

- Kubernetes cluster 1.25+ (EKS, GKE, AKS, or self-hosted)
- kubectl configured
- Helm 3.x installed
- 3+ worker nodes (recommended)
- Storage class for persistent volumes
- Ingress controller (nginx)
- Cert-manager for SSL

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│              Ingress / Load Balancer            │
│          (nginx-ingress + cert-manager)         │
└──────────┬──────────────────────────────────────┘
           │
    ┌──────┴───────┐
    │              │
┌───▼────┐    ┌───▼────┐
│Backend │    │Backend │  (3+ replicas, HPA)
│  Pod   │    │  Pod   │
└───┬────┘    └───┬────┘
    │              │
    └──────┬───────┘
           │
    ┌──────┴───────┐
    │              │
┌───▼────┐    ┌───▼────┐
│Worker  │    │Worker  │  (5+ replicas)
│  Pod   │    │  Pod   │
└───┬────┘    └───┬────┘
    │              │
    └──────┬───────┘
           │
    ┌──────┴──────────────┐
    │                     │
┌───▼────┐         ┌─────▼─────┐
│PostgreSQL        │   Redis    │
│ StatefulSet      │ StatefulSet│
│ (Primary +       │  (Cluster) │
│  Replicas)       │            │
└──────────┘       └────────────┘
```

---

## 📦 Deployment Files

### 1. Namespace

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nexus-production
  labels:
    name: nexus-production
```

### 2. PostgreSQL StatefulSet

```yaml
# postgres-statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: nexus-production
spec:
  ports:
    - port: 5432
      name: postgres
  clusterIP: None
  selector:
    app: postgres
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: nexus-production
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_DB
          valueFrom:
            secretKeyRef:
              name: nexus-secrets
              key: db-name
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: nexus-secrets
              key: db-user
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: nexus-secrets
              key: db-password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - nexus
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - nexus
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: standard
      resources:
        requests:
          storage: 50Gi
```

### 3. Redis StatefulSet

```yaml
# redis-statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: nexus-production
spec:
  ports:
    - port: 6379
      name: redis
  clusterIP: None
  selector:
    app: redis
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: nexus-production
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
          name: redis
        command:
        - redis-server
        - --appendonly
        - "yes"
        - --cluster-enabled
        - "yes"
        volumeMounts:
        - name: redis-storage
          mountPath: /data
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 30
          periodSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: redis-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: standard
      resources:
        requests:
          storage: 10Gi
```

### 4. Backend Deployment

```yaml
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-backend
  namespace: nexus-production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nexus-backend
  template:
    metadata:
      labels:
        app: nexus-backend
    spec:
      containers:
      - name: backend
        image: nexus/backend:2.2
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          value: "postgres-0.postgres.nexus-production.svc.cluster.local"
        - name: DB_PORT
          value: "5432"
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: nexus-secrets
              key: db-name
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: nexus-secrets
              key: db-user
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: nexus-secrets
              key: db-password
        - name: REDIS_URL
          value: "redis://redis-0.redis.nexus-production.svc.cluster.local:6379"
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: nexus-secrets
              key: jwt-secret
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: nexus-backend
  namespace: nexus-production
spec:
  selector:
    app: nexus-backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

### 5. Worker Deployment

```yaml
# worker-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-worker
  namespace: nexus-production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nexus-worker
  template:
    metadata:
      labels:
        app: nexus-worker
    spec:
      containers:
      - name: worker
        image: nexus/backend:2.2
        command: ["node", "workers/scan-worker.js"]
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          value: "postgres-0.postgres.nexus-production.svc.cluster.local"
        - name: REDIS_URL
          value: "redis://redis-0.redis.nexus-production.svc.cluster.local:6379"
        envFrom:
        - secretRef:
            name: nexus-secrets
        resources:
          requests:
            memory: "1Gi"
            cpu: "1000m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
```

### 6. HorizontalPodAutoscaler

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nexus-backend-hpa
  namespace: nexus-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nexus-backend
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nexus-worker-hpa
  namespace: nexus-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nexus-worker
  minReplicas: 5
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

### 7. Ingress with SSL

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nexus-ingress
  namespace: nexus-production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - nexus.yourcompany.com
    secretName: nexus-tls
  rules:
  - host: nexus.yourcompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nexus-backend
            port:
              number: 80
```

### 8. Secrets

```yaml
# secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: nexus-secrets
  namespace: nexus-production
type: Opaque
stringData:
  db-name: "nexus_production"
  db-user: "nexus"
  db-password: "CHANGE_ME_SECURE_PASSWORD_32_CHARS"
  jwt-secret: "CHANGE_ME_JWT_SECRET_64_CHARS"
```

---

## 🚀 Deployment Commands

```bash
# 1. Create namespace
kubectl apply -f namespace.yaml

# 2. Create secrets (EDIT FIRST!)
kubectl apply -f secrets.yaml

# 3. Deploy PostgreSQL
kubectl apply -f postgres-statefulset.yaml

# 4. Deploy Redis
kubectl apply -f redis-statefulset.yaml

# 5. Wait for databases
kubectl wait --for=condition=ready pod -l app=postgres -n nexus-production --timeout=300s
kubectl wait --for=condition=ready pod -l app=redis -n nexus-production --timeout=300s

# 6. Deploy backend
kubectl apply -f backend-deployment.yaml

# 7. Deploy workers
kubectl apply -f worker-deployment.yaml

# 8. Deploy HPA
kubectl apply -f hpa.yaml

# 9. Deploy ingress
kubectl apply -f ingress.yaml

# 10. Verify
kubectl get pods -n nexus-production
kubectl get svc -n nexus-production
kubectl get ingress -n nexus-production
```

---

## 📊 Monitoring

```yaml
# servicemonitor.yaml (for Prometheus Operator)
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nexus-backend
  namespace: nexus-production
spec:
  selector:
    matchLabels:
      app: nexus-backend
  endpoints:
  - port: http
    path: /metrics
    interval: 30s
```

---

## 🔄 Updates & Rollbacks

```bash
# Update image
kubectl set image deployment/nexus-backend backend=nexus/backend:2.3 -n nexus-production

# Check rollout status
kubectl rollout status deployment/nexus-backend -n nexus-production

# Rollback if needed
kubectl rollout undo deployment/nexus-backend -n nexus-production
```

---

## 📈 Capacity Planning

**For 10,000 scans/day:**
- Backend: 3-5 pods (HPA)
- Workers: 10-20 pods (HPA)
- PostgreSQL: 50GB storage, 4vCPU, 8GB RAM
- Redis: 10GB storage, 2vCPU, 4GB RAM

**For 100,000 scans/day:**
- Backend: 5-10 pods
- Workers: 30-50 pods
- PostgreSQL: 200GB storage, 8vCPU, 16GB RAM
- Redis: 50GB storage, 4vCPU, 8GB RAM

---

**NEXUS Kubernetes - Enterprise Scale Ready** 🚀
