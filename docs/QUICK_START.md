# 🚀 Quick Start Guide - Week 2 Setup
## Get Culture Threads Running Locally in 30 Minutes

**Last Updated**: February 1, 2026  
**Current Phase**: Week 2 - Local Development Setup

---

## ✅ Prerequisites Checklist

Before you begin, install these tools:

### 1. Docker Desktop ⭐ (REQUIRED)
```bash
# Download and install from:
https://www.docker.com/products/docker-desktop

# After installation, verify:
docker --version
# Should show: Docker version 24.x or higher

# ⚠️ IMPORTANT: Configure Docker Desktop
# Go to: Docker Desktop → Settings → Resources
# - Memory: 8GB minimum (12GB recommended)
# - CPUs: 4 minimum (6 recommended)
# - Swap: 1GB minimum

# Enable Kubernetes:
# Docker Desktop → Settings → Kubernetes
# ✅ Enable Kubernetes
# Click "Apply & Restart"
```

### 2. kubectl (Kubernetes CLI)
```bash
# macOS installation
brew install kubectl

# Verify installation
kubectl version --client
# Should show: Client Version: v1.28.x or higher

# Test Kubernetes connection
kubectl cluster-info
# Should show: Kubernetes control plane is running at...
```

### 3. Skaffold (Development Tool)
```bash
# macOS installation
brew install skaffold

# Verify installation
skaffold version
# Should show: v2.x or higher
```

### 4. Optional but Recommended
```bash
# grpcurl - for testing gRPC endpoints
brew install grpcurl

# k9s - Kubernetes UI in terminal (amazing!)
brew install k9s
```

---

## 🎯 Quick Start (3 Simple Steps)

### Step 1: Verify Everything Works
```bash
# Test Docker
docker ps
# Should show: CONTAINER ID   IMAGE   COMMAND   ...

# Test Kubernetes
kubectl get nodes
# Should show: NAME                 STATUS   ...

# Test Skaffold
skaffold version
# Should show version number

# ✅ If all three work, you're ready!
```

### Step 2: Deploy All Services
```bash
# Navigate to project directory
cd /Users/daquanmcdaniel/Documents/2026/culture-threads-platform/culture-threads-platform

# Start all services (this will take 10-20 minutes first time)
skaffold dev

# What happens:
# 1. Builds Docker images for all 12 services
# 2. Pushes to local Docker registry
# 3. Deploys to Kubernetes
# 4. Shows live logs from all services
# 5. Auto-rebuilds on code changes

# ⚠️ Keep this terminal open - it shows all logs!
```

### Step 3: Access the Application
```bash
# Open a NEW terminal (keep skaffold running in the other one)

# Check if all pods are running
kubectl get pods

# All pods should show "Running" - if not, wait a minute
# NAME                          READY   STATUS    RESTARTS   AGE
# adservice-xxx                 1/1     Running   0          5m
# cartservice-xxx               1/1     Running   0          5m
# ... (all 12 services)

# Access the frontend
kubectl port-forward svc/frontend 8080:80

# Open in browser
open http://localhost:8080

# 🎉 You should see the Culture Threads e-commerce site!
```

---

## 🎊 Success! Now What?

### Explore the Running System

#### Browse the Store
- Add items to cart
- Change currency
- Go through checkout
- See the AI shopping assistant

#### Check Service Logs
```bash
# In a new terminal, watch logs from different services

# Frontend logs
kubectl logs -f deployment/frontend

# Cart service logs
kubectl logs -f deployment/cartservice

# Watch all logs at once (optional with k9s)
k9s
# Press '0' to see all namespaces, use arrow keys, 'l' for logs
```

#### Test gRPC Endpoints
```bash
# Port forward to product catalog service
kubectl port-forward svc/productcatalogservice 3550:3550

# In another terminal, test the gRPC endpoint
grpcurl -plaintext localhost:3550 list
grpcurl -plaintext localhost:3550 hipstershop.ProductCatalogService/ListProducts

# You should see JSON output with all products!
```

#### Scale Services
```bash
# Scale frontend to 3 replicas
kubectl scale deployment/frontend --replicas=3

# Watch pods scale up
kubectl get pods -w

# Scale back down
kubectl scale deployment/frontend --replicas=1
```

---

## 🚨 Troubleshooting (Common Issues)

### Issue: "Port 8080 already in use"
```bash
# Find what's using port 8080
lsof -i :8080

# Kill it
kill -9 <PID>

# Or use a different port
kubectl port-forward svc/frontend 8081:80
open http://localhost:8081
```

### Issue: "Pod CrashLoopBackOff"
```bash
# Check the logs
kubectl logs <pod-name>

# Describe the pod for events
kubectl describe pod <pod-name>

# Common cause: Redis not ready before cart service
# Solution: Wait 1-2 minutes, it usually resolves itself
```

