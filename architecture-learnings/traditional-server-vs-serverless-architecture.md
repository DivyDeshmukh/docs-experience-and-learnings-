# Server vs Serverless Architecture — Short Notes

## Server Architecture (Traditional)

### Definition
Applications run on dedicated or virtual servers that are always running and managed by the development or DevOps team.

#### Architecture Diagram
```
┌─────────────────────────────────────────────┐
│         Load Balancer (nginx)               │
└──────────────┬──────────────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌─────────────┐ ┌─────────────┐
│  Server 1   │ │  Server 2   │
│             │ │             │
│  Express.js │ │  Express.js │
│  Always On  │ │  Always On  │
└──────┬──────┘ └──────┬──────┘
       │               │
       └───────┬───────┘
               │
               ▼
        ┌─────────────┐
        │  PostgreSQL │
        │  (Always On)│
        └─────────────┘
```

### Characteristics
- Long-lived processes
- Persistent server instances
- Manual or auto-managed scaling
- Direct database connections
- Fixed resource allocation

### Responsibilities
- Server provisioning
- OS updates and security patches
- Load balancer configuration
- Scaling and monitoring
- Downtime handling

### Pros
- Full control over infrastructure
- Predictable performance
- Suitable for long-running tasks
- Easier connection-based systems (WebSockets, queues)

### Cons
- Higher operational overhead
- Pay for idle resources
- Slower scaling
- Requires DevOps expertise

### Examples
- EC2
- DigitalOcean Droplets
- Bare metal servers
- Kubernetes clusters

---

## Serverless Architecture

### Definition
Code runs as short-lived functions executed on demand, without managing servers.

#### Architecture Diagram
```
┌─────────────────────────────────────────────┐
│         CDN Edge Network (Global)            │
└──────────────┬──────────────────────────────┘
               │
        Request comes in
               │
               ▼
┌─────────────────────────────────────────────┐
│   Serverless Function Orchestrator           │
│   (AWS Lambda / Vercel Functions)            │
└──────────────┬──────────────────────────────┘
               │
       ┌───────┴────────┐
       │                │
       ▼                ▼
┌────────────┐   ┌────────────┐
│ Function 1 │   │ Function 2 │
│ (Spawns on │   │ (Spawns on │
│  demand)   │   │  demand)   │
│            │   │            │
│ Lives: 5s  │   │ Lives: 3s  │
│ Then dies  │   │ Then dies  │
└─────┬──────┘   └─────┬──────┘
      │                │
      └────────┬───────┘
               │
               ▼
     ┌──────────────────┐
     │ Connection Pooler │
     │  (15 persistent)  │
     └────────┬──────────┘
              │
              ▼
       ┌─────────────┐
       │  Supabase   │
       │  (Always On)│
       └─────────────┘
```

### Characteristics
- Ephemeral execution (stateless)
- Auto-scaling by platform
- Pay-per-request model
- No persistent connections
- Event-driven execution

### Execution Model
- Each request spawns a function instance
- Functions live for milliseconds to seconds
- Instances are destroyed after execution

### Pros
- Zero server management
- Automatic scaling
- Cost-efficient for low to medium traffic
- Fast development and deployment
- Ideal for APIs and microservices

### Cons
- Cold start latency
- Difficult long-running tasks
- Database connection challenges
- Limited execution time
- Less infrastructure control

### Examples
- Vercel Functions
- AWS Lambda
- Cloudflare Workers
- Netlify Functions

---

### What Are Cloud Functions?
- Definition
- Cloud functions (also called serverless functions, lambda functions, or edge functions) are small, single-purpose pieces of code that run in isolated, ephemeral containers managed entirely by a cloud provider.
- They execute in response to events (HTTP requests, database changes, file uploads, etc.) and automatically shut down after completion.

## Key Differences

| Aspect | Server | Serverless |
|------|--------|------------|
| Runtime | Always running | On-demand |
| Scaling | Manual / auto-scaling | Automatic |
| Cost model | Pay for uptime | Pay per execution |
| State | Persistent | Stateless |
| DB connections | Persistent | Must be pooled |
| DevOps effort | High | Minimal |

---

## When to Use Server Architecture
- High sustained traffic
- Long-running background jobs
- WebSockets and real-time systems
- Heavy database write workloads
- Complex infrastructure requirements

## When to Use Serverless Architecture
- APIs and backend-for-frontend
- SaaS dashboards
- MVPs and startups
- Event-driven workloads
- Irregular or burst traffic

---

## Summary

Server architecture provides control and stability but requires operational management.  
Serverless architecture prioritizes scalability and speed with reduced infrastructure responsibility, at the cost of execution constraints and connection limitations.

Choosing between them depends on workload type, traffic pattern, and operational requirements.
