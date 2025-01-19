# Service Discovery in Microservices

This repository explores the concept of **Service Discovery** in microservices, a mechanism that enables services to locate and communicate with each other in dynamic, containerized environments. 

## Table of Contents

1. [Introduction](#introduction)
2. [What Is Service Discovery?](#what-is-service-discovery)
3. [The Need for Service Discovery](#the-need-for-service-discovery)
4. [How Does Service Discovery Work?](#how-does-service-discovery-work)
5. [Service Discovery Implementations](#service-discovery-implementations)
   - [Client-Side Service Discovery](#client-side-service-discovery)
   - [Server-Side Service Discovery](#server-side-service-discovery)
6. [Service Registry](#service-registry)
   - [Self-Registration](#self-registration)
   - [Third-Party Registration](#third-party-registration)
7. [Conclusion](#conclusion)

---

## Introduction

In microservices architecture, services need a reliable mechanism to locate and communicate with each other without hardcoding their locations. **Service Discovery** addresses this requirement by dynamically managing service locations and enabling seamless inter-service communication.

---

## What Is Service Discovery?

Service Discovery is a mechanism to dynamically determine the location (IP address and port) of microservices in an environment where service instances frequently scale up or down, or where their locations are dynamically assigned.

### Key Features:
- **Service Registration**: Tracks available service instances.
- **Service Resolution**: Allows clients to discover the location of the target service.
- **Load Balancing**: Distributes requests across multiple service instances.

---

## The Need for Service Discovery

### Challenges Addressed by Service Discovery:
- Dynamic scaling of services.
- Avoiding hardcoded service locations.
- Simplifying communication in containerized or cloud-based environments.

Without Service Discovery, service location management becomes cumbersome, especially in modern, dynamic applications.

---

## How Does Service Discovery Work?

Service Discovery typically involves the following:
1. **Service Registration**: Services register their locations with a **Service Registry**.
2. **Service Lookup**: Clients query the registry to find the desired service.
3. **Service Communication**: Clients communicate directly or through intermediaries with the target service.
   
![image](https://github.com/user-attachments/assets/a73ba7ad-a787-442d-98ec-b674c3dec3c7)

Two primary patterns exist:
- **Client-Side Discovery**
- **Server-Side Discovery**

---

## Service Discovery Implementations

### Client-Side Service Discovery
In this approach, the client directly queries the Service Registry, retrieves available service locations, and uses a load-balancing algorithm to interact with one of the instances.

![image](https://github.com/user-attachments/assets/e267a455-6e83-4005-a149-731366853c74)

#### Advantages:
- Reduced latency (avoids an extra network hop).
- Efficient for lightweight services.

#### Disadvantages:
- The client must implement discovery and load-balancing logic.
- Tight coupling between clients and the registry.

### Server-Side Service Discovery
In this approach, a **Load Balancer** acts as an intermediary between clients and the Service Registry, managing the discovery process.

![image](https://github.com/user-attachments/assets/d5f3faac-1eb1-44fb-b5c5-5ed4df032146)

#### Advantages:
- Simplified client logic.
- Centralized discovery and load balancing.

#### Disadvantages:
- Additional setup and maintenance of the load balancer.

---

## Service Registry

The **Service Registry** is the backbone of Service Discovery, containing dynamic network locations of all available service instances. It supports two primary registration mechanisms:

### 1. Self-Registration
Here, service instances register and deregister themselves directly with the Service Registry. They may also send periodic heartbeats to indicate their availability.

#### Workflow:
1. Service instances announce their presence to the Service Registry.
2. If a service terminates, it deregisters itself from the registry.

![image](https://github.com/user-attachments/assets/078034e1-8df6-4cfd-a0b6-55f317b36bf0)

#### Advantages:
- Simple and straightforward implementation.
- No additional components required for registration.

#### Disadvantages:
- Tight coupling between services and the Service Registry.
- Registration logic must be implemented for each language and framework used.

#### Example Use Case:
A microservice application using Consul for service discovery where each service instance directly communicates with the Consul agent for registration.

---

### 2. Third-Party Registration
In this approach, a separate **Service Register** component handles registration and deregistration of service instances. It monitors the deployment environment or listens for events to detect changes in service availability.

#### Workflow:
1. The Service Register detects new or terminated service instances and updates the Service Registry accordingly.
2. Clients query the Service Registry as usual.

![image](https://github.com/user-attachments/assets/93672376-2c08-4a13-8dbd-44d251d6dc45)

#### Advantages:
- Decouples service instances from the Service Registry.
- No need to implement registration logic in each service.

#### Disadvantages:
- Adds complexity by introducing a new component (Service Register).
- Requires high availability of the Service Register.

#### Example Use Case:
Kubernetes uses a third-party registration mechanism where kubelet agents register services with the Kubernetes API server, which acts as the Service Registry.

---

## Realtime Use Case
### **Microservices with Eureka (Spring Cloud)**
This repository includes an implementation of **Server-Side Discovery** using **Netflix Eureka**:

1. **Eureka Server**:
   - Acts as a centralized service registry.
2. **Service Provider**:
   - Registers with Eureka and exposes a REST API.
3. **Service Consumer**:
   - Queries Eureka to discover and communicate with the Service Provider.

#### Prerequisites:
- JDK 11 or higher
- Maven/Gradle
- Docker (for containerized setup)

#### Steps to Run:
1. Start Eureka Server:
   ```bash
   mvn spring-boot:run -pl eureka-server
   ```
2. Start Service Provider:
   ```bash
   mvn spring-boot:run -pl service-provider
   ```
3. Start Service Consumer:
   ```bash
   mvn spring-boot:run -pl service-consumer
   ```
4. Access the Eureka dashboard at `http://localhost:port` to view registered services.

---

## Conclusion

Service Discovery is a vital component of microservices architecture, ensuring scalability, fault tolerance, and dynamic service communication. By implementing Service Discovery, applications become resilient and easier to maintain in modern cloud-native environments.

---

## Reference
- [baeldung](https://www.baeldung.com/cs/service-discovery-microservices).
