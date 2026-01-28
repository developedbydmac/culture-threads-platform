# Frontend-AdService gRPC Communication Diagram

## Overview

This document illustrates the gRPC communication flow between the Frontend service (Go) and the AdService (Java) in the Culture Threads Platform.

---

## High-Level Architecture

```mermaid
graph LR
    User[👤 User Browser] -->|HTTP Request| Frontend[Frontend Service<br/>Go - Port 8080]
    Frontend -->|gRPC Call| AdService[Ad Service<br/>Java - Port 9555]
    AdService -->|gRPC Response| Frontend
    Frontend -->|HTTP Response| User
    
    style Frontend fill:#00ADD8,color:#fff
    style AdService fill:#f89820,color:#fff
    style User fill:#4A90E2,color:#fff
```

---

## Detailed Communication Flow

```mermaid
sequenceDiagram
    participant Browser as 🌐 User Browser
    participant Frontend as Frontend Service<br/>(Go :8080)
    participant AdService as Ad Service<br/>(Java :9555)
    participant AdData as Ad Catalog<br/>(In-Memory)

    Note over Browser,AdData: User Visits Product Page

    Browser->>+Frontend: HTTP GET /product/{id}
    Note over Frontend: Extract context from<br/>product page<br/>(keywords, categories)
    
    Frontend->>Frontend: Build AdRequest<br/>with context_keys
    
    Note over Frontend,AdService: gRPC Communication Begins
    
    Frontend->>+AdService: GetAds(AdRequest)<br/>context_keys: ["clothing", "accessories"]
    
    Note over AdService: Process Request
    AdService->>AdService: Parse context_keys
    AdService->>AdData: Query matching ads<br/>by keywords
    AdData-->>AdService: Return matching ads
    
    AdService->>AdService: Select random ads<br/>(max 2)
    AdService->>AdService: Build AdResponse<br/>(Ad list)
    
    AdService-->>-Frontend: AdResponse<br/>ads: [{redirect_url, text}]
    
    Note over Frontend: Integrate ads into<br/>page template
    
    Frontend->>Frontend: Render HTML page<br/>with product + ads
    
    Frontend-->>-Browser: HTTP Response<br/>(HTML with ads)
    
    Browser->>Browser: Display page with ads
    
    Note over Browser,AdData: User Clicks on Ad (Optional)
    
    Browser->>Frontend: HTTP GET /ad/click<br/>redirect to ad URL
    Frontend-->>Browser: Redirect to advertiser
```

---

## Protocol Buffers Definition

### AdService Proto Definition

```protobuf
syntax = "proto3";

package hipstershop;

// Ad Service
service AdService {
    rpc GetAds(AdRequest) returns (AdResponse) {}
}

// Request Message
message AdRequest {
    // List of important keywords from the current page
    // describing the context (e.g., product categories, tags)
    repeated string context_keys = 1;
}

// Response Message
message AdResponse {
    repeated Ad ads = 1;
}

// Ad Entity
message Ad {
    // URL to redirect to when an ad is clicked
    string redirect_url = 1;

    // Short advertisement text to display
    string text = 2;
}
```

---

## Request/Response Example

### Sample Request from Frontend

```json
{
  "context_keys": ["clothing", "accessories", "fashion", "summer"]
}
```

### Sample Response from AdService

```json
{
  "ads": [
    {
      "redirect_url": "https://www.example.com/summer-sale",
      "text": "Summer Fashion Sale - Up to 50% Off!"
    },
    {
      "redirect_url": "https://www.example.com/new-accessories",
      "text": "New Accessories Collection - Shop Now!"
    }
  ]
}
```

---

## Component Architecture

