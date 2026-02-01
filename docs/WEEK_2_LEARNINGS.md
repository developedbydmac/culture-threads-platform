# Week 2 Learnings - Local Deployment

**Date**: February 1, 2026  
**Focus**: Deploying Culture Threads Platform Locally with Kubernetes

---

## 🎯 What I Accomplished Today

- ✅ Deployed 12 microservices to local Kubernetes cluster
- ✅ Fixed Dockerfile build issues (`--platform=$BUILDPLATFORM` directive)
- ✅ All services running successfully
- ✅ Accessed the frontend application
- ✅ Tested gRPC endpoints with grpcurl
- ✅ Viewed real-time service logs
- ✅ Learned kubectl commands for managing Kubernetes resources

---

## 🏗️ Services Deployed

| # | Service | Language | Purpose |
|---|---------|----------|---------|
| 1 | **adservice** | Java | Serves contextual advertisements |
| 2 | **cartservice** | C# (.NET) | Shopping cart management with Redis |
| 3 | **checkoutservice** | Go | Orchestrates checkout flow |
| 4 | **currencyservice** | Node.js | Currency conversion |
| 5 | **emailservice** | Python | Sends order confirmation emails |
| 6 | **frontend** | Go | Web UI and HTTP server |
| 7 | **loadgenerator** | Python | Simulates user traffic |
| 8 | **paymentservice** | Node.js | Processes payments (mock) |
| 9 | **productcatalogservice** | Go | Manages product catalog |
| 10 | **recommendationservice** | Python | ML-based product recommendations |
| 11 | **shippingservice** | Go | Calculates shipping quotes |
| 12 | **redis-cart** | Redis | In-memory database for cart data |

---

## 🚧 Challenges Faced & Solutions

### Challenge 1: Dockerfile Platform Directive Issue
**Problem**: All 12 services failed to build with error:
```
removing unused default args: parsing dockerfile: file with no instructions
```

**Root Cause**: Dockerfiles contained `FROM --platform=$BUILDPLATFORM` which is for multi-architecture builds, but Docker Desktop's local builder didn't support this directive.

**Solution**: 
- Removed `--platform=$BUILDPLATFORM` from all 12 Dockerfiles using bulk sed command:
  ```bash
  find src -name Dockerfile -type f -exec sed -i '' 's/FROM --platform=\$BUILDPLATFORM /FROM /g' {} \;
  ```
- Cleared Docker cache: `docker system prune -f` (freed 2.9GB)
- Restarted `skaffold dev`

**Learning**: Multi-architecture directives are for CI/CD pipelines building for different CPU architectures (amd64, arm64), not needed for local development.

---

### Challenge 2: Some Services Restarted
**Problem**: `emailservice` (2 restarts) and `recommendationservice` (1 restart) showed restarts in pod status.

**Root Cause**: Services started before their dependencies were ready (e.g., other gRPC services they call).

**Solution**: No action needed - Kubernetes automatically restarted them until dependencies were available.

**Learning**: This is normal in microservices architecture. Kubernetes' self-healing is working as designed!

---

## 📚 What I Learned

### Kubernetes Concepts
- **Pods**: Smallest deployable unit, contains one or more containers
- **Deployments**: Manages pod replicas and updates
- **Services**: Provides stable networking for pods
  - **ClusterIP**: Internal-only access (default)
  - **LoadBalancer**: External access (frontend-external)
- **Self-Healing**: Kubernetes automatically restarts failed pods

### Microservices Communication
- **gRPC**: Services communicate via Remote Procedure Calls
- **Protocol Buffers**: Binary serialization format (faster than JSON)
- **Service Discovery**: Services find each other via Kubernetes DNS
  - Example: `cartservice:7070` resolves to cart service pod

### Tools & Commands
- **Skaffold**: Automates build-deploy-watch cycle
- **kubectl**: Command-line tool for Kubernetes operations
- **grpcurl**: Like `curl` but for gRPC services
- **k9s**: Terminal UI for Kubernetes (amazing!)

---

## 🔍 Key Observations

