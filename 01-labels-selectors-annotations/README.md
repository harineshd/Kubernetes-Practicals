# 🏷️ Topic 01: Kubernetes Labels, Selectors & Annotations

## 📌 Overview & Concept Fundamentals

In Kubernetes:
- **Labels:** Key-value pairs attached to Kubernetes objects (like Pods, Services, Deployments). They are used for **identifying, grouping, and selecting** subsets of objects.
- **Selectors:** Queries used by administrators (via `kubectl`) and controllers (Services, ReplicaSets, Deployments) to filter resources based on labels.
  - **Equality-based Selectors:** `=`, `==`, `!=`
  - **Set-based Selectors:** `in`, `notin`, `exists` (key only), `does not exist` (`!key`)
- **Annotations:** Non-identifying metadata used to store build info, release notes, timestamps, tooling configs, and audit logs (not used for selection/filtering).

---

## 📐 Architecture & Pod Distribution

```
                                      +------------------------------------+
                                      |     Kind Cluster: 'revise'         |
                                      +------------------------------------+
                                                        |
                         +------------------------------+------------------------------+
                         |                                                             |
            +---------------------------+                                 +---------------------------+
            |      revise-worker        |                                 |      revise-worker2       |
            +---------------------------+                                 +---------------------------+
            | [Pod] frontend-2          |                                 | [Pod] frontend-1          |
            |   app=medtrack            |                                 |   app=medtrack            |
            |   tier=frontend           |                                 |   tier=frontend           |
            |   environment=dev         |                                 |   environment=dev         |
            |                           |                                 |   version=v1  <-- Added   |
            | [Pod] backend-2           |                                 |                           |
            |   app=medtrack            |                                 | [Pod] backend-1           |
            |   tier=backend            |                                 |   app=medtrack            |
            |   environment=prod        |                                 |   tier=backend            |
            |                           |                                 |   environment=dev         |
            +---------------------------+                                 |   version=v2  <-- Added   |
                                                                          |                           |
                                                                          | [Pod] database-1          |
                                                                          |   app=medtrack            |
                                                                          |   tier=database           |
                                                                          |   environment=prod        |
                                                                          +---------------------------+
```

---

## 📋 Initial Cluster State Matrix

| Pod Name | App | Tier | Environment | Version | Node | Status |
|---|---|---|---|---|---|---|
| `frontend-1` | `medtrack` | `frontend` | `dev` | `v1` | `revise-worker2` | `Running` |
| `frontend-2` | `medtrack` | `frontend` | `dev` | *(none)* | `revise-worker` | `Running` |
| `backend-1` | `medtrack` | `backend` | `dev` | `v2` | `revise-worker2` | `Running` |
| `backend-2` | `medtrack` | `backend` | `prod` | *(none)* | `revise-worker` | `Running` |
| `database-1` | `medtrack` | `database` | `prod` | *(none)* | `revise-worker2` | `Running` |

---

## 🚀 Scenario 1: Easy – Basic Labeling & Equality Filtering

### Task 1: Add `version=v1` label to `frontend-1`
```bash
kubectl label pod frontend-1 version=v1
```
*Verification:*
```bash
kubectl get pod frontend-1 --show-labels
```

---

### Task 2: Add `version=v2` label to `backend-1`
```bash
kubectl label pod backend-1 version=v2
```
*Verification:*
```bash
kubectl get pod backend-1 --show-labels
```

---

### Task 3: Display all Pods with `app=medtrack`
```bash
kubectl get pods -l app=medtrack
```
*Expected Output:* All 5 pods (`frontend-1`, `frontend-2`, `backend-1`, `backend-2`, `database-1`).

---

### Task 4: Display only `tier=backend` Pods
```bash
kubectl get pods -l tier=backend
```
*Expected Output:* `backend-1`, `backend-2`.

---

### Task 5: Display only `environment=dev` Pods
```bash
kubectl get pods -l environment=dev
```
*Expected Output:* `frontend-1`, `frontend-2`, `backend-1`.

---

### Task 6: Display Pods matching BOTH `tier=frontend` AND `environment=dev`
```bash
kubectl get pods -l tier=frontend,environment=dev
```
*Expected Output:* `frontend-1`, `frontend-2`.

---

### Task 7: Identify Pod with `version=v1`
```bash
kubectl get pods -l version=v1
```
*Expected Output:* `frontend-1`.

---

### Task 8: Display all Pods showing their labels
```bash
kubectl get pods --show-labels
```

---

## ⚡ Scenario 2: Intermediate – Set-Based Filtering & Multi-Label Logic

Set-based requirements allow rich queries using `in`, `notin`, and key presence:

### 1. Assign Production Labels
```bash
# Add or overwrite labels using --overwrite flag
kubectl label pod frontend-1 frontend-2 app=medtrack tier=frontend environment=prod --overwrite
kubectl label pod backend-1 backend-2 tier=backend --overwrite
kubectl label pod database-1 tier=database --overwrite
```

### 2. Query: `tier in (frontend, backend)`
```bash
kubectl get pods -l 'tier in (frontend, backend)'
```

### 3. Query: `environment != dev`
```bash
kubectl get pods -l 'environment != dev'
# Or set-based:
kubectl get pods -l 'environment notin (dev)'
```

### 4. Query: Pods that have the `tier` label (Key exists)
```bash
kubectl get pods -l 'tier'
```

### 5. Query: Pods that DO NOT have the `tier` label (Key does not exist)
```bash
kubectl get pods -l '!tier'
```

---

## 🛠️ Scenario 3: Break & Fix – Service Selector Mismatch

### Problem:
A Service is created with:
```yaml
selector:
  app: medtrack
  tier: backend
```
However, the Backend Pods have:
```yaml
labels:
  app: medtrack
  tier: api
```
As a result:
```bash
kubectl get endpoints backend-service
# Output: ENDPOINTS: <none>
```

### Root Cause:
Service endpoints controller uses exact label matching. `tier: backend` does not match `tier: api`.

### Solution Options:
1. **Option A (Fix Service):** Update Service `spec.selector.tier: api`
2. **Option B (Fix Pods - Recommended in standard architecture):** Re-label Pods:
```bash
kubectl label pod backend-1 backend-2 tier=backend --overwrite
```
3. **Verify Endpoints:**
```bash
kubectl get endpoints <service-name>
```

---

## 📝 Scenario 4: Annotations (Metadata & Auditing)

Annotations are used for system information, audit logs, and tool integrations (e.g., Helm, Ingress, Istio).

### Task 1: Add annotations
```bash
# Add owner annotation
kubectl annotate pod frontend-1 owner="Frontend Team"

# Add git commit SHA
kubectl annotate pod backend-1 git-commit="a1b2c3d"

# Add database description
kubectl annotate pod database-1 description="MedTrack PostgreSQL Database"
```

### Task 2: Update an existing annotation (`--overwrite`)
```bash
kubectl annotate pod backend-1 git-commit="f4e5d6c" --overwrite
```

### Task 3: Inspect Pod annotations
```bash
kubectl describe pod backend-1 | Select-String -Pattern "Annotations" -Context 0,5
# Or using jsonpath:
kubectl get pod backend-1 -o jsonpath='{.metadata.annotations}'
```

### Task 4: Remove an annotation (suffix with `-`)
```bash
kubectl annotate pod frontend-1 owner-
```