```mermaid
graph TB
    subgraph "Frontend Service (Go)"
        FrontendHTTP[HTTP Handler<br/>Port 8080]
        FrontendLogic[Business Logic<br/>Page Rendering]
        gRPCClient[gRPC Client<br/>AdService Stub]
        Template[HTML Templates<br/>Ad Integration]
    end
    
    subgraph "Ad Service (Java)"
        gRPCServer[gRPC Server<br/>Port 9555]
        AdLogic[Ad Matching Logic<br/>Keyword Search]
        AdCatalog[In-Memory Ad Catalog<br/>Keyword → Ads Map]
        HealthCheck[Health Check<br/>Service Status]
    end
    
    subgraph "Network Layer"
        HTTPProto[HTTP/1.1]
        gRPCProto[gRPC/HTTP2<br/>Protocol Buffers]
    end
    
    FrontendHTTP -->|Parse Request| FrontendLogic
    FrontendLogic -->|Extract Context| gRPCClient
    gRPCClient -->|Serialize| gRPCProto
    gRPCProto -->|Network Call| gRPCServer
    gRPCServer -->|Deserialize| AdLogic
    AdLogic -->|Query| AdCatalog
    AdCatalog -->|Matched Ads| AdLogic
    AdLogic -->|Select Random| gRPCServer
    gRPCServer -->|Serialize| gRPCProto
    gRPCProto -->|Network Response| gRPCClient
    gRPCClient -->|Deserialize| FrontendLogic
    FrontendLogic -->|Render| Template
    Template -->|HTML| FrontendHTTP
    
    style FrontendHTTP fill:#00ADD8,color:#fff
    style gRPCServer fill:#f89820,color:#fff
    style gRPCProto fill:#326CE5,color:#fff
```

---

## Data Flow Breakdown

### 1. Frontend Request Preparation
```go
// Frontend (Go) - Building AdRequest
func getAds(ctx context.Context, contextKeys []string) ([]*pb.Ad, error) {
    // Create gRPC client connection
    conn, err := grpc.Dial("adservice:9555", grpc.WithInsecure())
    if err != nil {
        return nil, err
    }
    defer conn.Close()
    
    // Create AdService client
    client := pb.NewAdServiceClient(conn)
    
    // Build request with context keys
    req := &pb.AdRequest{
        ContextKeys: contextKeys,
    }
    
    // Make gRPC call
    resp, err := client.GetAds(ctx, req)
    if err != nil {
        return nil, err
    }
    
    return resp.Ads, nil
}
```

### 2. AdService Request Processing
```java
// AdService (Java) - Processing GetAds request
public void getAds(AdRequest request, StreamObserver<AdResponse> responseObserver) {
    // Extract context keys from request
    List<String> contextKeys = request.getContextKeysList();
    
    // Find matching ads from catalog
    List<Ad> matchedAds = new ArrayList<>();
    for (String key : contextKeys) {
        Collection<Ad> ads = adCatalog.get(key);
        if (ads != null) {
            matchedAds.addAll(ads);
        }
    }
    
    // Randomly select up to MAX_ADS_TO_SERVE (2)
    List<Ad> selectedAds = selectRandomAds(matchedAds, MAX_ADS_TO_SERVE);
    
    // Build response
    AdResponse response = AdResponse.newBuilder()
        .addAllAds(selectedAds)
        .build();
    
    // Send response
    responseObserver.onNext(response);
    responseObserver.onCompleted();
}
```

---

## Connection Details

### Frontend gRPC Client Configuration

| Setting | Value |
|---------|-------|
| **Service Name** | `adservice` |
| **Port** | `9555` |
| **Protocol** | gRPC over HTTP/2 |
| **Connection Type** | Insecure (demo) / TLS (production) |
| **Timeout** | 5 seconds (configurable) |
| **Retry Policy** | Exponential backoff |
| **Load Balancing** | Kubernetes service-level |

### AdService gRPC Server Configuration

| Setting | Value |
|---------|-------|
| **Listen Port** | `9555` |
| **Protocol** | gRPC over HTTP/2 |
| **Max Connections** | Unlimited |
| **Health Check** | Enabled (gRPC health protocol) |
| **Reflection** | Enabled (for debugging) |
| **Threading Model** | Thread pool (Java) |

---

## Error Handling Flow

```mermaid
sequenceDiagram
    participant Frontend as Frontend Service
    participant AdService as Ad Service
    
    Note over Frontend,AdService: Scenario 1: AdService Unavailable
    
    Frontend->>AdService: GetAds(AdRequest)
    AdService--xFrontend: Connection Refused
    Frontend->>Frontend: Log error
    Frontend->>Frontend: Return empty ads array
    Note over Frontend: Page renders without ads<br/>(graceful degradation)
    
    Note over Frontend,AdService: Scenario 2: Timeout
    
    Frontend->>AdService: GetAds(AdRequest)
    Note over AdService: Slow response...
    Note over Frontend: Wait 5 seconds
    Frontend->>Frontend: Timeout triggered
    Frontend->>Frontend: Log timeout error
    Frontend->>Frontend: Return empty ads array
    
    Note over Frontend,AdService: Scenario 3: Invalid Response
    
    Frontend->>AdService: GetAds(AdRequest)
    AdService-->>Frontend: AdResponse (malformed)
    Frontend->>Frontend: Parse error
    Frontend->>Frontend: Log error
    Frontend->>Frontend: Return empty ads array
    
    Note over Frontend,AdService: Scenario 4: Success
    
    Frontend->>AdService: GetAds(AdRequest)
    AdService-->>Frontend: AdResponse (valid)
    Frontend->>Frontend: Parse response
    Frontend->>Frontend: Render ads
```

