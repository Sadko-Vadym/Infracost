# Infracost Integration for GKE Terraform

This repository contains Terraform configuration for deploying a Google Kubernetes Engine (GKE) cluster with Infracost integration for cost estimation and monitoring.

## What is Infracost?

Infracost is a tool for estimating cloud infrastructure costs. It helps developers and DevOps teams understand and manage costs associated with cloud infrastructure changes before they are deployed.

## Setup Instructions

### Prerequisites

- GitHub account
- Google Cloud Platform (GCP) account with billing enabled
- Infracost account (free tier available at https://www.infracost.io/)

### 1. Fork and Clone Repository

This repository is already set up with the necessary Terraform configuration from the original [tf-google-gke-cluster](https://github.com/den-vasyliev/tf-google-gke-cluster) repository.

### 2. Configure GitHub Secrets

You need to add the following secrets to your GitHub repository:

#### INFRACOST_API_KEY

1. Register at https://www.infracost.io/ or run locally:
   ```bash
   infracost auth login
   ```

2. Get your API key:
   ```bash
   infracost configure get api_key
   ```

3. Add the API key to GitHub:
   - Go to your repository Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `INFRACOST_API_KEY`
   - Value: Your Infracost API key
   - Click "Add secret"

### 3. Enable Branch Protection (Optional but Recommended)

1. Go to repository Settings → Branches
2. Add branch protection rule for `main`:
   - Branch name pattern: `main`
   - Check "Require a pull request before merging"
   - Check "Require status checks to pass before merging"
   - Search and select "Infracost" status check (after first PR)

### 4. Test the Integration

Create a test pull request with infrastructure changes:

1. Create a new branch:
   ```bash
   git checkout -b test/increase-nodes
   ```

2. Modify the infrastructure (e.g., in `variables.tf`):
   ```terraform
   variable "GKE_NUM_NODES" {
     type        = number
     default     = 3  # Changed from 2 to 3
     description = "GKE nodes number"
   }
   ```

3. Commit and push:
   ```bash
   git add variables.tf
   git commit -m "Increase node count to 3"
   git push origin test/increase-nodes
   ```

4. Create a Pull Request on GitHub
5. Wait for Infracost to post a cost estimate comment

## How It Works

The GitHub Actions workflow (`.github/workflows/infracost.yml`) automatically:

1. Runs on every Pull Request that modifies Terraform files
2. Generates cost estimates for both the base branch and PR branch
3. Calculates the cost difference
4. Posts a detailed comment on the PR with:
   - Cost breakdown by resource
   - Monthly cost estimates
   - Cost difference compared to base branch

## Terraform Configuration

### Main Resources

- **GKE Cluster**: Google Kubernetes Engine cluster with Workload Identity
- **Node Pool**: Custom node pool with configurable machine type and node count
- **Authentication**: Automatic kubeconfig generation for cluster access

### Variables

| Name | Description | Default | Type |
|------|-------------|---------|------|
| GOOGLE_PROJECT | GCP project name | - | string |
| GOOGLE_REGION | GCP region | us-central1-c | string |
| GKE_CLUSTER_NAME | GKE cluster name | main | string |
| GKE_POOL_NAME | Node pool name | main | string |
| GKE_NUM_NODES | Number of nodes | 2 | number |
| GKE_MACHINE_TYPE | Machine type | g1-small | string |

## Local Development

### Initialize Terraform

```bash
terraform init
```

### Validate Configuration

```bash
terraform validate
```

### Plan Infrastructure

```bash
terraform plan -var="GOOGLE_PROJECT=your-project-id"
```

### Check Costs Locally

If you have Infracost installed locally:

```bash
# Install Infracost
curl -fsSL https://raw.githubusercontent.com/infracost/infracost/master/scripts/install.sh | sh

# Authenticate
infracost auth login

# Generate cost estimate
infracost breakdown --path .
```

## Cost Optimization Tips

1. **Machine Type**: Use `e2-small` or `e2-medium` instead of `g1-small` for better price/performance
2. **Node Count**: Start with minimum required nodes and use autoscaling
3. **Regional vs Zonal**: Zonal clusters are cheaper than regional clusters
4. **Preemptible Nodes**: Consider using spot/preemptible instances for non-critical workloads

## References

- [Infracost Documentation](https://www.infracost.io/docs/)
- [Infracost GitHub Actions](https://github.com/infracost/actions)
- [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator)
- [GKE Pricing](https://cloud.google.com/kubernetes-engine/pricing)

## License

This project is licensed under the MIT License.

