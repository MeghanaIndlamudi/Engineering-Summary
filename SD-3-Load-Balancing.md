# Load Balancing 

## 1. What is Load Balancing?
Load balancing is the process of distributing network or application traffic across multiple servers to ensure high availability and reliability. It helps in optimizing resource utilization, reducing latency, and preventing overload on a single server.

---

## 2. IP Hashing in Load Balancing
IP Hashing is a common technique used to allocate client requests to servers based on the client's IP address.

### 2.1 Example Problem with Hash Functions
Consider a scenario where a load balancer distributes requests among three servers (S1, S2, S3) using a simple hash function:

```plaintext
server_index = hash(client_IP) % 3
```

This ensures that the same client IP always gets assigned to the same server, providing session persistence.

### 2.2 Adding and Removing Servers in IP Hashing
- **Adding a Server:** The modulo function changes (`% new_server_count`), which results in a drastic reassignment of requests. Most clients will now be mapped to different servers, leading to cache misses and session disruptions.
- **Removing a Server:** Causes similar disruptions since the mapping changes for most clients, making IP Hashing impractical for dynamic scaling.

### 2.3 Disadvantages of IP Hashing
- **Uneven Distribution:** If some IPs are more frequent, those servers will receive more traffic, causing an imbalance.
- **Server Scalability Issue:** When a new server is added, the hash function needs to be recomputed, leading to a major redistribution of traffic.
- **Single Point of Failure:** If a server goes down, affected clients will lose their assigned sessions.

---

## 3. Introduction to Consistent Hashing
To overcome the limitations of IP Hashing, **Consistent Hashing** was introduced. It minimizes disruptions when adding or removing servers by mapping both servers and requests onto a hash ring.

### 3.1 How Consistent Hashing Works
1. **Mapping Servers to a Hash Ring:** Each server is assigned a position in a circular hash space using a hash function (e.g., `hash(server_id)`).
2. **Mapping Requests to the Hash Ring:** Each request's key (e.g., client IP) is also hashed and placed on the ring.
3. **Assigning Requests to Servers:** A request is handled by the next closest server in a clockwise direction.
4. **Adding/Removing Servers:** When a server is added or removed, only a subset of requests need to be reassigned instead of all, unlike traditional hashing.

### 3.2 Example Problem with Consistent Hashing
Assume we have servers **S1, S2, and S3** placed at hashed positions on a circular space:

```plaintext
|----S1(10)----|----S2(50)----|----S3(90)----|
```

When a client request arrives, its hash determines which server handles it. For example:
- Request **R1**: Hashes to 15 → Assigned to **S2**.
- Request **R2**: Hashes to 60 → Assigned to **S3**.
- Request **R3**: Hashes to 95 → Assigned to **S1**.

If **S2** is removed, its requests get reassigned to **S3** without affecting **S1**.

### 3.3 Adding and Removing Servers in Consistent Hashing
- **Adding a Server:** The new server is assigned a hashed position in the ring, and only the requests that fall between the new server and its predecessor are reassigned.
- **Removing a Server:** The requests assigned to that server are redirected to the next server in a clockwise direction, minimizing disruptions.

### 3.4 Advantages of Consistent Hashing
- **Minimizes Reassignments:** When adding/removing servers, only a small portion of requests need reassignment.
- **Better Load Distribution:** Virtual nodes can be used to evenly distribute traffic.
- **Scalability:** Servers can be added dynamically without massive rehashing.

### 3.5 Disadvantages of Consistent Hashing
- **Implementation Complexity:** More complex than simple modulo-based hashing.
- **Initial Skewness:** Without virtual nodes, servers might receive unbalanced loads.

---

## 4. Real-Time Use Cases of Load Balancing

### 4.1 Content Delivery Networks (CDN)
- Companies like **Cloudflare and Akamai** use consistent hashing to distribute traffic across multiple edge locations efficiently.

### 4.2 Distributed Databases
- **Amazon DynamoDB** and **Apache Cassandra** use consistent hashing to allocate data across multiple nodes, ensuring fault tolerance.

### 4.3 Caching Systems
- **Memcached and Redis** use consistent hashing to distribute keys across multiple caching servers for efficient lookups.

### 4.4 Microservices and API Gateways
- Load balancing helps distribute API requests across multiple instances of a microservice, reducing latency and improving reliability.

### 4.5 Gaming and Streaming Services
- **Netflix and Twitch** use load balancing to direct user requests to the closest or least-loaded server, ensuring smooth playback and low latency.

---

## 5. Additional Load Balancing Strategies
Apart from consistent hashing, other common load balancing techniques include:

### 5.1 Round Robin
- Requests are distributed sequentially to each server.
- **Pros:** Simple and easy to implement.
- **Cons:** Can cause uneven distribution if servers have different processing capabilities.

### 5.2 Least Connections
- Assigns requests to the server with the fewest active connections.
- **Pros:** Helps distribute traffic based on load.
- **Cons:** May not work efficiently in session-persistent applications.

### 5.3 Weighted Load Balancing
- Assigns different weights to servers based on their capacity.
- **Pros:** Ensures better distribution based on server performance.
- **Cons:** Requires manual tuning of weights.

---

## 6. Conclusion
Load balancing is an essential component of scalable system design, ensuring high availability, fault tolerance, and optimal resource utilization. **Consistent Hashing** provides an effective way to distribute requests without major disruptions, making it ideal for modern distributed systems.
