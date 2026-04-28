<!-- Use this file to provide workspace-specific custom instructions to Copilot. -->

# PCF Learning Project - Workspace Instructions

## Project Purpose
This is a comprehensive MkDocs documentation project for learning Pivotal Cloud Foundry (cf4k8s) with hands-on labs. The target audience is advanced cloud developers familiar with AWS/Azure/GCP who want to understand how PCF differs and gain practical experience with cf4k8s running locally.

## Documentation Structure
- **docs/**: All markdown documentation files organized by learning progression
- **mkdocs.yml**: MkDocs configuration with navigation structure
- **README.md**: Project overview and quick start guide

## Content Organization
The project uses 9 main sections (00-08), each with progressive complexity:
1. Introduction (why PCF, learning path)
2. Foundations (cloud basics for context)
3. Core Concepts (PCF architecture)
4. Local Setup (cf4k8s installation)
5. Deployment Patterns (real-world strategies)
6. Advanced Topics (security, monitoring, custom buildpacks)
7. Hands-On Labs (6 progressive labs)
8. PCF vs Cloud Providers (AWS/Azure/GCP comparisons)
9. Reference (CLI, manifests, troubleshooting)

## When Modifying Content
- Follow MkDocs Markdown conventions
- Keep labs hands-on and practical
- Emphasize PCF's unique approach compared to traditional cloud providers
- Use clear code examples and step-by-step instructions
- Add diagrams/flowcharts where helpful for architecture concepts

## Target Audience Notes
- Users have advanced cloud developer experience
- They understand container orchestration, PaaS concepts
- Focus on PCF differences, not basic cloud education
- Labs should assume production-grade expectations

## Build & Serve
```bash
pip install mkdocs mkdocs-material
mkdocs serve          # Run locally at http://localhost:8000
mkdocs build          # Build static site in /site directory
```
