Project Purpose
This repo contains original projects applying Docker, Kubernetes, and platform engineering concepts. No tutorial or course-exercise code — only work I've designed and built myself.
Current Focus
Building a Python API that evolves incrementally:

Containerized with Docker
Persistent storage via Docker volumes
Multi-container setup with Docker Compose (API + Postgres + Redis)
Deployed to Kubernetes (Deployments, Services, health checks, persistent storage)
CI/CD eventually added

Working Style

Explain concepts before writing code
Give hints instead of full solutions when possible
Explain why something is best practice, not just what to do
Point out common mistakes

Code Quality Standards

Proper project structure (src/, docker/, k8s/, tests/)
Each feature/project gets a clear README: what it does, why it's built that way
Commit history should show real iteration, not one large final commit
