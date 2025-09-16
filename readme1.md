

###  Step 1: Login to Azure

    az login
    
### Create Resource Group


     az group create --name myResourceGroup --location eastus



### your subscription has registered the resource provider
     az provider register --namespace Microsoft.OperationalInsights
  
  az provider show --namespace Microsoft.OperationalInsights --query "registrationState"

 
### If you’re also going to use Container Insights and Log Analytics, also register these (just once per subscription):

     az provider register --namespace Microsoft.Monitoring
  
    az provider register --namespace Microsoft.ContainerService

###  Before using AKS, register all the commonly required providers in one go:

    az provider register --namespace Microsoft.ContainerService
    az provider register --namespace Microsoft.OperationalInsights
    az provider register --namespace Microsoft.Monitor
    az provider register --namespace Microsoft.Network
    az provider register --namespace Microsoft.Compute



### Create AKS Cluster

     az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 2 --node-vm-size Standard_a2_v2 --generate-ssh-keys

   or 
### This creates a **2-node AKS cluster** with monitoring enabled.

     az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 2 --node-vm-size Standard_DS2_v2 --enable-addons monitoring --generate-ssh-keys








### Get Credentials for `kubectl`


     az aks get-credentials --resource-group myResourceGroup --name myAKSCluster



### Verify Cluster

    kubectl get nodes

    argocd cluster list

## argocd PASSWORD 

    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }

## ARGOCD LOGIN

    argocd login 20.124.143.23 --username admin

# ADD CLuster in argocd 
    kubectl config get-contexts

    argocd cluster add myAKSCluster

    argocd cluster list

    
### access via Browser 

    kubectl patch svc argocd-server -n argocd --type=merge -p '{\"spec\":{\"type\":\"LoadBalancer\"}}'

    kubectl get svc


   











###########install argocd 

 ######Argo CD Namespace Create 
kubectl create namespace argocd

3### rgo CD Install 
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


यह command Argo CD के सभी components install करेगा:

argocd-server

argocd-repo-server

argocd-application-controller

argocd-dex-server



 kubectl patch svc argocd-server -n argocd --type=merge -p '{\"spec\":{\"type\":\"LoadBalancer\"}}'
 
 kubectl get svc argocd-server -n argocd
 
 
 
 Install ARGO ROLLOUTS 
 
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml



 ######### 🔹 Argo Rollouts Dashboard Start
 
 kubectl argo rollouts dashboard

use in Browser http://localhost:3100

 
---

##  Main Commands in `kubectl-argo-rollouts`

### 1. `kubectl argo rollouts get rollout <name>`

 Shows the current status of a specific rollout (similar to `kubectl get deployment` but with rollout-specific details).

```ps
kubectl argo rollouts get rollout my-app -n default
```

 Example output shows replicas, steps completed, canary percentage, etc.

---

### 2. `kubectl argo rollouts list rollouts -n <namespace>`

 Lists all rollouts in a namespace.

```ps
kubectl argo rollouts list rollouts -n default
```

 Useful to see all rollouts and their status (Paused, Progressing, Completed).

---



---

### 4. `kubectl argo rollouts get rollout <name> --watch`

 Continuously watches the rollout status in real time.

```ps
kubectl argo rollouts get rollout my-app -n default --watch
```

 Helps monitor progress step by step (e.g., 20% traffic → pause → 50%).

---

### 5. `kubectl argo rollouts dashboard`

 Opens an interactive **local dashboard** in your browser.

```ps
kubectl argo rollouts dashboard
```

 You can visualize traffic shifting, replicas, and experiment results.

---

### 6. `kubectl argo rollouts promote <name>`

 Moves the rollout to the next step or fully promotes the new version.

```ps
kubectl argo rollouts promote my-app -n default
```

 Example: Canary was paused at 50% traffic → this command makes it go to 100%.

---

### 7. `kubectl argo rollouts restart <name>`

 Restarts the rollout (just like `kubectl rollout restart deployment`).

```ps
kubectl argo rollouts restart my-app -n default
```

 Good for applying new config changes without updating the image.

---

### 8. `kubectl argo rollouts terminate <name>`

 Immediately **stops a rollout in progress**.  only use  Blue-Green strategy 

```ps
kubectl argo rollouts terminate my-app -n default
```

 Useful if something goes wrong and you want to cancel mid-deployment.

---

### 9. `kubectl argo rollouts pause <name>`

 Manually pauses the rollout at its current step.

