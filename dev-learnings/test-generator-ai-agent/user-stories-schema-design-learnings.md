# What is an Entity?

## Simple Definition:
An entity is a "thing" or "object" that you need to store information about in your database.

Think of it like a filing cabinet:

- Each entity = a drawer in the cabinet  
- Each instance of that entity = a folder in that drawer  
- Each attribute = a piece of paper in that folder  

---

## Real-World Example:
In a library:

- Book is an entity (you store info about books)
- Each specific book (like "Harry Potter") is an instance of the Book entity
- Book attributes: title, author, ISBN, pages, publishedDate

In your database:

- Book = a table
- Each row = one specific book
- Each column = an attribute (title, author, etc.)

---

## Entity vs Attribute vs Relationship:

### 1. Entity (Noun - the main "thing")
Examples: User, Project, Product, Order, Post  

Test: Can you point to it and say "this is a ___"?

- "This is a User" (John)
- "This is a Project" (My E-commerce App)
- "This is an email" (email is just a property of User)

### 2. Attribute (Properties of the entity)
Examples: User has email, name, createdAt  

Test: Does it describe the entity?

- A User has an email
- A Project has a name
- A User is a Project (that's a relationship, not an attribute)

### 3. Relationship (How entities connect)
Examples: User owns Projects, Project has TestGenerations  

Test: Can you say "Entity A has/owns/belongs-to Entity B"?

- User owns many Projects
- TestGeneration belongs to a Project
- User creates TestGenerations

---

## Identifying Entities from Your User Stories

Let's analyze your stories together:

| User Story | Potential Entities | Why? |
|----------|------------------|------|
| Users sign in | User | User is the main actor |
| Users create Projects | Project | Project is a thing we store |
| Users generate Tests | TestGeneration | Each test generation is a separate thing with its own data |
| Users analyze Flaky Tests | FlakyAnalysis | Each analysis is a separate thing with results |
| Users can buy subscription (future) | Subscription | Subscription is a thing we track |

---

## How to Decide: Is This an Entity or Just an Attribute?

Ask yourself these questions:

### Question 1: Does it have multiple properties of its own?

- Email: Just a string → Attribute of User
- Project: Has name, description, createdAt, owner → Entity

### Question 2: Will you need to query it independently?

- Name: Only searched with User → Attribute
- TestGeneration: You want to list all test generations for a project → Entity

### Question 3: Does it have a lifecycle (created, updated, deleted)?

- CreatedAt: Just a timestamp → Attribute
- Project: Can be created, updated, archived, deleted → Entity

### Question 4: Can it exist independently or does it describe something else?

- Version: Just describes TestGeneration → Attribute (version number)
- TestGeneration: Exists as its own thing → Entity

---

## Examples from Your Project:

### Entities (Things to store):
- User - Has identity, can perform actions
- Project - Container for work, has lifecycle
- TestGeneration - Each generation is unique, has source code, tests, metadata
- FlakyAnalysis - Each analysis is unique, has results, metadata

### Not Entities (Just Attributes):
- Version - Describes a TestGeneration → Attribute: TestGeneration.version
- Source Code - Part of TestGeneration → Attribute: TestGeneration.sourceCode
- Test Name - Part of test run data → Attribute inside JSON
- Email - Part of User → Attribute: User.email

---

## Your Turn: Let's Practice!

From your user stories, you mentioned these things:

- Generated Tests - Entity or Attribute?
- GitHub Repo (future) - Entity or Attribute?
- Notification (future) - Entity or Attribute?
- Test Results (the data users upload for flaky analysis) - Entity or Attribute?

Think about:

- Does it have multiple properties?
- Will you query it separately?
- Does it have its own lifecycle?

---

### 1. Generated Tests → Entity

Your answer: Entity – CORRECT!

Why you're right:

* Each generation has: source code, generated tests, language, framework, version, timestamp
* Multiple properties
* You want to list them, view history
* Has lifecycle: created, can be edited (new version), deleted

We'll call this entity: **TestGeneration**

---

### 2. GitHub Repo → "if it is a link then attribute"

Let me challenge your thinking here.

#### Scenario A: Just storing the repo URL

Project:

* id
* name
* githubRepoUrl: "[https://github.com/user/repo](https://github.com/user/repo)" ← Attribute

Then yes, it's just an attribute of Project.

#### Scenario B: Actually integrating with GitHub

If in the future you want to:

* Pull code directly from GitHub
* Track which branch, which commits
* Store OAuth token to access private repos
* Sync multiple repos per project

Then it becomes an Entity.

GitHubRepo:

* id
* projectId
* repoUrl
* branch
* accessToken
* lastSyncAt

Question for you: For your future feature, which scenario matches your vision?

* Just storing a URL for reference? → Attribute
* Actually pulling code from GitHub? → Entity

---

### 3. Notification → "maybe attribute as I do not know how it works"

Let me explain how notifications typically work.

#### Scenario A: Simple – Just send and forget

When a task completes, you send an email. You don't store anything.
No entity needed, just a service that sends emails.

#### Scenario B: Notification History (more common)

You want to:

* See notification history ("You got notified on Jan 22")
* Track delivery status (sent, failed, read/unread)
* Let users manage notification preferences

Then it's an Entity.

Notification:

* id
* userId
* type: "test_generation_complete"
* message: "Your tests are ready!"
* isRead: false
* sentAt

Question for you: Do you just want to email users when done, or do you want to show a notification history/inbox in your app?

---

### 4. Test Results → Entity?

Your answer: Entity – Let me clarify this one.

This is tricky. The test results (CSV/JSON data user uploads) are actually part of the **FlakyAnalysis** entity.

Here’s why:

* User uploads test data
* Your system analyzes it
* The analysis output is what you store

So the structure is:

FlakyAnalysis (Entity):

* id
* projectId
* version
* rawTestData ← uploaded CSV/JSON stored as JSON
* analysisResults ← flaky tests detected, AI suggestions
* totalTests
* flakyCount
* createdAt

The test results are an **attribute (JSON field)** of the FlakyAnalysis entity.

Think of it this way:

* You don’t query individual test runs separately
* You query the entire analysis: "Show me flaky analysis v1"
* The test data is just input/storage for that analysis

The raw data is embedded in the analysis, not a separate entity.

---

### Updated Entity List

Based on your answers and clarifications:

#### Core Entities (MVP):

* User – People using your app
* Project – Container for work
* TestGeneration – AI-generated tests
* FlakyAnalysis – Flaky test analysis results (includes test data inside)

#### Future Entities (not MVP):

* Subscription – User payment/plan info
* Notification – Notification history (if you want to show history)
* GitHubRepo – GitHub integration (if doing actual sync, not just URL)

---

### Now Let’s Define Attributes

We have 4 core entities. Let’s start with **User**.

Question for you: What information do we need to store about a User?

Hints to consider:

* OAuth gives us: email, name, avatar, provider (Google/GitHub)
* We need: a unique ID
* We might want: sign-up time, last login time, role (user/admin)

---
