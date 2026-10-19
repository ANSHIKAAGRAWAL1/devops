DevOps-1BM23IS032

# DevOps Lab Exercises

A collection of hands-on DevOps lab exercises covering containerization, Docker networking, Kubernetes deployments and scaling, CI/CD automation, monitoring, alerting, and container security.

This repository contains practical implementations, configuration files, documentation, and screenshots demonstrating the concepts explored in each exercise.

## Repository Overview

The exercises are organized into separate folders so that each implementation, its supporting files, and its screenshots can be maintained independently.

| Exercise | Topic | Key Technologies |
|---|---|---|
| Exercise 1 | Kubernetes Getting Started | Kubernetes, Minikube, kubectl |
| Exercise 2 | Deploying a Flask Application on Minikube | Python, Flask, Docker, Kubernetes |
| Exercise 3 | Scaling a Flask Application with ReplicaSets | Python, Flask, Docker, Kubernetes |
| Exercise 4 | Docker Networking with Multiple Containers | Docker, Flask, MySQL, Redis |
| Exercise 5 | Docker Security with AppArmor | Docker, AppArmor, Python |
| Exercise 6 | Real-Time Operations Monitoring and Alerting | Python, Prometheus, Grafana, Docker, Jenkins |
| Exercise 7 | Jenkins CI Automation | Jenkins, Docker |
| Exercise 8 | Jenkins Hello World Job | Jenkins, Git, Shell scripting |
| Exercise 9 | Jenkins Multi-Stage Pipeline | Jenkins, Python, Flask |
| Exercise 10 | Minikube Multi-Node, Multi-App Deployment | Docker, Kubernetes, Minikube, Flask |

*Note: The table summarizes the intended topics of the exercises. Refer to each exercise folder for its actual implementation and current completion status.*

## Technologies Used

- **Containerization:** Docker, Dockerfiles, Docker networking
- **Orchestration:** Kubernetes, Minikube, kubectl, Pods, Services, ReplicaSets, Deployments
- **Programming:** Python, Flask, Shell scripting
- **CI/CD:** Jenkins, Freestyle jobs, Jenkins pipelines
- **Monitoring and alerting:** Prometheus, Grafana
- **Container security:** AppArmor
- **Databases and caching:** MySQL, Redis
- **Version control:** Git and GitHub

## Exercise Descriptions

### Exercise 1: Kubernetes Getting Started

Introduces fundamental Kubernetes concepts by deploying a basic application and exposing it through a Kubernetes Service.

**Concepts:** Pods, Services, kubectl, and Minikube.

### Exercise 2: Flask Application on Minikube

Deploys a Flask web application to a local Kubernetes cluster using Docker and Kubernetes YAML manifests.

**Concepts:** Container images, Deployments, Services, ports, and application access.

### Exercise 3: Scaling Flask with ReplicaSets

Demonstrates how Kubernetes maintains a desired number of application replicas and replaces failed or deleted Pods.

**Concepts:** ReplicaSets, horizontal scaling, health probes, self-healing, and Pod inspection.

### Exercise 4: Docker Networking

Creates a user-defined Docker bridge network connecting a Flask application, MySQL, and Redis containers.

**Concepts:** Bridge networking, container-name resolution, port publishing, network inspection, and container connectivity.

### Exercise 5: Docker Security with AppArmor

Explores container security by applying an AppArmor profile and examining restricted operations.

**Concepts:** Linux security profiles, container isolation, and security testing.

### Exercise 6: Real-Time Operations Monitoring and Alerting

Explores operational monitoring and alerting using Python and monitoring tools.

**Concepts:** Metrics, monitoring dashboards, alerting, containers, and automation.

### Exercise 7: Jenkins CI Automation

Introduces Jenkins and its role in automating continuous integration workflows.

**Concepts:** Jenkins setup, build automation, and CI fundamentals.

### Exercise 8: Jenkins Hello World Job

Creates and runs a basic Jenkins job that executes a shell script.

**Concepts:** Freestyle projects, Git integration, build steps, and console output.

### Exercise 9: Jenkins Multi-Stage Pipeline

Implements a Jenkins pipeline for a Flask application, organizing the workflow into stages such as building, testing, and running the application.

**Concepts:** Jenkinsfile, pipeline stages, automated testing, and CI/CD workflows.

### Exercise 10: Minikube Multi-Node, Multi-App Deployment

Deploys multiple Flask services in a Minikube environment using Kubernetes manifests.

**Concepts:** Multi-node clusters, multiple applications, Deployments, Services, replicas, and scheduling.

## Prerequisites

Install the tools required by the specific exercise you want to run.

- Git
- Docker Desktop or Docker Engine
- Python 3 and pip
- kubectl
- Minikube
- Jenkins, where required
- Prometheus and Grafana, where required
- A Linux environment with AppArmor support for the security exercise

Some exercises require only a subset of these tools. Check the README or instructions inside the relevant exercise folder before running it.

## Getting Started

### 1. Clone the repository

```bash
git clone <your-github-repository-url>
cd <your-repository-folder>
```

Replace the placeholders with your actual GitHub repository URL and local repository directory.

### 2. Open an exercise

Navigate to the required exercise directory. For example:

```bash
cd "Ex4-Docker-Networking"
```

### 3. Follow the exercise instructions

Read the exercise-specific README and configuration files before running commands. Start the required services, build images, apply manifests, or configure tools according to that exercise's instructions.

**Important:** Commands that create networks, start containers, modify Kubernetes resources, or stop services can affect your local environment. Check existing resources before running cleanup commands.



## Version Control

This repository uses Git for version control and GitHub for hosting the project.

Typical commands for committing changes are:

```bash
git status
git add <specific-files-or-exercise-folder>
git diff --cached
git commit -m "Describe the changes"
git push origin <your-branch-name>
```

Replace placeholders with the appropriate paths and branch name. Review staged changes before committing to avoid including unrelated files.

## Learning Outcomes

Through these exercises, the repository aims to demonstrate practical understanding of:

- Building and running containerized applications
- Connecting containers through Docker networks
- Deploying and scaling applications using Kubernetes
- Understanding replica management and self-healing
- Automating builds and tests with Jenkins
- Monitoring applications and configuring alerts
- Applying container security controls
- Using Git and GitHub to maintain and share project work

## Notes

- Each exercise should be run independently unless its instructions specify otherwise.
- Tool versions and environment requirements may differ between exercises.
- Review the configuration files before using commands that create, delete, or modify resources.
- The presence of an exercise folder does not by itself indicate that the exercise has been fully tested or completed.

