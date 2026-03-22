Day 1: Networking & Load Balancing
==================================

Topic 1.1: The Network Foundation (TCP, UDP, & DNS)
---------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** \* TCP is a certified phone call (you know they heard you).
    
    *   UDP is throwing a postcard in the mail (you hope it arrives).
        
    *   DNS is the internet's phonebook (translating names to numbers).
        
*   **Why it exists (Problem Solved):** Computers only understand IP addresses and raw bytes. We need reliable ways to transmit those bytes across failing physical wires, and human-readable names (like google.com) to find the servers.
    
*   **Core Idea:** TCP guarantees order and delivery at the cost of speed. UDP guarantees speed at the cost of reliability.
    
*   **Key Terms:** 3-Way Handshake, Packet, IP Address, DNS Record (A, CNAME), TTL (Time to Live).
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **TCP 3-Way Handshake:** SYN (Hello) -> SYN-ACK (I hear you, Hello) -> ACK (I hear you too. Let's talk).
        
    *   **DNS Resolution Chain:** Browser Cache -> OS Cache -> ISP Resolver -> Root Server -> TLD Server -> Authoritative Nameserver.
        
*   **Real-World Usage:**
    
    *   **TCP:** HTTP/REST APIs, Database Connections, File Uploads, Emails.
        
    *   **UDP:** Zoom calls, Multiplayer Gaming (Fortnite), Live Video Streaming.
        
*   **Trade-offs:** TCP requires network overhead for acknowledgments. UDP has zero overhead but suffers from packet loss out-of-order delivery.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Attempting to use TCP for real-time multiplayer gaming. The "Head-of-Line Blocking" problem will cause the game to freeze entirely while waiting for one dropped packet to be re-transmitted.
    
*   **Interview Pointers:** \* If an interviewer asks "What happens when you type https://www.google.com/url?sa=E&source=gmail&q=google.com?", start with DNS resolution, not the HTTP request.
    
    *   Be prepared to explain why DNS heavily relies on UDP (for speed) but falls back to TCP if the payload exceeds 512 bytes.
        

Topic 1.2: Load Balancing (Layer 4 vs Layer 7)
----------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** The traffic cop standing in front of your 50 identical restaurants, pointing new customers to the shortest line.
    
*   **Why it exists (Problem Solved):** A single server will eventually melt under high traffic (Vertical Scaling limits). We need to distribute traffic across many servers (Horizontal Scaling) while exposing only one public IP address.
    
*   **Core Idea:** Sit a reverse proxy in front of your microservices to mathematically distribute incoming requests.
    
*   **Key Terms:** Reverse Proxy, Layer 4 (Transport), Layer 7 (Application), Round Robin, SSL Termination.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **Layer 4 (L4) Load Balancing:** Operates at the Transport layer. It only looks at the IP address and TCP Port. It is "dumb" but blisteringly fast. It does not look at the HTTP payload.
        
    *   **Layer 7 (L7) Load Balancing:** Operates at the Application layer. It decrypts the SSL certificate, reads the actual HTTP URL path (e.g., /api/payments), and routes based on the content.
        
*   **Real-World Usage:**
    
    *   **L4:** AWS Network Load Balancer (NLB). Used for raw, massive-scale gaming or database traffic throughput.
        
    *   **L7:** AWS Application Load Balancer (ALB), NGINX. Used for API Gateways and routing specific URLs to specific microservices.
        
*   **Common Algorithms:** Round Robin (take turns), Least Connections (send to the least busy server), IP Hash (same user always goes to the same server).
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** \* Trying to route traffic based on a JWT token or URL path using an L4 Load Balancer. It physically cannot read them.
    
    *   Leaving SSL Decryption to the individual microservices instead of terminating it at the L7 Load Balancer (wastes massive backend CPU).
        
*   **Interview Pointers:**
    
    *   When asked how to scale a stateful system (like a multiplayer game server or a shopping cart cache), immediately mention **Consistent Hashing** or **IP Hash** at the Load Balancer level so a user is "sticky" to a specific server.