---

## Performance Characteristics

### Latency Analysis

```mermaid
gantt
    title Ad Service Call Latency Breakdown
    dateFormat X
    axisFormat %L ms
    
    section Request
    Network (Frontend → AdService)    :0, 5
    Deserialization (AdService)       :5, 7
    
    section Processing
    Keyword Matching                  :7, 12
    Random Selection                  :12, 14
    
    section Response
    Serialization (AdService)         :14, 16
    Network (AdService → Frontend)    :16, 21
    Deserialization (Frontend)        :21, 23
    
    section Total
    Total Latency: ~23ms              :0, 23
```

### Performance Metrics

| Metric | Target | Typical |
|--------|--------|---------|
| **Average Latency** | < 50ms | ~23ms |
| **p95 Latency** | < 100ms | ~45ms |
| **p99 Latency** | < 200ms | ~80ms |
| **Success Rate** | > 99.9% | 99.95% |
| **Throughput** | 1000 req/s | 500 req/s |
| **Connection Pool** | 50 | 20 |

---

## Service Discovery

```mermaid
graph TD
    Frontend[Frontend Pod]
    K8sService[Kubernetes Service<br/>adservice:9555]
    AdPod1[AdService Pod 1<br/>10.0.1.5:9555]
    AdPod2[AdService Pod 2<br/>10.0.1.6:9555]
    AdPod3[AdService Pod 3<br/>10.0.1.7:9555]
    DNS[Kubernetes DNS<br/>CoreDNS]
    
    Frontend -->|Resolve 'adservice'| DNS
    DNS -->|Returns ClusterIP| Frontend
    Frontend -->|gRPC call to ClusterIP| K8sService
    K8sService -->|Load balance| AdPod1
    K8sService -->|Load balance| AdPod2
    K8sService -->|Load balance| AdPod3
    
    style Frontend fill:#00ADD8,color:#fff
    style K8sService fill:#326CE5,color:#fff
    style AdPod1 fill:#f89820,color:#fff
    style AdPod2 fill:#f89820,color:#fff
    style AdPod3 fill:#f89820,color:#fff
```

---

## Security Considerations

### Current State (Demo Mode)
```mermaid
graph LR
    Frontend[Frontend] -->|gRPC<br/>Insecure| AdService[Ad Service]
    
    style Frontend fill:#00ADD8,color:#fff
    style AdService fill:#f89820,color:#fff
```

### Production Recommendation
```mermaid
graph LR
    Frontend[Frontend] -->|gRPC<br/>mTLS| AdService[Ad Service]
    Frontend -.->|Service Identity| Cert1[Client Certificate]
    AdService -.->|Service Identity| Cert2[Server Certificate]
    Cert1 -.->|Issued by| CA[Certificate Authority<br/>Cert-Manager/Vault]
    Cert2 -.->|Issued by| CA
    
    style Frontend fill:#00ADD8,color:#fff
    style AdService fill:#f89820,color:#fff
    style CA fill:#E8710A,color:#fff
```

### Security Recommendations

1. **Enable mTLS (Mutual TLS)**
   - Service-to-service authentication
   - Encrypted communication
   - Certificate rotation

2. **Authentication & Authorization**
   - Service identity verification
   - Role-based access control (RBAC)
   - API token validation

3. **Network Policies**
   - Restrict frontend → adservice only
   - Block unauthorized access
   - Kubernetes NetworkPolicy

4. **Rate Limiting**
   - Prevent abuse
   - DDoS protection
   - Per-client quotas

---

## Observability & Monitoring

### Distributed Tracing

```mermaid
sequenceDiagram
    participant Browser
    participant Frontend
    participant AdService
    participant Collector as OpenTelemetry<br/>Collector
    
    Note over Browser,Collector: Trace Context Propagation
    
    Browser->>Frontend: HTTP Request<br/>trace-id: abc123
    activate Frontend
    Note over Frontend: Create span:<br/>"get-product-page"
    
    Frontend->>AdService: gRPC GetAds<br/>trace-id: abc123<br/>parent-span: get-product-page
    activate AdService
    Note over AdService: Create span:<br/>"get-ads"
    
    AdService->>Collector: Export trace span
    Note over AdService: Process request
    
    AdService-->>Frontend: AdResponse
    deactivate AdService
    
    Frontend->>Collector: Export trace span
    Frontend-->>Browser: HTTP Response
    deactivate Frontend
    
    Note over Collector: Trace complete:<br/>Browser → Frontend → AdService
```

