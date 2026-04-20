---
title: "From Nothing to EKS: How I Built a DevSecOps Pipeline for 11 Microservices"
datePublished: 2026-04-20T19:26:40.168Z
cuid: cmo7l5z4y00362bpw6w7bhebx
slug: devsecops-pipeline-for-11-microservice-documentation
cover: https://cdn.hashnode.com/uploads/covers/69dad9e0aadf1107e26d5b69/ac89a88d-1120-4bb1-a464-d8e994346ee8.png

---

## Introduction

At some point, tutorials stop helping.  
I had been going through DevOps and DevSecOps content for a while, but everything felt too controlled. Everything worked. No real friction. No real confusion.  
So I decided to build something closer to what an actual system looks like.

Not one service. Not a simple pipeline.  
An **e-commerce microservices application with 11 services**, running through a full CI/CD pipeline — from code to deployment on Kubernetes.  
  
I didn’t fully know what I was getting into.  
But I started anyway.

## What I Was Trying to Build

The goal was simple (at least on paper):

*   Code hosted in a repo
    
*   Continious Integration handled by Jenkins
    
*   Code quality checks using SonarQube
    
*   Containerization with Docker
    
*   Deployment to Kubernetes (EKS on Amazon Web Services)
    

Basically:

> Code → Build → Scan → Package → Deploy

But once you start wiring these things together across multiple services… it stops being simple very quickly.

## Setting Up the Foundation (EC2 + Tools)

I started by creating an EC2 instance.

*   Instance type: `c5.xlarge`
    
*   Storage: 50GB
    

That wasn’t random — I knew I’d be running multiple heavy tools on the same machine (Jenkins, SonarQube, Docker), so I didn’t want resource issues later.

Before launching, I configured a security group and opened the necessary ports:

*   22 (SSH)
    
*   80 / 443 (web access)
    
*   8080 (Jenkins)
    
*   9000 (SonarQube)
    

After launching the instance, I SSH’d in, updated the system, and started installing the core tools:

*   Git
    
*   Java
    
*   Jenkins
    
*   Docker
    
*   Python
    
*   Trivy
    
*   SonarQube (later via Docker)
    

This part feels straightforward… until everything starts depending on everything else.

## Introduction

Installing Jenkins was the easy part.

Getting it ready for real use took more work.

After installation:

*   Accessed it via `http://<public-ip>:8080`
    
*   Retrieved the initial admin password from the server
    
*   Installed suggested plugins
    
*   Created an admin user
    

Then I added additional plugins for what I needed:

*   SonarQube integration
    
*   NodeJS
    
*   Dependency Check
    
*   Pipeline tools
    
*   Language support (Go, .NET, Python)
    

At this point, Jenkins was running… but not yet useful.

## Running SonarQube with Docker

Instead of installing SonarQube directly, I ran it in a Docker container.

The important part here was persistence.

I used Docker volumes to make sure:

*   data
    
*   logs
    
*   extensions
    

would survive container restarts.

Once it was up, I accessed it via:

```plaintext
http://<public-ip>:9000
```

Logged in with default credentials, changed the password, and generated a token.

That token later became important for Jenkins integration.

## Credentials & Integrations (Where Things Get Messy)

This is where things started getting… chaotic.

Different tools needed different credentials:

*   Gmail (for notifications)
    
*   SonarQube token
    
*   Docker Hub access token
    

All of these had to be stored properly inside Jenkins.

Inside Jenkins credentials:

*   `mail-cred`
    
*   `sonar-token`
    
*   `docker-cred`
    

Then I wired everything together:

*   Jenkins ↔ SonarQube
    
*   Jenkins ↔ Email (SMTP setup)
    

Tested email configuration — and it worked.

That was one of the few moments where something worked on the first try.

## Integrations

This was easily one of the hardest parts.

Writing one pipeline is fine.

Writing **11 pipelines**, each with:

*   different paths
    
*   different services
    
*   different configs
    

…is where mistakes start multiplying.

Each service had its own Jenkinsfile, stored in different directories.

In Jenkins:

*   Created a pipeline job
    
*   Linked it to the Git repo
    
*   Set script path (e.g. `jenkins/frontend`)
    
*   Triggered build
    

And then came the errors.

## First Real Issue: Docker Build Failure

The pipeline failed during the Docker build stage.

At first, nothing made sense.

Then I checked the logs properly.

The issue wasn’t the Dockerfile.  
It wasn’t the pipeline script.

It was permissions.

Jenkins didn’t have permission to run Docker.

Fix:

```plaintext
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

After that, the build worked.

That moment changes how you see debugging.

## Containerization & Image Verification

Once pipelines started passing:

*   Docker images were built
    
*   Images were pushed to Docker Hub
    

I verified this directly in my repository.

Seeing all services there confirmed that the CI part was actually working.

## Moving to Kubernetes (EKS Setup)

Next step was Kubernetes.

I installed:

*   AWS CLI
    
*   kubectl
    
*   eksctl
    
*   Helm
    

Configured AWS:

```plaintext
aws configure
```

Then created the cluster:

```plaintext
eksctl create cluster ...
```

Then node group.

This part required patience.

A lot of waiting.  
  
A lot of checking.

## Connecting to the Cluster

After cluster creation:

```plaintext
aws eks update-kubeconfig ...
```

Now `kubectl` could talk to the cluster.

That’s when things started feeling real.

## Deploying the Application

I applied my deployment YAML:

```plaintext
kubectl apply -f deployment.yml
```

Then watched pods:

```plaintext
watch kubectl get pods
```

Pods started coming up.

Then services:

```plaintext
kubectl get svc
```

I noticed:

*   frontend → LoadBalancer
    
*   others → ClusterIP
    

Which made sense.

## The Moment Everything Worked

I copied the LoadBalancer IP.

Opened it in my browser.

And the application loaded.

After everything — the setup, the errors, the confusion — it finally worked.

That moment was worth it.

## Challenges I Faced

This project wasn’t smooth. At all.

Some of the main issues:

*   Docker permission errors in Jenkins
    
*   Pipeline failures with unclear logs
    
*   Managing multiple Jenkinsfiles
    
*   Credential confusion
    
*   Kubernetes service exposure issues
    
*   Waiting for infrastructure without knowing if something failed
    

And sometimes:

> Just staring at an error and not even recognizing what was wrong.

## What I Learned

A few things became very clear:

*   DevOps is mostly debugging
    
*   Logs matter more than assumptions
    
*   Small misconfigurations cause big failures
    
*   Repetition creates opportunities for automation
    
*   Patience is part of the process
    

## What’s Next

This isn’t finished yet.

Next steps:

*   Ingress setup
    
*   Monitoring (Prometheus & Grafana)
    
*   GitOps
    

And possibly building tools to reduce the manual work I experienced.

## Final Thoughts

This project changed how I see DevOps.

It’s not about knowing tools.

It’s about:

*   connecting systems
    
*   understanding failures
    
*   and fixing things when they break
    

Because they will break.

Also, this project isn’t fully complete yet.

There are still a few things left to implement (Ingress, monitoring with Prometheus and Grafana, and GitOps). I didn’t want to rush and write another blog post just for those parts without going deeper into them — that would feel a bit incomplete.

Instead, I’ll be doing a full video where I implement everything step by step and explain it properly.

That way, you can actually see how it all comes together in a real setup

Follow my youtube channel and stay updated: https://youtube.com/@jamesadeniyi-j5f?si=b2XPs8Eya\_DQD5VY