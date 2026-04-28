# Mastering Pivotal Cloud Foundry (PCF) & cf4k8s

A comprehensive hands-on learning guide for mastering Pivotal Cloud Foundry locally using cf4k8s. This project is designed for cloud developers already familiar with cloud concepts (AWS, Azure, GCP) who want to understand PCF's unique approach and get practical experience.

[https://nitinkc.github.io/PCF-Learning/](https://nitinkc.github.io/PCF-Learning/)
## Overview

This MkDocs project provides:
- **Structured Learning Path**: From foundations to advanced deployment patterns
- **Hands-On Labs**: 6 progressive labs to practice key concepts
- **Local Setup Guide**: Get cf4k8s running on your machine
- **Comparative Analysis**: How PCF differs from AWS, Azure, and GCP
- **Reference Materials**: CLI commands, manifests, troubleshooting

## Project Structure

```
docs/
├── 00-introduction/          # Overview and learning path
├── 01-foundations/           # Cloud computing and PaaS basics
├── 02-core-concepts/         # PCF architecture and components
├── 03-setup-local/           # cf4k8s installation and configuration
├── 04-deployment-patterns/   # Real-world deployment strategies
├── 05-advanced-topics/       # Advanced PCF concepts
├── 06-labs/                  # Hands-on practice labs
├── 07-pcf-vs-cloud-providers/# Comparisons with AWS, Azure, GCP
├── 08-reference/             # Command reference and troubleshooting
└── index.md                  # Home page
```

## Getting Started

### Prerequisites
- Docker and kubectl installed
- Basic cloud computing knowledge
- Advanced cloud developer experience (assumed)
- Python 3.8+ installed

### Quick Start

#### Option 1: Using Virtual Environment (Recommended)
```bash
# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate  # macOS/Linux
# or
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt

# Serve locally
mkdocs serve
```

#### Option 2: Direct Installation
```bash
# Install dependencies
pip install -r requirements.txt

# Serve locally
mkdocs serve
```

Then open browser: `http://localhost:8000`

## Learning Path

1. **Start Here**: Introduction → Why PCF → Learning Path
2. **Build Foundation**: Cloud Basics → Container Orchestration → PaaS Model
3. **Learn Core**: PCF Architecture → Apps/Buildpacks → Services → Routing → Scaling
4. **Setup Local**: Prerequisites → cf4k8s Installation → CLI Configuration
5. **Hands-On Labs**: Deploy → Scale → Bind Services → Blue-Green → Monitoring
6. **Advanced Topics**: Resource Management → Security → Custom Buildpacks
7. **Compare**: PCF vs AWS/Azure/GCP with comparison matrix

## Labs Overview

- **Lab 1**: Deploy your first application to cf4k8s
- **Lab 2**: Practice scaling and high availability
- **Lab 3**: Bind and use services
- **Lab 4**: Implement blue-green deployment strategy
- **Lab 5**: Create a custom buildpack
- **Lab 6**: Set up monitoring and logging

## Key Learning Objectives

By completing this guide, you will:
- Understand PCF's unique architecture and components
- Know how PCF differs strategically from AWS/Azure/GCP
- Deploy and manage applications on cf4k8s locally
- Implement deployment patterns (blue-green, canary, etc.)
- Configure services and service brokers
- Monitor and troubleshoot PCF applications
- Master PCF CLI and deployment manifests

## For Advanced Cloud Developers

If you're coming from AWS/Azure/GCP, focus on:
- **Section 7**: PCF vs Cloud Providers (understand strategic differences)
- **Section 2**: Core Concepts (learn PCF's abstractions)
- **Section 4**: Deployment Patterns (see how PCF approaches DevOps)
- **Labs 1-4**: Practical experience with cf4k8s

## Contributing

Suggestions and improvements welcome! Feel free to expand labs, add examples, or clarify concepts.

## Resources

- [Cloud Foundry Official Docs](https://docs.cloudfoundry.org/)
- [cf4k8s Repository](https://github.com/cloudfoundry/cf-for-k8s)
- [CF CLI Documentation](https://cli.cloudfoundry.org/)
- [Buildpacks](https://buildpacks.io/)

## License

This learning material is provided as-is for educational purposes.
