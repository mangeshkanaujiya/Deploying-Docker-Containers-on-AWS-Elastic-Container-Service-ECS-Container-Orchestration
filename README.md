# Deploying Docker Containers on AWS Elastic Container Service (ECS) | Container Orchestration
# 🚀 Node.js Docker Example — Deploying on AWS ECS

This project demonstrates how to **containerize a Node.js application using Docker** and **deploy it to AWS Elastic Container Service (ECS)** via **Amazon ECR**.

---

## 🧩 Project Overview

**Project Name:** Nodejs-Docker-Example-Main  
**Language:** Node.js (Express)  
**Containerization:** Docker  
**Cloud Platform:** AWS (ECR + ECS)  
**Region:** eu-north-1  
**Account ID:** 071325923672

---

## 📁 Folder Structure

```
nodejs-docker-example-main/
│
├── src/
│   └── app.js              # Main Node.js Application File
│
├── package.json            # Project dependencies
├── Dockerfile              # Docker build instructions
├── tsconfig.json           # TypeScript Configuration (if using TS)
└── README.md               # Project documentation
```

---

## ⚙️ Step 1: Setup Node.js App

Initialize your Node.js application (if not already done):
```bash
npm init -y
npm install express
```

Sample `app.js`:
```javascript
const express = require("express");
const app = express();
const PORT = 3000;

app.get("/", (req, res) => {
  res.send("Hello from Node.js Docker App!");
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

## 🐳 Step 2: Create Dockerfile

Sample **Dockerfile** used in this project:

```dockerfile
# Step 1: Use official Node.js image
FROM node:18

# Step 2: Set working directory
WORKDIR /app

# Step 3: Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Step 4: Copy the entire app
COPY . .

# Step 5: Expose port
EXPOSE 3000

# Step 6: Run the app
CMD ["node", "src/app.js"]
```

---

## 🧱 Step 3: Build Docker Image

```bash
docker build -t my-img .
```

List the image:
```bash
docker images
```

Run the container locally to test:
```bash
docker run -d -p 3000:3000 my-img
```

Visit 👉 `http://localhost:3000`

---

## ☁️ Step 4: Configure AWS CLI

```bash
aws configure
```
Provide:
```
AWS Access Key ID [None]: <your-access-key>
AWS Secret Access Key [None]: <your-secret-key>
Default region name [None]: eu-north-1
Default output format [None]: json
```

Verify identity:
```bash
aws sts get-caller-identity
```

✅ Output Example:
```json
{
  "UserId": "AIDARBG23HFML4BLHCPNI",
  "Account": "071325923672",
  "Arn": "arn:aws:iam::071325923672:user/docker-user"
}
```

---

## 🏗️ Step 5: Create ECR Repository

```bash
aws ecr create-repository --repository-name my-img
```

---

## 🔑 Step 6: Authenticate Docker to ECR

```bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 071325923672.dkr.ecr.eu-north-1.amazonaws.com
```

Output:
```
Login Succeeded
```

---

## 🏷️ Step 7: Tag the Docker Image

```bash
docker tag my-img:latest 071325923672.dkr.ecr.eu-north-1.amazonaws.com/my-img:latest
```

---

## 📤 Step 8: Push Image to ECR

```bash
docker push 071325923672.dkr.ecr.eu-north-1.amazonaws.com/my-img:latest
```

Wait until upload completes successfully.

---

## 🧠 Step 9: Create ECS Cluster

1. Go to **ECS Console → Clusters → Create Cluster**
2. Choose **Fargate (Serverless)** or **EC2** type
3. Cluster Name: `docker-cluster-server-mangesh`
4. Create

If you get this error:
> “A CloudFormation stack already exists…”

Go to **CloudFormation Console → Delete the failed stack** named similar to:
```
Infra-ECS-Cluster-docker-cluster-server-mangesh-xxxxxx
```
Then recreate the cluster.

---

## 🪄 Step 10: Create Task Definition

1. Go to **ECS → Task Definitions → Create new**
2. Launch Type: **Fargate**
3. Add container:
   - Name: `my-container`
   - Image URI: `071325923672.dkr.ecr.eu-north-1.amazonaws.com/my-img:latest`
   - Port: `3000`
4. Create Task Definition

---

## 🚀 Step 11: Run ECS Service

1. Go to your **Cluster → Create Service**
2. Launch type: **Fargate**
3. Select your Task Definition
4. Choose number of tasks (e.g., 1)
5. Deploy

Wait for the service to become **ACTIVE**.

---

## 🧩 Step 12: Verify Deployment

