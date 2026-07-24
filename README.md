# Deploying a Microservices Application on Kubernetes with Production Best Practices

This project demonstrates the deployment of a cloud-native microservices application on a managed Kubernetes cluster.

The project was completed as part of the DevOps Bootcamp by **TechWorld with Nana**. Beyond deploying the application, my goal was to understand how Kubernetes orchestrates distributed applications and how production-oriented practices improve reliability, scalability, and maintainability.

# Architecture

<p align="center">
  <img src="./images/architecture.png" width="900">
</p>


# Business Problem

An e-commerce company is modernizing its platform by adopting a microservices architecture.

Instead of running a monolithic application, the platform is composed of multiple independent microservices responsible for different business capabilities. Managing deployments, networking, service discovery, and communication between these services manually quickly becomes complex, creating the need for a platform capable of orchestrating the entire application.

The company needs a platform capable of deploying all services in a standardized way while providing service discovery, workload isolation, health monitoring, and simplified application management.

# Solution Overview

To address these challenges, the application was deployed on a managed Kubernetes cluster using Kubernetes Deployments and Services to orchestrate all application components.

Each microservice runs independently within the cluster with its own Deployment and Service configuration. Internal communication is handled through Kubernetes Services and built-in DNS, while Redis provides centralized storage for the shopping cart service.

The deployment also incorporates several Kubernetes practices commonly adopted in production environments, including namespaces, labels, health probes, resource requests and limits, version-pinned container images, and multiple replicas for improved availability.

# Implementation

## Namespace Isolation

A dedicated namespace was created to isolate all application resources.

Namespaces provide logical separation within the cluster, making workloads easier to organize and manage while allowing multiple applications to coexist without resource conflicts.

```bash
kubectl create ns microservice

kubectl apply -f config.yaml -n microservice
```

📷 *Namespace / Cluster overview*

## Deployments

Each microservice is managed through its own Kubernetes Deployment.

Deployments define the desired state of an application, allowing Kubernetes to create Pods, replace unhealthy instances, and maintain the configured number of replicas automatically.

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: recommendationservice

spec:
  replicas: 2

  selector:
    matchLabels:
      app: recommendationservice
```

📷 *Deployment example*


## Service Discovery

One of Kubernetes' biggest advantages is its built-in service discovery.

Instead of communicating through Pod IP addresses, microservices communicate using Kubernetes Services and internal DNS names.

For example, the Recommendation Service discovers the Product Catalog Service through its DNS name:

```yaml
env:
- name: PRODUCT_CATALOG_SERVICE_ADDR
  value: "productcatalogservice:3550"
```

Likewise, the Cart Service communicates with Redis using another internal Service:

```yaml
env:
- name: REDIS_ADDR
  value: "redis-cart:6379"
```

This approach allows Pods to be recreated without disrupting communication between application components.

📷 *Microservice communication diagram*


## Services

Each Deployment is exposed internally through a Kubernetes Service.

Services provide a stable endpoint for applications while Kubernetes automatically routes traffic to healthy Pods.

```yaml
kind: Service

metadata:
  name: paymentservice

spec:
  selector:
    app: paymentservice

  ports:
    - port: 50051
```

📷 *Service configuration*


## External Access

For this project, the frontend is exposed through a NodePort Service, allowing external access for demonstration purposes.

```yaml
spec:
  type: NodePort

  ports:
    - port: 8080
      nodePort: 30007
```

In production environments, a LoadBalancer or Ingress would typically be preferred to provide a single entry point, TLS termination, and improved security.

📷 *Application running*


# Production Best Practices

Beyond simply deploying the application, this project incorporates several Kubernetes practices commonly used in production environments.

### Version-Pinned Images

Container images use fixed versions instead of floating tags to guarantee predictable deployments.

```yaml
image: gcr.io/google-samples/microservices-demo/frontend:v0.8.0
```

### Liveness Probes

Liveness probes allow Kubernetes to detect unhealthy containers and automatically restart them.

```yaml
livenessProbe:
  grpc:
    port: 8080
  periodSeconds: 5
```

### Readiness Probes

Readiness probes ensure that containers only begin receiving traffic after becoming fully operational.

```yaml
readinessProbe:
  grpc:
    port: 3550
  periodSeconds: 5
```

Different probe types (HTTP, gRPC, and TCP) were configured according to each service's communication protocol.

### Resource Requests and Limits

CPU and memory requests and limits help Kubernetes schedule workloads efficiently while preventing individual containers from consuming excessive resources.

```yaml
resources:
  requests:
    cpu: 70m
    memory: 200Mi

  limits:
    cpu: 125m
    memory: 300Mi
```

### Labels

Labels organize Kubernetes resources and allow Services to identify the Pods they should route traffic to.

```yaml
metadata:
  labels:
    app: frontend

selector:
  matchLabels:
    app: frontend
```

### High Availability Considerations

The project also follows several practices that improve application resiliency:

- Multiple replicas for each microservice
- Multiple Kubernetes worker nodes
- Automatic Pod recreation through Deployments
- Internal service discovery independent of Pod IP addresses

# Final Result

  ![Diagram](./images/k8s-project5-1.png)

The application was successfully deployed to a managed Kubernetes cluster, with all microservices communicating through Kubernetes Services and internal DNS.

# What I Learned

Building this project strengthened my understanding of:

- Deploying distributed applications with Kubernetes
- Managing Deployments and Services
- Kubernetes service discovery and internal DNS
- Namespace isolation
- Labels and selectors
- Health monitoring with Liveness and Readiness Probes
- CPU and memory requests and limits
- Production deployment best practices
- Running containerized microservices on a managed Kubernetes cluster


# Acknowledgements

This project was completed as part of the DevOps Bootcamp created by **TechWorld with Nana**.

The implementation, documentation, architectural analysis, and technical explanations in this repository reflect my own understanding and learning throughout the project.
