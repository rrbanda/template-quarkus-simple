# 🎯 Create Red Hat Developer Hub Software Templates for Quarkus based application.

## Table of Contents
1. [Workshop Overview](#workshop-overview)
2. [Agenda](#agenda)
3. [What to Know Before Writing Software Templates](#what-to-know-before-writing-software-templates)
   - [Understanding Software Catalog](#understanding-software-catalog)
   - [Entities in Red Hat Developer Hub](#entities-in-red-hat-developer-hub)
   - [How Backstage Templates Automate This Process](#how-backstage-templates-automate-this-process)
   - [How Entities Are Added to the Software Catalog](#how-entities-are-added-to-the-software-catalog)
   - [Example catalog-info.yaml](#example-catalog-infoyaml)
4. [Building a Software Template](#building-a-software-template)
   - [Folder Structure of Template Repository](#folder-structure-of-template-repository)
   - [Pre-existing Code (Quarkus & Manifests)](#pre-existing-code-quarkus--manifests)
   - [Creating a Template Step by Step](#creating-a-template-step-by-step)
     - [Defining Input Parameters](#defining-input-parameters)
     - [Fetching Source Code](#fetching-source-code)
     - [Creating Git Repositories](#creating-git-repositories)
     - [Publishing the Template](#publishing-the-template)
   - [Building catalog-info.yaml](#building-catalog-infoyaml)
     - [Start with a Blank catalog-info.yaml](#start-with-a-blank-catalog-infoyaml)
     - [Add Metadata (Description & Tags)](#add-metadata-description--tags)
     - [Deploying with GitOps & ArgoCD](#deploying-with-gitops--argocd)
     - [Add Developer Links for OpenShift Dev Spaces](#add-developer-links-for-openshift-dev-spaces)
     - [Register the API in Red Hat Developer Hub](#register-the-api-in-red-hat-developer-hub)
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


Here is the **corrected and well-structured** content, keeping the **Table of Contents intact** and ensuring smooth navigation:

---

# 📖 **What to Know Before Writing Software Templates**

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


Your folder structure explanation is **clear and well-organized**, but I have a few **minor improvements** to ensure clarity and consistency:

### **Suggested Improvements**
1. **Fix small typos**:  
   - "beloew" → "below"  
   - "elemnents" → "elements"  
   - "an example software template" → "an example of a software template"  
   - "in this regard we assume" → "Here, we assume"  

2. **Clarify the purpose of the folder structure**:  
   - Make it clearer that the **Quarkus service code** and **deployment files** are assumed to be pre-written.

3. **Formatting consistency**:  
   - Add descriptions to **all** top-level folders for better clarity.  
   - Ensure indentation remains consistent in code blocks.  

---

### **✅ Updated Version**
```md
# 🛠 Understanding the Folder Structure

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
```

---

### **🔹 Key Fixes & Enhancements**
✅ **Typos corrected** and improved sentence structure.  
✅ **Clarified** that the **Quarkus service code is pre-written**, focusing on RHDH elements.  
✅ **Better indentation and descriptions** in the folder structure.  
✅ **Consistency** in the use of "Red Hat Developer Hub (RHDH)".  

---

### 🎯 **Next Steps**
- **Does this look good?**  
- Do you want to **add any more details** about specific files or folders? 🚀



---
## Building template.yaml

### **A quick overview of `template.yaml`?**
The Software Templates part of Backstage is a tool that can help you create Components inside Backstage. By default, it has the ability to load skeletons of code, template in some variables, and then publish the template to some locations like GitHub or GitLab.

* Templates are stored in the Software Catalog under a kind Template.
* You can create your own templates with a small yaml definition which describes the template and its metadata, along with some input variables that your template will need, and then a list of actions which are then executed by the scaffolding service.
---
## **🛠 Step 2: Start with a Blank `template.yaml`** we explain that Red Hat Developer Hub uses templates to scaffold services and that we will **incrementally build `template.yaml`**.

### **Instructions**
1. Open **VS Code** and navigate to your Red Hat Developer Hub Software template repository.
2. **Create a new file**: `template.yaml`
3. **Start with the following code snippet in a blank template** and **add basic comments**:

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
---
 ### Explanation
### **What is `template.yaml`?**
- `template.yaml` is a **Red Hat Developer Hub Software Template** that automates **project scaffolding**.
- It defines **how new services are created** based on user input.
- The template includes **parameters, actions (steps), and output** to generate, register, and deploy a service.

### **Why do we need `apiVersion`, `kind`, and `metadata`?**
- **`apiVersion`**: Specifies the **version of the Red Hat Developer Hub API** being used (`scaffolder.backstage.io/v1beta3` ensures compatibility with Red Hat Developer Hub scaffolding).
- **`kind`**: Defines the **type of entity** (`Template`) so Red Hat Developer Hub knows how to process it.
- **`metadata`**: Provides essential **identifiers and descriptions**, such as:
  - **`name`** → Unique identifier for the template.
  - **`title` & `description`** → Human-readable details.
  - **`tags`** → Helps categorize and filter templates in the Red Hat Developer Hub UI.

 **In short:** `template.yaml` tells Red Hat Developer Hub **what to create, how to create it, and what metadata to assign** in the Software Catalog.

---
## **🛠 Step 2.1: Define User Input Parameters** to Collect user input for the service.

### **Instructions**
1. **Add the parameters section**:

```
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
        description: Name for the java package. eg (com.redhat.blah)
      description:
        title: Description
        type: string
        description: Help others understand what this website is for.
        default: A cool quarkus app
  - title: Provide Image Registry Information
    required:
      - image_registry
    properties:
      image_registry:
        title: Image Registry
        type: string
        enum:
          - Openshift
          - Quay
    dependencies:
      image_registry:
        oneOf:
          - properties:
              image_registry:
                enum:
                  - Openshift
              image_host:
                title: Image Host
                type: string
                description: Host for storing image
                default: image-registry.openshift-image-registry.svc:5000
              image_tag:
                title: Image Tag
                default: latest
                type: string
                description: Build Image tag
          - properties:
              image_registry:
                enum:
                  - Quay
              image_host:
                title: Image Host
                type: string
                description: Host for storing image
                default: quay.apps.cluster-5nf6t.5nf6t.sandbox722.opentlc.com
              image_password:
                title: Password
                type: string
                description: Your Quay password
                ui:field: Secret
              image_tag:
                title: Image Tag
                default: latest
                type: string
                description: Build Image tag


```

### Explanation
### **Why do we need `parameters`?**  
- `parameters` allow users to **input values** before generating a service.  
- These values are used to **customize the service** dynamically (e.g., setting the name, group ID, or repository host).  
- Without `parameters`, all generated services would be **identical and static**, limiting flexibility.  

### **What is `EntityNamePicker`?**  
- `EntityNamePicker` is a **UI field** that helps users select a **valid component name** in Red Hat Developer Hub.  
- It ensures **naming consistency** by validating the input against Red Hat Developer Hub's **naming rules**.  
- This prevents **duplicate names** or incorrect formats before a service is created.  

**In short:** `parameters` make templates flexible, and `EntityNamePicker` ensures valid service names.

**⏩ Test It in RHDH Template editor ** → Paste the template and check the form UI.


## **🛠 Step 2.2: Fetch Quarkus Boilerplate Code** to copy a **predefined Quarkus project**.

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
### **Why use `fetch:template`?**  
- `fetch:template` **copies predefined files** (e.g., a Quarkus project skeleton) into the new service directory.  
- It **ensures consistency** by using a standard project structure.  
- This eliminates the need for developers to **manually clone or copy boilerplate code**.  

### **How does `${{ parameters.component_id }}` work?**  
- `${{ parameters.component_id }}` is a **templating expression** in Red Hat Developer Hub that **injects user-provided input** into the template.  
- When a user fills out the **Red Hat Developer Hub form**, the value of `component_id` is dynamically **substituted** in places like file paths, names, or configurations.  
- This ensures that **each service has a unique name** based on the user’s input.  

**In short:** `fetch:template` automates code scaffolding, and `${{ parameters.component_id }}` dynamically customizes the generated service.


## **🛠 Step 2.3: Create a Git Repository and Push Code** to Automatically create a **GitLab repository**.

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

### **Why use `publish:gitlab`?**  
- `publish:gitlab` **automates repository creation and code push** to GitLab.  
- It eliminates the need for **manual Git operations** (`git init`, `git add`, `git commit`, `git push`).  
- This ensures that every generated service has **a properly initialized and version-controlled repository**.  

### **What does `sourcePath` do?**  
- `sourcePath` **defines which directory** should be pushed to the repository.  
- It prevents **unnecessary files** (like template metadata) from being included in the service repo.  
- Example:  
  - If `sourcePath: ./generated-service`, only the contents of `generated-service/` are pushed to GitLab.  

**In short:** `publish:gitlab` automates Git operations, while `sourcePath` ensures only the correct files are committed.

**⏩ Test It in Red Hat Developer Hub** → Run the template and check GitLab.

---

## **🛠 Step 2.4: Register the Service in Red Hat Developer Hub** to Add the service to the **Red Hat Developer Hub Software catalog**.

```yaml
    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: "/catalog-info.yaml"
```

### Explanation
- Why is this necessary?

Here is the **detailed write-up** covering the missing steps in `template.yaml` related to **GitOps deployment, publishing GitOps resources, and integrating ArgoCD**.

---

### 🛠 Step 2.5: Generate GitOps Deployment Resources to Automate the generation of **deployment manifests** for GitOps-based deployment.

  * Add the `fetch:template` step** to generate deployment resources.
  * Ensure all necessary GitOps configurations** (e.g., ArgoCD, Kubernetes) are included.

```yaml
    - id: template-gitops-deployment
      name: Generating Deployment Resources
      action: fetch:template
      input:
        url: ./manifests
        copyWithoutTemplating: []
        values:
          component_id: ${{ parameters.component_id }}
          source_repository_gitops: https://${{ parameters.repo.host }}/${{ user.entity.metadata.name }}/${{parameters.component_id}}-gitops.git
          source_repository: https://${{ parameters.repo.host }}/${{ user.entity.metadata.name }}/${{parameters.component_id}}.git
          owner: ${{ user.entity.metadata.name }}
          namespace: ${{ user.entity.metadata.name }}-${{ parameters.component_id }}-dev
          image_registry: ${{ parameters.image_registry }}
          image_host: ${{ parameters.image_host }}
          image_organization: ${{ user.entity.metadata.name }}
          image_password: ${{ secrets.image_password }}
          image_name: ${{ parameters.component_id }}
          image_tag: ${{ parameters.image_tag }}
          gitops_namespace: rhdh-gitops
          gitops_project: default
          port: 8080
          cluster_subdomain: apps.cluster-5nf6t.5nf6t.sandbox722.opentlc.com
          repository_host: ${{ parameters.repo.host }}
        targetPath: ./${{ user.entity.metadata.name }}-${{parameters.component_id}}-gitops
```

### **Explanation**
**Why use `fetch:template`?**  
- This step **copies predefined GitOps deployment manifests** from the `manifests` folder.  
- It **ensures that the correct Kubernetes resources (Deployments, Services, Routes, ConfigMaps, Secrets, etc.) are generated**.  
- Using **templating expressions** (`${{ parameters.* }}`), it dynamically inserts **component-specific values** into the deployment files.

**What are these values used for?**  
- **`component_id`**: The name of the microservice being deployed.  
- **`source_repository_gitops` & `source_repository`**: The repositories where GitOps configurations and application code are stored.  
- **`namespace`**: The Kubernetes namespace where the service will be deployed.  
- **`image_registry`, `image_host`, `image_name`, `image_tag`**: Docker image information used by Kubernetes.  
- **`gitops_namespace` & `gitops_project`**: Define where ArgoCD will manage GitOps-based deployments.  
- **`repository_host`**: The SCM (GitLab/GitHub) host where the service is registered.  

**Why is this important?**  
- Automates **infrastructure provisioning** by **creating deployment configurations**.
- **Ensures consistency** across deployments by **storing everything as code in a Git repository** (GitOps approach).
- Makes the deployment **repeatable and scalable** across environments.

---

### **🛠 Step 2.6: Publish GitOps Deployment Resources to push the generated deployment resources to a **Git repository** that ArgoCD will track.

### **Instructions**
1️⃣ **Add the `publish:gitlab` step** to **push GitOps configurations** to a separate repository.

```yaml
    - id: publish-gitops
      name: Publishing to Resource Repository
      action: publish:gitlab
      input:
        repoUrl: "${{ parameters.repo.host }}?owner=${{ user.entity.metadata.name }}&repo=${{parameters.component_id}}-gitops"
        repoVisibility: public
        defaultBranch: main
        title: gitops resources for ${{ parameters.component_id }}
        description: gitops resources for ${{ parameters.component_id }}
        sourcePath: ./${{ user.entity.metadata.name }}-${{parameters.component_id}}-gitops
```

### **Explanation**
**Why use `publish:gitlab`?**  
- **Creates a new Git repository** (`component_id-gitops`) to store Kubernetes deployment files.  
- Ensures **GitOps tools (e.g., ArgoCD) can track infrastructure changes** by watching this repo.  

**What does `sourcePath` do?**  
- Specifies **which folder** (`component_id-gitops`) to push to GitLab.  
- Ensures that only **GitOps-related deployment files** are committed and pushed, without unnecessary files.  

✅ **Why do we need a separate GitOps repo?**  
- In GitOps workflows, **application source code and deployment configurations should be managed separately**.  
- This ensures that **any infrastructure changes are tracked independently** and **trigger automated deployments via ArgoCD**.

---

### **🛠 Step 2.7: Create ArgoCD Resources** to Automatically configure **ArgoCD** to deploy and manage the microservice.

### **Instructions**
**Add the `argocd:create-resources` step** to integrate the service with ArgoCD.

```yaml
    - id: create-argocd-resources
      name: Create ArgoCD Resources
      action: argocd:create-resources
      input:
        appName: ${{ user.entity.metadata.name }}-${{ parameters.component_id }}-bootstrap
        argoInstance: main
        namespace: rhdh-gitops
        repoUrl: https://${{ parameters.repo.host }}/${{ user.entity.metadata.name }}/${{ parameters.component_id }}-gitops.git
        path: 'argocd/'
```

### **Explanation**
**What does this step do?**  
- Configures **ArgoCD to monitor the GitOps repo** (`component_id-gitops`).  
- Creates an **ArgoCD Application** (`component_id-bootstrap`) that deploys the service.  
- Watches the GitOps repository and **automatically applies changes** when new commits are made.

**Key Inputs**  
- **`appName`** → The name of the ArgoCD application (`component_id-bootstrap`).  
- **`argoInstance`** → Specifies the ArgoCD instance (`main`).  
- **`namespace`** → The Kubernetes namespace where ArgoCD manages the service (`rhdh-gitops`).  
- **`repoUrl`** → The **GitOps repository** that ArgoCD will track.  
- **`path`** → The **directory inside the repo** where ArgoCD looks for Kubernetes manifests (`argocd/`).  

**Why is this important?**  
- **Enables Continuous Deployment** → Every Git push triggers an automatic deployment via ArgoCD.  
- **Ensures deployment consistency** → Developers don’t need to manually apply Kubernetes resources.  
- **Allows rollback & history tracking** → If a bad deployment happens, ArgoCD can roll back to the last working state.

---

### **🛠 Step 2.8: Output Links for Easy Access** to Provide direct **clickable links** to the generated Git repositories and Backstage Component.

### **Instructions**
1️⃣ **Define output links at the end of `template.yaml`**.

```yaml
  output:
    links:
      - title: Source Code Repository
        url: ${{ steps.publish.output.remoteUrl }}
      - title: Open Component in catalog
        icon: catalog
        entityRef: ${{ steps.register.output.entityRef }}
```

### **Explanation**
**Why include output links?**  
- **Gives developers direct access** to the newly created Git repositories.  
- **Ensures services can be found in Red Hat Developer Hub** by linking to their catalog entry.  

**What does `entityRef` do?**  
- The `entityRef` ensures that after the template runs, users can **click a link to view the registered Component in the RHDH catalog**.  
- This helps **new developers quickly find the service** and access relevant information.

---


**Test It in Red Hat Developer Hub** → The new service should appear in the catalog.

Here’s your **corrected and well-structured** Markdown content for **Building `catalog-info.yaml`**, ensuring **clarity, logical flow, and easy readability**. You can copy and paste it into your `.md` file:


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

## 6.2 Add Metadata (Description & Tags)

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

### **Explanation**
- **What does `description` do?**  
  - Provides a **short summary** of the service to help developers understand its purpose.  
- **Why add `tags`?**  
  - Tags make the service **easier to find** inside **Red Hat Developer Hub**.

---

## 6.3 Deploying with GitOps & ArgoCD  

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

## 6.4 Add Developer Links for OpenShift Dev Spaces

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

### **Explanation**
- **Why add these links?**  
  - Developers can click these **to open the service in OpenShift Dev Spaces** for live coding.

---

## 6.5 Register the API in Red Hat Developer Hub

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

### **Explanation**

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

### **Final Thoughts on `catalog-info.yaml`**
✔️ `catalog-info.yaml` ensures **services and APIs** are registered correctly in **RHDH**.  
✔️ It includes **metadata, annotations, GitOps configuration, and API registration**.  
✔️ **With this file in place**, developers can **easily discover, use, and manage services** in Red Hat Developer Hub.  

✅ Now that **`catalog-info.yaml` is ready**, let’s move to the next steps! 🚀  

---
## Importing Template in Red Hat Developer Hub  

Once you have built your **Software Template** and defined `catalog-info.yaml`, the next step is to **register it in Red Hat Developer Hub (RHDH)** so that developers can use it to generate new services.  

### 8.1 How to Register a Template in RHDH  
1. **Navigate to "Register Component"** in the **Red Hat Developer Hub UI**.  
2. **Enter the Git repository URL** where the `catalog-info.yaml` file is stored.  
3. Click **Analyze → Import** to scan and register the template in the catalog.  
4. Once imported, navigate to **Create → Choose Your Template** to verify that it appears in the list of available templates.  

### 8.2 How to Register an API in RHDH  
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

### 9.1 Test the Template in RHDH  
1. **Go to Red Hat Developer Hub UI** and navigate to the **Create** section.  
2. **Select your newly created template** from the available options.  
3. **Fill in the required inputs**, such as service name, repository details, and other configurations.  
4. **Click "Run"** to execute the template and generate the new service.  

### 9.2 Verify Git Repository Creation  
After running the template:  
✔️ Check your **GitLab/GitHub repository** to confirm that the required **service repository** and **GitOps repository** have been created.  
✔️ Verify that the source code, CI/CD configuration, and necessary deployment files exist in the repositories.  

### 9.3 Validate Red Hat Developer Hub Registration  
✔️ Go to **Red Hat Developer Hub → Software Catalog**.  
✔️ Search for the **newly created Component**.  
✔️ Verify that all **linked APIs, annotations, and metadata** are correctly displayed.  
✔️ Check if the **TechDocs documentation is available** in the UI.  

### 9.4 Confirm GitOps Deployment  
✔️ Ensure that **ArgoCD detects and deploys the service** using the generated GitOps repository.  
✔️ Monitor logs to see if the microservice successfully starts in the cluster.  

---

Now that your **Backstage Software Template** is fully functional in **Red Hat Developer Hub**, you have successfully **automated service onboarding, GitOps deployments, and API registration**! 🚀🎉  
---
