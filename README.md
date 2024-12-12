# Assignment 2 - By Seerat Sawhney (041107886) 
# Best Buy Cloud-Native Application

## **Introduction**

Welcome to the Best Buy Cloud-Native Application project! This document details the deployment and management of a microservices-based architecture for Best Buy’s online store, enriched with modern cloud-native practices. The project demonstrates the use of Kubernetes for container orchestration, Azure Service Bus for order queuing, and Azure OpenAI for AI-powered functionalities.

---
## Step 1: Clone the BestBuyApplication Repository
To begin, clone the [**BestBuyApp**]  repository, which contains all necessary deployment files.

 **Review the Deployment Files**:
   - Navigate to the `Deployment Files` folder
   - This folder contains YAML files for deploying all necessary Kubernetes resources, including services, deployments, StatefulSets, ConfigMaps, and Secrets.

## **Updated Application Architecture**
## Architecture

The application has the following services: 

| Service | Description | Github Repo |
| --- | --- | --- |
| `store-front` | Web app for customers to place orders (Vue.js) | link |
| `store-admin` | Web app used by store employees to view orders in queue and manage products (Vue.js) |  |
| `order-service` | This service is used for placing orders (Javascript) |  |
| `product-service` | This service is used to perform CRUD operations on products (Rust) |  |
| `makeline-service` | This service handles processing orders from the queue and completing them (Golang) |  |
| `ai-service` | Optional service for adding generative text and graphics creation (Python) |  |
| `rabbitmq` | RabbitMQ for an order queue |  |
| `mongodb` | MongoDB instance for persisted data |  |
| `virtual-customer` | Simulates order creation on a scheduled basis (Rust) |  |
| `virtual-worker` | Simulates order completion on a scheduled basis (Rust) |  |




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
## Step 2: Create an Azure Kubernetes Cluster (AKS)

1. **Log in to Azure Portal:**
   - Go to [https://portal.azure.com](https://portal.azure.com) and log in with your Azure account.

2. **Create a Resource Group:**
   - In the Azure Portal, search for **Resource Groups** in the search bar.
   - Click **Create** and fill in the following:
     - **Resource group name**: `AlgonquinPetStoreRG`
     - **Region**: `Canada`.
   - Click **Review + Create** and then **Create**.

3. **Create an AKS Cluster:**
   - In the search bar, type **Kubernetes services** and click on it.
   - Click **Create** and select **Kubernetes cluster**
   - In the `Basics` tap fill in the following details:
     - **Subscription**: Select your subscription.
     - **Resource group**: Choose `AlgonquinPetStoreRG`.
     - **Cluster preset configuration**: Choose `Dev/Test`.
     - **Kubernetes cluster name**: `AlgonquinPetStoreCluster`.
     - **Region**: Same as your resource group (e.g., `Canada`).
     - **Availability zones**: `None`.
     - **AKS pricing tier**: `Free`.
     - **Kubernetes version**: `Default`.
     - **Automatic upgrade**: `Disabled`.
     - **Automatic upgrade scheduler**: `No schedule`.
     - **Node security channel type**: `None`.
     - **Security channel scheduler**: `No schedule`.
     - **Authentication and Authorization**: `Local accounts with Kubernetes RBAC`.
   - In the `Node pools` tap fill in the following details:
     - Select **agentpool**. Optionally change its name to `masterpool`. This nodes will have the controlplane.
        - Set **node size** to `D2as_v4`.
        - **Scale method**: `Manual`
        - **Node count**: `1`
        - Click `update`
     - Click on **Add node pool**:
        - **Node pool name**: `workerspool`.
        - **Mode**: `User` 
        - Set **node size** to `D2as_v4`.
        - **Scale method**: `Manual`
        - **Node count**: `2`
        - Click `add`
   - Click **Review + Create**, and then **Create**. The deployment will take a few minutes.

4. **Connect to the AKS Cluster:**
   - Once the AKS cluster is deployed, navigate to the cluster in the Azure Portal.
   - In the overview page, click on **Connect**. 
   - Select **Azure CLI** tap. You will need Azure CLI. If you don't have it: [**Install Azure CLI**](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
   - Login to your azure account using the following command:
      ```
      az login
      ```
   - Set the cluster subscription using the command shown in the portal (it will look something like this):
      ```
      az account set --subscription 'subscribtion-id'
      ```

   - Copy the command shown in the portal for configuring `kubectl` (it will look something like this):
     ```
     az aks get-credentials --resource-group AlgonquinPetStoreRG --name AlgonquinPetStoreCluster
     ```
      **Understanding the Command:**
      - The command `az aks get-credentials` pulls the necessary configuration files to enable `kubectl` to access your AKS cluster. Here’s a breakdown:
     - `--resource-group` specifies the resource group where your AKS cluster resides.
     - `--name` specifies the name of your AKS cluster.
     - `--overwrite-existing` can be used to overwrite any existing Kubernetes configuration files for the same cluster. This is useful if you’ve connected to the cluster before or if multiple configurations exist for it.
   - Verify Cluster Access:
      - Test your connection to the AKS cluster by listing all nodes:
        ```
        kubectl get nodes
        ```
        You should see details of the nodes in your AKS cluster if the connection is successful.

---


## 2. Set Up AI Backing Services 
## Azure OpenAI Service

### Step 1: Create an OpenAI Service
1. Navigate to **Azure OpenAI** in the Azure Portal.
2. Click **+ Create**.
3. Provide:
   - **Resource Group**: Select your existing resource group(e.g., `bestbuyss').
   - **Region**: Select a region that supports OpenAI (e.g., East US).
   - **Pricing Tier**: Choose **Standard**.
4. Click **Review + Create**, then **Create**.

### Step 2: Deploy GPT-4 and DALL-E Models
1. Open your OpenAI resource.
2. Go to **Model Deployments** and click **+ Add Deployment**.
3. Select:
   - **Model**: Choose GPT-4 and DALL-E.
   - **Deployment Name**: Enter descriptive names (e.g., `gpt4-deployment`, `dalle-deployment`).
4. Save the deployments.

### Step 3: Retrieve API Key
1. Open your OpenAI resource> Home> Click on json file .
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
