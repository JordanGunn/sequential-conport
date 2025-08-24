# brief.md — Spexi Technical Interview Prep

---

------------
title: Spexi Senior Backend Prep — Brief
owner: Jordan
version: 2.1
------------

## Purpose

To prepare effectively for the **30-minute technical screen** with Spexi Geospatial’s CTO. This preparation emphasizes **Python (FastAPI)**, **geospatial APIs**, and **AWS vocabulary** translated from Azure experience. The objective is to demonstrate not only coding ability but also **senior-level reasoning in scalability, cloud design, and data architecture**.

---

## Additional information
For more detailed information, review documents linked in: [.index.md](../../../docs/.index.md)

---

## Company Snapshot

* **Spexi**: Drone-powered geospatial imagery company, founded 2017.
* **Mission**: Empower humanity to make better decisions about planet Earth — “It’s time Earth got an update.”
* **Platform**: “Aerial Intelligence Layer,” world’s largest drone imagery network (4M+ acres, 130k+ flights, sub-3 cm resolution).
* **Products**:

  * **World Viewer** (Google Street View in the sky).
  * **APIs** (OGC-aligned, rich metadata).
  * **Flight plans** (oblique + vertical) → outputs: orthomosaics, 3-D point clouds, Gaussian splats.
* **Differentiators**: Faster, cheaper, more precise than satellites/planes.
* **Culture**: Remote-friendly, inclusive, sustainability-focused, with community impact.

**Interview relevance:** APIs and backend infrastructure are core to their product. Expect geospatial-domain prompts.

---

## Role Snapshot

**Senior Software Engineer (Backend)**

* Build scalable, secure, extensible geospatial data marketplace.
* Develop reliable APIs and services for imagery delivery.
* Collaborate across frontend/data, lead architecture improvements.
* Mentor engineers, improve processes, maintain code quality.
* **Required strengths**: Python, REST APIs, databases, cloud infra (AWS), geospatial data.
* **Bonus**: GIS familiarity (ESRI), 3D math, OSS contributions.

**Interview relevance:** You’re expected to show both **coding fluency** and **architectural maturity**.

---

## Interview Details

* **Stage**: Technical screen (CTO).
* **Length**: 30 minutes.
* **Format**: Virtual, screen share, code editor, solve problem in **Python**.
* **Focus**: Implement a small backend service + reasoning on scalability, data handling, and geospatial challenges.

---

## Likely Scenarios

1. **REST API Implementation (FastAPI)**

   * Most probable.
   * Example: `POST /datasets`, `GET /datasets/{id}`, `GET /datasets?bbox=&date=`.
   * Watch for: semantic endpoints, validation, pagination, bbox filters.

2. **Data Transformation / Handling**

   * Parse GeoJSON or imagery metadata, return structured response.
   * Libraries: `geojson`, `geopandas`, `shapely`.

3. **System Design Discussion**

   * “Design a scalable service to deliver drone imagery.”
   * Storage in S3/Blob, metadata in PostGIS, caching/CDN, K8s autoscaling.

4. **Cloud Problem (AWS Vocabulary)**

   * “How to scale to 100× traffic?”
   * Translate Azure → AWS confidently: Blob→S3, AKS→EKS, Entra→IAM, Bicep→CDK.

---

## Known Strengths

* **Deep geospatial background**: LiDAR, orthophotos, imagery pipelines, QA/QC.
* **Distributed systems**: Kubernetes, Dask, operator/CRD designs.
* **API design**: Clean, semantic, domain-driven REST APIs.
* **CI/CD & GitOps**: Proven track record with GitLab + ArgoCD.

## Potential Gaps

* Drone flight-planning metadata (waypoints, telemetry).
* AWS hands-on (strength is translation from Azure/GCP).
* ⚠️ Mitigation: Frame gaps as “extend metadata models” and emphasize **patterns over providers**.

---

## Prep Strategy

### Coding (Primary)

* Write and rehearse a **FastAPI microservice**:

  * CRUD dataset metadata.
  * Bbox/time query filter.
  * Pagination and error handling.

### Geospatial Python

* Review `geopandas`, `shapely`, `rasterio`.
* Be fluent with bounding box filters, CRS basics, metadata extraction.

### AWS ↔ Azure Mapping

* Prepare translation sheet: Blob→S3, AKS→EKS, Entra→IAM, Bicep→CDK.
* Be ready to restate Azure stories in AWS terms.

### Architecture Narrative

* 5-bullet pitch: “store imagery in S3, index metadata in PostGIS, serve via API, cache with CDN, autoscale with K8s.”

### Leadership & Mentorship

