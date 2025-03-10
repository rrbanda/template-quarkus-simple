# **🎯Learning Software templates using an example Quarkus Service**
By the end of this session, participants will:
- **Learn how to create a Red Hat Developer Hub Software Template from scratch for quarkus service.**
- **Understand the folder structure and role of each file.**
- **Build `template.yaml` and `catalog-info.yaml`.**
- **Register and import services in Red Hat Developer Hub (RHDH).**
- **Use the imported template from RHDH Software Catalog and create repos with necessary code that developer can start using to build further.**

---
## 📜 Agenda
| **Time** | **Activity** | **Why It’s Important** |
|---------|-------------|----------------------|
| **0-2 mins** | **Setting Context: Understanding Red Hat Developer Hub/Backstage Entities** | What are Templates and Components? (Quick, high-level intro) |
| **2-4 mins** | **Understanding the Folder Structure** | Where do `template.yaml`, `catalog-info.yaml`, and the Quarkus skeleton fit? (Brief overview) |
| **4-18 mins** | **Step 1: Build `template.yaml` Incrementally** | Define parameters, fetch boilerplate, create GitLab repo, register in Red Hat Developer Hub, deploy with ArgoCD. (Live coding) |
| **18-26 mins** | **Step 2: Build `catalog-info.yaml` Incrementally** | Why does Red Hat Developer Hub need it? How does it relate to GitOps? (Live coding + quick test) |
| **26-28 mins** | **Step 3: Importing and Registering in Red Hat Developer Hub (RHDH)** | How to manually register the service into RHDH. (Quick live demo) |
| **28-30 mins** | **Step 4: Q&A + Customization Challenge** | Let participants tweak the template for real-world use cases. |

---

# 🛠 Understanding Backstage Entities Before We Start

## Before writing any code, let's understand how **Red Hat Developer Hub** organizes software components using **entities**.

## **What are Red Hat Developer Hub Entities?**  
Everything inside **Red Hat Developer Hub (RHDH)** is considered an **entity**.  
An **entity** represents a **real-world object** that is **registered and managed within the Software Catalog**.  

### **Common Entity Types in RHDH:**  
- **A microservice** (e.g., a backend service running in production)  
- **An API** (e.g., an OpenAPI-defined REST or GraphQL endpoint)  
- **A CI/CD pipeline** (e.g., Tekton or GitHub Actions workflows)  
- **A team or user group** (e.g., an organizational unit managing services)  

Entities in **Red Hat Developer Hub** are defined using **YAML files**, specifically `catalog-info.yaml`, which RHDH reads to organize components in the **Software Catalog**.  

---

## **Key Entities We Will Work With**  
In this demo, we will use **three core entities**:  

| **Entity Type** | **Kind**            | **Purpose**  |  
|----------------|-----------------|----------------------|  
| **Template**   | `kind: Template`   | Defines how **new services are created** using a Red Hat Developer Hub form.  |  
| **Component**  | `kind: Component`  | Represents a **runnable microservice** registered in the catalog.  |  
| **API**        | `kind: API`        | Describes an **exposed API** that other services can use.  |  

---

## **How Do These Work Together?**  
- A **Template** (`kind: Template`) helps generate a **Component** (`kind: Component`).  
- A **Component** represents a **real microservice** deployed in production.  
- If a **Component exposes an API**, it is **linked to an API entity** (`kind: API`) in the catalog.  

---

## **Example Workflow:**  
* A **developer fills out a form** in **Red Hat Developer Hub** → The **Template** creates a **new Git repository** with Quarkus boilerplate.
* The service is **registered as a `Component` in the catalog** and linked to a **Kubernetes deployment**.
* If the service **exposes an API**, it is also **registered as an `API` entity** in RHDH.  

**Now that we understand these concepts, let’s build our Red Hat Developer Hub Software Template step by step!**  

---
# **🛠 Step 1: Setting Up the Demo**

