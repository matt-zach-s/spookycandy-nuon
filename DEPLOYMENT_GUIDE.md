# Spooky Candy Manager - Nuon Deployment Guide

## 🎉 What We've Built

A complete Nuon app configuration to deploy the Spooky Candy Manager (Next.js + PostgreSQL) to AWS EKS via Nuon's platform.

## 📁 Structure Created

```
spookycandy-nuon/
└── your-app/
    ├── metadata.toml              ✅ App metadata
    ├── installer.toml             ✅ Installer configuration
    ├── inputs.toml                ✅ User inputs (domain, db config)
    ├── runner.toml                ✅ Runner configuration
    ├── stack.toml                 ✅ CloudFormation stack config
    ├── README.md                  ✅ User-facing install guide
    │
    ├── components/                ✅ Component definitions
    │   ├── 1-postgres.toml            → PostgreSQL database
    │   ├── 2-db-secrets.toml          → Database connection secret
    │   ├── 3-candy-app.toml           → Next.js application
    │   ├── 4-certificate.toml         → TLS certificate (ACM)
    │   └── 5-alb.toml                 → Application Load Balancer
    │
    ├── actions/                   📁 Lifecycle scripts (empty, ready for use)
    ├── permissions/               ✅ IAM roles (provision, maintenance, deprovision)
    │
    └── src/components/            ✅ Source code for components
        ├── postgres/                  → Bitnami PostgreSQL Helm chart
        ├── candy-app/                 → Custom Helm chart for Next.js
        │   ├── Chart.yaml
        │   ├── values.yaml
        │   └── templates/
        │       ├── deployment.yaml
        │       └── service.yaml
        ├── certificate/               → Terraform module for ACM
        └── alb/                       → Helm chart for ALB ingress
```

## 🐳 Application Changes

Modified `spookycandy` repository:
- ✅ Added `Dockerfile` for production builds
- ✅ Added `.dockerignore`
- ✅ Updated `next.config.mjs` with `output: 'standalone'`

## 🏗️ Architecture

When deployed via Nuon, this creates:

```
AWS Account (Customer's)
│
├── EKS Cluster (via Nuon Sandbox)
│   │
│   ├── Namespace: spookycandy
│   │   │
│   │   ├── PostgreSQL (Helm - Bitnami)
│   │   │   └── PersistentVolume (10Gi EBS)
│   │   │
│   │   ├── DB Secret (Kubernetes Secret)
│   │   │   └── NEON_DATABASE_URL
│   │   │
│   │   └── Candy App (2 replicas)
│   │       ├── Container: Next.js (port 3000)
│   │       ├── Service: ClusterIP
│   │       └── Env: DB connection from secret
│   │
│   └── ALB Ingress Controller
│       └── Ingress → Service → Pods
│
├── Application Load Balancer (HTTPS:443)
│   └── Certificate: ACM wildcard cert
│
├── Route53 DNS
│   └── {subdomain}.{install-id}.nuon.run
│
└── IAM Roles
    ├── Provision role
    ├── Maintenance role
    └── Deprovision role
```

## 🚀 Deployment Flow

### 1. Component Deployment Order

Components are deployed in sequence based on dependencies:

```
1. postgres          (no dependencies)
   ↓
2. db_secrets        (depends on: postgres)
   ↓
3. candy_app         (depends on: postgres, db_secrets)
   ↓
4. certificate       (no dependencies, but needed by ALB)
   ↓
5. alb              (depends on: candy_app, certificate)
```

### 2. Database Connection

The app connects to PostgreSQL using Kubernetes internal DNS:

```
postgresql.spookycandy.svc.cluster.local:5432
```

Connection string format:
```
postgresql://{username}:{password}@postgresql.spookycandy.svc.cluster.local:5432/{dbname}?sslmode=disable
```

### 3. Environment Variables

The candy app receives `NEON_DATABASE_URL` from the Kubernetes secret, allowing seamless connection without code changes.

