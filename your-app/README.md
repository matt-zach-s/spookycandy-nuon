<center>
<h1>🎃 Spooky Candy Manager</h1>
A Halloween-themed candy recipe management application deployed on EKS with PostgreSQL.

Nuon Install Id: {{ .nuon.install.id }}

AWS Region: {{ .nuon.install_stack.outputs.region }}

</center>

## Access Your Application

Your Spooky Candy Manager is now live! Access it at:

**[https://{{.nuon.inputs.inputs.sub_domain}}.{{.nuon.install.sandbox.outputs.nuon_dns.public_domain.name}}](https://{{.nuon.inputs.inputs.sub_domain}}.{{.nuon.install.sandbox.outputs.nuon_dns.public_domain.name}})**

Or use curl:

```bash
curl https://{{.nuon.inputs.inputs.sub_domain}}.{{.nuon.install.sandbox.outputs.nuon_dns.public_domain.name}}
```

## What's Deployed

- **Next.js Application**: React 19 + TypeScript candy recipe manager
- **PostgreSQL Database**: Running in Kubernetes with persistent storage
- **Application Load Balancer**: HTTPS access with AWS Certificate Manager
- **Auto-scaling**: Kubernetes deployment with 2 replicas

## Database Connection

Your PostgreSQL database is deployed within the Kubernetes cluster:
- **Database**: {{.nuon.inputs.inputs.db_name}}
- **Username**: {{.nuon.inputs.inputs.db_user}}
- **Internal DNS**: postgresql.spookycandy.svc.cluster.local:5432

## Features

- 🎃 Browse Halloween candy recipes
- ✨ Add new spooky candy creations
- 📊 View prep time and difficulty levels
- 🔍 Search and filter by category
- 📱 Responsive mobile-friendly design

## Architecture

```
                    ┌─────────────────┐
                    │   AWS Route53   │
                    │   (DNS)         │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   ALB + ACM     │
                    │   (HTTPS)       │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │     EKS Cluster             │
              │  ┌────────────────────┐     │
              │  │  Candy App Pods    │     │
              │  │  (Next.js)         │     │
              │  └──────┬─────────────┘     │
              │         │                    │
              │  ┌──────▼─────────────┐     │
              │  │  PostgreSQL        │     │
              │  │  (Persistent Vol)  │     │
              │  └────────────────────┘     │
              └─────────────────────────────┘
```

### Full Install State

<details>
<summary>Full Install State</summary>
<pre>{{ toPrettyJson .nuon }}</pre>
</details>

---

**Happy Halloween! 🎃 👻 🍬**
