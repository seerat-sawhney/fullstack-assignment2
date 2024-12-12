# Assignment 2 - By Seerat Sawhney (041107886) 
# Best Buy Cloud-Native Application

## **Introduction**

Welcome to the Best Buy Cloud-Native Application project! This document details the deployment and management of a microservices-based architecture for Best Buy’s online store, enriched with modern cloud-native practices. The project demonstrates the use of Kubernetes for container orchestration, Azure Service Bus for order queuing, and Azure OpenAI for AI-powered functionalities.

---

## **Updated Application Architecture**

### **Architecture Diagram**

![alt text](Architecture.png)

### **Application Components**

| **Service**           | **Description**                                                       |
| --------------------- | --------------------------------------------------------------------- |
| **Store-Front**       | Customer-facing application for browsing and placing orders.          |
| **Store-Admin**       | Employee-facing application for managing products and viewing orders. |
| **Order-Service**     | Handles order creation and sends data to the managed order queue.     |
| **Product-Service**   | Manages CRUD operations for product data.                             |
| **Makeline-Service**  | Processes and completes orders by reading from the order queue.       |
| **AI-Service**        | Generates product descriptions and images using GPT-4 and DALL-E.     |
| **Database**          | MongoDB for persisting order and product data.                        |
| **Azure Service Bus** | Managed service for reliable order queuing.                           |

---

## **Application Functionality**

The Best Buy Cloud-Native Application provides:

- **Customer Features:**
  - Seamless browsing with AI-generated product descriptions and images.
  - Efficient order placement and processing.

- **Employee Tools:**
  - Inventory management and order tracking.
  - Real-time insights into order processing.

- **Cloud-Native Enhancements:**
  - Scalable microservices architecture.
  - Managed Azure resources for reliability and performance.

---

## **Deployment Guide**

### **Prerequisites**

# Prerequisites Setup for Best Buy Cloud-Native Application

## 1. Set Up Azure Kubernetes Service (AKS)