### **Prerequisites**
Before starting, ensure you have:
- **VS Code open** with your Red Hat Developer Hub repository.
- **A running Red Hat Developer Hub or Red Hat Developer Hub (RHDH) instance**.
- **A working GitLab (or GitHub) instance**.
- **ArgoCD configured** if testing deployments.

# 🛠 Preparing Your Repository for Backstage Software Templates

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

## Step 1: Fork and Clone the Repository

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
---
**At this point, you have a repository with an empty `template.yaml` and `catalog-info.yaml`, ready to build your Backstage Software Template.**
---

#  🛠 Understanding the Folder Structure

The folder structure beloew represents a an example software template and in this regard we assume that the code for quarkus service, manifests is already written for the purposes of focusing on developer hub related elemnents of a software template 

```
# 📁 template-quarkus-simple-main

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
### **A quick overview of `template.yaml`?**
The Software Templates part of Backstage is a tool that can help you create Components inside Backstage. By default, it has the ability to load skeletons of code, template in some variables, and then publish the template to some locations like GitHub or GitLab.

Templates are stored in the Software Catalog under a kind Template. You can create your own templates with a small yaml definition which describes the template and its metadata, along with some input variables that your template will need, and then a list of actions which are then executed by the scaffolding service.
---
## **🛠 Step 2: Start with a Blank `template.yaml`**
**Goal**: Explain that Red Hat Developer Hub uses templates to scaffold services and that we will **incrementally build `template.yaml`**.

### **Instructions**
1. Open **VS Code** and navigate to your Red Hat Developer Hub Software template repository.
2. **Create a new file**: `template.yaml`
3. **Start with a blank template** and **add basic comments**:




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
## **🛠 Step 3: Define User Input Parameters**
 **Goal**: Collect user input for the service.

### **Instructions**
1. **Add the parameters section**:

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
        java_package_name:
          title: Java Package Name
          type: string
          description: Package for Java classes
          default: com.redhat.rhdh
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


## **🛠 Step 4: Fetch Quarkus Boilerplate Code**
**Goal**: Copy a **predefined Quarkus project**.

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


## **🛠 Step 5: Create a Git Repository and Push Code**
**Goal**: Automatically create a **GitLab repository**.

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

## **🛠 Step 6: Register the Service in Red Hat Developer Hub**
**Goal**: Add the service to the **Red Hat Developer Hub Software catalog**.

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

**⏩ Test It in Red Hat Developer Hub** → The new service should appear in the catalog.


## **🛠 Step 6: Build `catalog-info.yaml` Incrementally**

**Goal**: Register the service in Red Hat Developer Hub by **building `catalog-info.yaml` step by step**.


### **1️⃣ Start with a Blank `catalog-info.yaml`**

**Why?** Every service created by Red Hat Developer Hub **must be registered** in the catalog.

#### **Instructions**

1.  **Create a new file**: `catalog-info.yaml`
2.  **Add the basic `Component` definition**:

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

### Explanation

-   **What is `kind: Component`?**
    -   It tells Red Hat Developer Hub that **this entity represents a microservice**.
-   **What is `lifecycle: development`?**
    -   It shows that the service is **still being developed, not yet in production**.
-   **Why do we need `owner`?**
    -   Red Hat Developer Hub **requires every service to have an owner** for accountability.

----------

### **2️⃣ Add Metadata (Description & Tags)**

**Why?** Helps users **identify** and **search** for the service.

#### **Update `metadata` section**:

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

###Explanation

-   **What does `description` do?**
    -   It provides **a short summary of the service** so developers can understand what it does.
-   **Why add `tags`?**
    -   It makes the service **easier to find** inside Red Hat Developer Hub.

----------

### **3️⃣ Add Annotations for GitOps and CI/CD**

**Why?** These annotations **link Red Hat Developer Hub to GitLab, ArgoCD, and Kubernetes**.

#### **Update `annotations` section**:

```yaml
  annotations:
    argocd/app-selector: rht-gitops.com/${{ values.gitops_namespace }}=${{ values.owner }}-${{values.component_id}}
    backstage.io/kubernetes-id: ${{values.component_id}}
    janus-idp.io/tekton: ${{values.component_id}}
    backstage.io/source-location: url:https://${{values.host}}/${{values.destination}}
    backstage.io/techdocs-ref: url:https://${{values.host}}/${{values.destination}}
    gitlab.com/project-slug: ${{values.destination}}

