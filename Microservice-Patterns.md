## 📘 Microservices Patterns

| S.No | Importance | Category      | Pattern Name                     | Pattern Description                               | Spring Boot Library / Tool |
| ---: | ---------- | ------------- | -------------------------------- | ------------------------------------------------- | -------------------------- |
|    1 | 🔴         | Design        | Decompose by Business Capability | Split services based on business functions        | Spring Boot + DDD          |
|    2 | 🔴         | Design        | Decompose by Subdomain (DDD)     | Use bounded contexts to define service boundaries | Spring Boot + DDD          |
|    3 | 🔴         | Design        | Strangler                     | Gradually migrate monolith to microservices          | Spring Cloud Gateway       |
|    5 | 🔴         | Data          | Database per Service             | Each service owns its own database                | Spring Data JPA / MongoDB  |
|    6 | 🔴         | Data          | Shared Database (Anti-Pattern)   | Tight coupling via shared DB                      | ❌ Avoid                   |
|    7 | 🔴         | Data          | Saga – Choreography              | Distributed transaction via events                | Spring Kafka               |
|    8 | 🔴         | Data          | Saga – Orchestration             | Central controller for saga flow                  | Camunda / Temporal         |
|    9 | 🔴         | Data          | CQRS                             | Separate read and write models                    | Axon Framework             |
|   13 | 🔴         | Communication | Asynchronous Messaging           | Event-driven service communication                | Spring Kafka               |
|   14 | 🔴         | Communication | API Gateway                      | Single entry point for clients                    | Spring Cloud Gateway       |
|   17 | 🔴         | Resilience    | Circuit Breaker                  | Prevent cascading failures                        | Resilience4j               |
|   18 | 🔴         | Resilience    | Retry                            | Retry transient failures                          | Spring Retry               |
|   19 | 🔴         | Resilience    | Bulkhead                         | Isolate resource usage                            | Resilience4j               |
|   20 | 🔴         | Resilience    | Timeout                          | Avoid indefinite waits                            | Resilience4j               |
|   21 | 🔴         | Resilience    | Fallback                         | Graceful degradation                              | Resilience4j               |
|   22 | 🔴         | Resilience    | Rate Limiting                    | Protect services from overload                    | Bucket4j                   |
|   23 | 🔴         | Discovery     | Client-Side Discovery            | Client finds service instances                    | Eureka Client              |
|   24 | 🔴         | Discovery     | Server-Side Discovery            | Platform resolves services                        | Kubernetes Service         |
|   29 | 🔴         | Deployment    | Sidecar                          | Infra helper container                            | Istio Envoy                |
|   30 | 🔴         | Deployment    | Ambassador                       | External communication proxy                      | Ambassador Gateway         |
|   31 | 🔴         | Deployment    | Blue-Green Deployment            | Zero-downtime releases                            | ArgoCD                     |
|   32 | 🔴         | Deployment    | Canary Deployment                | Gradual traffic rollout                           | Argo Rollouts              |
|   33 | 🔴         | Deployment    | Rolling Deployment               | Incremental updates                               | Kubernetes                 |
|   34 | 🔴         | Configuration | Externalized Configuration       | Central config management                         | Spring Cloud Config        |
|   35 | 🟢         | Configuration | Secret Management                | Secure secrets storage                            | Vault / AWS Secrets        |
|   36 | 🟢         | Observability | Centralized Logging              | Unified log analysis                              | ELK / OpenSearch           |
|   37 | 🟢         | Observability | Distributed Tracing              | Trace calls across services                       | Sleuth + Zipkin            |
|   38 | 🟢         | Observability | Correlation ID                   | Track request end-to-end                          | Spring Sleuth              |
|   39 | 🟢         | Observability | Health Check                     | Liveness/readiness probes                         | Actuator                   |
|   40 | 🟢         | Observability | Metrics Collection               | System & app metrics                              | Micrometer                 |
|   41 | 🟢         | Security      | API Gateway Security             | Central auth & authorization                      | Spring Security            |
|   42 | 🟢         | Security      | OAuth2 / OIDC                    | Token-based security                              | Spring Security OAuth2     |
|   43 | 🟢         | Security      | Mutual TLS (mTLS)                | Secure service-to-service traffic                 | Istio + Spring Security    |
|   44 | 🟢         | Security      | Zero Trust Architecture          | Verify every request                              | Spring Security            |
|   45 | 🟢         | Testing       | Consumer-Driven Contract         | Ensure API compatibility                          | Spring Cloud Contract      |
|   46 | 🟢         | Testing       | Integration Testing              | Test real interactions                            | Testcontainers             |
|   47 | 🟢         | Testing       | Chaos Engineering                | Failure injection testing                         | Chaos Monkey               |
|   48 | 🟢         | Anti-Pattern  | Distributed Monolith             | Tightly coupled services                          | ❌ Avoid                  |
|   49 | 🟢         | Anti-Pattern  | Chatty Services                  | Excessive sync calls                              | ❌ Avoid                  |
|   50 | 🟢         | Anti-Pattern  | God API Gateway                  | Gateway becomes bottleneck                        | ❌ Avoid                  |
|    4 | 🟡         | Design        | Anti-Corruption Layer            | Isolate legacy system models                      | Adapter Pattern            |
|   10 | 🟡         | Data          | Event Sourcing                   | Persist state as events                           | Axon Framework             |
|   11 | 🟡         | Data          | Transactional Outbox             | Reliable DB + Kafka publishing                    | Debezium                   |
|   12 | 🟡         | Communication | Synchronous Communication        | REST/gRPC request-response                        | Spring Web                 |
|   25 | 🟡         | Discovery     | Self-Registration                | Service registers itself                          | Eureka                     |
|   26 | 🟡         | Discovery     | Third-Party Registration         | External registration agent                       | Consul                     |
|   27 | 🟡         | Deployment    | Container per Service            | Containerized microservices                       | Docker + Kubernetes        |
|   28 | 🟡         | Deployment    | One Service per VM               | Strong isolation per service                      | VM / EC2                   |
|   15 | 🟡         | Communication | Backend for Frontend (BFF)       | Client-specific backend APIs                      | Spring Cloud Gateway       |
|   16 | 🟢         | Communication | Service Mesh                     | Infra-level service communication                 | Istio / Linkerd            |

