# API Gateway

An **API Gateway** is a central server that acts as an intermediary between clients (e.g., browsers, mobile apps) and backend services. It simplifies communication by routing requests, enforcing security, and managing interactions with multiple microservices.

## Table of Contents
- [Why Do We Need an API Gateway?](#why-do-we-need-an-api-gateway)
- [Core Features of an API Gateway](#core-features-of-an-api-gateway)
- [How Does an API Gateway Work?](#how-does-an-api-gateway-work)
- [Terminology](#terminology)

## Why Do We Need an API Gateway?
In modern applications built using microservices architecture, multiple backend services handle different functionalities. Without an API Gateway:
- Clients need to know the location and details of all backend services.
- Developers must manage security and authentication for each service individually.

![without](https://github.com/user-attachments/assets/92a2e0ad-7450-4797-8116-11d13c4cd219)

With an API Gateway:
- Clients send all requests to a single entry point.
- The gateway handles routing, authentication, and operational tasks, simplifying client interactions and backend management.

![with](https://github.com/user-attachments/assets/c8595cd5-2535-449d-a45a-0c3a8fa575e2)


## Core Features of an API Gateway

#### 1. Authentication and Authorization
- Verifies the identity of the client using tokens (e.g., OAuth, JWT), API keys, or certificates.
- Checks client permissions to access specific resources.

#### 2. Rate Limiting
- Controls the frequency of client requests to prevent abuse.
- Protects services from excessive traffic or DoS attacks.

#### 3. Load Balancing
- Distributes incoming requests across multiple service instances.
- Ensures traffic is directed to healthy instances using algorithms like round-robin or least connections.

#### 4. Caching
- Temporarily stores frequently requested data to reduce latency and operational costs.

#### 5. Request Transformation
- Modifies request or response formats to ensure compatibility between clients and backend services.

#### 6. Service Discovery
- Dynamically identifies the appropriate backend service instance for each request.

#### 7. Circuit Breaking
- Temporarily blocks requests to failing services to maintain overall system stability.

#### 8. Logging and Monitoring
- Tracks metrics like request rates, errors, and latency to optimize performance.
  
![features](https://github.com/user-attachments/assets/4151e429-05a2-4c97-88cd-e0ebb2868bb2)

## How Does an API Gateway Work?

### 1. Request Reception
The client sends a request to the API Gateway, which acts as the single entry point to the backend system.

### 2. Request Validation
- Ensures the request has required parameters and headers.
- Validates data format and schema.

```javascript
app.post('/api/v1/orders', async (req, res) => {
    if (!req.headers['content-type'].includes('application/json')) {
        return res.status(400).send('Invalid content type');
    }
    // Continue processing...
});
```

### 3. Authentication and Authorization
- Verifies the client’s identity using tokens (e.g., OAuth, JWT).
- Checks permissions for accessing specific services.

```javascript
const authenticateRequest = async (req) => {
    const token = req.headers.authorization?.split(' ')[1];
    const user = await verifyToken(token);
    return user.permissions.includes('place_orders');
};
```

### 4. Rate Limiting
- Prevents excessive requests by limiting the frequency per client.

```javascript
const checkRateLimit = async (userId) => {
    const key = `rate_limit:order:${userId}`;
    const current = await redis.incr(key);
    if (current === 1) {
        await redis.expire(key, 60); // 1 minute window
    }
    return current <= 10;
};
```

### 5. Request Transformation
- Adapts request formats for compatibility with backend services.

```javascript
const transformRequest = async (originalRequest) => {
    const address = originalRequest.deliveryAddress;
    const coordinates = await getCoordinatesFromAddress(address);
    if (!coordinates) {
        throw new Error('Failed to fetch GPS coordinates');
    }
    return {
        orderId: originalRequest.orderId,
        customerName: originalRequest.customerName,
        deliveryLocation: {
            latitude: coordinates.lat,
            longitude: coordinates.lng
        },
        deliveryInstructions: originalRequest.instructions || ""
    };
};
```

### 6. Request Routing
- Directs requests to appropriate backend services using service discovery.

```javascript
const routeRequest = async (req, serviceType) => {
    const services = await serviceDiscovery.getServices(serviceType);
    const targetService = selectServiceInstance(services);
    return await axios.post(
        `${targetService.url}/api/orders`,
        req.body,
        { headers: req.headers }
    );
};
```

### 7. Response Handling
- Adjusts response formats or caches responses if applicable.

```javascript
const handleResponse = async (serviceResponse) => {
    const transformedResponse = {
        orderId: serviceResponse.order_reference,
        estimatedDelivery: serviceResponse.eta,
        status: serviceResponse.current_status
    };
    if (serviceResponse.cacheable) {
        await cacheResponse(
            transformedResponse.orderId, 
            transformedResponse
        );
    }
    return transformedResponse;
};
```

### 8. Logging and Monitoring
- Records metrics to track and analyze requests.

```javascript
const logRequest = async (req, res, timing) => {
    await logger.log({
        timestamp: new Date(),
        path: req.path,
        method: req.method,
        responseTime: timing,
        statusCode: res.statusCode,
        userId: req.user?.id
    });
};
```

## Terminology

### Latency
- Refers to the delay between a client request and the server's response.
- Example: The time it takes to load a webpage after clicking a link.

### Redundancy
- Refers to the duplication of resources or services to improve reliability and fault tolerance.
- Example: Running multiple instances of a service to ensure availability during failures.

## Reference
- Learn more about API Gateways and their practical applications at [Substack](https://substack.com/inbox/post/152508834).

