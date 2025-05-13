# Configuration-management-with-helms
Great — now that I understand you're working on the "Configuration Management With Helm" Capstone Project, I’ll provide a clear, structured walkthrough for each step you need to complete, including what to do, where to do it, and why.

---

### **Step-by-Step Implementation Guide**

---

### **STEP 1: Set Up Your Jenkins Server**

**Objective:** Get Jenkins running and ready to automate deployments.

#### **Where to do it:**

* On your local machine (Windows/Linux) or a virtual machine (e.g., EC2 on AWS).

#### **What to do:**

1. **Install Jenkins:**

   * **Windows:** Use Chocolatey:

     ```bash
     choco install jenkins-lts
     ```
   * **Linux (Debian-based):**

     ```bash
     sudo apt update
     sudo apt install openjdk-11-jdk
     curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
     sudo apt update
     sudo apt install jenkins
     ```

2. **Start Jenkins and Access Dashboard:**

   * Navigate to `http://localhost:8080` or `http://<your-vm-ip>:8080`
   * Unlock Jenkins with the initial admin password (shown in the terminal or located in `C:\Program Files (x86)\Jenkins\secrets\initialAdminPassword`)

3. **Install Recommended Plugins:**

   * Select “Install Suggested Plugins”

4. **Basic Security Setup:**

   * Create an admin user and set permissions
   * Install the **Git** and **Pipeline** plugins manually if not automatically installed.

---

### **STEP 2: Create a Basic Helm Chart**

**Objective:** Learn and build a Helm chart structure for a web app.

#### **Where to do it:**

* Your local development folder (e.g., `~/projects/helm-web-app`)

#### **What to do:**

1. **Install Helm:**

   * **Windows (via Chocolatey):**

     ```bash
     choco install kubernetes-helm
     ```
   * **Linux/macOS:**

     ```bash
     curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
     ```

2. **Create Helm Chart:**

   ```bash
   helm create webapp
   ```

3. **Explore & Customize the Chart:**

   * `Chart.yaml`: Metadata

   * `values.yaml`: Set these values:

     ```yaml
     replicaCount: 2
     image:
       repository: nginx
       tag: stable
       pullPolicy: IfNotPresent
     ```

   * `templates/deployment.yaml`: Add resource requests:

     ```yaml
     resources:
       requests:
         memory: "128Mi"
         cpu: "100m"
       limits:
         memory: "256Mi"
         cpu: "200m"
     ```

---

### **STEP 3: Push Code to GitHub**

**Objective:** Version control and make it accessible to Jenkins.

#### **Where to do it:**

* Terminal in the project folder.

#### **What to do:**

```bash
git init
git remote add origin <your-repo-url>
git add .
git commit -m "Initial Helm chart for webapp"
git push -u origin main
```

---

### **STEP 4: Deploy Using Helm Locally**

**Objective:** Test manual deployment before automation.

#### **Where to do it:**

* Terminal (ensure Kubernetes cluster is running, e.g., Minikube or Kind)

#### **What to do:**

```bash
helm install my-webapp ./webapp
kubectl get all
```

---

### **STEP 5: Integrate Jenkins with Helm for CI/CD**

**Objective:** Automate the deployment process.

#### **Where to do it:**

* Jenkins Dashboard

#### **What to do:**

1. **Create New Jenkins Pipeline Job**

   * Go to Jenkins → New Item → Name it `Helm-Webapp-CI` → Select “Pipeline”

2. **Set GitHub Repo:**

   * Under “Pipeline” → “Pipeline Script from SCM” → Git → Add your repo URL

3. **Find Helm Path (Windows Example):**

   ```powershell
   Get-Command helm | Select-Object -ExpandProperty Source
   ```

4. **Use Helm in Jenkinsfile (example):**

   ```groovy
   pipeline {
     agent any
     stages {
       stage('Deploy with Helm') {
         steps {
           script {
             def helmPath = "C:\\tools\\helm\\helm.exe"  // Replace with your path
             bat "${helmPath} upgrade --install my-webapp ./webapp"
           }
         }
       }
     }
   }
   ```

---

### **STEP 6: Trigger Build via Git Push**

1. **Make a Change:**

   * E.g., change `replicaCount` to 3 in `values.yaml`

2. **Push to GitHub:**

   ```bash
   git add .
   git commit -m "Update replica count to 3"
   git push
   ```

3. **Observe Jenkins:**

   * A new build should start.
   * Check Jenkins console log to ensure Helm is deploying the new chart.

---

### **STEP 7: Validate Deployment**

```bash
kubectl get deployments
kubectl get pods
kubectl port-forward <pod-name> 8081:<container-port>
```

Visit `http://localhost:8081` to access your app.

---

