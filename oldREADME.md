# **📖 Step-by-Step Guide: Running the Backstage Template Demo in VS Code**

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

## 📌 Step 1: Fork and Clone the Repository

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
📌 **At this point, you have a repository with an empty `template.yaml` and `catalog-info.yaml`, ready to build your Backstage Software Template.**

🚀 **Now this is optimized for documentation, README files, or onboarding guides!** Let me know if you need any refinements. 🔥

📌 **At this point, you have a repository with an empty `template.yaml` and `catalog-info.yaml`, ready to build your Backstage Software Template.**
---

### **2. Other Required Tools**
- **VS Code** installed.
- **A running instance of Backstage** (or **Red Hat Developer Hub**).
- **Your Backstage repository cloned** on your machine.
- **A GitLab (or GitHub) instance** to publish repositories.
- **ArgoCD configured** if testing deployments.
---

## **🛠 Step 1: Open VS Code and Start with a Blank `template.yaml`**
📌 **Goal**: Explain that Backstage uses templates to scaffold services and that we will **incrementally build `template.yaml`**.

### **Instructions:**
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

### **What to Explain to the Audience**
✅ **What is this file?** → It defines how Backstage scaffolds new services.  
✅ **What does `apiVersion` mean?** → We use `scaffolder.backstage.io/v1beta3` for templates.  
✅ **Why do we need `metadata`?** → It provides **searchable tags** and a **friendly title**.  

**🎯 Now, save the file and move to Step 2.**  

---

## **🛠 Step 2: Define User Input Parameters**
📌 **Goal**: Explain that we need **parameters** to collect user input for the service.

### **Instructions:**
1. Add the **parameters** section to `template.yaml`:

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

### **Explanation**
✅ **Why do we need parameters?** → To allow users to provide values before service creation.  
✅ **How does `required` work?** → It ensures certain fields must be filled.  
✅ **What is `ui:field: EntityNamePicker`?** → A UI helper for selecting component names.

### **⏩ Test It in Backstage**
1. **Run Backstage** (`yarn dev` if running locally).  
2. Go to **"Create"** and select your template.  
3. You should see a **form asking for `component_id` and `java_package_name`**.

---

## **🛠 Step 3: Fetch Quarkus Boilerplate Code**
📌 **Goal**: Copy a **predefined Quarkus project** into the new repository.

### **Instructions:**
1. **Add this step** to `template.yaml` under `steps:`:

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

### **What to Explain to the Audience**
✅ **What does `fetch:template` do?** → It **copies the Quarkus skeleton project** into a new folder.  
✅ **Why use `${{ parameters.component_id }}`?** → To **rename the project dynamically**.  
✅ **What is `targetPath`?** → The location where the new service is generated.

### **⏩ Test It in Backstage**
1. **Start Backstage** and fill in the form.
2. The **Quarkus project should now be copied** into a new directory.

---

## **🛠 Step 4: Create a Git Repository and Push Code**
📌 **Goal**: Automatically **create a GitLab repository** and push the generated service.

### **Instructions:**
1. **Add the `publish:gitlab` step**:

```yaml
    - id: publish
      name: Publish
      action: publish:gitlab
      input:
        repoUrl: "${{ parameters.repo.host }}?owner=${{ user.entity.metadata.name }}&repo=${{parameters.component_id}}"
        repoVisibility: public
        defaultBranch: main
        sourcePath: ./${{ user.entity.metadata.name }}-${{ parameters.component_id }}
```




**What does this step do?**  
This step **automates the creation of a Git repository** where the generated service code will be stored. Instead of manually creating a repository on GitLab or GitHub, initializing it, and pushing the files, Backstage does this **automatically** using the `publish:gitlab` (or `publish:github`) action.  

💡 **Without automation**, users would have to:  
1. Manually create a new Git repository.  
2. Clone the empty repository locally.  
3. Copy their generated project files into the repo.  
4. Run `git add .`, `git commit -m "Initial commit"`, and `git push`.  
✅ **With this Backstage step**, all of this happens **in one click**.  

---

✅ **Why use `${{ parameters.repo.host }}`?**  
`${{ parameters.repo.host }}` is a **user-defined input parameter** that ensures the template is **flexible and reusable** for different source code management (SCM) hosts.  

💡 **Example Scenarios**:  
- If a company uses **GitLab**, the user could input:  
  ```yaml
  parameters:
    repo:
      host: gitlab.company.com
  ```  
- If they use **GitHub**, the user could input:  
  ```yaml
  parameters:
    repo:
      host: github.com
  ```  

This makes the template **adaptable** instead of **hardcoding a specific repository provider**.  

---

✅ **What is `sourcePath`?**  
The `sourcePath` field tells Backstage **which folder to push** to the Git repository.  

💡 **Example Scenario**:  
If the generated project files are placed in:
```
/workspace/my-project
```
Then using:
```yaml
sourcePath: ./${{ user.entity.metadata.name }}-${{ parameters.component_id }}
```
ensures that **only the generated service files** (not the entire Backstage template repo) get pushed.  

✅ **Why is this important?**  
- Without `sourcePath`, Backstage might **push unnecessary files** like template configuration files.  
- Ensures **only relevant service code** is added to the repository.  

---

🚀 **With this step, we have a streamlined, flexible, and automated approach to repository creation, making the onboarding process seamless and efficient!** Let me know if you need refinements. 🔥  
```

### **⏩ Test It in Backstage**
1. Run the template.
2. **Check GitLab** – a new repo should be created!

---

## **🛠 Step 5: Register the Service in Backstage**
📌 **Goal**: Add the service to the **Backstage catalog** for tracking.

### **Instructions:**
1. **Add the `catalog:register` step**:

```yaml
    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: "/catalog-info.yaml"
