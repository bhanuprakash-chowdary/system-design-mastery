Day 7: Resilience & Distributed Transactions
============================================

Topic 7.1: Resilience Patterns (Preventing Cascading Failures)
--------------------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** \* **Circuit Breaker:** The electrical panel in your house. If the microwave shorts out, it trips the breaker to save the rest of the house from catching fire.
    
    *   **Bulkhead:** The waterproof compartments in a submarine. If the front floods, you seal the door so the whole ship doesn't sink.
        
*   **Why it exists (Problem Solved):** In microservices, one slow database in Service B will cause Service A to wait. Service A's threads will block, its RAM will fill up, and it will crash. This chain reaction (Cascading Failure) can take down an entire company.
    
*   **Core Idea:** Assume network calls will fail. Isolate failures and "Fail Fast" to protect system resources.
    
*   **Key Terms:** Cascading Failure, Circuit Breaker, Bulkhead, Timeout, Retry with Jitter, Fallback Method.
    

### Layer 2: Deep Dive

*   **Key Internals (The 4 Core Shields):**
    
    1.  **Timeouts:** The bare minimum. Never wait infinitely. Cut the connection after 200ms to free up the thread.
        
    2.  **Retries + Jitter:** Retry transient errors, but use Exponential Backoff (wait 1s, 2s, 4s) and Jitter (random variance) to avoid a "Thundering Herd" of synchronized retries melting the struggling server.
        
    3.  **Circuit Breaker:** A state machine. **CLOSED** (normal, traffic flows). If error rate > 50%, it trips **OPEN** (instantly rejects all local calls to save CPU). After 30s, it goes **HALF-OPEN** (lets 1 request through to test the waters).
        
    4.  **Bulkheads:** Physically partitioning Thread Pools. Give 50 threads to /checkout and 50 to /search. If search hangs, checkout is completely unaffected.
        
*   **Real-World Usage:** Netflix (Resilience4j/Hystrix). If the Recommendation engine dies, the Circuit Breaker trips and returns a cached "Top 10 Movies" list instead of crashing the UI.
    
*   **Trade-offs:** Circuit Breakers live in local memory. If you have 50 instances of Service A, Instance 1 might trip its breaker while Instance 2 keeps blindly firing.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Retrying an HTTP 400 Bad Request. A 400 means your payload is malformed; retrying it will mathematically always fail. Only retry 500, 502, 503, or Network Timeouts.
    
*   **Interview Pointers:** A Circuit Breaker is functionally useless without a **Fallback Method**. If the breaker trips, you must explicitly code what happens next (e.g., return default data, return cached data, or drop the non-critical feature gracefully).
    

Topic 7.2: Distributed Transactions (Saga & 2PC)
------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** Booking a vacation. You successfully book the Flight and Hotel, but your credit card declines for the Rental Car. You cannot magically "rollback" the database; you must explicitly call the airline and cancel the flight to get your money back.
    
*   **Why it exists (Problem Solved):** When you split a monolith into microservices, you lose ACID Atomicity. You cannot wrap a START TRANSACTION around two different databases across a network.
    
*   **Core Idea:** A strategy to guarantee that either all microservices succeed, or the system safely reverts to its original state.
    
*   **Key Terms:** Two-Phase Commit (2PC), Saga Pattern, Choreography, Orchestration, Compensating Transaction.
    

### Layer 2: Deep Dive

*   **Key Internals (The Two Approaches):**
    
    *   **Two-Phase Commit (2PC):** A central coordinator asks all databases to "Prepare" (lock their rows). If all say yes, it sends "Commit". _Verdict:_ **Do not use this.** It physically locks rows across the network and violates the CAP theorem's Availability.
        
    *   **The Saga Pattern:** Breaks the transaction into a sequence of local ACID transactions. If Step 3 fails, it executes a **Compensating Transaction** (explicit code to reverse Step 1 and Step 2).
        
*   **Saga Architectures:**
    
    1.  **Choreography (Event-Driven):** No boss. Services publish events to Kafka. Other services listen and react. (Great for simple workflows, becomes "Spaghetti Architecture" for complex ones).
        
    2.  **Orchestration (Command-Driven):** The Boss. A central microservice (like AWS Step Functions) explicitly tells other services what to do and tracks the global state.
        
*   **Real-World Usage:** Uber trip payments (Cadence/Temporal Orchestration), E-commerce returns.
    
*   **Trade-offs:** You lose the "I" (Isolation) in ACID. Other users might see intermediate states (e.g., money deducted but order not yet confirmed) before the Saga completes.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Failing to make Compensating Transactions **Idempotent**. If the Orchestrator commands the Payment Service to "Refund $50" because the Saga failed, and the network blips, the Orchestrator will retry. If not idempotent, you will refund the user $100.
    
*   **Interview Pointers:** If the interviewer asks about database transactions in a microservice architecture, immediately reject 2PC. State confidently: _"Because we cannot lock resources across a distributed network without destroying availability, we must embrace Eventual Consistency using the Saga Pattern."_
