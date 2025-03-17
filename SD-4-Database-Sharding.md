# Database Sharding

## Table of Contents

1. What is Sharding?
2. Methods of Sharding
   - Key-Based Sharding
   - Horizontal or Range-Based Sharding
   - Vertical Sharding
   - Directory-Based Sharding
3. Ways to Optimize Database Sharding for Even Data Distribution
4. Alternatives to Database Sharding
5. Advantages of Sharding in System Design
6. Disadvantages of Sharding in System Design

## What is Sharding?

Sharding is a database architecture pattern that splits a large dataset into smaller chunks (logical shards) and distributes them across multiple database nodes (physical shards). Each shard has the same database schema as the original database, and each row appears in exactly one shard. Sharding improves application scalability by distributing the data load.
![Sharding](https://github.com/user-attachments/assets/e499becf-021a-41ca-af12-a9e1a923b36d)
**Example Use Case:**

- **E-commerce Platforms:** Large e-commerce websites like Amazon or Flipkart store customer orders in sharded databases to handle high traffic and large datasets efficiently. For example, orders may be sharded based on customer region to ensure even distribution and faster query performance.

## Methods of Sharding

### 1. Key-Based Sharding (Hash-Based Sharding)

A shard key (e.g., customer ID, email, IP address) is hashed to determine the shard where data will be stored. This method ensures uniform distribution if the hash function is well-designed.

![Keybased](https://github.com/user-attachments/assets/a904e925-5183-4784-aa06-c110d95ad8cd)

**Example:**

- Suppose there are 3 database servers.
- Each new application ID is incremented by 1.
- We use the modulo operation (Application ID % 3) to determine the shard for storing data.

```java
// Example: Key-Based Sharding using Modulo Operation

public class KeyBasedSharding {
    public static int keyBasedSharding(int customerId, int numShards) {
        return customerId % numShards;
    }

    public static void main(String[] args) {
        int numShards = 3;
        int customerId = 1024;
        int shard = keyBasedSharding(customerId, numShards);
        System.out.println("Customer " + customerId + " will be stored in Shard " + shard);
    }
}
```

**Real-World Use Case:**

- **Banking Systems:** Banks use key-based sharding to distribute customer transaction data. Each customer ID is hashed and mapped to a specific shard to ensure fast retrieval and updates.

### 2. Horizontal or Range-Based Sharding

Data is split based on value ranges (e.g., customers with first names A-P in one shard, Q-Z in another).

![Horizontal](https://github.com/user-attachments/assets/e06b0c53-40a6-4ba7-84b0-06990d52af74)

**Advantages:**

- Easy scalability by distributing data across multiple shards.
- Improved query performance.

**Disadvantages:**

- Complex querying across shards.
- Risk of uneven data distribution.
  
```java
// Example: Range-Based Sharding

public class RangeBasedSharding {
    public static String rangeBasedSharding(String customerName) {
        // Split by first letter of name
        if (customerName.toUpperCase().charAt(0) <= 'M') {
            return "Shard 1";  // A-M
        } else {
            return "Shard 2";  // N-Z
        }
    }
    public static void main(String[] args) {
        String customerName = "Rahul";
        String shard = rangeBasedSharding(customerName);
        System.out.println("Customer " + customerName + " will be stored in " + shard);
    }
}
```
**Real-World Use Case:**

- **Social Media Platforms:** Facebook uses range-based sharding to store user data. Users are assigned to different shards based on a predefined range of user IDs.

### 3. Vertical Sharding

Entire columns from a table are placed into different shards. Each shard contains a unique set of rows and columns.

![Vertical](https://github.com/user-attachments/assets/75f20ea5-ba9e-42e6-99da-3148fb9bf24d)

**Example:**

- Twitter data can be split into separate shards for user profiles, followers, and tweets.

```java
// Example: Vertical Sharding

public class VerticalSharding {
    static class UserData {
        int userId;
        String name;
        String posts;
        String followers;

        public UserData(int userId, String name, String posts, String followers) {
            this.userId = userId;
            this.name = name;
            this.posts = posts;
            this.followers = followers;
        }
    }

    public static String getUserData(int userId, String shardType) {
        if ("posts".equals(shardType)) {
            return "Fetching posts for user " + userId + " from Post Shard";
        } else if ("followers".equals(shardType)) {
            return "Fetching followers for user " + userId + " from Followers Shard";
        } else {
            return "Fetching name for user " + userId + " from Name Shard";
        }
    }
    public static void main(String[] args) {
        int userId = 1;
        String shard = getUserData(userId, "posts");
        System.out.println(shard);
    }
}

```
**Real-World Use Case:**

- **Healthcare Systems:** Hospitals maintain separate shards for patient records, billing information, and appointment schedules, ensuring that queries for specific data are efficient.

### 4. Directory-Based Sharding

A lookup table maintains mappings between entities and shards. The lookup service directs queries to the correct shard.

![Directory](https://github.com/user-attachments/assets/a49a1b27-6960-45a9-bba4-94c9209e5150)

**Advantages:**

- Flexible data distribution.
- Efficient query routing.
- Scalable as new shards can be added dynamically.

**Disadvantages:**

- Single point of failure in the lookup table.
- Potential for increased latency.
```java
// Example: Directory-Based Sharding

public class DirectoryBasedSharding {
    static Map<String, String> shardLookup = new HashMap<>();
    
    static {
        shardLookup.put("zone_1", "Shard 1");
        shardLookup.put("zone_2", "Shard 2");
        shardLookup.put("zone_3", "Shard 1");
    }

    public static String directoryBasedSharding(String userId) {
        // Look up the shard for the user
        String shard = shardLookup.getOrDefault(deliveryZone, "Unknown Shard");
        return deliveryZone + " is in " + shard;
    }
    public static void main(String[] args) {
        String deliveryZone = "zone_1";
        String shard = directoryBasedSharding(deliveryZone);
        System.out.println(shard);
    }
  }
```
**Real-World Use Case:**

- **Gaming Applications:** Online multiplayer games like Fortnite use directory-based sharding to store player game data, where a lookup table maps players to the appropriate game server.

## Ways to Optimize Database Sharding for Even Data Distribution

1. **Use Consistent Hashing:** Helps distribute data evenly across shards.
2. **Choose a Well-Balanced Sharding Key:** Prevents hotspots.
3. **Range-Based Sharding with Caution:** Ensures even load distribution.
4. **Regular Monitoring and Rebalancing:** Helps prevent performance bottlenecks.
5. **Automate Sharding Logic:** Uses automation tools to balance the load dynamically.

## Alternatives to Database Sharding

1. **Vertical Scaling:** Upgrading server resources instead of sharding.
2. **Replication:** Maintaining copies of the database across multiple servers.
3. **Partitioning:** Dividing data within a single server instead of across multiple servers.
4. **Caching:** Using cache layers like Redis or Memcached to reduce database load.
5. **CDNs:** Offloading read-heavy operations to content delivery networks.

## Advantages of Sharding in System Design

- **Enhances Performance:** Distributes workloads, improving response times.
- **Scalability:** Easily accommodates growing datasets.
- **Improved Resource Utilization:** Prevents overloading a single server.
- **Fault Isolation:** If one shard fails, others remain unaffected.
- **Cost Efficiency:** Uses smaller, cost-effective servers.

## Disadvantages of Sharding in System Design

- **Increased Complexity:** Requires careful planning and maintenance.
- **Rebalancing Challenges:** Moving data between shards can be difficult.
- **Cross-Shard Joins:** Queries across multiple shards can be inefficient.
- **Backup and Recovery Complexity:** Each shard needs separate backups.

By implementing the right sharding strategy based on business needs, organizations can ensure optimal database performance, scalability, and efficiency.