### Frontend Service Logs
Captured real-time HTTP traffic:
- Users browsing product pages (`/product/<id>`)
- Adding items to cart (`POST /cart`)
- Currency conversions (USD → GBP, JPY, TRY)
- Checkout flow (`/cart/checkout`)
- Health checks every few seconds (`/_healthz`)

**Example log entry**:
```json
{
  "http.req.method": "POST",
  "http.req.path": "/cart/checkout",
  "message": "order placed",
  "order": "5f3eea44-ff80-11f0-a3bb-327f0468cd8a",
  "session": "b9969655-5de8-44ae-8941-63ec0b6fbebb",
  "severity": "info"
}
```

---

### Cart Service Logs
Captured gRPC calls from frontend:
- `GetCartAsync` called with unique user IDs
- Health checks from Kubernetes liveness/readiness probes
- Redis integration working smoothly

**Observation**: Each user has a session ID (UUID) that persists their cart across requests.

---

### gRPC Testing Success
Successfully called `ProductCatalogService/ListProducts`:
- Port forwarded to `localhost:3550`
- Used command: `grpcurl -plaintext -proto protos/demo.proto localhost:3550 hipstershop.ProductCatalogService/ListProducts`
- Received JSON response with **9 products**: Sunglasses, Tank Top, Watch, Loafers, Hairdryer, Candle Holder, Salt & Pepper Shakers, Bamboo Glass Jar, Mug

**Interesting**: gRPC represents money as:
```json
"priceUsd": {
  "currencyCode": "USD",
  "units": "19",
  "nanos": 990000000
}
```
This means $19.99 = 19 units + 990,000,000 nanos (avoids floating-point precision issues!)

---

## ❓ Questions for Next Time

1. **Service Discovery**: How exactly does Kubernetes DNS resolve service names to pod IPs?
2. **LoadBalancer vs ClusterIP**: When should I use each type of service?
3. **Pod Crashes**: What's the best way to debug when a pod enters CrashLoopBackOff?
4. **Redis Integration**: How does cartservice store data in Redis? What's the key structure?
5. **gRPC Money Format**: Why use `units` and `nanos` instead of float? (Answer: avoids 0.1 + 0.2 = 0.30000000004 problems!)
6. **Self-Healing**: What triggers Kubernetes to restart a pod? (Answer: liveness/readiness probes)
7. **Skaffold**: How does it detect code changes and know what to rebuild?
8. **Proto Files**: How do I read `.proto` files to understand gRPC service contracts?

---

## 🎓 Commands I Mastered Today

### Deployment
```bash
# Deploy all services with live reload
skaffold dev

# Deploy with pre-built images (alternative)
kubectl apply -f ./kubernetes-manifests/
```

### Monitoring
```bash
# Check pod status
kubectl get pods
kubectl get pods -w  # watch mode

# Check deployments
kubectl get deployments

# Check services
kubectl get services

# View logs
kubectl logs deployment/frontend
kubectl logs deployment/cartservice --tail=50
kubectl logs -f deployment/frontend  # follow mode
```

### Port Forwarding
```bash
# Forward service port to localhost
kubectl port-forward svc/frontend 8080:80
kubectl port-forward svc/productcatalogservice 3550:3550
```

### gRPC Testing
```bash
# List available services (requires reflection API)
grpcurl -plaintext localhost:3550 list

# Call a specific method with proto file
grpcurl -plaintext -proto protos/demo.proto localhost:3550 \
  hipstershop.ProductCatalogService/ListProducts
```

### Scaling
```bash
# Scale up
kubectl scale deployment/frontend --replicas=3

# Watch pods scale
kubectl get pods -w

# Scale down
kubectl scale deployment/frontend --replicas=1
```

### Self-Healing Test
```bash
# Delete a pod (Kubernetes will recreate it)
kubectl delete pod <pod-name>

# Or delete by label
kubectl delete pod $(kubectl get pods -l app=frontend -o jsonpath='{.items[0].metadata.name}')

# Watch it recreate
kubectl get pods -w
```

### Context Management
```bash
# List contexts
kubectl config get-contexts

# Switch context
kubectl config use-context docker-desktop
```

