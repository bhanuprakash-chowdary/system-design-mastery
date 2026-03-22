Day 8: Deployment & Orchestration (Shipping the Code)
=====================================================

Topic 8.1: Containerization (Docker)
------------------------------------

### Layer 1: Core Recall

*   **Intuition:** The standardized steel shipping container. Before containers, you hand-packed ships with barrels and sacks (custom server setup). Now, you put your code in a standard box, and the server just needs to know how to run the box.
    
*   **Why it exists (Problem Solved):** "It works on my machine!" Solves dependency hell and environment parity (Mac vs. Linux, Java 11 vs. Java 17).
    
*   **Core Idea:** Package the application, its runtime (JRE), and OS dependencies into a single, immutable, runnable unit.
    
*   **Key Terms:** Docker Image, Docker Container, Dockerfile, Namespaces, Cgroups, Ephemeral Storage.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **Not a Virtual Machine:** VMs run a heavy, full "Guest OS" (takes minutes to boot, wastes GBs of RAM). Docker shares the _Host OS Linux Kernel_.
        
    *   **Namespaces:** Provides isolation (tricks the app into thinking it's the only one on the computer).
        
    *   **Cgroups:** Provides resource limiting (strictly limits CPU and RAM usage).
        
    *   **Images vs. Containers:** An Image is a read-only blueprint. A Container is a running process with a temporary "Read-Write" layer on top.
        
*   **Real-World Usage:** Local developer onboarding (docker-compose up to boot DBs and queues instantly), completely clean CI/CD test environments.
    
*   **Trade-offs:** Containers are _ephemeral_. If a container restarts, all data written to its local disk is permanently deleted. (You must use **Docker Volumes** to map host hard drives for databases).
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** \* Massive Images (using ubuntu:latest instead of alpine). **Fix:** Use **Multi-Stage Builds** to compile the code and then drop only the tiny compiled .jar into a fresh, minimal runtime image.
    
    *   Running processes as root inside the container (massive security vulnerability if the app is breached).
        
*   **Interview Pointers:** If asked the difference between a VM and a Container, immediately say: _"Containers virtualize the Operating System (sharing the kernel), while VMs virtualize the physical hardware."_
    

Topic 8.2: Container Orchestration (Kubernetes / K8s)
-----------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** The automated port authority. Docker makes the boxes; Kubernetes (K8s) is the massive crane system that loads, unloads, and replaces the boxes if they fall in the ocean.
    
*   **Why it exists (Problem Solved):** You cannot manually type docker run across 500 servers. You need automated self-healing, auto-scaling, and zero-downtime deployments.
    
*   **Core Idea:** You write a "Declarative Blueprint" (YAML) saying "I want 5 copies of this app." K8s constantly monitors the servers to ensure exactly 5 copies are running.
    
*   **Key Terms:** Control Plane, Worker Node, Pod, Deployment, Service, kubelet, HPA (Horizontal Pod Autoscaler).
    

### Layer 2: Deep Dive

*   **Key Internals (The Architecture):**
    
    *   **The Brain (Control Plane):** kube-apiserver (the front door), etcd (the database tracking cluster state), kube-scheduler (finds servers with free RAM for new Pods).
        
    *   **The Muscle (Worker Nodes):** Physical EC2 servers running the kubelet agent.
        
    *   **The Objects:** \* **Pod:** The smallest unit. A mortal, disposable wrapper around your Docker container.
        
        *   **Deployment:** Manages the Pods (e.g., handles scaling and rolling updates).
            
        *   **Service:** Provides a permanent, stable internal IP address and load balances traffic across the constantly dying/respawning Pods.
            
*   **Real-World Usage:** Spotify's autonomous squads, Pokemon GO's massive global launch.
    
*   **Trade-offs:** Massive learning curve. It is architectural overkill for a 2-service startup. Historically terrible for Stateful apps (databases).
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Forgetting to set **CPU/Memory Limits** in the YAML. If a Java Pod leaks memory without a limit, it will consume 100% of the host server's RAM and the Linux kernel will violently crash the entire node.
    
*   **Interview Pointers:** \* Be ready to explain a **Rolling Update**: K8s spins up one v2 Pod, waits for it to be healthy, kills one v1 Pod, and repeats. This guarantees zero downtime.
    
    *   Emphasize that apps in K8s _must_ be stateless. Session data must live in Redis, not in the Pod's RAM, because K8s will murder Pods without warning to rebalance the cluster.
        

Topic 8.3: CI/CD Pipelines (The Automated Assembly Line)
--------------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** Henry Ford's automated car assembly line. If a robot detects a flaw in the brakes, the belt physically stops. No broken car leaves the factory.
    
*   **Why it exists (Problem Solved):** "Deployment Friday" panic. Manual deployments involve human error, downtime, and massive stress.
    
*   **Core Idea:** Every time code is pushed to Git, a server automatically builds it, runs all tests, and deploys it to production without human hands.
    
*   **Key Terms:** CI (Continuous Integration), CD (Continuous Delivery/Deployment), Blue/Green Deployment, Canary Release.
    

### Layer 2: Deep Dive

*   **Key Internals (The Flow):**
    
    *   **CI (Proving it works):** Checkout Code -> Run Security Linter -> **Run Unit Tests** -> Compile -> docker build -> Push Image to remote Registry. (If _any_ step fails, the pipeline halts immediately).
        
    *   **CD (Shipping it):** Update K8s Manifests -> kubectl apply -> Monitor deployment.
        
*   **Real-World Usage:** GitHub Actions, GitLab CI, Jenkins. Amazon deploying code every 11 seconds.
    
*   **Trade-offs:** A pipeline is only as good as its tests. "Flaky tests" (tests that randomly fail 10% of the time due to bad code) will constantly block deployments and infuriate the engineering team.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Testing against a live, shared "Dev Database". If two pipelines run simultaneously, they overwrite each other's data and tests fail. _(Fix: Use disposable in-memory DBs like H2, or Testcontainers)._
    
*   **Interview Pointers:** Know your deployment strategies:
    
    *   **Blue/Green:** Spin up a fully identical v2 environment alongside v1. Flip the Load Balancer to point 100% of traffic to v2 instantly. Easy rollback.
        
    *   **Canary Release:** Deploy v2 to exactly 1% of users. Monitor error rates for 10 minutes. If stable, roll out to 100%. This aggressively minimizes the "Blast Radius" of a hidden bug.
