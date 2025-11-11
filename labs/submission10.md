# Lab 10 — Cloud Computing Fundamentals

## Task 1 — Artifact Registries Research

### 1.1 Services overview

- **AWS**
  - **Amazon Elastic Container Registry (ECR):** Private Docker/OCI image registry with vulnerability scanning, cross-region/account replication, encryption at rest, IAM integration, and CI/CD hooks.
  - **AWS CodeArtifact:** Managed package registry for Maven, npm, PyPI, NuGet, and more; integrates with standard package managers and AWS build tools.

- **GCP**
  - **Artifact Registry:** Unified registry for container images and language packages (e.g., Maven, npm, Python). Integrated IAM, vulnerability scanning, attestations, and Cloud Build integration.

- **Azure**
  - **Azure Container Registry (ACR):** Private Docker/OCI registry with geo-replication, content trust/signing, Private Link, and ACR Tasks.  
  - *(Note:* Azure has **Azure Artifacts** for language packages within Azure DevOps.)

### 1.2 Supported artifact types

- **Containers:** AWS ECR, GCP Artifact Registry, Azure ACR (all OCI/Docker).  
- **Language packages:** AWS CodeArtifact (Maven/npm/PyPI/NuGet/etc.), GCP Artifact Registry (Maven/npm/Python/etc.). Azure Artifacts covers packages on the Azure side.

### 1.3 Key features

- **Security & Compliance**
  - ECR: image scanning, KMS/SSE encryption, IAM policies.
  - Artifact Registry: vulnerability scanning and attestations, IAM.
  - ACR: image signing/content trust, Defender integrations, private networking.

- **Networking & Replication**
  - ECR: cross-region and cross-account replication; VPC endpoints.
  - Artifact Registry: regional repositories; Private Service Connect.
  - ACR: geo-replication (Premium tier), Private Link.

- **CI/CD & Ecosystem**
  - ECR: tight integration with ECS/EKS/CodeBuild/CodePipeline.
  - Artifact Registry: Cloud Build, Cloud Deploy, GKE.
  - ACR: AKS, GitHub Actions/Azure Pipelines, ACR Tasks (builds, base-image updates).

### 1.4 Comparison table

| Factor | AWS ECR | GCP Artifact Registry | Azure ACR |
|---|---|---|---|
| Artifact formats | Docker/OCI | Docker/OCI + Maven/npm/Python | Docker/OCI |
| Vulnerability scanning | Yes | Yes | Via Defender/partner |
| Replication | Cross-region/account | Regional repos | Geo-replication (Premium) |
| Access control | IAM | IAM | RBAC/AAD |
| Private networking | VPC endpoints | Private Service Connect | Private Link |
| CI/CD | ECS/EKS/Code* | Cloud Build/Deploy/GKE | AKS/ACR Tasks/Pipelines |
| Pricing basics | Storage + egress | Storage + egress | SKU tier + features |

### 1.5 Analysis

- **Multi-cloud teams** benefit from standardizing on OCI images and common scanning/signing policies. If you also need package registries in the same service, **GCP Artifact Registry** is the most unified option.  
- **AWS-centric stacks** pair **ECR** (images) with **CodeArtifact** (packages) for full coverage and deep AWS integration.  
- **Azure-centric stacks** that require geo-replication and private networking often prefer **ACR Premium**.

**Bottom line:** Choose per platform affinity and network/replication needs; keep artifacts OCI-compliant and policies portable.

---

## Task 2 — Serverless Computing Platform Research

### 2.1 Services overview

- **AWS Lambda:** Functions-as-a-Service (FaaS). Rich event ecosystem (S3, SNS, EventBridge, API Gateway). Max runtime typically **15 minutes**. Options to mitigate cold starts: Provisioned Concurrency, SnapStart (Java).
- **GCP Cloud Functions (Gen2) / Cloud Run:** Functions on top of Cloud Run or direct serverless containers with HTTP/event triggers. Cloud Run allows per-request runtimes up to **60 minutes** and supports minimum instances to keep containers warm.
- **Azure Functions:** FaaS with multiple hosting plans. **Consumption** has a default timeout up to **10 minutes**; **Premium** reduces cold starts via pre-warmed instances and allows longer runtimes and VNet integration.

### 2.2 Runtimes and execution model

- **AWS Lambda:** multiple managed runtimes or custom container images; automatic scaling; concurrency controls; wide event sources.  
- **GCP Cloud Functions/Run:** HTTP and event triggers, Pub/Sub, Eventarc; min/max instances for scale and cold-start control.  
- **Azure Functions:** HTTP/queue/timer/event triggers; Premium keeps instances pre-warmed; deep Azure integrations.

### 2.3 Performance characteristics

- **Cold starts:** Lambda’s Provisioned Concurrency and SnapStart reduce startup latency; Cloud Run’s min instances keep containers hot; Azure Functions Premium keeps workers warm.  
- **Throughput & concurrency:** All three provide automatic scaling with per-platform concurrency and quota levers.  
- **Observability:** CloudWatch (AWS), Cloud Logging/Trace (GCP), Application Insights (Azure).

### 2.4 Limits and timeouts (at a glance)

| Platform | Max per-request duration | Cold-start mitigation |
|---|---|---|
| AWS Lambda | ~15 minutes | Provisioned Concurrency, SnapStart (Java) |
| GCP Cloud Run | ~60 minutes | Min instances |
| GCP Cloud Functions (Gen2) | inherits Cloud Run characteristics | Min instances |
| Azure Functions | 10 minutes (Consumption), longer on Premium/Dedicated | Pre-warmed instances (Premium) |

### 2.5 Comparison table

| Factor | AWS Lambda | GCP Cloud Functions / Cloud Run | Azure Functions |
|---|---|---|---|
| Model | FaaS | FaaS / serverless containers | FaaS |
| Max duration | 15 min | Functions via Cloud Run; Cloud Run HTTP 60 min | 10 min (Consumption), longer in Premium |
| Cold start mitigation | Provisioned Concurrency, SnapStart | Min instances (Cloud Run) | Pre-warmed instances (Premium) |
| Triggers | Broad AWS events + HTTP | HTTP, Pub/Sub, Eventarc | HTTP, Timer, Queues, Event Hub |
| Networking | VPC integration | VPC/serverless VPC access | VNet integration |
| Pricing | Requests + GB-s + optional provisioned | Requests + time/CPU/mem | Requests + time; Premium warm cost |

### 2.6 Analysis — best fit for REST API backend

- **Low latency, AWS-native:** Lambda with Provisioned Concurrency provides predictable start-up at extra cost.  
- **Containerized HTTP with more control:** Cloud Run offers standard containers, long HTTP timeouts, and min instances.  
- **Azure-native with stable latency:** Functions on Premium plan for pre-warmed workers and VNet.

### 2.7 Pros & Cons of serverless

**Pros**
- No server management, automatic scaling, pay-for-use, scale-to-zero.

**Cons**
- Cold starts, per-platform limits, tuning for latency, possible vendor coupling.