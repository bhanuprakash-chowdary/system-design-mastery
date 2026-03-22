Day 3: Database Scaling (Replication, Sharding, & CAP)
======================================================

Topic 3.1: Replication (Scaling Reads & Availability)
-----------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** Having one original Master document and making multiple photocopies. If someone just wants to read it, you hand them a photocopy so the Master doesn't get crowded.
    
*   **Why it exists (Problem Solved):** A single database crashes under too many read requests, and if the hard drive dies, the data is gone forever (Single Point of Failure).
    
*   **Core Idea:** Keep identical copies of your database on multiple servers.
    
*   **Key Terms:** Leader/Follower (Master/Slave), Read Replica, Synchronous vs. Asynchronous Replication, Replication Lag.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **The Flow:** All INSERT/UPDATE/DELETE (Writes) go strictly to the Leader. The Leader writes to its Write-Ahead Log (WAL), and immediately streams that log over the network to the Followers. Followers apply the log to their own disks.
        
    *   All SELECT (Reads) are routed to the Followers, removing 90% of the load from the Leader.
        
*   **Real-World Usage:** E-commerce catalogs, blogs, analytical dashboards (anything Read-Heavy).
    
*   **Trade-offs:** If replication is _Asynchronous_ (standard), the Follower is always a few milliseconds behind the Leader (Eventual Consistency).
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** The "Where is my comment?" problem. A user posts a comment (Write to Leader), the page refreshes instantly (Read from Follower), and the comment is missing because of Replication Lag.
    
*   **Interview Pointers:** When an interviewer asks how to solve the missing comment issue, introduce **Read-After-Write Consistency**. Solution: If a user modified data in the last 5 seconds, route their _reads_ directly to the Leader temporarily. Everyone else reads from the replicas.
    

Topic 3.2: Sharding vs. Partitioning (Scaling Writes & Capacity)
----------------------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** \* **Partitioning:** Splitting a massive, heavy encyclopedia into 26 smaller books (A-Z) but keeping them on the _same physical bookshelf_.
    
    *   **Sharding:** Taking those 26 books and putting them in completely _different libraries_ across the city.
        
*   **Why it exists (Problem Solved):** The database is too large to fit on one physical hard drive, or the write volume is melting a single CPU.
    
*   **Core Idea:** Break the database physically into smaller chunks based on a specific key.
    
*   **Key Terms:** Horizontal Partitioning, Vertical Partitioning, Shard Key, Cross-Shard JOIN, Colocation.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **Sharding Algorithm:** When user\_id = 50 creates an order, the application hashes the ID (50 % 3 = 2). The order goes strictly to Shard Server 2.
        
    *   **Vertical Partitioning:** Moving the massive 5MB profile\_picture\_blob column into a separate table on a slower, cheaper hard drive so the users\_core table stays lightning fast in RAM.
        
*   **Real-World Usage:** Twitter/X user data, Stripe transaction histories, massive multiplayer games.
    
*   **Trade-offs:** Sharding is an operational nightmare. You physically cannot perform a standard SQL JOIN between User 50 (on Server 1) and their Company (on Server 4).
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Choosing an Auto-Incrementing ID or a Timestamp as a Shard Key. This causes **Hotspotting**. All new writes will go to the exact same shard, melting Server 4 while Servers 1-3 sit idle. Use a hashed identifier (like user\_id).
    
*   **Interview Pointers:** Always proactively address the Cross-Shard JOIN problem. Tell the interviewer you will solve it via **Data Colocation** (forcing related tables to share the exact same Shard Key) or **Global Tables** (broadcasting small reference tables to every shard).
    

Topic 3.3: Read vs. Write Scaling (CQRS & Fan-Out)
--------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** \* Read-Heavy is a massive buffet (optimize for serving pre-cooked food to crowds).
    
    *   Write-Heavy is a busy drive-thru (optimize for taking rapid orders without blocking the line).
        
*   **Why it exists (Problem Solved):** Tuning a database for fast writes (dropping indexes, NoSQL) actively destroys read performance. You cannot perfectly optimize for both simultaneously.
    
*   **Core Idea:** Physically separate the Read architecture from the Write architecture.
    
*   **Key Terms:** CQRS (Command Query Responsibility Segregation), Fan-out on Write (Push), Fan-out on Read (Pull).
    

### Layer 2: Deep Dive

*   **Key Internals (CQRS):** You use two separate databases. The user POSTs data to a Write-Optimized DB (like Cassandra or Event Store). A background worker syncs the data to a Read-Optimized DB (like ElasticSearch or a denormalized Postgres table). The user GETs from the Read DB.
    
*   **Key Internals (Fan-out on Write):** Used by Twitter. When you tweet, the server does not wait for readers. It physically copies your tweet and "pushes" it into the pre-computed Redis timeline caches of all your followers. Reads become $O(1)$.
    
*   **Real-World Usage:** Uber GPS tracking (Write-Heavy), Netflix Catalog (Read-Heavy).
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Applying "Fan-out on Write" to a celebrity account with 100 million followers. One tweet will cause 100 million cache writes, crashing the queue (The Justin Bieber Problem).
    
*   **Interview Pointers:** For social feeds, suggest a **Hybrid Architecture**. Fan-out on Write for normal users (fast reads). Fan-out on Read for celebrities (do not push; force the client to pull and merge the celebrity's tweets at load time).
    

Topic 3.4: The CAP Theorem & PACELC
-----------------------------------

### Layer 1: Core Recall

*   **Intuition:** Two bank managers share an account via telephone. The phone line gets cut. A customer wants to withdraw money. Do you refuse the transaction (Consistency) or guess the balance to keep the bank open (Availability)? You can't do both.
    
*   **Why it exists (Problem Solved):** Network cables get cut. Servers lose power. You must mathematically define how your system behaves during a failure.
    
*   **Core Idea:** During a network partition (P), a distributed system must choose between Consistency (C) and Availability (A).
    
*   **Key Terms:** Consistency (Latest data or error), Availability (Non-error, but maybe stale), Partition Tolerance (Network breaks), Eventual Consistency.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **CP System:** If the network drops between Node A and B, Node A refuses all writes to prevent data mismatch. (Databases like MongoDB in strict mode, HBase).
        
    *   **AP System:** If the network drops, Node A accepts the write and promises to sync it to Node B whenever the network heals. (Databases like Cassandra, DynamoDB).
        
*   **Real-World Usage:** \* CP: Financial ledgers, ATMs, Inventory checkout.
    
    *   AP: Amazon Shopping Cart (never block a sale!), Social media likes, Video views.
        

### Layer 3: Interview Mastery

*   **Common Mistakes:** Saying "I will build a CA system." In a distributed cloud network, Partitions (P) are inevitable. You _must_ choose P. The choice is strictly between CP or AP.
    
*   **Interview Pointers:** Drop the **PACELC Theorem** to impress the interviewer. It extends CAP: "If there is a Partition (P), choose Availability (A) or Consistency (C). **E**lse (when the network is fine), choose **L**atency or **C**onsistency." This proves you know that even without failures, syncing data for perfect consistency always costs latency.
