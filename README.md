Here’s the `README.md` in a properly formatted Markdown block that you can copy and paste directly into your `my-web-app/` directory. It’s ready to use with your current setup.

```markdown
# My Web App Helm Chart

This Helm chart deploys a web application with two independent services (`app1` and `app2`) on an AWS EKS cluster, using the AWS Load Balancer Controller to manage an Application Load Balancer (ALB). The Ingress routes traffic to different paths (`/app1` and `/app2`) under a single domain.

## Prerequisites

- **EKS Cluster**: Named `dev-cluster` in `us-east-1` (or your region), with at least 2 nodes (`t3.medium`).
- **Tools**: `helm`, `kubectl`, `aws-cli`, `eksctl` installed.
- **AWS Load Balancer Controller**: Installed in `kube-system` with `IngressClass: alb`.
- **Namespace**: `dev` created (`kubectl create namespace dev`).
- **ACM Certificate**: For `dev.myapp.com` (replace with your domain and ARN).
- **IAM Role**: Attached to the controller’s service account (`aws-load-balancer-controller`).

## Chart Structure

```
my-web-app/
├── Chart.yaml              # Chart metadata
├── values-dev.yaml         # Dev environment values
├── templates/
│   ├── deployment-app1.yaml  # Deployment for app1
│   ├── deployment-app2.yaml  # Deployment for app2
│   ├── service-app1.yaml     # Service for app1
│   ├── service-app2.yaml     # Service for app2
│   ├── ingress.yaml          # Ingress with path-based routing
```

## Features

- **Two Deployments**: `app1` and `app2`, each running Nginx.
- **Two Services**: Expose `app1` and `app2` within the cluster.
- **Ingress**: Routes `dev.myapp.com/app1` to `app1` and `dev.myapp.com/app2` to `app2` via ALB.

## Installation

### 1. Clone or Prepare the Chart
Ensure the chart files are in a `my-web-app` directory.

### 2. Deploy to EKS
Deploy the chart to the `dev` namespace:
```bash
helm install my-web-dev ./my-web-app -f values-dev.yaml --namespace dev
```

### 3. Update (After Changes)
If you modify templates (e.g., `ingress.yaml`):
```bash
helm upgrade my-web-dev ./my-web-app -f values-dev.yaml --namespace dev
```

### 4. Rollback (If Needed)
Revert to the previous release:
```bash
helm rollback my-web-dev --namespace dev
```

## Configuration

### values-dev.yaml
```yaml
app1:
  replicas: 1
  image:
    repository: nginx
    tag: latest
  service:
    type: ClusterIP
    port: 80
  resources:
    requests:
      cpu: "100m"
      memory: "128Mi"
    limits:
      cpu: "200m"
      memory: "256Mi"
app2:
  replicas: 1
  image:
    repository: nginx
    tag: latest
  service:
    type: ClusterIP
    port: 80
  resources:
    requests:
      cpu: "100m"
      memory: "128Mi"
    limits:
      cpu: "200m"
      memory: "256Mi"
ingress:
  host: "dev.myapp.com"
  certificateArn: "arn:aws:acm:us-east-1:<your-account-id>:certificate/<dev-cert-id>"
```
- Update `<your-account-id>` and `<dev-cert-id>` with your AWS account ID and ACM certificate ARN.

## Verification

### Check Resources
```bash
kubectl get all -n dev
```
- **Pods**: `my-web-dev-app1-xxx` and `my-web-dev-app2-xxx` should be `1/1 Running`.
- **Services**: `my-web-dev-app1-service` and `my-web-dev-app2-service`.
- **Ingress**: `my-web-dev-web-ingress` with an ALB address.

### Test Paths
```bash
curl dev.myapp.com/app1
curl dev.myapp.com/app2
```
- Expected: Nginx welcome page for each.

### Ingress Details
```bash
kubectl describe ingress my-web-dev-web-ingress -n dev
```
- Look for rules: `/app1` → `app1-service`, `/app2` → `app2-service`.

## Troubleshooting

- **Pods Not Running**: Check logs (`kubectl logs <pod-name> -n dev`).
- **Ingress Not Working**: Verify ALB controller (`kubectl get pods -n kube-system | grep aws-load-balancer`) and `IngressClass` (`kubectl get ingressclass`).
- **Events**: `kubectl get events -n dev`.

## Uninstallation

Remove the release:
```bash
helm uninstall my-web-dev --namespace dev
```

## Notes

- **Scaling**: Adjust `replicas` in `values-dev.yaml` for each app.
- **Customization**: Modify `image.repository` or add health checks in `deployment-*.yaml`.
- **EKS Setup**: Ensure the cluster (`dev-cluster`) and controller are configured as per prior steps.
```

---

## **Instructions**
1. Copy this entire block.
2. Paste it into a file named `README.md` in your `my-web-app/` directory.
3. Replace `<your-account-id>` and `<dev-cert-id>` with your actual AWS account ID and ACM certificate ARN (e.g., from your AWS Console).
4. Save and commit to your repository.

This is formatted for easy copy-paste and includes all necessary details. Let me know if you want to add more (e.g., setup steps for the cluster) or adjust anything!