```
###Explanation

-   **Why do we need `argocd/app-selector`?**
    -   This **links the service to ArgoCD**, allowing Red Hat Developer Hub to track GitOps deployments.
-   **What is `backstage.io/source-location`?**
    -   It tells Red Hat Developer Hub **where the service’s source code lives** (GitLab, GitHub, etc.).
-   **What is `janus-idp.io/tekton`?**
    -   It **enables CI/CD tracking in Red Hat Developer Hub** for Tekton pipelines.

----------

### **4️⃣ Add Developer Links for OpenShift Dev Spaces**

**Why?** Allows developers to **open the service directly in VS Code or JetBrains**.

#### **Update `links` section**:

```yaml
  links:
    - url: https://devspaces.${{values.cluster}}/#https://${{values.host}}/${{values.destination}}
      title: OpenShift Dev Spaces (VS Code)
      icon: web
    - url: https://devspaces.${{values.cluster}}/#https://${{values.host}}/${{values.destination}}?che-editor=che-incubator/che-idea/latest
      title: OpenShift Dev Spaces (JetBrains IntelliJ)
      icon: web

```
###Explanation

-   **Why add these links?**
    -   Developers can click these **to open the service in OpenShift Dev Spaces** for live coding.

----------

### **5️⃣ Register the API in Red Hat Developer Hub**

📌 **Why?** If the service **exposes an API**, we need to **document it** in Red Hat Developer Hub.

#### **Add API registration to `catalog-info.yaml`**:

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
- The `kind: API` declaration in **`catalog-info.yaml`** tells Red Hat Developer Hub that the entity being registered is an **API** rather than a standard microservice (`kind: Component`).  
- This allows Red Hat Developer Hub to manage the API separately from services, making it **discoverable, reusable, and integrable** with other systems.  
- APIs registered this way can be **documented, versioned, and consumed** by other teams directly from the Red Hat Developer Hub UI.  

#### **What is `definition: $text: ./openapi.yaml`?**  
- This field **links the API entity** to an **OpenAPI specification** stored in `openapi.yaml`.  
- It enables **automatic API documentation rendering** in Red Hat Developer Hub, allowing developers to view, interact with, and test the API within the Red Hat Developer Hub UI.  
- This helps teams maintain **consistency in API documentation** while ensuring APIs are easily discoverable and accessible.  

#### **Why does RHDH need this step?**  
- **Red Hat Developer Hub (RHDH)** uses **`catalog-info.yaml`** to recognize and register software components and APIs into its **Software Catalog**.  
- Manually importing a component ensures that the service/API is **visible, traceable, and manageable** within the enterprise developer portal.  
- The registration process allows **teams to standardize API documentation**, monitor API usage, and enable seamless **API lifecycle management** within RHDH.  

#### **How to Register an API in RHDH?**  
1. Navigate to **"Register Component"** in RHDH.  
2. Enter the **Git repository URL** where `catalog-info.yaml` is stored.  
3. Click **Analyze → Import** to add the API to the catalog.  

This process ensures **all APIs and services are properly documented and tracked** within RHDH, improving discoverability and governance. 🚀

---

## **🎯 Final Steps**
1️⃣ **Review the full `template.yaml` and `catalog-info.yaml`.**  
2️⃣ **Run a final test in template editor UI for Red Hat Developer Hub .**  
3️⃣ **Open Q&A and let participants modify the template.**  

