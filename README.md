# 🚀 Jenkins Declarative Pipeline – Complete Reference Guide

> A comprehensive guide covering Jenkins installation, Terraform setup on Ubuntu, and every major Declarative Pipeline concept with ready-to-use examples.

---

## 📋 Table of Contents

- [Jenkins Installation (Ubuntu)](#-jenkins-installation-ubuntu)
- [Terraform Installation (Ubuntu)](#-terraform-installation-ubuntu)
- [Pipeline Structure](#-pipeline-structure)
- [Agent](#-agent)
- [Stages & Steps](#-stages--steps)
- [Environment Variables](#-environment-variables)
- [Credentials](#-credentials)
- [When (Conditions)](#-when-conditions)
- [Tools](#-tools)
- [Parallel Execution](#-parallel-execution)
- [Post Actions](#-post-actions)
- [Triggers](#-triggers)
- [Build Status – Unstable](#-marking-build-as-unstable)

---

## ☕ Jenkins Installation (Ubuntu)

### Step 1 – Install Java 17
```bash
sudo apt install fontconfig openjdk-17-jre -y
java -version
```

### Step 2 – Add Jenkins Repository Key
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
```

### Step 3 – Add Jenkins Apt Repository
```bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Step 4 – Install Jenkins
```bash
sudo apt update
sudo apt install jenkins -y
```

### Step 5 – Enable & Start Jenkins
```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

> 🌐 Access Jenkins at: `http://<your-server-ip>:8080`

---

## 🟦 Terraform Installation (Ubuntu)

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

sudo apt-add-repository \
  "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

sudo apt update
sudo apt install terraform -y

terraform -v
```

---

## 🏗️ Pipeline Structure

A Jenkins Declarative Pipeline always follows this skeleton:

```
pipeline {
    agent      → Where to run
    tools      → Tools to load (Maven, JDK, etc.)
    environment → Variables
    triggers   → Schedule / webhook triggers
    stages {
        stage('name') {
            when   → Conditions
            steps  → Actual commands
            post   → Stage-level post actions
        }
    }
    post       → Pipeline-level post actions
}
```

### Minimal "Hello World" Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Welcome Step') {
            steps {
                echo 'Welcome to Jenkins!'
            }
        }
    }
}
```

---

## 🤖 Agent

The `agent` directive tells Jenkins **where** to run the pipeline.

### `agent any` – Run on any available node
```groovy
pipeline {
    agent any
}
```

### `agent none` – No global agent (define per stage)
```groovy
pipeline {
    agent none
}
```

### `agent label` – Run on a specific node
```groovy
pipeline {
    agent {
        label 'linux-machine'
    }
}
```

---

## 📦 Stages & Steps

### `stages` – Container for all stages
```groovy
pipeline {
    agent { label 'linux-machine' }
    stages {
        // stages go here
    }
}
```

### `stage` – A single named phase
```groovy
pipeline {
    agent { label 'linux-machine' }
    stages {
        stage('Build') {
            // steps go here
        }
    }
}
```

### `steps` – Commands inside a stage
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Build stage is running"
            }
        }
    }
}
```

---

## 🌍 Environment Variables

### 1. Refer to a Variable Defined in Jenkins UI
```groovy
pipeline {
    agent any
    stages {
        stage('Initialization') {
            steps {
                echo "${JAVA_INSTALLATION_PATH}"
            }
        }
    }
}
```

### 2. Define Variables in the Jenkinsfile
```groovy
pipeline {
    agent any
    environment {
        DEPLOY_TO = 'production'
    }
    stages {
        stage('Initialization') {
            steps {
                echo "${DEPLOY_TO}"
            }
        }
    }
}
```

### 3. Initialize Using Shell Command Output
```groovy
pipeline {
    agent any
    stages {
        stage('Initialization') {
            environment {
                JOB_TIME = sh(returnStdout: true, script: "date '+%A %W %Y %X'").trim()
            }
            steps {
                sh 'echo $JOB_TIME'
            }
        }
    }
}
```

---

## 🔐 Credentials

Load credentials stored in Jenkins Credential Manager:

```groovy
pipeline {
    agent any
    environment {
        MY_CRED = credentials('MY_SECRET')
    }
    stages {
        stage('Load Credentials') {
            steps {
                echo "Username is $MY_CRED_USR"
                echo "Password is $MY_CRED_PSW"
            }
        }
    }
}
```

> 💡 Jenkins automatically appends `_USR` and `_PSW` suffixes for username/password credential types.

---

## 🔀 When (Conditions)

The `when` block controls whether a stage should execute.

### 1. Run Only on a Specific Branch
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            when {
                branch 'dev'
            }
            steps {
                echo "Working on dev branch"
            }
        }
    }
}
```

### 2. Using a Groovy Expression
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            when {
                expression {
                    return env.BRANCH_NAME == 'dev'
                }
            }
            steps {
                echo "Working on dev branch"
            }
        }
    }
}
```

### 3. Match an Environment Variable
```groovy
pipeline {
    agent any
    environment {
        DEPLOY_TO = 'production'
    }
    stages {
        stage('Deploy') {
            when {
                environment name: 'DEPLOY_TO', value: 'production'
            }
            steps {
                echo 'Deploying to production'
            }
        }
    }
}
```

### 4. `allOf` – ALL Conditions Must Match
```groovy
pipeline {
    agent any
    environment {
        DEPLOY_TO = 'production'
    }
    stages {
        stage('Deploy') {
            when {
                allOf {
                    branch 'master'
                    environment name: 'DEPLOY_TO', value: 'production'
                }
            }
            steps {
                echo 'Deploying master to production'
            }
        }
    }
}
```

### 5. `anyOf` – ANY Condition Can Match
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            when {
                anyOf {
                    branch 'master'
                    branch 'staging'
                }
            }
            steps {
                echo 'Deploying master or staging branch'
            }
        }
    }
}
```

---

## 🔧 Tools

Load pre-configured tools (Maven, JDK, etc.) from Jenkins Global Tool Configuration:

```groovy
pipeline {
    agent any
    tools {
        maven 'MAVEN_PATH'
    }
    stages {
        stage('Load Tools') {
            steps {
                sh "mvn -version"
            }
        }
    }
}
```

---

## ⚡ Parallel Execution

Run stages simultaneously on different agents:

```groovy
pipeline {
    agent any
    stages {
        stage('Parallel Stage') {
            parallel {
                stage('Windows Script') {
                    agent {
                        label "windows"
                    }
                    steps {
                        echo "Running in windows agent"
                        bat 'echo %PATH%'
                    }
                }
                stage('Linux Script') {
                    agent {
                        label "linux"
                    }
                    steps {
                        sh "echo Running in Linux agent"
                    }
                }
            }
        }
    }
}
```

> 💡 Use `parallel` to speed up independent tasks like multi-platform testing.

---

## 📬 Post Actions

Run actions **after** pipeline stages complete, based on build result:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Build stage is running"
            }
        }
    }
    post {
        always {
            echo "This always runs regardless of result"
        }
        success {
            echo "Pipeline completed successfully ✅"
        }
        unstable {
            echo "Build is unstable ⚠️ — check test results"
        }
        failure {
            echo "Pipeline FAILED ❌ — check logs"
        }
    }
}
```

### Post Condition Reference

| Condition | When It Runs |
|---|---|
| `always` | Every time, no matter the result |
| `success` | Only when the build succeeds |
| `failure` | Only when the build fails |
| `unstable` | When the build is marked unstable |
| `changed` | When the build result differs from the last run |
| `aborted` | When the build was manually stopped |

---

## ⏰ Triggers

Automatically trigger pipelines on a schedule using cron syntax:

```groovy
pipeline {
    agent any
    triggers {
        cron('H/15 * * * *')   // Every 15 minutes
    }
    stages {
        stage('Scheduled Run') {
            steps {
                echo 'Hello from scheduled pipeline!'
            }
        }
    }
}
```

### Cron Quick Reference

| Expression | Schedule |
|---|---|
| `H/15 * * * *` | Every 15 minutes |
| `H * * * *` | Every hour |
| `H 8 * * 1-5` | Every weekday at 8 AM |
| `@daily` | Once per day |
| `@weekly` | Once per week |

---

## ⚠️ Marking Build as UNSTABLE

### Mark Build Based on Code Coverage (JaCoCo)

```groovy
pipeline {
    agent any
    tools {
        maven 'MAVEN_PATH'
        jdk 'jdk8'
    }
    stages {
        stage('Tools Initialization') {
            steps {
                sh "mvn --version"
                sh "java -version"
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/your-org/your-repo.git'
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Code Coverage') {
            steps {
                jacoco(
                    execPattern:      '**/target/**.exec',
                    classPattern:     '**/target/classes',
                    sourcePattern:    '**/src',
                    inclusionPattern: 'com/yourpackage/**',
                    changeBuildStatus: true,
                    minimumInstructionCoverage: '30',
                    maximumInstructionCoverage: '80'
                )
            }
        }
    }
}
```

> ⚠️ If code coverage falls below `minimumInstructionCoverage` (30%), the build is marked **UNSTABLE** automatically.

---

## 📊 Directive Quick Reference

| Directive | Purpose |
|---|---|
| `pipeline` | Root block — wraps everything |
| `agent` | Where the pipeline runs |
| `stages` | Container for all `stage` blocks |
| `stage` | A named phase of the pipeline |
| `steps` | Commands executed in a stage |
| `environment` | Define or load env variables |
| `credentials` | Load Jenkins-stored secrets |
| `when` | Conditional stage execution |
| `tools` | Load pre-configured tools |
| `parallel` | Run stages simultaneously |
| `post` | Actions after build completes |
| `triggers` | Schedule or webhook-based runs |