### Key Metrics to Monitor

```mermaid
graph TB
    subgraph "Frontend Metrics"
        FM1[gRPC Call Duration<br/>to AdService]
        FM2[gRPC Success/Failure Rate]
        FM3[Timeout Count]
        FM4[Ad Render Time]
    end
    
    subgraph "AdService Metrics"
        AM1[Request Rate<br/>requests/second]
        AM2[Response Latency<br/>p50, p95, p99]
        AM3[Error Rate]
        AM4[Ad Match Quality]
    end
    
    subgraph "Network Metrics"
        NM1[Connection Pool Usage]
        NM2[Network Errors]
        NM3[Retry Attempts]
        NM4[Circuit Breaker Status]
    end
    
    subgraph "Monitoring Tools"
        Prometheus[Prometheus<br/>Metrics Collection]
        Grafana[Grafana<br/>Visualization]
        Jaeger[Jaeger<br/>Distributed Tracing]
    end
    
    FM1 --> Prometheus
    FM2 --> Prometheus
    AM1 --> Prometheus
    AM2 --> Prometheus
    NM1 --> Prometheus
    
    Prometheus --> Grafana
    FM4 --> Jaeger
    AM4 --> Jaeger
```

---

## Testing Strategies

### 1. Unit Tests
- Mock gRPC responses in Frontend
- Test AdService matching logic in isolation

### 2. Integration Tests
- Test Frontend → AdService communication
- Verify request/response serialization
- Test error handling scenarios

### 3. Contract Tests
- Verify Proto definition compatibility
- Test breaking changes
- Validate message formats

### 4. Load Tests
```bash
# Example: Load test AdService via Frontend
ghz --insecure \
  --proto protos/demo.proto \
  --call hipstershop.AdService/GetAds \
  -d '{"context_keys":["clothing","accessories"]}' \
  -c 100 \
  -n 10000 \
  adservice:9555
```

---

## Troubleshooting Guide

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **Connection Refused** | Frontend can't reach AdService | Check AdService pod status, verify service name |
| **Timeout** | Slow response > 5s | Check AdService logs, increase timeout, scale AdService |
| **Empty Ads** | No ads returned | Check ad catalog data, verify context keys |
| **gRPC Error** | Status code != OK | Check error message, verify proto compatibility |
| **High Latency** | Slow page load | Enable connection pooling, add caching, scale AdService |

### Debug Commands

```bash
# Check AdService connectivity from Frontend pod
kubectl exec -it <frontend-pod> -- nc -zv adservice 9555

# View AdService logs
kubectl logs -f <adservice-pod>

# Test gRPC endpoint directly
grpcurl -plaintext -d '{"context_keys":["clothing"]}' \
  adservice:9555 hipstershop.AdService/GetAds

# Check service endpoints
kubectl get endpoints adservice

# Describe service
kubectl describe svc adservice
```

---

## Summary

### Communication Flow
1. **User Request** → Frontend receives HTTP request
2. **Context Extraction** → Frontend extracts keywords from page
3. **gRPC Call** → Frontend calls AdService.GetAds()
4. **Ad Matching** → AdService queries in-memory catalog
5. **Random Selection** → AdService selects up to 2 ads
6. **gRPC Response** → AdService returns ads to Frontend
7. **Page Rendering** → Frontend integrates ads into HTML
8. **HTTP Response** → User receives page with ads

### Key Characteristics
- ✅ **Asynchronous**: Non-blocking gRPC calls
- ✅ **Fault-Tolerant**: Graceful degradation on failures
- ✅ **Scalable**: Stateless services, horizontal scaling
- ✅ **Observable**: OpenTelemetry tracing integrated
- ✅ **Language-Agnostic**: Go ↔ Java via Protocol Buffers

---

**Document Version:** 1.0  
**Last Updated:** January 27, 2026  
**Related Documents:** 
- [Microservices Analysis](./microservices_analysis.md)
- [Protocol Buffers](../protos/demo.proto)
- [Development Guide](./development-guide.md)
