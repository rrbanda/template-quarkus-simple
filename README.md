# **📖 Step-by-Step Guide: Running the Incremental Backstage Template Demo in VS Code**

### ✅ **Prerequisites**
Before starting, ensure you have:
- **VS Code** installed.
- **A running instance of Backstage** (or Red Hat Developer Hub).
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

### **What to Explain to the Audience**
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

### **What to Explain to the Audience**
✅ **What does this step do?** → It **automates repository creation** instead of users doing it manually.  
✅ **Why use `${{ parameters.repo.host }}`?** → To allow flexibility for different SCM hosts.  
✅ **What is `sourcePath`?** → It tells Backstage **what folder to push** to GitLab.

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

