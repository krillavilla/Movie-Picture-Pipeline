# Movie Picture Pipeline - Deployment Guide

## Infrastructure Successfully Provisioned ✅

### AWS Resources Created

**EKS Cluster:**
- Cluster Name: `cluster`
- Kubernetes Version: `1.30`
- Region: `us-east-1`
- Node Group: `udacity` (1x t3.small instance)
- Status: ✅ Ready

**ECR Repositories:**
- Frontend: `647451364529.dkr.ecr.us-east-1.amazonaws.com/frontend`
- Backend: `647451364529.dkr.ecr.us-east-1.amazonaws.com/backend`

**IAM:**
- GitHub Action User: `github-action-user`
- ARN: `arn:aws:iam::647451364529:user/github-action-user`
- Permissions: ECR, EKS, EC2, IAM (GetUser)
- EKS Access: ✅ Configured (system:masters)

**Networking:**
- VPC: `10.0.0.0/16`
- Public Subnet: `10.0.1.0/24` (us-east-1a)
- Private Subnet: `10.0.2.0/24` (us-east-1b)
- Internet Gateway: ✅ Configured

---

## Next Steps to Complete Deployment

### 1. Create AWS Access Keys for GitHub Actions

You need to create access keys for the `github-action-user` IAM user:

1. Go to AWS Console → IAM → Users → `github-action-user`
2. Click "Security credentials" tab
3. Click "Create access key"
4. Select "Application running outside AWS"
5. Click "Next" → "Create access key"
6. **SAVE THESE CREDENTIALS SECURELY:**
   - Access Key ID
   - Secret Access Key

### 2. Add GitHub Secrets

Go to your GitHub repository settings and add these secrets:

**Required Secrets:**
```
AWS_ACCESS_KEY_ID: <from step 1>
AWS_SECRET_ACCESS_KEY: <from step 1>
AWS_DEFAULT_REGION: us-east-1
AWS_ACCOUNT_ID: 647451364529
EKS_CLUSTER_NAME: cluster
REACT_APP_MOVIE_API_URL: http://localhost:5000
```

**Note:** The REACT_APP_MOVIE_API_URL will be updated later once we know the backend service URL in Kubernetes.

### 3. Check Kubernetes Service Names

Your workflows reference these service names. Let's verify they match:

```bash
# Check backend service name
cat starter/backend/k8s/service.yaml | grep "name:"

# Check frontend service name
cat starter/frontend/k8s/service.yaml | grep "name:"
```

### 4. Commit and Push Your Code

```bash
cd /home/krillavilla/Documents/Cloud\ DevOps\ Udacity/Movie-Picture-Pipeline

# Check what needs to be committed
git status

# Add all files
git add -A

# Commit
git commit -m "chore: add workflows and update terraform config for EKS 1.30"

# Push to main
git push origin main
```

### 5. Trigger CD Workflows Manually

Once secrets are added:

1. Go to GitHub → Actions
2. Select "Frontend Continuous Deployment"
3. Click "Run workflow" → "Run workflow"
4. Repeat for "Backend Continuous Deployment"

### 6. Verify Deployment

```bash
# Check pods and services
kubectl get pods,svc -n default

# Check deployments
kubectl get deployments -n default

# View logs
kubectl logs deployment/frontend -n default
kubectl logs deployment/backend -n default
```

### 7. Test the Applications

**Backend API:**
```bash
# Port forward
kubectl port-forward service/backend 5000:5000

# In another terminal, test the API
curl http://localhost:5000/movies
```

**Frontend UI:**
```bash
# Port forward
kubectl port-forward service/frontend 3000:3000

# Open browser to http://localhost:3000
```

### 8. Update REACT_APP_MOVIE_API_URL Secret

Once you know the backend service name, update the secret:

If backend service is `backend-service`:
```
REACT_APP_MOVIE_API_URL: http://backend-service.default.svc.cluster.local:5000
```

Then re-run the Frontend CD workflow.

