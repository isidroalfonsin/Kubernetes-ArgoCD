# Kubernetes ArgoCD Project on Civo

This project documents the setup of a CI/CD pipeline using ArgoCD on a Civo Kubernetes cluster.

## Prerequisites
- Civo Account
- `kubectl` installed locally

## 1. Setup Access

1. **Configure Kubeconfig**:
   - Save your kubeconfig as `kubeconfig.yaml` in the project root.
   - Export the variable for the current session:
     ```bash
     export KUBECONFIG=$(pwd)/kubeconfig.yaml
     ```
2. **Verify Connection**:
   ```bash
   kubectl get nodes
   ```

## 2. ArgoCD Installation

**Install ArgoCD Manifests:**
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

> **Note on Resources:** We upgraded the node pool to a standard size (e.g., Medium/Small) because smaller instances (Extra Small) caused memory pressure and pod evictions during installation.

**Expose UI Service (LoadBalancer):**
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

**Retrieve Admin Password:**
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

## 3. Deployed Applications

We have configured the following applications using ArgoCD:

### Guestbook (Official Demo)
- **Repository:** `https://github.com/argoproj/argocd-example-apps.git`
- **Path:** `guestbook`
- **Access:** Exposed via LoadBalancer.


### Podinfo (Professional Demo)
- **Repository:** `https://github.com/stefanprodan/podinfo.git`
- **Path:** `kustomize`
- **Configuration:** `selfHeal` disabled to allow manual LoadBalancer patching.
- **Access:** Exposed via LoadBalancer.

## 4. Custom CI/CD Pipeline (GitHub Actions)

We implemented a full Level 4 CI/CD pipeline for the custom application `my-app`.

**Workflow:**
1. **Source Code**: Located in `./my-app` (HTML + Dockerfile).
2. **GitHub Action**: Trigged on push to `main`.
   - Builds Docker image.
   - Pushes to DockerHub (`isidroalfonsin/my-app`).
   - Updates `my-app/k8s/deployment.yaml` with the new image SHA.
3. **ArgoCD**: Detects the change in the manifest and syncs the cluster.

**Required Secrets (GitHub Repo):**
- `DOCKER_USERNAME`: Your DockerHub username.
- `DOCKER_PASSWORD`: Your DockerHub Access Token (Read/Write/Delete).

---
*Created as part of the Civo Kubernetes request.*

All the screenshots are in the `screenshots` folder.

The cluster has been deleted for cost saving reasons.