* Prepare **1 anecdote** of mentoring a teammate or influencing architecture.

---

## Prep Timeline

* Draft FastAPI skeleton.
* Start AWS translation sheet.
* bbox/date filter practice.
* Afternoon: Python geospatial libraries.
* Evening: 5-min system design rehearsal.
* Warm-up: code one FastAPI endpoint from scratch.
* Review AWS translations.
* Skim CRD Lifecycle doc for operator pattern reference.

---

## Success Criteria

* Deliver working FastAPI endpoint in <10 minutes.
* Explain scaling strategy in <5 minutes.
* Confidently map Azure → AWS primitives.
* Demonstrate geospatial domain knowledge in API design.
* End interview showing **senior-level thinking** (mentorship, architecture, trade-offs).

---

## AWS ↔ Azure Translation Cheat Sheet (Extended: Storage & Kubernetes)

| Concept / Area              | Azure Equivalent                                                                              | AWS Equivalent                                           | Notes for Interview                                                                            |
| --------------------------- | --------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Object Storage**          | Blob Storage                                                                                  | S3                                                       | Same pattern: buckets, lifecycle policies, versioning. Use for large imagery (COGs).           |
| **Block Storage**           | Managed Disks (Premium/Standard SSD, HDD)                                                     | EBS (Elastic Block Store)                                | Used for K8s PVs; bound to single AZ/VM.                                                       |
| **File / Shared Storage**   | Azure Files (SMB/NFS)                                                                         | EFS (Elastic File System)                                | Shared POSIX file system, often mounted by pods for shared config/state.                       |
| **Cold / Archival Storage** | Azure Archive Storage                                                                         | S3 Glacier / Glacier Deep Archive                        | Long-term storage of rarely accessed data; stress cost efficiency.                             |
| **Kubernetes Service**      | AKS (Azure Kubernetes Service)                                                                | EKS (Elastic Kubernetes Service)                         | Both run upstream K8s, managed control plane. Nodes are VM-based (VMSS vs EC2 ASG).            |
| **Ingress / Load Balancer** | Azure Load Balancer + App Gateway                                                             | ELB (Classic/ALB/NLB)                                    | AWS has more granular LB types (ALB for HTTP, NLB for TCP).                                    |
| **Service Mesh**            | Open Service Mesh add-on (optional)                                                           | App Mesh (native AWS service mesh)                       | Rarely core to interviews, but note AWS has a first-party option.                              |
| **Storage Classes (K8s)**   | AzureDisk, AzureFile                                                                          | gp2/gp3 (EBS), io2, sc1, st1 (EBS types); EFS            | AWS exposes more SC options directly; Azure is simpler but annotation-driven.                  |
| **Identity Integration**    | Managed Identity / Workload Identity                                                          | IAM Roles for Service Accounts (IRSA)                    | Azure: annotations + AAD pod identities. AWS: IRSA integrates IAM directly into pod SA tokens. |
| **K8s Config Extensions**   | Azure provides pre-installed controllers for CSI (disk/file), CNI (network), and pod identity | AWS provides EBS/EFS CSI drivers, VPC CNI, IRSA natively | Both rely on annotations & labels, but AWS’s IRSA is widely used and more “official.”          |
| **Networking (Pods)**       | Azure CNI (VNet IPs per pod)                                                                  | AWS VPC CNI (ENI/IP per pod)                             | Both allow IP-per-pod networking, but quotas/scale differ.                                     |
| **Registry**                | Azure Container Registry (ACR)                                                                | Elastic Container Registry (ECR)                         | Both integrate tightly with IAM/Entra for CI/CD.                                               |

---

### Teaching Angle in the Interview

When asked about Kubernetes differences:

* **Emphasize the pattern**: “AKS and EKS are just managed control planes. Both are upstream K8s; the differences are mostly in how storage and identity are integrated.”
* **Identity**: “On Azure we used workload identities via annotations and Entra. In AWS, that maps to IRSA, which binds IAM roles directly to pod service accounts.”
* **Storage**: “Azure mostly abstracts storage classes to AzureDisk/AzureFile, while AWS exposes more granular SCs (gp2/gp3/io2). But the pattern is the same: CSI drivers with labels/annotations for PV/PVC provisioning.”

---

👉 This way, if Peter digs into **Kubernetes idioms**, you can show you understand both **vendor quirks** and the **common control loop pattern** (CSI, CNI, CRDs, annotations).

---

## Practice Project

Great instinct to pause and **re-tailor the practice project directly to the role**. Let’s ground this in what the job description explicitly emphasizes and order practice areas by **priority (inverse to time available)**:

---

# practice\_project.md — Spexi Prep API Project (Role-Tailored)

