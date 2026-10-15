### **step-by-step tutorial** to deploy an application as a Docker container using Jenkins in a fully automated way.

---

### **Objective**
Deploy a sample application as a Docker container on a Docker host using a Jenkins pipeline (Jenkinsfile).

---

### **Prerequisites**
1. **Jenkins** installed and running on your server.
2. Jenkins user with permission to execute Docker commands (`dockeradmin` user must be part of the Docker group).
3. Docker installed on the host machine.
4. **GitHub repository** for the source code.
5. Plugins installed in Jenkins:
   - Pipeline
   - Docker Pipeline
   - Publish Over SSH (if remote deployment is needed)

---

### **Step 1: Prepare the Sample Source Code**

Create the following structure for the application:
```bash
my-app/
 ├── app.py               # Sample Python web app
 ├── requirements.txt     # Dependencies file
 └── Dockerfile           # Dockerfile to containerize the app
```

#### **app.py**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, World! This is my Jenkins CI/CD pipeline demo."

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### **requirements.txt**
```
flask
```

#### **Dockerfile**
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 5000
CMD ["python", "app.py"]
```

Push this code to your GitHub repository.

---

### **Step 2: Create a Docker Host**

If not already done, set up a Docker host (or use the Jenkins server itself).

Ensure Docker is installed and running:
```bash
sudo apt update
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

Add Jenkins user (`dockeradmin`) to the Docker group:
```bash
sudo usermod -aG docker dockeradmin
```

Restart Jenkins for the changes to take effect.

---

### **Step 3: Configure Jenkins for Docker**

1. Navigate to **Manage Jenkins** > **Global Tool Configuration**.
2. Configure the Docker installation path.
3. Ensure that Docker is accessible by Jenkins (test with: `docker ps`).

---

### **Step 4: Create Jenkins Credentials**

Add credentials for accessing the GitHub repository and Docker registry if needed:
- **GitHub Credentials:** If your repo is private, add SSH or HTTPS credentials.
- **DockerHub Credentials:** If pushing to a private DockerHub repository.

---

### **Step 5: Create a Jenkins Pipeline Job**

1. Go to **New Item** > **Pipeline**.
2. Name the job and select **Pipeline**.
3. Scroll to **Pipeline** > **Pipeline script from SCM**.
4. Choose **Git** and provide the URL of your GitHub repo.
5. Add credentials if necessary and specify the branch (e.g., `main`).
6. In the **Script Path**, specify the name of your Jenkinsfile (e.g., `Jenkinsfile`).

---

### **Step 6: Write the Jenkinsfile**

Create a file named **Jenkinsfile** in the root of your repo.

#### **Jenkinsfile**
```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-docker-repo/my-app:latest'   // Change to your Docker repo
    }

    stages {
        stage('Clone Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    script {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh """
                    docker stop my-app || true
                    docker rm my-app || true
                    docker run -d --name my-app -p 5000:5000 ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
```

---

### **Step 7: Run the Pipeline**

1. Click **Build Now** on your Jenkins job.
2. Monitor the pipeline stages in the **Build History**.

---

### **Step 8: Verify the Deployment**

After the pipeline completes successfully, verify that the application is running:
```bash
docker ps
```

Access the application in your browser using the Docker host's IP:
```
http://<docker-host-ip>:5000
```

You should see: **Hello, World! This is my Jenkins CI/CD pipeline demo.**

---

### **Step 9: Automate Pipeline Execution (Optional)**

Set up a webhook in GitHub to trigger the Jenkins pipeline on every code push:
1. In your GitHub repository, go to **Settings** > **Webhooks**.
2. Click **Add webhook**.
3. Enter the Jenkins webhook URL (e.g., `http://<jenkins-server>/github-webhook/`).
4. Select **Just the push event** and save.

---

### **Step 10: Troubleshooting Tips**

- If Docker commands fail in Jenkins, check user permissions (`dockeradmin` must be part of the Docker group).
- Verify that Docker is installed and running on the host.
- Ensure credentials are correctly set for GitHub and DockerHub.

---

### **Summary**

This tutorial covered the steps to:
1. Write a sample Python web application.
2. Create a Dockerfile to containerize the application.
3. Automate the build, push, and deployment process using a Jenkins pipeline.

