# 🧵 Culture Threads: Enterprise Microservices Platform

![Project Status](https://img.shields.io/badge/status-in%20development-yellow)
![Microservices](https://img.shields.io/badge/microservices-12-blue)
![Languages](https://img.shields.io/badge/languages-5-green)
![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)

> **A production-grade e-commerce microservices demonstration platform, rebuilt from the Google Cloud Microservices Demo with enterprise-level Site Reliability Engineering (SRE) practices.**

## 📖 Project Overview

Culture Threads Platform is an advanced cloud-native microservices architecture project designed to demonstrate real-world enterprise practices in distributed systems. This platform showcases:

- **Polyglot Microservices**: 12 services across 5 programming languages
- **Modern Communication**: gRPC-based inter-service communication with Protocol Buffers
- **Cloud-Native Architecture**: Kubernetes orchestration with multiple deployment options
- **Production Practices**: Observability, resilience patterns, and security best practices

### 🎯 Project Purpose

This project serves as a comprehensive learning platform and portfolio piece, demonstrating:
- Manual deconstruction and analysis of complex microservices systems
- Implementation of enterprise-grade SRE practices
- Infrastructure as Code (IaC) with Terraform and Helm
- CI/CD pipeline development and deployment automation
- Comprehensive technical documentation and architecture analysis

**Original Source**: Based on [Google Cloud Platform's Microservices Demo](https://github.com/GoogleCloudPlatform/microservices-demo)

---

## 🏗️ Architecture

### Microservices Overview

**Total Microservices:** 12  
**Communication Protocol:** gRPC (inter-service) + HTTP (client-facing)  
**Deployment Platform:** Kubernetes (GKE, EKS, AKS compatible)

[![Architecture Diagram](/docs/img/architecture-diagram.png)](/docs/img/architecture-diagram.png)

### Service Breakdown

| Service | Language | Port | Purpose |
|---------|----------|------|---------|
| **Frontend** | Go | 8080 | Web UI and API Gateway |
| **Cart Service** | C# | 7070 | Shopping cart management (Redis) |
| **Product Catalog** | Go | 3550 | Product listing and search |
| **Currency Service** | Node.js | 7000 | Currency conversion (ECB API) |
| **Payment Service** | Node.js | 50051 | Payment processing (mock) |
| **Shipping Service** | Go | 50051 | Shipping cost calculation |
| **Email Service** | Python | 8080 | Order confirmation emails |
| **Checkout Service** | Go | 5050 | Order orchestration |
| **Recommendation** | Python | 8080 | Product recommendations |
| **Ad Service** | Java | 9555 | Contextual advertising |
| **Shopping Assistant** | Python | 8080 | AI-powered shopping help (Gemini) |
| **Load Generator** | Python/Locust | - | Traffic simulation |

### Language Distribution

```
Go        ████████████ 33% (4 services)
Python    ████████ 25% (3 services)
Node.js   █████ 17% (2 services)
C#        ██ 8% (1 service)
Java      ██ 8% (1 service)
```

### Key Design Patterns

- ✅ API Gateway (Frontend)
- ✅ Database per Service (Redis for Cart)
- ✅ Service Discovery (Kubernetes DNS)
- ✅ Circuit Breaker & Retry
- ✅ Sidecar Pattern (OpenTelemetry)
- ✅ Strangler Fig (Shopping Assistant)
- ✅ Backend for Frontend (BFF)
- ✅ Orchestration (Checkout Service)

📚 **Detailed Analysis**: See [Microservices Architecture Analysis](./docs/microservices_analysis.md)

---

## 📅 Day 1 Findings

### ✅ Completed Tasks

1. **Initial Repository Setup**
   - Forked and cloned original Google Cloud demo
   - Established project structure and organization
   - Created backup of original documentation

2. **Microservices Analysis**
   - Identified and documented all 12 microservices
   - Analyzed language choices and service responsibilities
   - Mapped inter-service communication patterns
   - Created comprehensive architecture documentation

3. **Service Communication Mapping**
   - Documented gRPC protocol buffer definitions
   - Analyzed Frontend ↔ AdService communication flow
   - Created detailed sequence diagrams
   - Identified dependency chains

4. **Documentation Creation**
   - [Microservices Analysis](./docs/microservices_analysis.md) - Complete service breakdown
   - [Frontend-AdService Communication](./docs/frontend_adservice_communication.md) - gRPC deep dive
   - Repository README with project overview

### 🔍 Key Insights

- **Polyglot Architecture**: 5 languages chosen for specific strengths
  - Go: Performance-critical services (Frontend, Checkout)
  - Python: AI/ML features (Recommendation, Shopping Assistant)
  - Node.js: I/O-heavy services (Currency, Payment)
  - C#: Redis integration (Cart)
  - Java: Enterprise patterns (Ad Service)

- **Communication Patterns**: gRPC chosen for efficiency
  - Protocol Buffers for type-safe contracts
  - HTTP/2 for performance
  - Service mesh ready (Istio manifests included)

- **Design Trade-offs Identified**:
  - ⚠️ Checkout service orchestrates 6+ services (potential SPOF)
  - ⚠️ Redis single point of failure for cart data
  - ⚠️ No distributed transaction management
  - ⚠️ Mock services not production-ready

### 📊 Architecture Statistics

```
Total Lines of Code: ~15,000+
Configuration Files: 50+
Docker Images: 12
Kubernetes Manifests: 40+
Protocol Buffer Definitions: 1 (multi-service)
Deployment Options: 5 (K8s, Helm, Kustomize, Terraform, Istio)
```

---

## 🚀 Upcoming Milestones

### Phase 1: Foundation (Weeks 1-2)
- [x] ✅ Repository setup and documentation
- [x] ✅ Microservices analysis and mapping
- [ ] 🔄 Local development environment setup
- [ ] 🔄 Docker containerization verification
- [ ] 🔄 Basic Kubernetes deployment testing

### Phase 2: Infrastructure (Weeks 3-4)
- [ ] 📋 Terraform infrastructure as code
  - VPC and networking
  - GKE cluster provisioning
  - Cloud SQL / Redis setup
  - IAM and security policies
- [ ] 📋 Helm chart optimization
- [ ] 📋 Kustomize overlays for environments

### Phase 3: Observability (Weeks 5-6)
- [ ] 📋 OpenTelemetry full implementation
- [ ] 📋 Distributed tracing (Jaeger/Cloud Trace)
- [ ] 📋 Centralized logging (ELK/Loki)
- [ ] 📋 Metrics and dashboards (Prometheus/Grafana)
- [ ] 📋 Service health monitoring

### Phase 4: Resilience (Weeks 7-8)
- [ ] 📋 Circuit breakers implementation
- [ ] 📋 Retry policies and timeouts
- [ ] 📋 Rate limiting
- [ ] 📋 Chaos engineering tests
- [ ] 📋 Disaster recovery procedures

### Phase 5: CI/CD (Weeks 9-10)
- [ ] 📋 GitHub Actions workflows
- [ ] 📋 Automated testing pipeline
- [ ] 📋 Container image building and scanning
- [ ] 📋 GitOps deployment (ArgoCD/Flux)
- [ ] 📋 Canary and blue-green deployments

### Phase 6: Security (Weeks 11-12)
- [ ] 📋 Service mesh implementation (Istio)
- [ ] 📋 mTLS between services
- [ ] 📋 OAuth 2.0 / OIDC authentication
- [ ] 📋 Network policies
- [ ] 📋 Secrets management (Vault)

### Phase 7: Production Readiness (Weeks 13-14)
- [ ] 📋 Replace mock services with real integrations
- [ ] 📋 Load testing and performance optimization
- [ ] 📋 Cost optimization analysis
- [ ] 📋 Documentation finalization
- [ ] 📋 Production deployment

---

## 💻 Technical Approach

### Development Philosophy

This project follows a **manual deconstruction and rebuild** approach:

1. **Analyze** - Deep dive into existing architecture
2. **Document** - Comprehensive technical documentation
3. **Understand** - Identify patterns, trade-offs, and decisions
4. **Enhance** - Implement enterprise-grade improvements
5. **Deploy** - Production-ready deployment pipelines

### Enterprise-Grade SRE Practices

- **Observability First**: Metrics, logs, and traces from day one
- **Security by Design**: mTLS, RBAC, and secrets management
- **Resilience Patterns**: Circuit breakers, retries, and graceful degradation
- **Infrastructure as Code**: Terraform for reproducible infrastructure
- **GitOps**: Declarative configuration and automated deployments
- **Chaos Engineering**: Proactive failure testing

### Documentation Strategy

- 📖 Architecture decision records (ADRs)
- 📊 Service communication diagrams
- 🔧 Operational runbooks
- 📝 Developer onboarding guides
- 🎯 SRE playbooks

---

## 🛠️ Technology Stack

### Core Technologies

| Category | Technologies |
|----------|-------------|
| **Languages** | Go, Python, Node.js, C#, Java |
| **Frameworks** | gRPC, ASP.NET Core, Express.js |
| **Communication** | Protocol Buffers, HTTP/2, REST |
| **Containerization** | Docker, Skaffold |
| **Orchestration** | Kubernetes (GKE, EKS, AKS) |
| **Service Mesh** | Istio (optional) |
| **IaC** | Terraform, Helm, Kustomize |
| **Observability** | OpenTelemetry, Prometheus, Grafana, Jaeger |
| **CI/CD** | GitHub Actions, ArgoCD |
| **Data Stores** | Redis, Cloud SQL, JSON |

### Cloud Platforms

- **Primary**: Google Cloud Platform (GKE)
- **Supported**: AWS (EKS), Azure (AKS), On-premises

---

## 🚦 Getting Started

### Prerequisites

```bash
# Required tools
- Docker Desktop or Docker Engine
- kubectl (Kubernetes CLI)
- Skaffold (for local development)
- Terraform (for infrastructure)
- Helm (for package management)
- gcloud CLI (for GCP deployment)
```

### Quick Start (Local Development)

```bash
# 1. Clone the repository
git clone https://github.com/developedbydmac/culture-threads-platform.git
cd culture-threads-platform

# 2. Install dependencies
# (Instructions vary by service - see individual service READMEs)

# 3. Build and run with Skaffold
skaffold dev

# 4. Access the application
open http://localhost:8080
```

### Deploy to Kubernetes

```bash
# Deploy using kubectl
kubectl apply -f ./kubernetes-manifests/

# Or deploy using Helm
helm install culture-threads ./helm-chart/

# Or deploy using Kustomize
kubectl apply -k ./kustomize/
```

### Infrastructure Provisioning (Terraform)

```bash
cd terraform/

# Initialize Terraform
terraform init

# Review the plan
terraform plan

# Apply infrastructure
terraform apply
```

---

## 📚 Documentation

### Project Documentation

- [Microservices Architecture Analysis](./docs/microservices_analysis.md)
- [Frontend-AdService Communication](./docs/frontend_adservice_communication.md)
- [Development Guide](./docs/development-guide.md)
- [Product Requirements](./docs/product-requirements.md)
- [Adding New Microservices](./docs/adding-new-microservice.md)

### Service-Specific READMEs

Each microservice has its own README with setup instructions:
- [Frontend Service](./src/frontend/README.md)
- [Cart Service](./src/cartservice/)
- [Product Catalog Service](./src/productcatalogservice/README.md)
- [And more...](./src/)

---

## �� Testing

### Test Strategy

```bash
# Unit tests (per service)
cd src/<service>
# Run language-specific tests

# Integration tests
# Coming soon...

# Load testing
cd src/loadgenerator
locust -f locustfile.py --host=http://localhost:8080

# Chaos engineering
# Coming soon with Chaos Mesh
```

---

## 🔧 Potential Challenges and Solutions

### Identified Challenges

| Challenge | Impact | Mitigation Strategy |
|-----------|--------|---------------------|
| **Multi-language complexity** | Higher maintenance overhead | Team specialization, shared patterns |
| **Service orchestration** | Checkout SPOF | Implement saga pattern, add compensations |
| **Data consistency** | Partial failures | Event sourcing, idempotency |
| **Redis SPOF** | Cart data loss | Redis Sentinel/Cluster, persistence |
| **Network latency** | Poor UX | Parallel calls, caching, timeouts |
| **Observability gaps** | Long MTTR | Full OpenTelemetry, centralized logging |
| **No authentication** | Security risk | OAuth 2.0, mTLS, RBAC |
| **Mock services** | Not production-ready | Real API integrations |

### Solutions in Progress

- 🔄 Implementing distributed tracing
- 🔄 Adding circuit breakers
- 🔄 Redis high availability setup
- 🔄 Service mesh for mTLS
- 🔄 OAuth 2.0 authentication

---

## 📖 Learning Objectives Achieved

### ✅ Week 1 Accomplishments

- [x] Understand microservices architecture fundamentals
- [x] Practice GitHub repository management
- [x] Document technical journey comprehensively
- [x] Analyze inter-service communication patterns
- [x] Identify architectural patterns and anti-patterns
- [x] Create professional technical documentation

### 🎓 Technical Skills Developed

- **Architecture**: Microservices, polyglot systems, service mesh
- **Communication**: gRPC, Protocol Buffers, REST APIs
- **Documentation**: Technical writing, diagrams, ADRs
- **Analysis**: Code review, dependency mapping, pattern recognition
- **Tools**: Git, Markdown, Mermaid diagrams

---

## 🔮 Next Steps Preparation

### Immediate Focus (Week 2)

1. **Research Topics**
   - [ ] gRPC and Protocol Buffers deep dive
   - [ ] Kubernetes architecture and networking
   - [ ] Terraform best practices
   - [ ] OpenTelemetry implementation

2. **Hands-On Practice**
   - [ ] Set up local Kubernetes cluster (minikube/kind)
   - [ ] Deploy services locally
   - [ ] Test gRPC calls manually
   - [ ] Explore Kubernetes manifests

3. **Documentation**
   - [ ] Create Kubernetes deployment guide
   - [ ] Document local development workflow
   - [ ] Write Terraform module documentation

### Medium-Term Goals (Weeks 3-6)

- Terraform infrastructure implementation
- CI/CD pipeline setup
- Observability stack deployment
- Service resilience patterns

### Long-Term Vision (Months 2-3)

- Production-ready deployment
- Complete SRE documentation
- Performance optimization
- Portfolio presentation materials

---

## 🤝 Contributing

This is a personal learning project, but suggestions and feedback are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add some improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

---

## 📝 Project Timeline

```
Week 1-2:   Setup & Analysis          [████████████████████] 100% ✅
Week 3-4:   Infrastructure (IaC)       [░░░░░░░░░░░░░░░░░░░░]   0%
Week 5-6:   Observability              [░░░░░░░░░░░░░░░░░░░░]   0%
Week 7-8:   Resilience Patterns        [░░░░░░░░░░░░░░░░░░░░]   0%
Week 9-10:  CI/CD Pipelines            [░░░░░░░░░░░░░░░░░░░░]   0%
Week 11-12: Security & Service Mesh    [░░░░░░░░░░░░░░░░░░░░]   0%
Week 13-14: Production Deployment      [░░░░░░░░░░░░░░░░░░░░]   0%
```

---

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

**Original Project**: Copyright © Google LLC  
**Culture Threads Platform**: Copyright © 2026 Daquan McDaniel

---

## 🙏 Acknowledgments

- **Google Cloud Platform** - Original microservices demo application
- **CNCF** - Kubernetes, gRPC, OpenTelemetry, and other cloud-native projects
- **Open Source Community** - For the amazing tools and frameworks

---

## 📧 Contact

**Project Maintainer**: Daquan McDaniel  
**GitHub**: [@developedbydmac](https://github.com/developedbydmac)  
**Repository**: [culture-threads-platform](https://github.com/developedbydmac/culture-threads-platform)

---

<p align="center">
  <strong>⭐ If you find this project helpful, please consider giving it a star! ⭐</strong>
</p>

<p align="center">
  Built with ❤️ and ☕ | Learning through building | 2026
</p>