---

## Current Workflow Files Status

Your repository already has these workflow files:

✅ `.github/workflows/frontend-ci.yaml` - Frontend Continuous Integration
✅ `.github/workflows/backend-ci.yaml` - Backend Continuous Integration  
✅ `.github/workflows/frontend-cd.yaml` - Frontend Continuous Deployment
✅ `.github/workflows/backend-cd.yaml` - Backend Continuous Deployment

**Note:** These workflows use a "preflight" check that allows them to pass even without AWS secrets configured. Once you add the secrets, they'll automatically push to ECR and deploy to EKS.

---

## Workflow Behavior

### CI Workflows (Pull Requests)
- Triggered on PRs to `main` branch
- Path filters ensure they only run when relevant code changes
- Run lint, test, and build in sequence
- **No AWS access required** (local Docker build only)

### CD Workflows (Push to Main)
- Triggered on push to `main` branch
- Path filters ensure efficiency
- Run lint, test in parallel
- Build Docker images with proper tagging
- **Push to ECR** (requires AWS secrets)
- **Deploy to EKS** using kustomize (requires AWS secrets)

---

## Testing Failure Scenarios

To verify workflows fail correctly on bad code:

**Frontend Lint Failure:**
```bash
# Temporarily set FAIL_LINT
FAIL_LINT=true npm run lint
```

**Frontend Test Failure:**
```bash
# Temporarily set FAIL_TEST
FAIL_TEST=true CI=true npm test
```

**Backend Test Failure:**
```bash
# In backend directory
FAIL_TEST=true pipenv run test
```

---

## Teardown Instructions (IMPORTANT!)

When you're done with the project:

### 1. Delete Kubernetes Resources
```bash
kubectl delete all --all -n default
```

### 2. Destroy Terraform Infrastructure
```bash
cd setup/terraform
export PATH="$HOME/.tfenv/bin:$PATH"
terraform destroy
```

This will delete:
- EKS cluster and node group
- ECR repositories and images
- VPC and networking
- IAM users and roles
- All other AWS resources

**Estimated Monthly Cost if Left Running:**
- EKS Control Plane: ~$73/month
- EC2 Instance (t3.small): ~$15/month
- Data transfer and other services: ~$5/month
- **Total: ~$93/month**

---

## Troubleshooting

### Workflow fails with "Error: error getting credentials"
- Verify GitHub secrets are set correctly
- Check that Access Key ID and Secret Access Key are valid

### Pod fails to pull image from ECR
- Verify ECR login step succeeded in workflow
- Check that image was pushed successfully
- Verify node group has ECR read permissions (should be automatic)

### kubectl commands fail locally
- Run: `aws eks update-kubeconfig --name cluster --region us-east-1`
- Verify: `kubectl get nodes`

### Deployment shows ImagePullBackOff
- Check that image exists in ECR
- Verify image tag matches what's in deployment
- Check ECR repository URI is correct in secrets

---

## Summary

**Completed:**
✅ Terraform tooling installed
✅ AWS infrastructure provisioned
✅ EKS cluster created (Kubernetes 1.30)
✅ ECR repositories created
✅ kubectl access configured
✅ github-action-user added to cluster auth
✅ Workflow files exist and are rubric-compliant

**TODO:**
1. Create AWS access keys for github-action-user
2. Add GitHub Secrets
3. Commit and push code to GitHub
4. Run CD workflows to deploy applications
5. Test deployed applications
6. Document results for submission
7. Teardown infrastructure when complete

---

## Quick Reference Commands

```bash
# View Terraform outputs
cd setup/terraform && terraform output

# Update kubeconfig
aws eks update-kubeconfig --name cluster --region us-east-1

# View cluster resources
kubectl get all -n default

# View workflow logs (in GitHub Actions UI)
# https://github.com/krillavilla/Movie-Picture-Pipeline/actions

# Tear down (when done)
kubectl delete all --all -n default
cd setup/terraform && terraform destroy
```
