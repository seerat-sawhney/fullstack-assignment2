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

- Azure account with Kubernetes Service (AKS) and Service Bus set up.
- Azure OpenAI Service configured for GPT-4 and DALL-E models.
- Docker and Kubernetes CLI installed locally.

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
