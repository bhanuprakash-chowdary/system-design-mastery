Day 10: The System Design Interview Blueprint
=============================================

Topic 10.1: The 5-Step Interview Framework
------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** A System Design interview is not a coding test; it is a collaborative technical meeting. You must physically drive the agenda.
    
*   **Why it exists (Problem Solved):** "Design Twitter" is an impossibly vague prompt. If you jump straight to drawing boxes, you will design the wrong system and fail.
    
*   **Core Idea:** A standardized, 45-minute roadmap to break down massive ambiguity into a scalable architecture.
    
*   **Key Terms:** Functional Requirements, Non-Functional Requirements, SPOF (Single Point of Failure), Happy Path.
    

### Layer 2: Deep Dive

*   **Key Internals (The 5 Steps):**
    
    1.  **Clarify Requirements (5 mins):** Lock down the _Functional_ (What does it do? "Users can tweet and follow") and _Non-Functional_ (Scale, Latency, CAP theorem choices).
        
    2.  **Capacity Estimation (5 mins):** Back-of-the-envelope math (RPS, Storage).
        
    3.  **API & Data Model (10 mins):** Define REST/gRPC endpoints and choose the right databases _before_ drawing servers.
        
    4.  **High-Level Architecture (10 mins):** Draw the "Happy Path" (Client -> DNS -> CDN -> LB -> Gateway -> Service -> DB).
        
    5.  **Deep Dive & Bottlenecks (15 mins):** Identify Single Points of Failure and fix them with Caches, Queues, and Sharding.
        

### Layer 3: Interview Mastery

*   **Common Mistakes:** Wasting 15 minutes designing a video-encoding pipeline when the interviewer only cared about the real-time news feed. _Always ask what features to prioritize._
    
*   **Interview Pointers:** Talk out loud constantly. If you are silently thinking for 2 minutes, you are losing points. Use phrases like, _"Assuming we prioritize Availability over perfect Consistency, I will design this as an AP system."_
    

Topic 10.2: Capacity Estimation (The Math Cheat Codes)
------------------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** The architect's mental scratchpad. You don't need a calculator; you need to know if you are building a system for 10 users or 100 million.
    
*   **Why it exists (Problem Solved):** Proves to the interviewer that you understand physical hardware limitations and can mathematically justify your database choices.
    
*   **Core Idea:** Memorize a few conversion ratios to quickly estimate Traffic (RPS), Bandwidth, and Storage.
    
*   **Key Terms:** RPS (Requests Per Second), DAU (Daily Active Users), Read-to-Write Ratio, Peak Traffic.
    

### Layer 2: Deep Dive

*   **Key Internals (The Cheat Codes):**
    
    *   **Traffic Magic Number:** **100,000 requests per day $\\approx$ 1.2 RPS.** (Therefore, 10 Million/day $\\approx$ 120 RPS).
        
    *   **Peak Traffic:** Always assume peak traffic is **2x to 3x** the average RPS.
        
    *   **Storage Standards:** \* 1 simple JSON/DB row $\\approx$ **1 KB**
        
        *   1 standard image $\\approx$ **1 MB**
            
        *   1 short video $\\approx$ **50 MB**
            
*   **The Math in Action:** 10 Million users uploading 1 image a day = 10,000,000 MB = **10 Terabytes (TB) per day**.
    

### Layer 3: Interview Mastery

*   **Common Mistakes:** Calculating exact decimals like "124.7 RPS". It wastes time. Round up aggressively to easy numbers (e.g., "Let's call it 150 RPS").
    
*   **Interview Pointers:** Do not just do the math; _react_ to the math. If you calculate 18 Petabytes of storage over 5 years, turn to the interviewer and say: _"Because 18 PB won't fit in a relational database, I must split the storage. Images go to AWS S3, and metadata goes to a sharded NoSQL database."_
    

Topic 10.3: Interview Psychology & Execution
--------------------------------------------

### Layer 1: Core Recall

*   **Intuition:** The interviewer is your new coworker. They are testing if they actually want to work with you on a difficult project.
    
*   **Why it exists (Problem Solved):** Brilliant coders fail this interview because they get defensive or refuse to accept feedback.
    
*   **Core Idea:** Manage trade-offs gracefully. Every architectural choice has a downside; you must be the one to point it out.
    
*   **Key Terms:** Trade-offs, Defensive Engineering, Collaboration, Redirection.
    

### Layer 2: Deep Dive

*   **Key Internals:**
    
    *   **State your Assumptions:** "I am going to assume a 10:1 Read-to-Write ratio. Does that sound reasonable to you?"
        
    *   **Acknowledge the Negative:** Whenever you introduce a technology, state its flaw. "I'll use a Write-Behind Redis cache for speed, _but the trade-off is we might lose data if the server loses power._"
        
    *   **Handling Pushback:** If the interviewer says, "I think that database will crash," do not argue. Say, "That is a great point. If we hit that limit, I would introduce a Kafka queue to act as a shock absorber."
        

### Layer 3: Interview Mastery

*   **Common Mistakes:** Trying to design the "perfect" system. There is no perfect system. A system optimized for read-speed will have high storage costs. A system optimized for consistency will be slow.
    
*   **Interview Pointers:** If you get completely stuck, zoom out. Go back to the client and trace the life of a single HTTP request step-by-step. The bottleneck will usually reveal itself naturally.