### Issue: "Cannot connect to Docker daemon"
```bash
# Make sure Docker Desktop is running
# Check the Docker icon in menu bar
# If it's not running, start Docker Desktop

# Restart Docker if needed
# Docker Desktop → Troubleshoot → Restart Docker
```

### Issue: "Skaffold build takes forever"
```bash
# First build is slow (10-20 min) - this is normal!
# Subsequent builds use cache and are much faster

# If it's really stuck:
# 1. Press Ctrl+C to stop
# 2. Clear Skaffold cache: skaffold delete
# 3. Try again: skaffold dev
```

### Issue: "imagePullBackOff"
```bash
# This usually means Docker couldn't build the image
# Check Skaffold terminal for build errors

# Try:
skaffold delete
skaffold dev
```

---

## 📋 Alternative Method (If Skaffold Has Issues)

If Skaffold is giving you trouble, use direct kubectl:

```bash
# Deploy using pre-built images
kubectl apply -f ./kubernetes-manifests/

# Wait for pods (2-5 minutes)
kubectl get pods -w

# When all show "Running", access frontend
kubectl port-forward svc/frontend 8080:80
open http://localhost:8080
```

---

## ✅ Week 2 Checklist

Track your progress:

### Day 1-2: Setup ✅
- [ ] Docker Desktop installed and configured
- [ ] kubectl installed and working
- [ ] Skaffold installed
- [ ] Kubernetes enabled in Docker Desktop
- [ ] All tools verified

### Day 3: Deployment ✅
- [ ] `skaffold dev` runs successfully
- [ ] All 12 pods show "Running" status
- [ ] Frontend accessible at localhost:8080
- [ ] Can browse products and add to cart

### Day 4-5: Exploration ✅
- [ ] grpcurl installed
- [ ] Tested gRPC endpoints
- [ ] Viewed logs from multiple services
- [ ] Scaled services up and down
- [ ] Understand pod lifecycle

### Documentation ✅
- [ ] Created WEEK_2_LEARNINGS.md
- [ ] Documented challenges faced
- [ ] Captured screenshots
- [ ] Listed questions for Week 3

---

## 🎓 What You're Learning

By completing Week 2 setup, you'll understand:

1. **Docker Containers**
   - How applications run in containers
   - Image building and layering
   - Container networking

2. **Kubernetes Orchestration**
   - Pod lifecycle management
   - Service discovery and networking
   - Self-healing and scaling

3. **Microservices Communication**
   - gRPC request/response flow
   - Service dependencies
   - Load balancing

4. **DevOps Tools**
   - Skaffold development workflow
   - kubectl for operations
   - Debugging distributed systems

---

## 📚 Next Steps

Once you have everything running:

### This Week
1. **Explore**: Browse the app, add to cart, checkout
2. **Test**: Use grpcurl to call different services
3. **Break**: Delete a pod, watch it restart
4. **Learn**: Read logs, understand request flow
5. **Document**: Write what you discovered

### Week 3 Preview
- Study Terraform basics
- Review existing Terraform code in `/terraform`
- Plan Google Cloud Platform setup
- Prepare for cloud deployment

---

## 🆘 Need Help?

### Resources
- **Main README**: `/README.md` - Complete project overview
- **Architecture Docs**: `/docs/microservices_analysis.md` - Deep dive
- **Week 2 Guide**: `/docs/WEEK_2_NEXT_STEPS.md` - Detailed walkthrough

### Getting Unstuck
1. Read error messages carefully
2. Check logs: `kubectl logs <pod-name>`
3. Describe resource: `kubectl describe pod <pod-name>`
4. Google the error message
5. Check Stack Overflow
6. Document the issue for learning

### Community
- Kubernetes Slack: https://slack.k8s.io
- Stack Overflow: kubernetes, docker, grpc tags
- GitHub Issues: Original repo for known issues

---

## 🎯 Your Goal for Week 2

**By Friday, you should be able to:**
- Deploy all 12 services with one command
- Access the frontend in your browser
- Understand how services communicate
- Debug basic Kubernetes issues
- Feel confident with kubectl commands

**This foundation is critical** for Week 3's Infrastructure as Code work!

---

## 🎉 Ready? Let's Go!

```bash
# 1. Make sure Docker Desktop is running
# 2. Open terminal
# 3. Navigate to project:
cd /Users/daquanmcdaniel/Documents/2026/culture-threads-platform/culture-threads-platform

# 4. Deploy everything:
skaffold dev

# 5. Open new terminal and check:
kubectl get pods

# 6. Access the app:
kubectl port-forward svc/frontend 8080:80

# 7. Open browser:
open http://localhost:8080

# 🚀 You're running a 12-service microservices platform!
```

---

**Document Version**: 1.0  
**Created**: February 1, 2026  
**For**: Week 2 - Local Development Setup  
**Time Estimate**: 30-60 minutes (after tools installed)

**Good luck! You've got this! 💪**
