# Chapter 2 — UML Diagrams

## 1. Overview of UML

**UML (Unified Modeling Language)** is used to model the Object-Oriented Analysis of a software system.

- It is a **collection of diagrams** that helps engineers, businesspeople, and system architects understand the **behavior and structure** of the system being designed.
- It is a **standard visual language** — universally understood across teams and organizations.

### Benefits of UML
- Develop a quick understanding of a software system
- Breaks complex systems into discrete, understandable pieces
- Graphical notations can be used to communicate design decisions
- Easy to hand over the system to the next team

---

## 2. Types of UML Diagrams

| Structural UML Diagrams | Behavioural UML Diagrams |
|------------------------|--------------------------|
| **Class diagram** ⭐ | **Sequence diagram** ⭐ |
| Object diagram | **Use case diagram** ⭐ |
| Package diagram | **Activity diagram** ⭐ |
| Component diagram | State diagram |
| Composite structure diagram | Communication diagram |
| Deployment diagram | Interaction overview diagram |
| Profile diagram | Timing diagram |

> ⭐ = Most important for LLD interviews

- **Structural** → describes *what* exists in the system (static view)
- **Behavioural** → describes *what happens* in the system (dynamic view)

---

## 3. Use Case Diagram

**Purpose:** Analyze the system's **high-level requirements** from the user's point of view.

> Answers: *What does the system do? Who uses it? What will it NOT do?*

### Characteristics
- Shows **high-level functional behaviour**
- Describes what the system does from the **user's perspective**
- Defines clear boundaries — what the system **will** and **will not** do

### 5 Components of a Use Case Diagram

| Component | Description | Notation |
|-----------|-------------|----------|
| **System Boundary** | Rectangle that defines the scope of the system | `[ System Box ]` |
| **Actor** | A user or external system that interacts with the system | Stick figure |
| **Use Case** | A feature or action the system provides | Oval / Ellipse |
| **Include** | Use case A always triggers use case B (mandatory) | `--<<include>>-->` |
| **Extend** | Use case B optionally extends use case A (optional) | `--<<extend>>-->` |

### Key Difference: Include vs Extend

| | Include | Extend |
|--|---------|--------|
| Trigger | Always happens | Optionally happens |
| Example | Create User always includes User Address Create | Search User can optionally extend to Search by First Name |

### Example
```
Admin ──→ Create User ──<<include>>──→ User Address Create
Admin ──→ Update User
Admin ──→ Delete User
Admin ──→ Search User ←──<<extend>>── Search by First Name
                      ←──<<extend>>── Search by Last Name
Guest ──→ Search User
```

---

## 4. Activity Diagram

**Purpose:** Shows the **flow of control** for a system functionality — like a flowchart for a use case.

- Emphasizes the **conditions** and **sequence** in which actions happen
- Each activity = an operation on a class that **changes the state** of the system
- Used to model **workflows, business processes, and internal operations**
- Can also describe the **steps involved in executing a use case**

### Symbols Used

| Symbol | Meaning |
|--------|---------|
| Rounded rectangle (filled) | Start point |
| Rounded rectangle (empty) | End point |
| Rectangle | Activity / Action |
| Diamond | Decision point (condition) |
| Arrow | Flow direction |

### Example — Delete User Flow
```
Start
  ↓
Customer opens User Web Portal
  ↓
[Decision] ──Yes──→ User Search ──→ [Decision: Found?]
                                        ↓ Yes
                                   View User Details
                                        ↓
                                   Delete User
                                        ↓
                                  [Decision: is deleted?]
                                        ↓ Yes
                                       End
```

---

## 5. Sequence Diagram

**Purpose:** Shows **interactions among classes** in terms of exchange of messages over time.

- Represents **calls between different objects** in their sequence
- Answers: *Who calls whom? In what order? What is returned?*

### Two Dimensions

| Dimension | Represents |
|-----------|-----------|
| **Vertical** (top → bottom) | Sequence of messages in **chronological order** |
| **Horizontal** (left → right) | Different **object instances** to which messages are sent |

### Key Elements

| Element | Description |
|---------|-------------|
| **Actor** | Initiator of the flow (stick figure) |
| **Lifeline** | Vertical dashed line for each object |
| **Activation bar** | Thick bar on lifeline = object is active/processing |
| **Solid arrow →** | Method call (request) |
| **Dashed arrow ←** | Return / Response |

### Example — Delete User Sequence
```
Actor → UserService    : searchUser
UserService → Transaction : searchUser
Transaction → UserDAO  : searchUser
UserDAO → Transaction  : User  (response)
Transaction → UserService : User (response)

Actor → UserService    : deleteUser
UserService → Transaction : deleteUser
Transaction → UserDAO  : deleteUser
UserDAO → Transaction  : Success
Transaction → UserService : Success
UserService → Actor    : Success
```

> The flow is **layered** — Actor talks to Service, Service talks to Transaction layer, Transaction talks to DAO (Data Access Object). This mirrors real-world layered architecture.

---

## Summary

```
Use Case Diagram  → "What can actors do in the system?" (requirements view)
Activity Diagram  → "How does a feature flow step by step?" (process/workflow view)
Sequence Diagram  → "Who calls whom and in what order?" (interaction/time view)
```

### Quick Comparison

| Diagram | Focus | Analogy |
|---------|-------|---------|
| Use Case | Features & actors | User stories / Requirements doc |
| Activity | Step-by-step flow with decisions | Flowchart |
| Sequence | Object-to-object communication over time | WhatsApp group chat timeline |
