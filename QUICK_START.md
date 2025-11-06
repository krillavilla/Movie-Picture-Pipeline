# Quick Start - Complete Your Deployment

## ✅ What's Been Done

1. **Terraform & Tools Installed**
   - Terraform 1.3.9
   - kustomize v5.4.1
   - jq, kubectl, AWS CLI (already present)

2. **AWS Infrastructure Provisioned**
   - EKS Cluster: `cluster` (Kubernetes 1.30)
   - EKS Node: 1x t3.small (Ready)
   - ECR Repositories: `frontend` and `backend`
   - IAM User: `github-action-user` with EKS access
   - VPC & Networking configured

3. **Local Configuration**
   - kubectl configured for EKS cluster
   - github-action-user added to cluster auth (system:masters)
   - Cluster verified: `kubectl get nodes` shows 1 node Ready

4. **Documentation Created**
   - `DEPLOYMENT_GUIDE.md` - Complete deployment instructions
   - `WORKFLOW_RUBRIC_COMPLIANCE.md` - Rubric compliance details
   - `QUICK_START.md` (this file) - Quick reference

5. **Git Changes**
   - All files committed to `feature/ci-cd-workflows` branch
   - Ready to push (needs GitHub authentication)

---

## 🚀 Next Steps (You Need to Do These)

### Step 1: Authenticate with GitHub

You need to push your code to GitHub. Choose one method:

**Option A: Using Personal Access Token (Recommended)**
```bash
# Generate a token at: https://github.com/settings/tokens
# Select: repo (full control)

# Configure git to use token
git config credential.helper store

# Push (it will prompt for username and token)
git push origin feature/ci-cd-workflows
# Username: krillavilla
# Password: <paste-your-token>
```

**Option B: Using GitHub CLI**
```bash
# Install gh CLI if not present
# Then authenticate
gh auth login

# Push
git push origin feature/ci-cd-workflows
```

### Step 2: Merge to Main

```bash
# Create PR via GitHub web UI or:
gh pr create --title "Deploy Movie Picture Pipeline" --body "Initial deployment with CI/CD workflows"

# Or merge directly:
git checkout main
git merge feature/ci-cd-workflows
git push origin main
```

### Step 3: Create AWS Access Keys

1. Go to AWS Console → IAM → Users → `github-action-user`
2. Security credentials → Create access key
3. Select "Application running outside AWS"
4. **Save these securely:**
   - Access Key ID
   - Secret Access Key

### Step 4: Add GitHub Secrets

Go to: https://github.com/krillavilla/Movie-Picture-Pipeline/settings/secrets/actions

Add these secrets:

```
AWS_ACCESS_KEY_ID: <from step 3>
AWS_SECRET_ACCESS_KEY: <from step 3>
AWS_DEFAULT_REGION: us-east-1
AWS_ACCOUNT_ID: 647451364529
EKS_CLUSTER_NAME: cluster
REACT_APP_MOVIE_API_URL: http://backend.default.svc.cluster.local:80
```

### Step 5: Run CD Workflows

1. Go to: https://github.com/krillavilla/Movie-Picture-Pipeline/actions
2. Click "Backend Continuous Deployment"
3. Click "Run workflow" (on main branch)
4. Wait for it to complete
5. Repeat for "Frontend Continuous Deployment"

### Step 6: Verify Deployment

```bash
# Check deployments
kubectl get deployments,pods,services -n default

# Wait for services to get external IPs (LoadBalancers)
kubectl get services -w

# Test backend (once LoadBalancer has EXTERNAL-IP)
BACKEND_IP=$(kubectl get service backend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$BACKEND_IP/movies

# Test frontend (open in browser)
FRONTEND_IP=$(kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Frontend URL: http://$FRONTEND_IP"
```

### Step 7: Take Screenshots for Submission

Capture screenshots of:
1. GitHub Actions showing successful CI and CD runs
2. Backend API response (curl output with movie list)
3. Frontend UI showing movie list in browser
4. kubectl get all output showing running pods and services

### Step 8: Test Failure Scenarios (for rubric)

Create a test branch and intentionally break linting:

```bash
git checkout -b test/lint-failure
# Edit starter/frontend/src/App.js and add: const unused = 'test';
git add starter/frontend/src/App.js
git commit -m "test: intentional lint failure"
git push origin test/lint-failure
# Create PR and verify CI fails
```

Repeat for test failures and lint failures in backend.

### Step 9: Teardown (When Complete!)

**IMPORTANT:** Don't forget to destroy infrastructure to avoid charges!

```bash
# Delete Kubernetes resources
kubectl delete all --all -n default

# Wait for LoadBalancers to be deleted (check AWS Console)

# Destroy Terraform infrastructure
cd setup/terraform
export PATH="$HOME/.tfenv/bin:$PATH"
terraform destroy
# Type: yes
```

---

## 📋 Key Information

**AWS Account:** 647451364529
**AWS Region:** us-east-1
**EKS Cluster:** cluster
**Frontend ECR:** 647451364529.dkr.ecr.us-east-1.amazonaws.com/frontend
**Backend ECR:** 647451364529.dkr.ecr.us-east-1.amazonaws.com/backend

**GitHub Repository:** https://github.com/krillavilla/Movie-Picture-Pipeline
**Workflow Files:**
- `.github/workflows/frontend-ci.yaml`
- `.github/workflows/backend-ci.yaml`
- `.github/workflows/frontend-cd.yaml`
- `.github/workflows/backend-cd.yaml`

---

## 🔧 Troubleshooting

**Git push fails:**
- Need to authenticate (see Step 1 above)

**Workflow says "have_aws=false":**
- GitHub Secrets not set yet (see Step 4 above)

**Pods show ImagePullBackOff:**
- CD workflow needs to run first to push images to ECR
- Check GitHub Actions logs for ECR push step

**LoadBalancer stuck in <pending>:**
- Wait 2-3 minutes for AWS to provision ELB
- Check: `kubectl describe service frontend`

**Can't access kubectl:**
```bash
aws eks update-kubeconfig --name cluster --region us-east-1
kubectl get nodes
```

---

## 📚 Additional Resources

- Full deployment guide: `DEPLOYMENT_GUIDE.md`
- Rubric compliance: `WORKFLOW_RUBRIC_COMPLIANCE.md`
- Terraform outputs: `cd setup/terraform && terraform output`
- GitHub Actions: https://github.com/krillavilla/Movie-Picture-Pipeline/actions

---

## ✨ Success Criteria

Your project is complete when:

✅ All four workflow files exist and pass syntax validation
✅ CI workflows run on pull requests and pass
✅ CD workflows run on push to main and deploy to EKS
✅ Backend API returns movie list (accessible via LoadBalancer)
✅ Frontend UI displays movie list (accessible via LoadBalancer)
✅ Failure scenarios tested (lint/test failures cause CI to fail)
✅ Screenshots captured for submission
✅ Infrastructure teardown completed

---

## 💰 Cost Warning

**Monthly cost if left running:** ~$93/month
- EKS Control Plane: $73
- t3.small instance: $15
- Data transfer: ~$5

**Always run `terraform destroy` when done!**