## Goal

Prepare for the 30-minute technical screen by practicing exactly what the role demands:

* **Python REST API design and implementation**
* **Database & data modeling (PostGIS awareness)**
* **Cloud scaling awareness (AWS vocabulary, Azure experience mapped)**
* **System design narratives for geospatial marketplaces**
* **Collaboration & leadership framing**

---

## Priorities (highest → lowest)

1. **Python / REST APIs**

   * Certain to be tested (language clarified during recruiter screen).
   * Focus: FastAPI idioms, Pydantic models, request/response validation, pagination, error handling.

2. **Database Systems / Geospatial Data**

   * Role explicitly calls for database + geospatial data expertise.
   * Practice PostGIS schema design + GeoJSON/bbox filters.

3. **Cloud Infrastructure / AWS Translation**

   * Must demonstrate comfort mapping Azure → AWS.
   * Emphasize vendor-agnostic patterns: object storage, IAM, managed K8s.

4. **System Design for Scaling**

   * Responsibility: design scalable geospatial marketplace.
   * Practice quick 3–5 bullet explanations (store → index → serve → scale → secure).

5. **Collaboration / Leadership**

   * Responsibility: mentorship, process improvement.
   * Have 1 anecdote ready (mentoring, architectural pushback, GitOps improvement).

6. **Bonus / Edge** (low likelihood in coding screen)

   * ESRI ecosystem familiarity.
   * 3D math (LiDAR/point clouds).
   * OSS contributions.

---

## Practice Tasks

### 1. FastAPI Project Setup

* Scaffold project (`app/routers`, `app/models`, `main.py`).
* Add `/health` endpoint.
* Configure `uvicorn`.

### 2. Pydantic Models

* Define request/response payloads:

  * `DatasetCreate`: name, bbox, acquired\_at, description.
  * `DatasetOut`: id, name, acquired\_at, description.
* Validate bbox as 4 floats, acquired\_at as ISO8601.
* Add nested example (e.g. dataset contains images).

### 3. REST Endpoints

* CRUD: `POST /datasets`, `GET /datasets/{id}`, `PUT`, `DELETE`.
* Query: `GET /datasets?bbox=...&date=...&limit=...&offset=...`.
* Return JSON via Pydantic models.
* Implement pagination + validation errors.

### 4. Database & PostGIS

* Draft schema:

```sql
CREATE TABLE datasets (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    geom GEOMETRY(POLYGON, 4326) NOT NULL,
    acquired_at TIMESTAMP NOT NULL,
    description TEXT
);
CREATE INDEX idx_datasets_geom ON datasets USING GIST (geom);
```

* Practice queries: `ST_Intersects`, `ST_Within`.
* Optional: spin up Dockerized PostGIS + seed sample data.

### 5. Cloud Mapping (Azure → AWS)

* Object storage: Blob ↔ S3.
* K8s: AKS ↔ EKS.
* Identity: Entra ↔ IAM/IRSA.
* IaC: Bicep ↔ CDK/CloudFormation.
* Storage classes: AzureDisk/AzureFile ↔ EBS/EFS.
* Practice one-liner mappings verbally.

### 6. System Design Mini-Rehearsal

* Prompt: “Design a service to serve drone imagery datasets at scale.”
* Outline:

  1. Store imagery in S3 (COG format).
  2. Index metadata in PostGIS.
  3. API layer for CRUD + bbox/time queries.
  4. Cache frequent queries (Redis/CDN).
  5. Scale horizontally with K8s autoscaling.
* Keep to <3 minutes.

### 7. Collaboration & Leadership

* Prepare 1–2 examples:

  * Mentorship (helping junior engineer adopt GitOps best practices).
  * Architectural advocacy (pushing for operator/CRD lifecycle pattern).

### 8. Extras (Optional, if time)

* Add caching middleware (Redis).
* Unit test one endpoint with pytest.
* Generate OpenAPI docs (`/docs`).
* Sketch a quick architecture diagram (client → API → PostGIS/S3 → CDN).

---

## STAC Consideration

* The role description **does not explicitly mention STAC**.
* They do emphasize **standardized APIs, geospatial data, and integration**.
* STAC is widely used in the EO/geospatial world, so awareness could score bonus points if mentioned naturally.
* Suggested position:

  > “I’m aware of STAC for dataset discovery and metadata standardization. In this API, we could expose dataset metadata in a STAC-like format for interoperability, though for the exercise I’d stick to simpler Pydantic models.”

---

✅ This structure we're strongest where it matters most (Python REST + PostGIS + AWS translation), while still prepared to mention scaling, leadership, and optional standards like STAC.

---