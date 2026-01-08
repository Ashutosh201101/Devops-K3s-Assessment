# Devops-k3s-Assessment
---
## Part 1: VM & Cluster Setup
 - Install Docker Engine
 - Ensure Docker is running
 - Your user should be able to run Docker commands
 
  Validation:
  - docker version
  - docker ps
 ```
   sudo apt update
   sudo apt install docker.io -y

   sudo systemctl start docker
   sudo systemctl enable docker

   sudo usermod -aG docker $USER
   newgrp docker
 ```
 ```
    docker version
    docker ps
```
![Docker version,ps](images/1.png)


- Install single-node k3s
- kubectl must work for your user
- Cluster must be operational
 
  Validation:
  - kubectl get nodes
  - kubectl get pods -A

```
  curl -sfL https://get.k3s.io | sh -
  sudo systemctl status k3s

  mkdir -p $HOME/.kube
  sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  export KUBECONFIG=$HOME/.kube/config
```
```
  kubectl get nodes
  kubectl get pods -A
```
![Get-nodes](images/2.png)
![Get all pods](images/4.png)

### Q-Explain briefly why this setup is suitable (or not) for an early-stage startup.
   - For an early-stage startup, this setup works well because it keeps things simple and             affordable. Using a single-node k3s cluster lets the team deploy applications quickly            without spending too much time or money on infrastructure management. It’s lightweight,          easy to maintain, and perfect for MVPs or low-traffic production environments where speed        matters more than complexity.   
     That said, since everything runs on one node, there’s no backup if the server goes down. So      while this setup is great in the beginning, it will need to be upgraded to a multi-node or       managed Kubernetes solution as the startup grows.

## Part 2: Application Deployment (Helm)
   helm repo add open-webui https://helm.openwebui.com/
   helm repo update
  
  Deploy Open WebUI with:
- Release name: webui
- Namespace: openwebui
- Helm v3+
- Service type: ClusterIP
- Perform a dry-run before install
 
Validation:
- kubectl get all -n openwebui

```
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   helm version
    kubectl create namespace openwebui
```
   ![Helm install](images/5.png)

   helm repo add open-webui https://helm.openwebui.com/
   helm repo update
   ![](images/6.png)
   ![](images/7.png)
   ![](images/8.png)
   ![](images/9.png)
   ![](images/10.png)
   ![](images/11.png)
   ![](images/12.png)
   ![](images/13.png)
   ```
     helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP \
  --dry-run
   ```
   ![](images/14.png)
   ```
     helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP \
  --dry-run
   ```
   ![](images/15.png)
   ![](images/16.png)

```
   kubectl get all -n openwebui
```
   ![](images/17.png)

   ## Part 3: Authentication Configuration (OIDC)
   Create values-oidc.yaml:
   ```
     oidc: clientId: "test" clientSecret: "" issuer: "https://<YOUR_DOMAIN_HERE>/auth/realms/hyperplane/.well-known/openid-configuration"
 scopes:
  - openid
  - profile
  - email
```
Apply using:
```
  helm upgrade webui open-webui/open-webui --namespace openwebui --values values-oidc.yaml
```
![](images/18.png)

```
  kubectl get pods -n openwebui
```
![](images/20.png)
# Completely stuck in this ,,,

# Ownership Questions
Answer clearly and concretely: 
1. Production Readiness.
  Top 5 risks before going live
      - Single-node failure
      - Authentication dependency risk
      - No proper ingress & TLS
      - Lack of monitoring and alerting
      - Secrets management
   
  First 2 things you would fix.
     - Add High Availability & Basic Resilience
     - Secure and Stabilize Authentication
2. Failure Scenario
  ## Traffic spikes 10x and node goes down at 2 AM
 ### What breaks first?  
     -  The single node gets overloaded due to high CPU and memory usage.
     -  Since it’s a single-node k3s cluster, the entire cluster goes down.
     -  The application becomes unavailable.
     -  Users are unable to access or log in.

 ### How do you recover?
     -  Restart the server or node from the cloud provider dashboard.
     -  Check node and pod status after recovery.
     -  Let k3s recreate the pods automatically.
     -  Restart the application deployment if required.
  
  ### What do you change the next day?
    -  Move from single-node to multi-node k3s setup.
    -  Add monitoring and alerting to detect issues early.
    -  Add traffic handling like autoscaling or rate limiting.
    -  Improve overall reliability before scaling features.
  

 3. Security & Secrets
    ## How do you manage secrets?
        - I keep passwords, API keys, and other secrets in Kubernetes Secrets or tools like               Vault / AWS Secrets Manager.
        - Pods get secrets through environment variables or files.
        - Only the apps or people who need the secrets can access them.
    
    ## What must never be in Git?
        - Passwords, API keys, OIDC secrets, database credentials.
        - Private certificates or keys.
        - Basically, anything that can give access to production.

    ## What should be rotated?
        - All passwords, API keys, OIDC client secrets.
        - TLS certificates.
        - Any keys used in CI/CD or cloud access.
        - Rotation makes sure if a secret leaks, it won’t be usable for long.

 4. Backups & Recovery
    ## What data must be backed up
        - The database (all customer, order, and product data)
        - Configs and Helm values
        - Any important files or uploads
        - Basically, anything that we can’t easily rebuild
    
   ## Backup frequency
      - Database: daily or hourly if traffic is high
      - Configs / files: after any change
      - The goal is to always have a recent copy

  ## How to test recovery
     - Restore the backup in a test environment
     - Check that the app, database, and configs all work
     - Make sure nothing is broken and data is intact
     
  Cost Ownership (Hetzner)
  
  ## How I keep infra costs low
    - I start with minimum resources and scale only when required
    - I use k3s because it is lightweight and cheap to run
    - I avoid running unnecessary services
    - I regularly clean up unused servers, volumes, and resources

  ## What I avoid early
    - Big clusters without enough traffic
    - Costly managed services in the early stage
    - Over-allocating CPU, RAM, or storage
    - Complex setups that increase running cost

  ## When I move away from k3s
    - When the product grows and needs high availability
    - When traffic becomes high and stability is critical
    - When managing things manually becomes difficult
    
     
  
