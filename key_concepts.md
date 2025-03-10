# 🛠 Understanding Backstage Entities Before We Start

## 📌 Goal
Before writing any code, let's understand how **Backstage** organizes software components using **entities**.

## What are Backstage Entities?

Everything inside Backstage is considered an **entity**.  
An entity represents a **real-world object**, such as:

- A **microservice**
- An **API**
- A **CI/CD pipeline**
- A **team or user group**

Entities in Backstage are defined using **YAML files**, which Backstage reads and organizes in the **Software Catalog**.

## Key Entities We Will Work With

In this demo, we will use **three core entities**:

| **Entity Type**  | **Kind**         | **Purpose** |
|------------------|-----------------|-------------|
| **Template**     | `kind: Template` | Defines how **new services are created** from a Backstage form. |
| **Component**    | `kind: Component` | Represents a **running microservice** in Backstage. |
| **API**         | `kind: API`      | Describes an **exposed API** that other services can use. |


## How Do These Work Together?

1️⃣ **A `Template` helps generate a `Component`.**  
2️⃣ **A `Component` represents a real microservice in production.**  
3️⃣ **An `API` is linked to a `Component` if it provides a public API.**


## 📌 Example Workflow:

- A **developer fills out a Backstage form** → The **Template** creates a **new Git repository** with Quarkus boilerplate.
- The service is **registered as a `Component` in Backstage** and linked to a **Kubernetes deployment**.
- If the service **exposes an API**, it is also **registered as an `API` entity** in Backstage.

