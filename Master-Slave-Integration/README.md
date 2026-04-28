# 🛠️ Jenkins Agent (Slave) Integration Guide

This guide details the architectural setup for a Distributed Jenkins Environment. By offloading heavy build tasks to a dedicated Slave, we protect the 4GB Master Node from crashing, ensuring the UI and orchestration remain stable.

## 🏗️ Architecture at a Glance

Jenkins Master: Handles Orchestration, UI, and Security.

Jenkins Slave (172.31.47.211): Dedicated execution environment for Maven builds and Java compilation.

Protocol: Secured via SSH (RSA 4096-bit) key-based authentication.

## 📂 Setup Phases

### Phase 1: Key Generation (Master Node)

Log into your Master instance as the jenkins user to create the "Identity" for the agent connection.

Generate the PEM Key:

Bash

ssh-keygen -t rsa -b 4096 -m PEM -f ~/.ssh/id_rsa_jenkins -N ""

Capture the Public Key:

Bash

cat ~/.ssh/id_rsa_jenkins.pub

Copy the entire string (starting with ssh-rsa) to your clipboard.

### Phase 2: Agent Preparation (Slave Node)

Log into your Slave instance (172.31.47.211) to prepare the environment for work.

User & SSH Configuration:

sudo dnf install java-21-amazon-corretto-devel maven -y

sudo useradd -m jenkins

sudo su - jenkins

mkdir -p ~/.ssh && chmod 700 ~/.ssh

echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys

JVM settings: -Djava.io.tmpdir=/home/jenkins/tmp


### Phase 3: Jenkins UI Configuration (Web Dashboard)

Add Credentials: Manage Jenkins > Credentials > Global > Add Credentials

Kind: SSH Username with private key

ID: slave-ssh-key

Username: jenkins

Private Key: Paste the content of ~/.ssh/id_rsa_jenkins from the Master.

Create the Node: Manage Jenkins > Nodes > New Node

Name: Slave-01

Remote root directory: /home/jenkins

Launch Method: Launch agents via SSH

Host: 172.31.47.211

Credentials: slave-ssh-key

Host Key Verification: Non-verifying Verification Strategy.

SRE Optimization (Crucial for t2.micro/Small Disks):

Navigate to Nodes > Configure Monitors.

Change Free Disk Space and Free Temp Space thresholds from 1GiB to 200MiB. This prevents Jenkins from marking the node as "Offline" due to low disk space common in cloud environments.

mkdir /home/jenkins/tmp

sudo chown jenkins:jenkins /home/jenkins/tmp

chmod 755 /home/jenkins/tmp


### 🚀 Validation Pipeline
Use the following declarative pipeline to verify that the Slave is correctly assuming the build workload.

pipeline {
    agent { label 'Slave-01' } 

    stages {
        stage('Identify Environment') {
            steps {
                echo "Running on Node: ${env.NODE_NAME}"
                sh 'hostname && uptime'
            }
        }

        stage('Check Build Tools') {
            steps {
                echo 'Verifying toolchains...'
                sh 'java -version && mvn -version'
            }
        }
    }
}