```ps
kubectl argo rollouts pause my-app -n default
```

 Example: Pause at 20% traffic to monitor metrics before continuing.

---

### 10. `kubectl argo rollouts resume <name>`

 Resumes a previously paused rollout.

```ps
kubectl argo rollouts resume my-app -n default
or 
kubectl-argo-rollouts resume hotstar-rollout -n default
```

 Rollout continues from where it was paused.

---

### 11. `kubectl argo rollouts undo <name> --to-revision=<num>`

 Rollback to a previous revision of the rollout.

```ps
kubectl argo rollouts undo my-app -n default --to-revision=2
```

 Useful if the new version is bad → instantly goes back to old stable.

---

### 12. `kubectl argo rollouts get experiments`

 Shows rollout **experiments** (used for A/B testing new versions).

```ps
kubectl argo rollouts get experiments -n default
```

---

### 13. `kubectl argo rollouts get analysisruns`

 Shows history of **analysis runs** (used with Prometheus, Datadog, etc., to decide promotion).

```ps
kubectl argo rollouts get analysisruns -n default
```

---

### 14. `kubectl argo rollouts version`

 Displays the installed CLI version.

```ps
kubectl argo rollouts version
```

---

With INgress 
######Install Helm with winget:

    winget install Helm.Helm
 
##  add repo 

    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update


## INSTALL  nginx ingress controller 

    helm install ingress-nginx ingress-nginx/ingress-nginx `
      --namespace ingress-nginx --create-namespace `
      --set controller.replicaCount=2 `
      --set controller.nodeSelector."kubernetes\.io/os"=linux `
      --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux `
      --set controller.service.type=LoadBalancer
      
  
  
    kubectl get svc -n ingress-nginx




### delete Cluster 

     
    
    
    az provider unregister --namespace Microsoft.ContainerService
    az provider unregister --namespace Microsoft.OperationalInsights
    az provider unregister --namespace Microsoft.Monitor
    az provider unregister --namespace Microsoft.Network
    az provider unregister --namespace Microsoft.Compute 

### you want to unregister a resource provider (namespace) in Azure.

    az provider unregister --namespace Microsoft.ContainerService

    az aks delete --resource-group myResourceGroup --name myAKSCluster --yes


# rollout file :
        apiVersion: argoproj.io/v1alpha1   # Argo Rollouts API version
        kind: Rollout                      # Resource type: Rollout
        metadata:
          name: example-rollout            # Rollout name (must be unique)
          namespace: default               # Namespace where rollout will be deployed
        
        spec:
          replicas: 10                     # Total desired pods (stable + canary)
          revisionHistoryLimit: 3          # Keep last 3 revisions for rollback
          minReadySeconds: 30              # Pod must be ready for 30s before next step
        
          selector:                        # Pod selection criteria
            matchLabels:
              app: nginx                   # Only pods with app=nginx label
        
          template:                        # Pod template (like Deployment)
            metadata:
              labels:
                app: nginx                 # Pod will get this label
            spec:
              containers:
                - name: nginx              # Container name
                  image: nginx:1.15.4      # Container image
                  ports:
                    - containerPort: 80    # App listens inside pod on port 80
        
          strategy:
            canary:                        # Use Canary deployment strategy
        
              canaryService: hotstar-canary    # Service for Canary pods
              stableService: hotstar-stable    # Service for Stable pods
        
              trafficRouting:                  # Traffic split rules
                nginx:                         # Using NGINX ingress controller
                  stableIngress: hotstar-ingress  # Stable ingress name
        
              maxSurge: '25%'              # Allow up to 25% extra pods during update
              maxUnavailable: 0            # Ensure no pods are unavailable during rollout
        
              steps:                       # Canary rollout steps
                - setWeight: 10            # Send 10% traffic to Canary
                - pause:
                    duration: 30s          # Auto-pause for 30 seconds (then continue)
                - setWeight: 20            # Send 20% traffic to Canary
                - pause: {}                # Indefinite pause (manual approval required)
        
                # -------------------------------
                # Additional CanaryScale examples
                # -------------------------------
        
                # Explicitly scale Canary pods to 3 (fixed number)
                - setCanaryScale:
                    replicas: 3
        
                # Scale Canary pods to 25% of total spec.replicas (10 * 25% = 2 or 3 pods)
                - setCanaryScale:
                    weight: 25
        
                # Return to default behavior: Canary replicas = traffic weight %
                - setCanaryScale:
                    matchTrafficWeight: true




 
