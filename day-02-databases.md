Day 2: Databases (Internals, ACID, & Optimization)
==================================================

Topic 2.1: Database Storage Internals (B+ Trees)
------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** The library's alphabetical index card catalog. Instead of searching every book on every shelf (a Full Table Scan), you jump straight to the exact row you need.
    
*   **Why it exists (Problem Solved):** Searching an unsorted hard drive for one record out of 1 billion takes minutes. We need to find data in milliseconds ($O(\\log N)$ time).
    
*   **Core Idea:** Relational databases don't store data randomly; they maintain a mathematically balanced tree structure on the hard drive.
    
*   **Key Terms:** B+ Tree, Node/Page, Leaf Node, Full Table Scan, $O(\\log N)$ Time Complexity.
    

### Layer 2: Deep Dive

*   **Key Internals:** \* A **B+ Tree** has a Root, Branches, and Leaves.
    
    *   The **Leaves** contain the actual database rows (or pointers to them).
        
    *   _The Superpower:_ All Leaf nodes are connected via a **Doubly Linked List**. This makes Range Queries (WHERE price BETWEEN 10 AND 50) incredibly fast. Once it finds 10, it just walks sideways down the list.
        
*   **Real-World Usage:** The default storage and indexing engine for almost all Relational Databases (Postgres, MySQL, Oracle).
    
*   **Trade-offs:** Reads are lightning fast. Writes are slower. Every time you INSERT, the database has to physically rebalance the tree on the hard drive to keep it perfectly sorted.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** "Indexing every column just to be safe." Every index creates a _new_ hidden B+ Tree on the disk. If you index 10 columns, every INSERT statement has to update 10 different trees, completely destroying your write performance.
    
*   **Interview Pointers:** If an interviewer asks, "Why is finding a specific user by ID fast, but finding all users created last week slow?", the answer is that the table is clustered/sorted by ID, not by date. You must explicitly create a Secondary Index on the date column.
    

Topic 2.2: ACID Transactions & The WAL
--------------------------------------

### Layer 1: Core Recall

*   **Intuition:** A bank transfer. If I send you $100, my account must go down by $100 AND your account must go up by $100. If the power goes out in between, the entire transaction must be cancelled. All or nothing.
    
*   **Why it exists (Problem Solved):** Hardware fails. Software crashes. Databases must guarantee mathematical correctness even if the server physically catches fire mid-query.
    
*   **Core Idea:** A strict set of rules (ACID) that relational databases follow to guarantee data integrity.
    
*   **Key Terms:** Atomicity (All or nothing), Consistency (Valid data only), Isolation (Concurrent queries don't step on each other), Durability (Saved permanently), Write-Ahead Log (WAL).
    

### Layer 2: Deep Dive

*   **Key Internals (The WAL):** How does a database survive a power outage mid-write? Before Postgres actually updates the complex B+ Tree, it writes a simple text string describing the change (e.g., "Add $100 to Bob") to a purely sequential, append-only file on disk called the **Write-Ahead Log (WAL)**. Sequential disk writes are blazing fast. If the DB crashes, upon reboot, it reads the WAL and finishes applying the changes.
    
*   **Real-World Usage:** Financial systems, billing engines, inventory management.
    
*   **Trade-offs:** Guaranteeing ACID properties requires locking rows and extensive disk I/O, which makes Relational Databases very difficult to scale horizontally (shard) across multiple servers.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Assuming NoSQL databases like MongoDB or Cassandra are fully ACID compliant by default. (Most are fundamentally BASE—Basically Available, Soft state, Eventual consistency).
    
*   **Interview Pointers:** Be ready to explain the "I" in ACID (Isolation). If two users buy the last concert ticket at the exact same millisecond, the database uses "Row-Level Locks" to force one transaction to wait in line behind the other, preventing a double-booking race condition.
    

Topic 2.3: Normalization vs. Denormalization
--------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** \* **Normalization:** Storing a fact exactly once. (e.g., Storing the company name in one companies table, and linking to it with an ID).
    
    *   **Denormalization:** Copying facts everywhere for speed. (e.g., Storing the company name directly inside the users table next to the user's name).
        
*   **Why it exists (Problem Solved):** CPU-heavy JOIN operations become the biggest bottleneck in a massive database.
    
*   **Core Idea:** Optimize for writes (Normalization) vs. optimize for reads (Denormalization).
    
*   **Key Terms:** 3rd Normal Form (3NF), JOIN, Foreign Key, Read-Heavy vs. Write-Heavy.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **Normalization** saves disk space and prevents data anomalies (if a company changes its name, you update exactly 1 row). But reading a user's profile requires matching IDs across multiple tables (slow).
        
    *   **Denormalization** pre-computes the data. Reading a profile is one lightning-fast disk fetch (no JOINs). But if the company changes its name, you must execute a massive, slow UPDATE query on 1 million user rows.
        
*   **Real-World Usage:** \* Normalization: Used in OLTP (Online Transaction Processing) systems like checkout carts.
    
    *   Denormalization: Used heavily in NoSQL document databases and read-heavy feeds (like Twitter/Instagram timelines).
        

### Layer 3: Interview Mastery

*   **Common Mistakes:** Defaulting strictly to 3rd Normal Form in a system design interview for a system that gets 100,000 reads per second and 10 writes per second.
    
*   **Interview Pointers:** If an interviewer asks how to speed up a Read-Heavy relational database _before_ suggesting Redis or Sharding, tell them you would use **Denormalization** or **Materialized Views** to eliminate the expensive runtime JOIN computations.