### Step 1: Create a Resource Group
1. Sign in to the [Azure Portal](https://portal.azure.com).
2. Navigate to **Resource Groups**.
3. Click **+ Create**.
4. Provide:
   - **Subscription**: Select your subscription.
   - **Resource Group Name**: Enter a unique name (e.g., `bestbuyss').
   - **Region**: Choose Canada Central
5. Click **Review + Create**, then **Create**.

### Step 2: Create the AKS Cluster
1. In the Azure Portal, navigate to **Azure Kubernetes Service**.
2. Click **+ Create** > **Kubernetes Cluster**.
3. Provide:
   - **Cluster Name**: Enter a name (e.g., `bestbuyclusterseerat`).
   - **Region**: Select the same region as your resource group i.e Canada Central.
   - **Node Count**: Start with 3 nodes (default).
   - **Node Image**: No schedule
4.### Configure Node Pools

 **First Node Pool: systemnode**
   - **Size**: D2as-V4
   - **Scale Method**: Manual
   - **Node Count**: 1
   - **Max Pods per Node**: 110

 **First Node Pool: workernode**
   - **Size**: D2as-V4
   - **Scale Method**: Manual
   - **Node Count**: 2
   - **Max Pods per Node**: 30


6. Click **Review + Create**, then **Create**.

### Step 3: Configure kubectl Access
1. Install the Azure CLI if you haven't already ([download link](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)).
2. Sign in to Azure CLI:
   ```bash
   az login
   ```
3. Get AKS credentials:
   ```bash
   az aks get-credentials --resource-group bestbuy-app-rg --name bestbuy-aks
   ```
4. Verify connection to the cluster:
   ```bash
   kubectl get nodes
   ```

---


## 2. Set Up Azure OpenAI Service

### Step 1: Enable OpenAI in Azure
1. If not already available in your subscription, request access to Azure OpenAI Service [here](https://aka.ms/oai/access).

### Step 2: Create an OpenAI Service
1. Navigate to **Azure OpenAI** in the Azure Portal.
2. Click **+ Create**.
3. Provide:
   - **Resource Group**: Select your existing resource group (`bestbuy-app-rg`).
   - **Region**: Select a region that supports OpenAI (e.g., East US).
   - **Pricing Tier**: Choose **Standard**.
4. Click **Review + Create**, then **Create**.

### Step 3: Deploy GPT-4 and DALL-E Models
1. Open your OpenAI resource.
2. Go to **Model Deployments** and click **+ Add Deployment**.
3. Select:
   - **Model**: Choose GPT-4 and DALL-E.
   - **Deployment Name**: Enter descriptive names (e.g., `gpt4-deployment`, `dalle-deployment`).
4. Save the deployments.

### Step 4: Retrieve API Key
1. Open your OpenAI resource.
2. Go to **Keys and Endpoints**.
3. Copy the **API Key** and **Endpoint URL** for later use in your application.

---

## 3. Install Docker and Kubernetes CLI

### Install Docker
1. Download Docker Desktop for your operating system from [docker.com](https://www.docker.com/products/docker-desktop).
2. Install Docker Desktop following the instructions for your OS.
3. Verify Docker installation:
   ```bash
   docker --version
   ```

### Install Kubernetes CLI
1. Download and install kubectl for your OS from [Kubernetes CLI Docs](https://kubernetes.io/docs/tasks/tools/).
2. Verify kubectl installation:
   ```bash
   kubectl version --client
   ```


### **Steps to Deploy**

#### **1. Clone the Repository**

```bash
git clone <repository-link>
cd <repository>
```

#### **2. Build Docker Images**

```bash
docker build -t <dockerhub-username>/store-front .
docker push <dockerhub-username>/store-front
# Repeat for other services
```

#### **3. Set Up AKS Cluster**

```bash
az aks create --resource-group <resource-group> --name <cluster-name> --node-count 3
```

#### **4. Apply Kubernetes Configurations**

```bash
kubectl apply -f DeploymentFiles/store-front-deployment.yaml
kubectl apply -f DeploymentFiles/order-service-deployment.yaml
# Repeat for other services
```

#### **5. Configure Kubernetes Resources**

- **Secrets**:
  ```bash
  kubectl create secret generic ai-keys --from-literal=API_KEY=<your-api-key>
  ```
- **ConfigMaps**:
  ```bash
  kubectl create configmap app-config --from-file=config.json
  ```

#### **6. Set Up Azure OpenAI Service**

- Navigate to Azure Portal.
- Create and configure GPT-4 and DALL-E models.
- Retrieve API keys and encode them:

  ```bash
  echo -n "<your-api-key>" | base64
  ```

- Update `secrets.yaml` and deployment files with the encoded keys.

#### **7. Access the Application**

- Use the Kubernetes LoadBalancer IP to access the Store-Front and Store-Admin interfaces.

---

## **Key Kubernetes Resources**

### **Deployments**

- Ensures specified replicas of pods are running.
- Provides rolling updates and self-healing capabilities.

### **StatefulSets**

- Manages stateful applications like MongoDB.
- Ensures stable storage and unique network identifiers for pods.

### **Secrets**

- Stores sensitive information securely, such as API keys.
- Can be injected into pods as environment variables.

### **ConfigMaps**

- Decouples application configuration from code.
- Stores non-sensitive data like environment variables.

---

## **Microservices and Docker Images**

| **Service**          | **Repository Link**        | **Docker Image Link**             |
| -------------------- | -------------------------- | --------------------------------- |
| **Store-Front**      | [GitHub Link](github-link) | [Docker Hub Link](dockerhub-link) |
| **Order-Service**    | [GitHub Link](github-link) | [Docker Hub Link](dockerhub-link) |
| **Product-Service**  | [GitHub Link](github-link) | [Docker Hub Link](dockerhub-link) |
| **Makeline-Service** | [GitHub Link](github-link) | [Docker Hub Link](dockerhub-link) |
| **AI-Service**       | [GitHub Link](github-link) | [Docker Hub Link](dockerhub-link) |

---

## **Scaling and Monitoring**

- Scale services using Kubernetes horizontal pod autoscalers:

  ```bash
  kubectl autoscale deployment <deployment-name> --cpu-percent=50 --min=1 --max=10
  ```

- Monitor application health with Azure Monitor and Kubernetes tools:
  - Configure alerts for resource usage.
  - Use Prometheus and Grafana for detailed metrics visualization.

---

## **Demo Video**

- [Demo Video Link](youtube-link)
  - Showcases Store-Front and Store-Admin interfaces.
  - Demonstrates order processing and AI integrations.

---

## **Issues or Limitations**

(Optional) Document any challenges, bugs, or limitations faced during the implementation process.

---

## **CI/CD Pipelines**

- Implemented using GitHub Actions.
- Automates Docker builds and Kubernetes deployments.

Thank you for reviewing the Best Buy Cloud-Native Application. For any queries or issues, feel free to raise a GitHub issue in the respective repository!