---

## ⏱️ Time Breakdown

- **Prerequisites Setup**: 15 minutes (verified Docker, installed Skaffold, grpcurl, k9s)
- **First Deployment Attempt**: 10 minutes (failed with platform directive error)
- **Troubleshooting & Fix**: 20 minutes (diagnosed issue, fixed all Dockerfiles, cleared cache)
- **Successful Deployment**: 20 minutes (all services built and deployed)
- **Exploration & Testing**: 45 minutes (logs, gRPC, scaling)
- **Documentation**: 20 minutes (writing this document)

**Total Time**: ~2 hours 10 minutes

---

## 📸 Screenshots TODO

- [ ] Browser showing Culture Threads e-commerce store
- [ ] Terminal showing `kubectl get pods` with all services running
- [ ] k9s dashboard with colorful pod view
- [ ] gRPC response in terminal showing product JSON
- [ ] Skaffold dev output showing successful builds
- [ ] Frontend logs showing HTTP requests

---

## 🚀 Next Steps for Week 2

### Short Term (This Week)
- [ ] Continue exploring the running application
- [ ] Browse the store UI (add to cart, checkout)
- [ ] Test scaling different services (not just frontend)
- [ ] Try port-forwarding to other services
- [ ] Test self-healing by deleting random pods
- [ ] Use k9s to navigate and view logs interactively
- [ ] Read the proto file (`protos/demo.proto`) to understand service contracts

### Learning Goals
- [ ] Understand Kubernetes networking in depth
- [ ] Learn how service mesh works
- [ ] Study liveness vs readiness probes
- [ ] Understand resource limits and requests
- [ ] Learn about ConfigMaps and Secrets

### Week 3 Preparation
- [ ] Study Terraform basics (HCL syntax)
- [ ] Review existing Terraform code in `/terraform` directory
- [ ] Research Google Cloud Platform (GKE, Cloud SQL, etc.)
- [ ] Plan cloud deployment strategy
- [ ] Understand Infrastructure as Code (IaC) concepts

---

## 💡 Key Insights

1. **Microservices are Complex**: 12 services means 12 potential failure points, but Kubernetes makes it manageable.

2. **gRPC is Fast**: Binary protocol is more efficient than REST/JSON for inter-service communication.

3. **Kubernetes is Self-Healing**: I don't need to manually restart failed pods - it just works!

4. **Skaffold is Magic**: Changed code, it auto-rebuilds and redeploys. Game changer for development!

5. **Docker Desktop is Powerful**: Full Kubernetes cluster running on my laptop with 12 services!

6. **Logs Tell Stories**: Watching logs in real-time shows exactly how microservices interact.

7. **Proto Files are Contracts**: They define the API between services - like OpenAPI but for gRPC.

---

## 🎉 Success Metrics

- ✅ **12/12 services** running and healthy
- ✅ **0 CrashLoopBackOff** errors (after initial startup)
- ✅ **Frontend accessible** at `localhost:80` and `localhost:31313`
- ✅ **gRPC working** (successfully called ProductCatalogService)
- ✅ **Self-healing validated** (pods restarted automatically)
- ✅ **Load generation working** (constant traffic visible in logs)

---

## 📝 Personal Notes

This was more challenging than expected, but incredibly rewarding! The Dockerfile platform issue taught me:
- Always read error messages carefully
- Docker build context matters
- Multi-architecture builds are advanced use cases

The platform is actually running real e-commerce logic:
- Shopping cart persists across requests
- Currency conversion is real-time
- Checkout orchestrates multiple services
- Recommendations use product data

I'm amazed that this entire system runs on my laptop. Can't wait to deploy this to the cloud in Week 3!

---

**Status**: ✅ **Week 2 Day 1 COMPLETE!**  
**Next Session**: Continue exploration, test more features, prepare for Week 3  
**Confidence Level**: 🟢 High - Ready to move forward!

---

**Document Created**: February 1, 2026  
**Last Updated**: February 1, 2026  
**Author**: Daquan McDaniel
