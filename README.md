# argorollout-project
#  **Argo CD** and **Argo Rollouts**  Install step-by-step guide** with IAM permissions and RBAC configurations 

---

## ✅ Pre-requisites

* AWS CLI configured (`aws configure`)
* `kubectl`, `helm`, `eksctl` installed
* EKS cluster already created and `kubectl` configured

---

## 📦 Step 1: Install Argo CD


# 1. Create namespace
  kubectl create namespace argocd

# 2. Install Argo CD using Helm (latest way)
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --set server.service.type=LoadBalancer \  
  --set dex.enabled=false


# ➡️ `argocd-server` provide  LoadBalancer IP  (AWS ALB/ELB)

### 🔐 Get Admin Password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

---

## 🌐 Access Argo CD Web UI

```bash
kubectl get svc -n argocd argocd-server
```

# Open the `EXTERNAL-IP` in browser
# Login using:

* Username: `admin`
* Password: (from above command)

---

## 📘 Step 2: Install Argo Rollouts

```bash
# Create namespace (optional)
 kubectl create namespace argo-rollouts

# Install Argo Rollouts controller
  kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

✅ This will deploy the Rollouts controller (CRDs + controller deployment)

---

## 🔍 Step 3: Install Argo Rollouts Kubectl Plugin (Optional but recommended)

```bash
curl -sLO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```

# ✅ Now you can use commands like:

```bash
 kubectl argo rollouts get rollout <name>
 kubectl argo rollouts dashboard
```

---

## 🛡️ Step 4: Permissions (IAM Roles for Service Account - IRSA)

> This is only needed if:
>
> * Argo CD needs to access AWS services (like S3, DynamoDB)
> * You're using ALB ingress (for traffic shifting)

### Use `eksctl` to enable IAM roles:

```bash
eksctl utils associate-iam-oidc-provider --cluster <your-cluster> --approve

eksctl create iamserviceaccount \
  --name argocd-role \
  --namespace argocd \
  --cluster <your-cluster> \
  --attach-policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
  --approve \
  --override-existing-serviceaccounts
```

Then, update Argo CD's `argocd-server` and `argocd-repo-server` deployments to use this service account (if you want AWS resource access via IAM).

---

## ✅ Verification

* Argo CD UI: https\://<elb-dns>
* Argo Rollouts:

  ```bash
  kubectl argo rollouts version
  ```

---

## 🎯 Optional: Argo Rollouts Dashboard

```bash
kubectl argo rollouts dashboard
```

     This will open a local dashboard UI for Argo Rollouts via port-forward.

---

## Pause duration can be specified with an optional time unit suffix. Valid time units are "s", "m", "h". Defaults to "s" if not specified.

    spec:
      strategy:
        canary:
          steps:
            - pause: { duration: 10 } # 10 seconds
            - pause: { duration: 10s } # 10 seconds
            - pause: { duration: 10m } # 10 minutes
            - pause: { duration: 10h } # 10 hours
            - pause: {} # pause indefinitely
    If no duration is specified for a pause step, the rollout will be paused indefinitely. To unpause, use the argo kubectl plugin promote command.

# promote to the next step
    kubectl argo rollouts promote <rollout>