---

## 🔄 Spring Boot Libraries — Legacy vs Modern

| Area                    | Legacy / Deprecated Library        | Status        | Modern / Recommended Replacement  |
| ----------------------- | ---------------------------------- | ------------- | --------------------------------- |
| Service Discovery       | Netflix Eureka                     | Legacy        | Kubernetes Service / DNS          |
| Client Load Balancer    | Ribbon                             | Deprecated    | Spring Cloud LoadBalancer         |
| API Gateway             | Zuul 1.x                           | Deprecated    | Spring Cloud Gateway              |
| API Gateway             | Zuul 2                             | Discontinued  | Spring Cloud Gateway / Envoy      |
| Resilience              | Hystrix                            | End-of-Life   | Resilience4j                      |
| Resilience Dashboard    | Hystrix Dashboard                  | Deprecated    | Micrometer + Grafana              |
| Circuit Aggregation     | Turbine                            | Deprecated    | Prometheus                        |
| Distributed Tracing     | Spring Cloud Sleuth (old)          | Deprecated    | Micrometer Tracing                |
| Tracing Backend         | Zipkin (only)                      | Legacy        | OpenTelemetry                     |
| Security OAuth          | Spring Security OAuth (legacy)     | Deprecated    | Spring Security OAuth2            |
| REST Client             | RestTemplate                       | Legacy (Soft) | WebClient                         |
| Declarative REST        | Feign + Ribbon                     | Legacy combo  | Feign + Spring Cloud LoadBalancer |
| Config Management       | bootstrap.yml                      | Deprecated    | application.yml (Config Data API) |
| Config Framework        | Archaius                           | Deprecated    | Spring Cloud Config / K8s Config  |
| Central Config          | Git-only Config Server             | Situational   | Vault / K8s ConfigMaps            |
| Messaging Abstraction   | Spring Cloud Stream (binder-heavy) | Situational   | Spring Kafka                      |
| Metrics                 | Dropwizard Metrics                 | Legacy        | Micrometer                        |
| Monitoring              | Old Actuator                       | Legacy        | Actuator + Micrometer             |
| Container Discovery     | Eureka in K8s                      | Legacy        | Native Kubernetes Discovery       |
| Security Client         | OAuth2RestTemplate                 | Deprecated    | WebClient + OAuth2                |
| Tracing Instrumentation | Sleuth Brave                       | Legacy        | OpenTelemetry SDK                 |
| Chaos Testing           | Netflix Chaos Monkey               | Legacy        | Chaos Mesh / Litmus               |
| Testing Framework       | JUnit 4 / SpringRunner             | Deprecated    | JUnit 5 (Jupiter)                 |
| Mocking                 | MockitoRunner                      | Deprecated    | MockitoExtension                  |
| ORM                     | Hibernate 5.x (older)              | Legacy        | Hibernate 6.x                     |
| Configuration Style     | XML-based Spring Config            | Deprecated    | Java Config / Annotations         |

---

## 🧠 **Interview-Ready Summary Line**

> *“Netflix OSS components like Eureka, Ribbon, Hystrix, and Zuul are legacy. Modern Spring Boot systems prefer Kubernetes-native discovery, Spring Cloud Gateway, Resilience4j, WebClient, Micrometer, and OpenTelemetry.”*

---

## ✅ **Modern Spring Boot 3.x Stack (Reference)**

```text
API Gateway        → Spring Cloud Gateway
Service Discovery  → Kubernetes DNS
Resilience         → Resilience4j
Tracing            → OpenTelemetry + Tempo
Metrics            → Micrometer + Prometheus
Security           → Spring Security OAuth2
REST Client        → WebClient
Messaging          → Spring Kafka
Config             → Vault + ConfigMap
```


Just tell me 👍
