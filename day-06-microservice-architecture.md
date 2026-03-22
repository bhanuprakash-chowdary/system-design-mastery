Day 6: Microservices Architecture (Split, Route, & Connect)
===========================================================

Topic 6.1: Monolith vs. Microservices
-------------------------------------

### Layer 1: Core Recall

*   **Intuition:** Building a house. A Monolith is one giant contractor doing everything (plumbing, electrical, framing). Microservices are 5 specialized teams working simultaneously.
    
*   **Why it exists (Problem Solved):** In a massive Monolith, 500 developers working in one Git repository cause **Deployment Friction** (one typo breaks the whole app) and **Resource Waste** (scaling the whole app just because the image-processing module needs more CPU).
    
*   **Core Idea:** Break the massive application into small, independent, separately deployable services grouped by business capability.
    
*   **Key Terms:** Monolith, Microservices, Domain-Driven Design (DDD), Bounded Context, Distributed Monolith.
    

### Layer 2: Deep Dive

*   **Key Internals (The Golden Rule):** Microservices **must not share a database**. If OrderService and BillingService read/write to the exact same Postgres table, you have built a "Distributed Monolith" (all the deployment pain, none of the decoupling).
    
*   **Real-World Usage:** Amazon's "Two-Pizza Teams" (a team small enough to be fed by two pizzas owns exactly one microservice end-to-end), Netflix, Uber.
    
*   **Trade-offs:** You trade CPU speed (in-memory function calls) for Organizational speed (independent deployments). Microservices introduce horrific network latency and the loss of ACID distributed transactions.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Splitting by technical layer (DatabaseService, BusinessLogicService) instead of vertical business domains (CheckoutService).
    
*   **Interview Pointers:** \* Never start a system design interview by saying "I will build 50 microservices." State that you would **start with a modular monolith** and only split it when organizational scaling or independent hardware scaling becomes necessary.
    
    *   If asked how Service A gets data from Service B without sharing a DB, mention **API Composition** (sync fetching) or **Eventual Consistency** (async caching via Kafka events).
        

Topic 6.2: API Gateway
----------------------

### Layer 1: Core Recall

*   **Intuition:** The Concierge desk at a 5,000-room hotel. Instead of wandering the halls to find the chef or the accountant, you make all requests to the Concierge, who routes them to the right room.
    
*   **Why it exists (Problem Solved):** Mobile apps shouldn't have to memorize 50 different IP addresses and make 50 slow 3G network calls to load one screen. Also, internal microservices shouldn't be exposed directly to the public internet.
    
*   **Core Idea:** A specialized reverse proxy that sits at the edge of your network, acting as the single entry point for all client traffic.
    
*   **Key Terms:** Reverse Proxy, SSL Termination, API Aggregation, BFF (Backend For Frontend), Rate Limiting.
    

### Layer 2: Deep Dive

*   **Key Internals (The Gateway Pipeline):** When an HTTP request arrives, it passes through a strict chain:
    
    1.  **Edge Security:** Decrypts SSL/TLS.
        
    2.  **AuthZ/AuthN:** Validates the JWT token cryptographically.
        
    3.  **Rate Limiting:** Checks Redis to see if the IP is spamming.
        
    4.  **Routing/Aggregation:** Forwards the packet to the internal IP, or fans out to 3 services and merges the JSON responses into one.
        
*   **Real-World Usage:** Netflix Zuul, AWS API Gateway, Kong.
    
*   **Trade-offs:** It creates a Single Point of Failure (SPOF). It must be highly available and sitting behind an L4 Network Load Balancer.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** The "Enterprise Service Bus (ESB)" Anti-pattern: Putting heavy business logic or complex data transformations inside the API Gateway. The Gateway should be "Dumb Pipes, Smart Endpoints."
    
*   **Interview Pointers:** Bring up the **BFF (Backend For Frontend)** pattern. If designing an app with a Web UI and an iOS UI, explain that you would build a separate Gateway for each, so the iOS Gateway can strictly format the JSON for a mobile screen size, saving bandwidth.
    

Topic 6.3: Service Discovery
----------------------------

### Layer 1: Core Recall

*   **Intuition:** The city phonebook. If your friend moves apartments every single day (ephemeral containers), you cannot hardcode their address. You must check the phonebook right before you send the letter.
    
*   **Why it exists (Problem Solved):** In the cloud, servers auto-scale and die constantly. Hardcoded IP addresses (e.g., 10.0.1.55) will break within hours.
    
*   **Core Idea:** A central registry that dynamically maps a logical service name (http://billing-service) to a live list of healthy IP addresses.
    
*   **Key Terms:** Service Registry, Heartbeat, Client-Side Discovery, Server-Side Discovery.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **The Registry:** A highly available database (Consul, Eureka, Zookeeper).
        
    *   **The Heartbeat:** Microservices _must_ ping the registry every ~10 seconds. If a service crashes, the heartbeat stops, and the registry deletes the IP from the phonebook.
        
    *   **Client-Side vs. Server-Side:** In Client-Side, the Java app queries the registry and load-balances itself. In Server-Side (like Kubernetes), the app talks to an internal proxy, which talks to the registry transparently.
        
*   **Real-World Usage:** Kubernetes CoreDNS (Server-side), HashiCorp Consul.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** The "Zombie Service". A service's HTTP web threads freeze, but its background "Heartbeat" thread keeps telling the registry it is alive. Traffic routes to a dead server. _(Fix: Heartbeats must be tied to Deep Health Checks, like querying the local DB)._
    
*   **Interview Pointers:** Service Registries must favor **Availability (AP) over Consistency (CP)** in the CAP theorem. Returning a slightly stale list of IPs (some might be dead, which the client can retry) is infinitely better than locking up the registry and blinding the entire cluster.
    

Topic 6.4: Inter-Service Communication (REST vs. gRPC)
------------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** REST is writing a letter in plain English (readable, bulky). gRPC is using a secret decoder ring to tap Morse code (unreadable to humans, but blisteringly fast and compressed).
    
*   **Why it exists (Problem Solved):** When 50 microservices talk to each other, parsing JSON strings over HTTP/1.1 wastes massive amounts of CPU cycles and network bandwidth.
    
*   **Core Idea:** Swap text-based JSON for strictly typed, binary serialization over persistent HTTP/2 connections.
    
*   **Key Terms:** gRPC, Protocol Buffers (Protobuf), HTTP/2, Multiplexing, Binary Serialization.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **Protobuf (.proto):** You define a strict schema (string name = 1;). gRPC does not send the word "name" over the network. It just sends the binary tag 1 and the value. This compresses payloads by 3x-10x.
        
    *   **HTTP/2 Multiplexing:** Instead of opening and closing a TCP connection for every request, HTTP/2 keeps _one_ pipe open and sends thousands of parallel binary frames through it simultaneously.
        
*   **Real-World Usage:** Uber's internal service mesh, Google.
    
*   **Trade-offs:** You cannot read a gRPC payload in Wireshark or Postman without the .proto file. Web browsers do not natively support it (forcing the industry standard: **REST for Frontend, gRPC for Backend**).
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Changing a Protobuf field tag number (e.g., changing string name = 1; to string name = 2;). This permanently breaks backward compatibility, causing older services to drop the data.
    
*   **Interview Pointers:** When calculating capacity or latency in an interview, mention that you will use gRPC for internal backend communication because it skips the CPU-heavy text-parsing layer, mapping binary bytes directly into RAM objects.