## 📋 Next Steps to Deploy

### Step 1: Push to GitHub

```bash
cd /Users/nuonuser/spookycandy-nuon
git init
git add .
git commit -m "Initial Nuon app config for Spooky Candy Manager"
git remote add origin https://github.com/matt-zach-s/spookycandy-nuon.git
git push -u origin main
```

### Step 2: Build and Push Docker Image

You'll need to build the Docker image and push it to a registry (ECR, DockerHub, etc.):

```bash
cd /Users/nuonuser/spookycandy

# Option A: Docker Hub
docker build -t matt-zach-s/spookycandy:latest .
docker push matt-zach-s/spookycandy:latest

# Option B: AWS ECR (recommended for Nuon)
# First create ECR repo in AWS console
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin {account-id}.dkr.ecr.us-west-2.amazonaws.com
docker build -t spookycandy .
docker tag spookycandy:latest {account-id}.dkr.ecr.us-west-2.amazonaws.com/spookycandy:latest
docker push {account-id}.dkr.ecr.us-west-2.amazonaws.com/spookycandy:latest
```

### Step 3: Update Image Reference

Update `spookycandy-nuon/your-app/src/components/candy-app/values.yaml`:

```yaml
image:
  repository: matt-zach-s/spookycandy  # or ECR URL
  tag: latest
```

### Step 4: Create Nuon App

```bash
nuon login
nuon apps create --name spookycandy
cd /Users/nuonuser/spookycandy-nuon/your-app
nuon apps sync .
```

### Step 5: Deploy via Nuon Dashboard

1. Go to https://app.nuon.co
2. Select "Spooky Candy Manager" app
3. Click "Install"
4. Fill in inputs:
   - Subdomain: spookycandy
   - DB Name: candydb
   - DB User: candyuser
   - DB Password: (secure password)
5. Follow CloudFormation stack creation
6. Access at: `https://spookycandy.{install-id}.nuon.run`

## 🔧 Configuration Options

### User Inputs

Defined in `inputs.toml`:

- **dns.domain**: Root domain (default: nuon.run)
- **dns.sub_domain**: Subdomain prefix (default: spookycandy)
- **database.db_name**: PostgreSQL database name (default: candydb)
- **database.db_user**: PostgreSQL username (default: candyuser)
- **database.db_password**: PostgreSQL password (default: changeme123)

### Resource Limits

Defined in `src/components/candy-app/values.yaml`:

```yaml
resources:
  limits:
    cpu: "1"
    memory: "512Mi"
  requests:
    cpu: "250m"
    memory: "256Mi"
```

### Replicas

Default: 2 replicas for high availability

## 🎯 Key Features

- ✅ **Self-contained database**: PostgreSQL runs in-cluster
- ✅ **Persistent storage**: 10Gi EBS volume for database
- ✅ **HTTPS by default**: ACM certificate with ALB
- ✅ **Auto-scaling ready**: Kubernetes HPA can be added
- ✅ **Multi-tenant**: Each install gets isolated namespace
- ✅ **Secure secrets**: Database credentials in Kubernetes secrets

## 🐛 Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n spookycandy
```

### View Logs
```bash
kubectl logs -n spookycandy deployment/spooky-candy-app
kubectl logs -n spookycandy statefulset/postgresql
```

### Database Connection Test
```bash
kubectl exec -it -n spookycandy deployment/spooky-candy-app -- env | grep DATABASE
```

### Check Ingress
```bash
kubectl get ingress -n spookycandy
```

## 📚 References

- [Nuon Documentation](https://docs.nuon.co)
- [AWS EKS Sandbox](https://github.com/nuonco/aws-eks-sandbox)
- [Original App Repo](https://github.com/matt-zach-s/spookycandy)

---

**Built with Nuon 🚀 | Deployed on AWS EKS ☁️ | Spooky Season 🎃**
