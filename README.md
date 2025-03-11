# Create Red Hat Developer Hub Software Templates for Quarkus based application.

## Table of Contents
1. [Workshop Overview](#workshop-overview)
2. [Agenda](#agenda)
3. [What to Know Before Writing Software Templates](#what-to-know-before-writing-software-templates)
4. [Building a Software Template](#building-a-software-template)
   - [Understanding the Software Template for Quarkus](#understanding-the-software-template-for-quarkus)
   - [Creating a Template Step by Step](#creating-a-template-step-by-step)
     - [Defining User Input Parameters](#defining-user-input-parameters)
     - [Fetching Boilerplate Code](#fetching-boilerplate-code)
     - [Creating a Git Repository](#creating-a-git-repository)
     - [Registering the Service in RHDH](#registering-the-service-in-rhdh)
     - [Deploying with GitOps & ArgoCD](#deploying-with-gitops--argocd)
   - [Building catalog-info.yaml](#building-catalog-infoyaml)
     - [Adding Metadata](#adding-metadata)
     - [Adding Annotations for GitOps & CI/CD](#adding-annotations-for-gitops--cicd)
     - [Adding Developer Links](#adding-developer-links)
     - [Registering an API in RHDH](#registering-an-api-in-rhdh)
5. [Importing Template in Red Hat Developer Hub](#importing-template-in-red-hat-developer-hub)
6. [Final Steps](#final-steps)

## Workshop Overview  
This workshop will guide you through creating **Backstage software templates** using **Red Hat Developer Hub (RHDH)**...

By the end of this session, participants will:
- **Learn how to create a Red Hat Developer Hub Software Template from scratch for a Quarkus based application.**
- **Understand the folder structure and role of each file.**
- **Build `template.yaml` and `catalog-info.yaml`.**
- **Register and import services in Red Hat Developer Hub (RHDH).**
- **Use the imported template from RHDH Software Catalog and create repos with necessary code that developer can start using to build further.**

---
## Agenda

| **Time**  | **Activity**                     | **Purpose**  |
|-----------|----------------------------------|-------------|
| **0-2 mins**  | **Intro to RHDH/Backstage Entities** | What are Templates and Components? (Quick overview) |
| **2-4 mins**  | **Folder Structure**  | Where `template.yaml`, `catalog-info.yaml`, and the Quarkus skeleton fit. |
| **4-18 mins**  | **Step 1: Build `template.yaml`**  | Define parameters, fetch boilerplate, create repo, register in RHDH, deploy with ArgoCD. |
| **18-26 mins**  | **Step 2: Build `catalog-info.yaml`**  | Why RHDH needs it? How it connects to GitOps. |
| **26-28 mins**  | **Step 3: Register in RHDH**  | Manually import the service. (Quick demo) |
| **28-30 mins**  | **Step 4: Q&A + Customization**  | Let participants tweak the template. |

---
# Building a Software Template

## What to Know Before Writing Software Templates

Before writing any code, let's understand how **Red Hat Developer Hub (RHDH)** organizes software components using **entities**.

---

## Understanding Software Catalog

The **Software Catalog** in **Red Hat Developer Hub (RHDH)** is a **centralized asset tracker** that stores and manages all software-related entities in your organization.

It helps teams:

✔️ **Track ownership** of services and APIs.  
✔️ **Discover and document** microservices and dependencies.  
✔️ **Ensure consistency** in software and infrastructure.  
✔️ **Automate software lifecycle management** with GitOps.

---

## Entities in Red Hat Developer Hub

An **entity** is any **software-related object** that needs to be tracked, documented, and managed inside RHDH.

Entities are typically **stored in YAML files** (e.g., `catalog-info.yaml`) and are **registered** in the **Software Catalog**.

### **Entity Types in Red Hat Developer Hub**

| **Entity Type** | **Kind**           | **Purpose** |
|---------------|-----------------|-------------------------------|
| **Component** | `kind: Component` | Represents a microservice, website, or library running in production. |
| **API**       | `kind: API`       | Describes an exposed API that other services can use. |
| **Resource**  | `kind: Resource`  | A database, S3 bucket, or cloud service used by Components. |
| **System**    | `kind: System`    | A collection of Components that work together as a product. |
| **Domain**    | `kind: Domain`    | A higher-level grouping of multiple Systems. |
| **Template**  | `kind: Template`  | Defines how new services are created using Red Hat Developer Hub. |
| **User**      | `kind: User`      | Represents an individual developer or team member. |
| **Group**     | `kind: Group`     | Represents a team or organization within RHDH. |

---

### Key Focus Areas in This Workshop

In this workshop, we will focus on **three key entities**:

- **Templates** → Automate service creation.
- **Components** → Represent microservices in production.
- **APIs** → Define interfaces that services expose.

---

## How Backstage Templates Automate This Process

With a **Backstage Software Template**, **this entire setup is automated**:

1️⃣ **A Template (`kind: Template`) asks for user input** (e.g., service name, repo details).  
2️⃣ **It generates a Component (`kind: Component`)** inside Red Hat Developer Hub.  
3️⃣ **If an API is included, it is also registered** as an `API` entity.  
4️⃣ **CI/CD, GitOps, and Kubernetes setup are automated**.

### Example: How Our Template Works

- A **developer fills out a form** in **Red Hat Developer Hub**.  
- The **Template** generates a **new Git repository** with Quarkus boilerplate.  
- The **service is automatically registered** as a **Component** in the catalog.  
- If the service **exposes an API**, it is **linked to an API entity**.  
- The service is **deployed automatically using GitOps** and ArgoCD.

---

## Example catalog-info yaml

When an entity is registered in RHDH, it is defined in a `catalog-info.yaml` file.

This is an example of how a **Component** is registered:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-amazing-microservice
  description: A microservice written to do amazing things

  # Annotations provide extra context to plugins, e.g., TechDocs
  annotations:
    backstage.io/techdocs-ref: dir:.

  # Tags help filter Entities in the Software Catalog
  tags:
    - docs

spec:
  type: Documentation
  lifecycle: development
  owner: "pe1"
```

---

## How Entities Are Added to the Software Catalog

Entities are **imported and synchronized** in **three ways**:

### 1. Automatically using plugins
- Backstage can scan **GitHub, GitLab, or Kubernetes** to discover entities.  

### 2. Manually registering entities via the UI
- Developers can **add a service manually** by providing a repository link.

### 3. Declaring entities in the Backstage configuration
- Administrators can define **static entities** inside **Backstage configuration files**.

---

## How Does Red Hat Developer Hub Keep Entities Up to Date?

✔️ RHDH is **configured to automatically sync** entity information on a regular schedule.  
✔️ If an **entity is stored in Git**, it follows the convention of using a `catalog-info.yaml` file.  

---

## Summary

✔️ The **Software Catalog** is the **source of truth** for all software in your organization.  
✔️ **Entities** such as **Components, APIs, Resources, and Templates** are registered inside the Software Catalog.  
✔️ **`catalog-info.yaml`** is used to **define and register services in Red Hat Developer Hub**.  
✔️ Our **Backstage Template** will **automate service onboarding, CI/CD, and deployment**.  

🚀 **Now let’s start building our Backstage Software Template!**  

---
# Creating a Template Step by Step

## **Understanding the Software Template for Quarkus**
Before we start building the software template step by step, let’s first understand the **key elements of our setup**.

### What Are We Working With?
For this workshop, we are using an **existing repository** that follows a **standard folder structure** for a **Red Hat Developer Hub (RHDH) software template**.

✔️ The repository contains **pre-existing code** for a **Quarkus microservice**.  
✔️ It also includes **deployment manifests** required for GitOps-based deployment.  
✔️ Our focus will be on the **two key building blocks** that make this a software template:  
   - **`template.yaml`** → Defines how new services are created in RHDH.  
   - **`catalog-info.yaml`** → Registers the generated service in the **Software Catalog**.

---

### Repository Structure Overview

This repository consists of two main parts:
1️⃣ **Skeleton (`skeleton/`)** → Contains the **Quarkus microservice code** and additional files that will be copied when a new service is created.  
2️⃣ **Manifests (`manifests/`)** → Holds **deployment resources** such as Helm charts and ArgoCD configurations, used for deploying the service.

```
# 📁 template-quarkus-simple

## ├── 📁 manifests
│   ├── 📁 argocd               # ArgoCD application definitions
│   ├── 📁 helm                 # Helm charts for deployment

## ├── 📁 skeleton
│   ├── 📄 catalog-info.yaml     # Service metadata for Red Hat Developer Hub
│   ├── 📄 README.md             # Documentation for the generated service
│   ├── 📁 docs                  # TechDocs content for Red Hat Developer Hub
│   ├── 📄 mvnw                  # Maven wrapper script (Linux/Mac)
│   ├── 📄 mvnw.cmd              # Maven wrapper script (Windows)
│   ├── 📄 openapi.yaml          # OpenAPI specification for API registration
│   ├── 📄 pom.xml               # Maven build file
│   ├── 📁 src                   # Quarkus service source code
│   ├── 📁 target                # Compiled build artifacts

## ├── 📄 template.yaml           # Red Hat Developer Hub template definition
## ├── 📄 README.md               # Documentation for the template

```
---

###  Why Focus on `template.yaml` and `catalog-info.yaml`?
To convert this repository into a **Red Hat Developer Hub Software Template**, we need to ensure:
1️⃣ **`template.yaml` is correctly defined** → It will guide RHDH on how to scaffold new services based on this template.  
2️⃣ **`catalog-info.yaml` is properly structured** → This will ensure that every generated service is correctly registered in the Software Catalog.

---

### What’s Next?
Now that we understand the **repository structure** and our **focus areas**, let’s start **building the software template step by step**! 🚀

---

### ***Key Takeaways**
- **The new TOC** correctly organizes content **before** jumping into hands-on sections.  
- **The new section "Understanding the Software Template for Quarkus"** provides **clear context** about the repo structure and key elements.  
- **Everything is now logically structured** under "Building a Software Template."  


**Now you can directly copy and paste this into your `.md` file!** 🎯 Let me know if you'd like any final refinements. 🚀

## Step-by-Step Guide
### **🛠 Step 1: Setting Up the Demo**

### **Prerequisites**
Before starting, ensure you have:
- VS Code or any other IDE .
- A running Red Hat Developer Hub instance**.
- An accessible GitLab (or GitHub) Org **.
- ArgoCD configured** if testing deployments.

### 🛠 Preparing Your Repository for Backstage Software Templates

Before starting, ensure you have the following:

###  1. GitHub CLI (`gh`) is Installed and Authenticated**

To check if GitHub CLI is installed, run:

```
gh --version
```
If it’s **not installed**, follow the [GitHub CLI installation instructions](https://cli.github.com/).

Now, check if you're logged into GitHub CLI:
```sh
gh auth status
```
If it says **"You are not logged in"**, run:
```sh
gh auth login
```
_(Follow the prompts to authenticate with GitHub.)_

### Step 1: Fork and Clone the Repository

### 1. Run the following command to fork and clone the repository:
```sh
gh repo fork rh-product-demos/create-software-template --clone --remote
```
#### **What This Does:**
- **Creates a fork** under your GitHub account (e.g., `your-github-username/create-software-template`).
- **Clones your fork to your local machine**.
- **Automatically sets up `origin` as your fork**.

### 2. Move into the cloned repository:
```sh
cd create-software-template
```

* At this point, you have a repository with an empty `template.yaml` and `catalog-info.yaml`, ready to build your Backstage Software Template.

---

# Understanding the Folder Structure

The folder structure below represents an **example of a software template**.  
Here, we assume that the **Quarkus service code and manifests are already written**, allowing us to focus on the **Red Hat Developer Hub (RHDH) elements** of a software template.

```
# 📁 template-quarkus-simple-main

## ├── 📁 manifests
│   ├── 📁 argocd               # ArgoCD application definitions for GitOps deployment
│   ├── 📁 helm                 # Helm charts for Kubernetes deployment

## ├── 📁 skeleton
│   ├── 📄 catalog-info.yaml     # Service metadata for Red Hat Developer Hub
│   ├── 📄 README.md             # Documentation for the generated service
│   ├── 📁 docs                  # TechDocs content for RHDH
│   ├── 📄 mvnw                  # Maven wrapper script (Linux/Mac)
│   ├── 📄 mvnw.cmd              # Maven wrapper script (Windows)
│   ├── 📄 openapi.yaml          # OpenAPI specification for API registration
│   ├── 📄 pom.xml               # Maven build file
│   ├── 📁 src                   # Source code for the Quarkus microservice
│   ├── 📁 target                # Compiled build artifacts

## ├── 📄 template.yaml           # RHDH Software Template definition
## ├── 📄 README.md               # Documentation for the template
```

---

# Building `template.yaml`

## A Quick Overview of `template.yaml`
The **Software Templates** feature in **Red Hat Developer Hub (RHDH)** helps you **scaffold and automate the creation of services**. A template defines:
- **How a new service is created**
- **What input parameters it requires**
- **Where it should be stored (GitHub, GitLab, etc.)**
- **How it should be deployed using GitOps (ArgoCD, Kubernetes, etc.)**

### Why Use `template.yaml`?
✔️ Automates service creation  
✔️ Ensures consistency across all microservices  
✔️ Standardizes deployment and registration  
✔️ Reduces manual setup time for developers  

---

## Step 2: Start with a Blank `template.yaml`
### **Instructions**
1. Open **VS Code** and navigate to your **Red Hat Developer Hub Software Template** repository.
2. **Create a new file**: `template.yaml`
3. **Add the following starting template**:

```yaml
# Red Hat Developer Hub Software Template Definition
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: quarkus-web-template
  title: Quarkus Service
  description: Create a simple microservice using Quarkus with ArgoCD
  tags:
    - recommended
    - java
    - quarkus
    - maven
spec:
  owner: rhdh
  type: service
```

### Explanation
- **`apiVersion`** → Defines the API version used by Red Hat Developer Hub  
- **`kind`** → Defines the entity as a `Template`  
- **`metadata`** → Includes a **name, title, description, and tags** to make it searchable  
- **`spec`** → Defines ownership and type of service  

**Next, we will add input parameters to make this template dynamic.**

---

## Step 2.1: Define User Input Parameters
Templates allow user input to customize each new service. These **parameters** will be collected from the user when running the template.

### **Instructions**
1. **Add the `parameters` section** in `template.yaml`:

```yaml
parameters:
  - title: Provide Information for Application
    required:
      - component_id
      - java_package_name
    properties:
      component_id:
        title: Name
        type: string
        description: Unique name of the component
        default: my-quarkus-app
        ui:field: EntityNamePicker
        maxLength: 18
      group_id:
        title: Group Id
        type: string
        default: com.redhat.rhdh
        description: Maven Group Id
      artifact_id:
        title: Artifact Id
        type: string
        default: quarkus-app
        description: Maven Artifact Id
      java_package_name:
        title: Java Package Name
        default: com.redhat.rhdh
        type: string
        description: Name for the Java package (e.g., com.redhat.app)
      description:
        title: Description
        type: string
        description: Help others understand what this service does.
        default: A cool Quarkus app
```

### Explanation
- **`parameters`** → Defines user inputs required to generate the service  
- **`component_id`** → Unique service name (with validation via `EntityNamePicker`)  
- **`group_id` & `artifact_id`** → Used for **Maven build configuration**  
- **`java_package_name`** → Ensures the correct **Java package structure**  
- **`description`** → Helps developers understand the service  

---

## Step 2.2: Fetch Quarkus Boilerplate Code
Once the user provides input, the template should **fetch a predefined Quarkus project skeleton**.

### Instructions
1. **Add the `fetch:template` step**:

```yaml
  steps:
    - id: template
      name: Fetch Skeleton + Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          component_id: ${{ parameters.component_id }}
          java_package_name: ${{ parameters.java_package_name }}
        targetPath: ./${{ user.entity.metadata.name }}-${{ parameters.component_id }}
```

### Explanation
- **`fetch:template`** → Copies the Quarkus boilerplate into the new service directory  
- **`${{ parameters.component_id }}`** → Injects user-provided values dynamically  
- **Ensures consistency** across all new services  

---

## Step 2.3: Create a Git Repository and Push Code
### **Instructions**
1. **Add the `publish:gitlab` step**:

```yaml
    - id: publish
      name: Publish
      action: publish:gitlab
      input:
        repoUrl: "${{ parameters.repo.host }}?owner=${{ user.entity.metadata.name }}&repo=${{parameters.component_id}}"
        repoVisibility: public
        defaultBranch: main
        sourcePath: ./${{ user.entity.metadata.name }}-${{parameters.component_id}}
```

### Explanation
- **`publish:gitlab`** → Automates repository creation in GitLab  
- **Ensures the project is version-controlled and stored correctly**  

---

## Step 2.4: Register the Service in Red Hat Developer Hub
Once the repository is created, the service **must be registered in the RHDH Software Catalog**.

### Instructions
1. **Add the `catalog:register` step**:

```yaml
    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: "/catalog-info.yaml"
```

### Explanation
- **Registers the generated service in RHDH**  
- **Ensures the service appears in the Software Catalog**  

---

## Step 2.5: Deploy with GitOps & ArgoCD
Once the service is created, we need to **deploy it using GitOps**.

### Instructions**
1. **Add the `fetch:template` step to generate deployment resources**:

```yaml
    - id: template-gitops-deployment
      name: Generating Deployment Resources
      action: fetch:template
      input:
        url: ./manifests
        values:
          component_id: ${{ parameters.component_id }}
          namespace: ${{ user.entity.metadata.name }}-${{ parameters.component_id }}-dev
        targetPath: ./${{ user.entity.metadata.name }}-${{parameters.component_id}}-gitops
```

### Explanation
- **Generates Kubernetes manifests**  
- **Ensures the service is deployed via GitOps in ArgoCD**  

---

## Step 2.6: Publish GitOps Deployment Resources
### Instructions
1. **Add the `publish:gitlab` step**:

```yaml
    - id: publish-gitops
      name: Publishing to Resource Repository
      action: publish:gitlab
      input:
        repoUrl: "${{ parameters.repo.host }}?owner=${{ user.entity.metadata.name }}&repo=${{parameters.component_id}}-gitops"
        repoVisibility: public
        sourcePath: ./${{ user.entity.metadata.name }}-${{parameters.component_id}}-gitops
```

### Explanation
- **Stores GitOps configurations in a dedicated repo**  
- **Ensures ArgoCD can monitor and deploy changes**  

---

## Step 2.7: Configure ArgoCD for Automated Deployment
### Instructions
1. **Add the `argocd:create-resources` step**:

```yaml
    - id: create-argocd-resources
      name: Create ArgoCD Resources
      action: argocd:create-resources
      input:
        appName: ${{ user.entity.metadata.name }}-${{ parameters.component_id }}-bootstrap
        namespace: rhdh-gitops
        repoUrl: https://${{ parameters.repo.host }}/${{ user.entity.metadata.name }}/${{ parameters.component_id }}-gitops.git
        path: 'argocd/'
```

### **Explanation**
- **Ensures continuous deployment via ArgoCD**  
- **Automates Kubernetes deployment tracking**  

---

## Step 2.8: Provide Output Links for Easy Access
```yaml
  output:
    links:
      - title: Source Code Repository
        url: ${{ steps.publish.output.remoteUrl }}
      - title: Open Component in catalog
        icon: catalog
        entityRef: ${{ steps.register.output.entityRef }}
```
 
 **Your template is now complete!**

---

# Building `catalog-info.yaml`

In this section, we will **incrementally build `catalog-info.yaml`** to properly register a service in **Red Hat Developer Hub (RHDH)**. This process ensures that your service is **discoverable, well-documented, and manageable** in the **Software Catalog**.

---

## Start with a Blank `catalog-info.yaml`

### **Why is this needed?**  
Every service created in **RHDH** must be **registered in the catalog** so that it can be tracked and managed.

### **Instructions**  
1. **Create a new file**: `catalog-info.yaml`  
2. **Add the basic `Component` definition**:  

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: ${{values.component_id | dump}}
spec:
  type: service
  lifecycle: development
  owner: ${{values.owner | dump}}
```

### **Explanation**
- **What is `kind: Component`?**  
  - This tells **RHDH** that the entity represents a **microservice**.  
- **What is `lifecycle: development`?**  
  - This indicates that the service is **still in development**, not yet in production.  
- **Why do we need `owner`?**  
  - **RHDH** requires every service to have an **owner** for accountability.

---

## Add Metadata (Description & Tags)

### **Why is this important?**  
Adding **metadata** allows users to **identify and search** for the service inside **RHDH**.

### **Update `metadata` section**:

```yaml
metadata:
  name: ${{values.component_id | dump}}
  {%- if values.description %}
  description: ${{values.description | dump}}
  {%- endif %}
  tags:
    - java
    - quarkus
```

### Explanation
- **What does `description` do?**  
  - Provides a **short summary** of the service to help developers understand its purpose.  
- **Why add `tags`?**  
  - Tags make the service **easier to find** inside **Red Hat Developer Hub**.

---

## Deploying with GitOps & ArgoCD  

To **enable automation with GitOps**, we must add **annotations** that link **RHDH** to GitLab, ArgoCD, and Kubernetes.

### **Update `annotations` section**:

```yaml
  annotations:
    argocd/app-selector: rht-gitops.com/${{ values.gitops_namespace }}=${{ values.owner }}-${{values.component_id}}
    backstage.io/kubernetes-id: ${{values.component_id}}
    janus-idp.io/tekton: ${{values.component_id}}
    backstage.io/source-location: url:https://${{values.host}}/${{values.destination}}
    backstage.io/techdocs-ref: url:https://${{values.host}}/${{values.destination}}
    gitlab.com/project-slug: ${{values.destination}}
```

### **Explanation**
- **What is `argocd/app-selector`?**  
  - **Links the service to ArgoCD**, allowing **RHDH** to track **GitOps deployments**.  
- **What is `backstage.io/source-location`?**  
  - Specifies where the **service’s source code is stored** (e.g., **GitLab or GitHub**).  
- **What is `janus-idp.io/tekton`?**  
  - Enables **CI/CD pipeline tracking** in **RHDH**.

---

## Add Developer Links for OpenShift Dev Spaces

To **enhance the developer experience**, we can provide **direct links** for **opening the service in VS Code or JetBrains**.

### **Update `links` section**:

```yaml
  links:
    - url: https://devspaces.${{values.cluster}}/#https://${{values.host}}/${{values.destination}}
      title: OpenShift Dev Spaces (VS Code)
      icon: web
    - url: https://devspaces.${{values.cluster}}/#https://${{values.host}}/${{values.destination}}?che-editor=che-incubator/che-idea/latest
      title: OpenShift Dev Spaces (JetBrains IntelliJ)
      icon: web
```

### Explanation
- **Why add these links?**  
  - Developers can click these **to open the service in OpenShift Dev Spaces** for live coding.

---

## Register the API in Red Hat Developer Hub

If the service **exposes an API**, we must document it in **RHDH**.

### **Add API registration to `catalog-info.yaml`**:

```yaml
---
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: ${{values.component_id | dump}}
  {%- if values.description %}
  description: ${{values.description | dump}}
  {%- endif %}
spec:
  type: openapi
  lifecycle: development
  owner: ${{values.owner | dump}}
  definition:
    $text: ./openapi.yaml
```

### Explanation

#### **What is `kind: API`?**  
- This tells **RHDH** that the entity being registered is an **API** rather than a microservice (`kind: Component`).  
- This ensures that **APIs are properly documented, reusable, and discoverable**.  

#### **What is `definition: $text: ./openapi.yaml`?**  
- This links the API to an **OpenAPI spec file** (`openapi.yaml`).  
- Enables **automatic API documentation rendering** inside **RHDH**.  

#### **Why does RHDH need this step?**  
- Ensures that **all APIs and services are properly documented**.  
- Standardizes API **versioning, governance, and discovery** in the **Software Catalog**.  

---

### Final Thoughts on `catalog-info.yaml`
✔️ `catalog-info.yaml` ensures **services and APIs** are registered correctly in **RHDH**.  
✔️ It includes **metadata, annotations, GitOps configuration, and API registration**.  
✔️ **With this file in place**, developers can **easily discover, use, and manage services** in Red Hat Developer Hub.  

Now that **`catalog-info.yaml` is ready**, let’s move to the next steps! 🚀  

---
## Importing Template in Red Hat Developer Hub  

Once you have built your **Software Template** and defined `catalog-info.yaml`, the next step is to **register it in Red Hat Developer Hub (RHDH)** so that developers can use it to generate new services.  

### How to Register a Template in RHDH  
1. **Navigate to "Register Component"** in the **Red Hat Developer Hub UI**.  
2. **Enter the Git repository URL** where the `catalog-info.yaml` file is stored.  
3. Click **Analyze → Import** to scan and register the template in the catalog.  
4. Once imported, navigate to **Create → Choose Your Template** to verify that it appears in the list of available templates.  

### How to Register an API in RHDH  
If your **template includes an API**, you need to ensure that it is also registered in **RHDH** for proper discoverability and documentation.  

1. **Navigate to "Register Component"** in RHDH.  
2. **Enter the Git repository URL** where the `catalog-info.yaml` is stored.  
3. Click **Analyze → Import** to add the API to the catalog.  

This process ensures that **all APIs and services are properly documented and tracked** within RHDH, improving discoverability and governance. 🚀  

---

Once the template and its associated APIs are successfully registered, developers can **instantly create new services** by selecting it from the **Create** menu in RHDH.  

Now that the **template is available**, let’s move on to **final validation steps!**


## Final Steps  

After building and registering your **Red Hat Developer Hub (RHDH) Software Template**, it's time to **test and verify** that everything works as expected.  

### Test the Template in RHDH  
1. **Go to Red Hat Developer Hub UI** and navigate to the **Create** section.  
2. **Select your newly created template** from the available options.  
3. **Fill in the required inputs**, such as service name, repository details, and other configurations.  
4. **Click "Run"** to execute the template and generate the new service.  

### Verify Git Repository Creation  
After running the template:  
✔️ Check your **GitLab/GitHub repository** to confirm that the required **service repository** and **GitOps repository** have been created.  
✔️ Verify that the source code, CI/CD configuration, and necessary deployment files exist in the repositories.  

### Validate Red Hat Developer Hub Registration  
✔️ Go to **Red Hat Developer Hub → Software Catalog**.  
✔️ Search for the **newly created Component**.  
✔️ Verify that all **linked APIs, annotations, and metadata** are correctly displayed.  
✔️ Check if the **TechDocs documentation is available** in the UI.  

### Confirm GitOps Deployment  
✔️ Ensure that **ArgoCD detects and deploys the service** using the generated GitOps repository.  
✔️ Monitor logs to see if the microservice successfully starts in the cluster.  

---

Now that your **Backstage Software Template** is fully functional in **Red Hat Developer Hub**, you have successfully **automated service onboarding, GitOps deployments, and API registration**! 🚀🎉  
---
