# **🎯 Workshop Goal**
By the end of this session, participants will:
- ✅ **Learn how to create a Backstage Software Template from scratch.**
- ✅ **Understand the folder structure and role of each file.**
- ✅ **Build `template.yaml` and `catalog-info.yaml`.**
- ✅ **Automate Git repo creation and GitOps deployment.**
- ✅ **Register and import services in Red Hat Developer Hub (RHDH).**
- ✅ **Deploy a microservice using ArgoCD.**

---

To **fit everything into 30 minutes** while maintaining **clarity and hands-on engagement**, we need to **condense explanations** and **focus on the most critical coding parts**. Below is the **optimized agenda for 30 minutes**:

---
## 📜 Agenda
| **Time** | **Activity** | **Why It’s Important** |
|---------|-------------|----------------------|
| **0-2 mins** | **Setting Context: Understanding Backstage Entities** | What are Templates and Components? (Quick, high-level intro) |
| **2-4 mins** | **Understanding the Folder Structure** | Where do `template.yaml`, `catalog-info.yaml`, and the Quarkus skeleton fit? (Brief overview) |
| **4-18 mins** | **Step 1: Build `template.yaml` Incrementally** | Define parameters, fetch boilerplate, create GitLab repo, register in Backstage, deploy with ArgoCD. (Live coding) |
| **18-26 mins** | **Step 2: Build `catalog-info.yaml` Incrementally** | Why does Backstage need it? How does it relate to GitOps? (Live coding + quick test) |
| **26-28 mins** | **Step 3: Importing and Registering in Red Hat Developer Hub (RHDH)** | How to manually register the service into RHDH. (Quick live demo) |
| **28-30 mins** | **Step 4: Q&A + Customization Challenge** | Let participants tweak the template for real-world use cases. |

---

# **🛠 Step 1: Setting Up the Live Coding Demo**
### ✅ **Prerequisites**
Before starting, ensure you have:
- **VS Code open** with your Backstage repository.
- **A running Backstage or Red Hat Developer Hub (RHDH) instance**.
- **A working GitLab (or GitHub) instance**.
- **ArgoCD configured** if testing deployments.

---

## **🛠 Step 2: Start with a Blank `template.yaml`**
📌 **Goal**: Explain that Backstage uses templates to scaffold services and that we will **incrementally build `template.yaml`**.

### **Instructions**
1. Open **VS Code** and navigate to your Backstage template repository.
2. **Create a new file**: `template.yaml`
3. **Start with a blank template** and **add basic comments**:

```yaml
# Backstage Software Template Definition
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

✅ **Explain**:
- What is `template.yaml`?  
- Why do we need `apiVersion`, `kind`, and `metadata`?

---

## **🛠 Step 3: Define User Input Parameters**
📌 **Goal**: Collect user input for the service.

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

✅ **Explain**:
- Why do we need parameters?
- What is `EntityNamePicker`?

**⏩ Test It in Backstage** → Run the template and check the form UI.

---

## **🛠 Step 4: Fetch Quarkus Boilerplate Code**
📌 **Goal**: Copy a **predefined Quarkus project**.

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

✅ **Explain**:
- Why use `fetch:template`?
- How does `${{ parameters.component_id }}` work?

---

## **🛠 Step 5: Create a Git Repository and Push Code**
📌 **Goal**: Automatically create a **GitLab repository**.

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

✅ **Explain**:
- Why use `publish:gitlab`?
- What does `sourcePath` do?

**⏩ Test It in Backstage** → Run the template and check GitLab.

---

## **🛠 Step 6: Register the Service in Backstage**
📌 **Goal**: Add the service to the **Backstage catalog**.

```yaml
    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: "/catalog-info.yaml"
```

✅ **Explain**:
- Why is this necessary?

**⏩ Test It in Backstage** → The new service should appear in the catalog.


---


## **🛠 Step 6: Build `catalog-info.yaml` Incrementally**

📌 **Goal**: Register the service in Backstage by **building `catalog-info.yaml` step by step**.

----------

### **1️⃣ Start with a Blank `catalog-info.yaml`**

📌 **Why?** Every service created by Backstage **must be registered** in the catalog.

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

✅ **Explain**:

-   **What is `kind: Component`?**
    -   It tells Backstage that **this entity represents a microservice**.
-   **What is `lifecycle: development`?**
    -   It shows that the service is **still being developed, not yet in production**.
-   **Why do we need `owner`?**
    -   Backstage **requires every service to have an owner** for accountability.

----------

### **2️⃣ Add Metadata (Description & Tags)**

📌 **Why?** Helps users **identify** and **search** for the service.

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

✅ **Explain**:

-   **What does `description` do?**
    -   It provides **a short summary of the service** so developers can understand what it does.
-   **Why add `tags`?**
    -   It makes the service **easier to find** inside Backstage.

----------

### **3️⃣ Add Annotations for GitOps and CI/CD**

📌 **Why?** These annotations **link Backstage to GitLab, ArgoCD, and Kubernetes**.

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

✅ **Explain**:

-   **Why do we need `argocd/app-selector`?**
    -   This **links the service to ArgoCD**, allowing Backstage to track GitOps deployments.
-   **What is `backstage.io/source-location`?**
    -   It tells Backstage **where the service’s source code lives** (GitLab, GitHub, etc.).
-   **What is `janus-idp.io/tekton`?**
    -   It **enables CI/CD tracking in Backstage** for Tekton pipelines.

----------

### **4️⃣ Add Developer Links for OpenShift Dev Spaces**

📌 **Why?** Allows developers to **open the service directly in VS Code or JetBrains**.

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

✅ **Explain**:

-   **Why add these links?**
    -   Developers can click these **to open the service in OpenShift Dev Spaces** for live coding.

----------

### **5️⃣ Register the API in Backstage**

📌 **Why?** If the service **exposes an API**, we need to **document it** in Backstage.

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

✅ **Explain**:

-   **What is `kind: API`?**
    -   It tells Backstage that **this is an API** (not just a microservice).
-   **What is `definition: $text: ./openapi.yaml`?**
    -   It **links the API to an OpenAPI spec file**, allowing Backstage to display **API documentation**.

----------



---

te → Register Component**.
2. Enter the **Git repository URL**.
3. Click **Analyze → Import**.

✅ **Explain**:
- Why does RHDH need this step?

---

## **🎯 Final Steps**
1️⃣ **Review the full `template.yaml` and `catalog-info.yaml`.**  
2️⃣ **Run a final test in Backstage.**  
3️⃣ **Open Q&A and let participants modify the template.**  

🚀 **Now you have the ultimate live demo!** Would you like any final refinements?