Click on the **Task → Public IP** link.  
Open in browser — you should see:
```
Hello from Node.js Docker App!
```

---

## 🔍 Useful Docker Commands

```bash
docker ps                  # List running containers
docker stop <container-id> # Stop container
docker rm <container-id>   # Remove container
docker rmi my-img          # Remove image
docker logs <container-id> # View logs
```

---

## 🧹 Clean Up AWS Resources

When done, delete to avoid charges:
```bash
aws ecs delete-cluster --cluster docker-cluster-server-mangesh
aws ecr delete-repository --repository-name my-img --force
```

---


### Docker & Amazon ECS (Elastic Container Service) Full Guide
## 1️⃣ Docker Containers – Basics

**Docker** is a containerization platform that packages your app and dependencies into a portable container.

### Why Docker?
- Eliminates "works on my machine" issues
- Consistent environment across development & production
- Lightweight and fast

### Common Terms
| Term | Description |
|------|--------------|
| Image | Blueprint of containers |
| Container | Running instance of an image |
| Dockerfile | Instructions to build an image |
| Registry | Stores Docker images (Docker Hub / AWS ECR) |

### Example
```bash
docker build -t myapp .
docker run -d -p 8000:8000 myapp
```

---

## 2️⃣ Docker Orchestration

When many containers are running, orchestration automates deployment, scaling, and management.

**Popular tools:**
- Docker Compose (local orchestration)
- Kubernetes
- AWS ECS (Amazon Elastic Container Service)

---

## 3️⃣ Container Orchestration with AWS

AWS provides two orchestration services:
| Service | Description |
|----------|--------------|
| ECS | AWS-native orchestration for Docker |
| EKS | AWS-managed Kubernetes |

---

## 4️⃣ Amazon ECS in Detail

ECS = Elastic Container Service → AWS-managed orchestration for Docker containers.

### ECS Components
| Component | Description |
|------------|--------------|
| Cluster | Group of EC2/Fargate resources |
| Task Definition | Template describing how to run containers |
| Task | Running instance of Task Definition |
| Service | Keeps tasks running and balanced |
| ECR | Elastic Container Registry for Docker images |

### ECS Workflow Example
1. Build and tag Docker image:
```bash
docker build -t myapp .
docker tag myapp:latest <aws-account-id>.dkr.ecr.<region>.amazonaws.com/myapp
```
2. Push image to ECR:
```bash
aws ecr create-repository --repository-name myapp
docker push <aws-account-id>.dkr.ecr.<region>.amazonaws.com/myapp
```
3. Create ECS Cluster (Fargate or EC2)
4. Create Task Definition using ECR image
5. Create Service to run tasks
6. ECS automatically manages scaling and availability

---

## 5️⃣ ECS + Fargate = Serverless Containers

Fargate removes the need to manage EC2 servers. You pay only for CPU and memory used.

---

## 6️⃣ Summary

| Feature | Docker | ECS |
|----------|--------|-----|
| Local container runs | ✅ | ❌ |
| Cloud deployment | ❌ | ✅ |
| Scaling | ❌ | ✅ |
| Load balancing | ❌ | ✅ |
| Serverless | ❌ | ✅ (via Fargate) |

---

## 7️⃣ ECS Internal Flow

1. Developer builds Docker image and pushes to ECR.  
2. ECS Task Definition references the image.  
3. ECS Service runs tasks inside the Cluster.  
4. ECS monitors and scales containers automatically.  

---

## 💡 Key AWS Services Involved
- **ECR:** Stores Docker images  
- **ECS:** Runs containers  
- **Fargate:** Serverless compute option  
- **CloudWatch:** Monitors logs and metrics  
- **IAM:** Manages permissions

---

### ✅ Final Command Reference

```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<region>.amazonaws.com

# Build & Push Image
docker build -t myapp .
docker tag myapp:latest <aws-account-id>.dkr.ecr.<region>.amazonaws.com/myapp
docker push <aws-account-id>.dkr.ecr.<region>.amazonaws.com/myapp
```


## 👨‍💻 Author

**Name:** Mangesh Kanaujiya  
**LinkedIn:** [Mangesh Kanaujiya](https://www.linkedin.com/in/mangesh-kanaujiya-0438bb2a5/)

**GitHub:** [mangeshkanaujiya](https://github.com/mangeshkanaujiya)

---

## 🏁 Conclusion

This project shows the **end-to-end containerization workflow** —  
from local Docker build → AWS ECR push → ECS Fargate deployment.  

You now have a **Node.js app running serverlessly on AWS!** 🚀

