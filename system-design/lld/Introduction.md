# Chapter 1 — Introduction to LLD

## 1. High-Level Design (HLD)

HLD is a **general system design** — it gives the bird's-eye view of the entire system.

It covers:
- Selection of components, platforms, and tools
- Database design (which DB to use, replication strategy, etc.)
- Brief description of relationships between services and modules

> **Example:** User → Load Balancer → User Service → DB (primary + replica)  
> HLD just shows *what* components exist and *how* they connect — not the internal code structure.

---

## 2. Low-Level Design (LLD)

LLD is a **component-level design** that follows a step-by-step refinement process.

- LLD is created *based on* the HLD
- It describes **class diagrams** — methods, attributes, and relationships between classes
- It defines program specs in enough detail that a programmer can **directly write code from the document**

> **Example:** The `UserService` from HLD becomes a class with:
> - `List<User> getUserList()`
> - `void deleteUser(id)`
> - And relationships to `User`, `Address`, `Profile` classes shown via a class diagram

---

## 3. How to Form LLD from HLD

Each service/component in the HLD is broken down into detailed class diagrams using:

| Tool | Purpose |
|------|---------|
| **UML Diagrams** | Visually represent classes, relationships, and flow |
| **Object Oriented Principles** | Encapsulation, Inheritance, Abstraction, Polymorphism |
| **SOLID Principles** | Guidelines to write clean, maintainable OO code |

> HLD → (using above tools) → LLD (Class Diagrams with methods & relations)

---

## 4. UML Diagrams

UML (Unified Modeling Language) diagrams are divided into two categories:

### Structural UML Diagrams
> Describe the *static structure* of the system (what exists)

| Diagram | Notes |
|---------|-------|
| **Class Diagram** ⭐ | Most important for LLD — shows classes, attributes, methods, relationships |
| Object Diagram | Snapshot of objects at a point in time |
| Package Diagram | Groups related classes |
| Component Diagram | Shows system components and their interfaces |
| Composite Structure Diagram | Internal structure of a class |
| Deployment Diagram | Physical deployment of system |
| Profile Diagram | Extensions to UML |

### Behavioural UML Diagrams
> Describe the *dynamic behaviour* of the system (what happens)

| Diagram | Notes |
|---------|-------|
| **Sequence Diagram** ⭐ | Shows interaction between objects over time |
| **Use Case Diagram** ⭐ | Shows actors and their interactions with the system |
| **Activity Diagram** ⭐ | Shows workflow / flow of control |
| State Diagram | Shows states of an object and transitions |
| Communication Diagram | Like sequence diagram but focuses on object links |
| Interaction Overview Diagram | High-level flow of interactions |
| Timing Diagram | Behaviour over time |

> ⭐ = Most commonly used in LLD interviews

---

## Summary

```
HLD  →  "What components exist and how they talk"
LLD  →  "How each component is internally structured (classes, methods, relations)"

LLD is built using:
  - UML Diagrams (especially Class Diagrams)
  - OOP Principles
  - SOLID Principles
```
