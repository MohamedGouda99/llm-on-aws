# 🚀 Web Application Deployment on Amazon EKS using Terraform & GitHub Actions

This project demonstrates a fully automated **CI/CD pipeline** for deploying a containerized web application on **Amazon EKS** using **Terraform**, **Docker**, and **GitHub Actions**.

---

## 📁 Project Structure

```
project-root/
│
├── app/                      # Application code (backend + frontend)
│   ├── backend/              # Flask app and related services
│   ├── frontend/             # Static frontend files (HTML, JS, CSS)
│   └── Dockerfile            # Combined Dockerfile for full app
│
├── infra/                    # Terraform IAC (Infrastructure as Code)
│   ├── vpc, eks, ec2, ecr    # Modular files for AWS setup
│   └── *.tfvars              # Auto tfvars files for provisioning
│
├── k8s/
│   └── deployment.yml        # Kubernetes deployment manifest
│
└── .github/workflows/        # CI/CD pipeline workflows
    ├── terraform_deployment.yml  # Infra provisioning (EKS, VPC, etc)
    ├── build_push.yml            # Docker build & push to ECR
    └── k8s_deploy.yml            # Pull image & deploy to EKS
```

---

## ⚙️ CI/CD Pipeline Overview

### 🛠️ 1. Terraform Deployment
- **File:** `.github/workflows/terraform_deployment.yml`
- Provisions:
  - ✅ VPC
  - ✅ EKS Cluster
  - ✅ EC2 Bastion Host
  - ✅ ECR Repositories
- **Trigger:** Push to any file under `/infra`

**Run Locally:**
```bash
cd infra
terraform init
terraform apply -auto-approve
```

---

### 🐳 2. Docker Build & Push to ECR
- **File:** `.github/workflows/build_push.yml`
- Builds the Docker image from `app/`
- Tags it with `${{ github.run_number }}`
- Pushes to AWS ECR

**Run Locally:**
```bash
cd app
docker build -t myrepo:latest .
aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
docker push <account>.dkr.ecr.<region>.amazonaws.com/myrepo:latest
```

---

### ☸️ 3. Deploy to EKS
- **File:** `.github/workflows/k8s_deploy.yml`
- Fetches the latest ECR image tag
- Replaces the `image:` field in `k8s/deployment.yml`
- Deploys to the EKS cluster via `kubectl`

**Run Locally:**
```bash
aws eks update-kubeconfig --name <your-cluster> --region <region>
kubectl apply -f k8s/deployment.yml
```

---

## 🔐 Required GitHub Secrets

| Secret Name             | Description                        |
|------------------------|------------------------------------|
| `AWS_ACCESS_KEY_ID`    | IAM Access Key                     |
| `AWS_SECRET_ACCESS_KEY`| IAM Secret Key                     |
| `AWS_DEFAULT_REGION`   | e.g., `us-east-1`                  |
| `ECR_REPOSITORY`       | ECR repo name used for pushing     |
| `AWS_ACCOUNT_ID`       | Your AWS account number            |

---

## 🧭 Architecture Diagram (Text-Based)

```
Developer Push ➝ GitHub Actions
   ├── Terraform Deploy ➝ AWS (EKS, ECR, EC2, VPC)
   ├── Docker Build ➝ Push to ECR
   └── K8s Manifest Update ➝ Deploy to EKS
```

---

## ✅ Tips

- Use `github.run_number` as a clean CI version tag.
- Always set up EKS `kubeconfig` on your GitHub runner or local env before deploying.
- Modularize your Terraform for reusability across environments.

---

## 🧠 Notes

- Terraform backend configuration is not included — recommend using remote state (e.g. S3 + DynamoDB).
- Backend is a Flask API and frontend is static NGINX — all bundled into Docker.
- Kubernetes deploy job uses `sed` to patch in latest Docker image URI before applying.

---

## 🤝 Contributions

Pull requests are welcome. For major changes, open an issue first.

---

## 📜 License

MIT License