```

### **What to Explain to the Audience**
✅ **Why do we need this step?** → Without it, Backstage won’t know the service exists.  
✅ **What is `repoContentsUrl`?** → It points to the **new repository contents**.  

### **⏩ Test It in Backstage**
1. Run the template.
2. The new service should now appear in **Backstage’s Software Catalog**.

---

## **🛠 Step 6: Deploy with ArgoCD**
📌 **Goal**: **Automatically deploy the service** using ArgoCD.

### **Instructions:**
1. **Add the `argocd:create-resources` step**:

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

### **What to Explain to the Audience**
✅ **Why use GitOps?** → It enables **automated deployments**.  
✅ **How does ArgoCD work?** → It watches the Git repo and deploys changes.

### **⏩ Test It in ArgoCD**
1. Open **ArgoCD UI**.
2. You should see the **new application deployed**.

---

## **🎯 Final Steps**
1️⃣ **Review the full `template.yaml`.**  
2️⃣ **Manually import the service into RHDH** using the UI.  
3️⃣ **Confirm the service appears in the catalog.**  
4️⃣ **Run a final test and answer Q&A.**  


Yes! You can **incrementally build** the `catalog-info.yaml` file **step by step in VS Code**, just like we did for `template.yaml`. This approach will help **explain each section** before adding it, making it **clear and engaging for the audience**.

---

# **📖 Step-by-Step Guide: Building `catalog-info.yaml` in VS Code**

### ✅ **Prerequisites**
Before starting, ensure you have:
- **VS Code open** with your Backstage repository.
- **A running Backstage or Red Hat Developer Hub (RHDH) instance**.
- **Your `template.yaml` is working**, so it generates `catalog-info.yaml`.

---

## **🛠 Step 1: Start with a Blank `catalog-info.yaml` with Comments**
📌 **Goal**: Explain that `catalog-info.yaml` **registers the service in Backstage**.

### **Instructions**
1. Open **VS Code**.
2. **Create a new file**: `catalog-info.yaml`
3. **Start with a blank template** and **add basic comments**:

```yaml
# Backstage Component registration file
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: ${{values.component_id | dump}}
spec:
  type: service
  lifecycle: development
  owner: ${{values.owner | dump}}
```

### **What to Explain to the Audience**
✅ **Why is this file important?** → Backstage needs `catalog-info.yaml` to **discover the service**.  
✅ **What is `apiVersion: backstage.io/v1alpha1`?** → It defines this as a **Backstage entity**.  
✅ **What does `kind: Component` mean?** → It tells Backstage this is a **service**.  
✅ **Why use `${{values.component_id}}`?** → It dynamically assigns the **name from user input**.

---

## **🛠 Step 2: Add Metadata Section**
📌 **Goal**: Add **description and tags** to make the component **easier to find**.

### **Instructions**
1. **Update `metadata` section**:

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

### **What to Explain to the Audience**
✅ **Why add a description?** → Helps other developers understand **what the service does**.  
✅ **What are tags for?** → Makes it **easier to search** in Backstage.  

---

## **🛠 Step 3: Add Annotations for Backstage and ArgoCD**
📌 **Goal**: Connect the service to **ArgoCD, Kubernetes, GitLab, and TechDocs**.

### **Instructions**
1. **Add the `annotations` section**:

```yaml
  annotations:
    argocd/app-selector: rht-gitops.com/${{ values.gitops_namespace }}=${{ values.owner }}-${{values.component_id}}
    backstage.io/kubernetes-id: ${{values.component_id}}
    janus-idp.io/tekton: ${{values.component_id}}
    backstage.io/source-location: url:https://${{values.host}}/${{values.destination}}
    backstage.io/techdocs-ref: url:https://${{values.host}}/${{values.destination}}
    gitlab.com/project-slug: ${{values.destination}}
```

### **What to Explain to the Audience**
✅ **Why is `argocd/app-selector` needed?** → It **links the service to ArgoCD** for GitOps deployment.  
✅ **What does `backstage.io/kubernetes-id` do?** → It **connects Backstage to the Kubernetes deployment**.  
✅ **Why `backstage.io/source-location`?** → This allows **clicking on the repo link inside Backstage**.  
✅ **What does `gitlab.com/project-slug` do?** → It **links the service to GitLab for CI/CD visibility**.  

**⏩ Run and test** in Backstage → The new service should appear **with GitLab and Kubernetes integrations!**  

---

## **🛠 Step 4: Add Developer Links**
📌 **Goal**: Provide **direct links** to VS Code and JetBrains in OpenShift Dev Spaces.

### **Instructions**
1. **Add the `links` section**:

```yaml
  links:
    - url: https://devspaces.${{values.cluster}}/#https://${{values.host}}/${{values.destination}}
      title: OpenShift Dev Spaces (VS Code)
      icon: web
    - url: https://devspaces.${{values.cluster}}/#https://${{values.host}}/${{values.destination}}?che-editor=che-incubator/che-idea/latest
      title: OpenShift Dev Spaces (JetBrains IntelliJ)
      icon: web
```

### **What to Explain to the Audience**
✅ **Why add links?** → They allow **developers to open the service directly in OpenShift Dev Spaces**.  
✅ **What do the URLs do?** → They **point to the exact Git repo** for easy access.  

---

## **🛠 Step 5: Add API Component**
📌 **Goal**: Register the API inside Backstage **so it appears under "APIs"**.

### **Instructions**
1. **Add a new API definition** **below** the `Component`:

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

### **What to Explain to the Audience**
✅ **What is `kind: API`?** → It registers the API in Backstage’s **API catalog**.  
✅ **Why use `definition: $text: ./openapi.yaml`?** → It tells Backstage to **use the OpenAPI spec**.  
✅ **How does this help developers?** → They can **see and test the API directly in Backstage**.  



