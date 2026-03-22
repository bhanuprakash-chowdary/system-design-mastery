Day 4: Caching Systems (Speed & Memory)
=======================================

Topic 4.1: Caching Fundamentals & Strategies
--------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** Your database (SSD) is a massive filing cabinet down the hall. Your cache (RAM) is your physical desk. Your desk is tiny, but grabbing a paper from it is instant.
    
*   **Why it exists (Problem Solved):** Hard drives are physically too slow to handle millions of reads. RAM is roughly 1,000x faster than a network SSD (3 microseconds vs. 1-2 milliseconds).
    
*   **Core Idea:** Store the results of heavy, frequent database queries in RAM so subsequent users get the answer instantly.
    
*   **Key Terms:** Cache Hit, Cache Miss, Cache-Aside (Lazy Loading), Write-Through, Write-Behind (Write-Back).
    

### Layer 2: Deep Dive

*   **Key Internals (The 3 Main Strategies):**
    
    1.  **Cache-Aside:** The application code coordinates everything. App checks Cache -> Miss -> App queries DB -> App saves to Cache -> Returns to user. (Safest, most common).
        
    2.  **Write-Through:** App writes to Cache, and Cache _synchronously_ writes to DB. The user waits for both. (Perfect consistency, slow writes).
        
    3.  **Write-Behind:** App writes to Cache. Cache instantly returns 200 OK. Asynchronously, the Cache flushes the data to the DB. (Blazing fast writes, high risk of data loss on power failure).
        
*   **Real-World Usage:** Facebook caching user profiles (Memcached), High-Frequency Trading (Write-Behind).
    
*   **Trade-offs:** RAM is vastly more expensive than SSD storage. You cannot cache everything.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Caching highly unique, dynamic data (like free-text search queries: "blue shoes size 10"). You will fill up your expensive RAM with millions of unique JSON blobs that nobody else will ever request.
    
*   **Interview Pointers:** If asked to design a system with overwhelming **Write** traffic, suggest a **Write-Behind** cache. But immediately warn the interviewer: _"If the Redis server loses power before flushing to disk, we permanently lose those writes."_
    

Topic 4.2: Cache Eviction & Invalidation (The Stale Data Nightmare)
-------------------------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** \* **Eviction:** Your desk is physically full. You must throw a paper in the trash to make room for a new one.
    
    *   **Invalidation:** The bank balance paper on your desk says $50, but the user just deposited $100 in the master filing cabinet. Your paper is a lie. You must destroy it.
        
*   **Why it exists (Problem Solved):** Preventing Out-Of-Memory (OOM) server crashes and preventing the system from serving mathematically incorrect/stale business data to users.
    
*   **Core Idea:** Rules for deleting data from RAM when it is no longer useful or accurate.
    
*   **Key Terms:** LRU (Least Recently Used), LFU (Least Frequently Used), TTL (Time-To-Live), Active Invalidation, Cache Stampede.
    

### Layer 2: Deep Dive

*   **Key Internals (The LRU Cache):** The industry standard for eviction. To achieve $O(1)$ reads and writes, it combines two data structures:
    
    1.  **Hash Map:** For $O(1)$ lookups. The value is a memory pointer to a linked list node.
        
    2.  **Doubly Linked List:** Tracks recency. When a key is read/written, you snip its node and move it to the "Head". When RAM is full, you delete the "Tail".
        
*   **Real-World Usage:** Twitter trending topics (Time-To-Live expiration), E-commerce inventory (Active Invalidation).
    
*   **Trade-offs:** Active Invalidation (App deleting cache on DB update) is prone to race conditions if network packets arrive out of order.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** \* The "Cache Stampede" (Thundering Herd). A heavy 10-second query's cache expires. 5,000 users hit the API at that exact second. All 5,000 get a Cache Miss and hit the database simultaneously, instantly melting the DB. _(Fix: Use Mutex Locks so only the first thread queries the DB while the others wait 100ms)._
    
    *   Using a pure Doubly Linked List for LRU without a Hash Map, resulting in disastrous $O(N)$ lookup times.
        
*   **Interview Pointers:** Always state that you will use a **TTL (Time-To-Live) as a safety net**, even if you are using Active Invalidation. If the "delete cache" network call fails, the TTL guarantees the stale data will eventually destroy itself.
    

Topic 4.3: Redis Architecture Deep Dive
---------------------------------------

### Layer 1: Core Recall

*   **Intuition:** It's not just a giant HashMap on another computer. It is a data-structure server that understands how to sort lists and do math purely in RAM.
    
*   **Why it exists (Problem Solved):** Relational databases use B+ Trees on disks, which are too slow and lock-heavy for operations like incrementing a live counter 100,000 times a second.
    
*   **Core Idea:** A lightning-fast, single-threaded, in-memory engine for atomic operations.
    
*   **Key Terms:** Single-Threaded Event Loop, I/O Multiplexing, ZSET (Sorted Set), RDB Snapshot, AOF (Append-Only File).
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **Single-Threaded:** Redis executes all commands sequentially on _one CPU thread_. Because RAM is nanosecond-fast, the CPU isn't the bottleneck—the network is. Single-threading completely eliminates the need for Mutex Locks, making every operation mathematically **Atomic**.
        
    *   **ZSET (Sorted Set):** Uses a "Skip List" under the hood to keep items perfectly sorted by a score in $O(\\log N)$ time.
        
    *   **Persistence:** RDB takes full RAM snapshots (Copy-on-Write). AOF logs every single command to a text file for zero-data-loss recovery.
        
*   **Real-World Usage:** Live Gaming Leaderboards (ZSET), API Rate Limiting (INCR), Distributed Locks (SETNX), Session Storage.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Running KEYS \* in production. Because Redis is single-threaded, asking it to scan 50 million keys will freeze the entire server for seconds, timing out every other microservice relying on it. _(Fix: Use the SCAN command for pagination)._
    
*   **Interview Pointers:**
    
    *   If asked to build a **Global Leaderboard**, instantly say "Redis Sorted Sets (ZSET)".
        
    *   If asked how to prevent race conditions when 10,000 users click "Buy" on 100 flash-sale iPhones, say "Redis DECR (Decrement). Because Redis is single-threaded, it naturally queues the requests in perfect order without explicit row locking."
