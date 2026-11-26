# 📄 Complete YAML Guide & Kubernetes Manifest Architecture

Welcome to the **Kubernetes Manifests & YAML Architecture Guide**. Before writing and troubleshooting Kubernetes configurations, mastering YAML (YAML Ain't Markup Language) syntax is an essential prerequisite. Understanding YAML data structures prevents syntax failures, schema validation errors, and runtime misconfigurations.

---

## 📑 Table of Contents
1. [What is YAML?](#1-what-is-yaml)
2. [Core Syntax Rules](#2-core-syntax-rules)
3. [Data Types in YAML](#3-data-types-in-yaml)
4. [Lists and Arrays in YAML](#4-lists-and-arrays-in-yaml)
5. [Dictionaries and Maps in YAML](#5-dictionaries-and-maps-in-yaml)
6. [Nested Dictionaries and Maps](#6-nested-dictionaries-and-maps)
7. [Lists of Dictionaries (Crucial for Kubernetes)](#7-lists-of-dictionaries-crucial-for-kubernetes)
8. [Multiline Strings: Literal Block (`|`) vs Folded Block (`>`)](#8-multiline-strings-literal-block--vs-folded-block-)
9. [Empty Collections (`[]` and `{}`)](#9-empty-collections--and-)
10. [Special Characters & Quoting Rules](#10-special-characters--quoting-rules)
11. [YAML Anchors (`&`), Aliases (`*`), and Merges (`<<`)](#11-yaml-anchors--aliases--and-merges-)
12. [End-to-End Kubernetes Manifest Dissection](#12-end-to-end-kubernetes-manifest-dissection)
13. [Top Common Mistakes & Best Practices](#13-top-common-mistakes--best-practices)
14. [Hands-On Practice Exercises & Solutions](#14-hands-on-practice-exercises--solutions)
15. [Comprehensive YAML Revision Cheat Sheet](#15-comprehensive-yaml-revision-cheat-sheet)
16. [Core Kubernetes Manifest Reference Files](#16-core-kubernetes-manifest-reference-files)

---

## 1. What is YAML?

**YAML** stands for **YAML Ain't Markup Language**. It is a human-readable, cross-language data serialization standard commonly used for configuration files.

### Primary Use Cases in DevOps:
- **Kubernetes Manifests:** Pods, Deployments, Services, Ingress, ConfigMaps, Secrets, RBAC.
- **Infrastructure as Code (IaC):** Ansible Playbooks, CloudFormation templates.
- **Container Orchestration:** Docker Compose definitions.
- **CI/CD Pipelines:** GitHub Actions workflows, GitLab CI, Azure DevOps pipelines.
- **Application Configuration:** Helm charts, Spring Boot, Prometheus alerting rules.

### Core Building Blocks:
```yaml
# 1. String
name: medtrack

# 2. Number / Integer
replicas: 3

# 3. Boolean
enabled: true

# 4. List / Sequence
ports: [80, 443]

# 5. Dictionary / Map
app:
  name: medtrack
  tier: backend
```

---

## 2. Core Syntax Rules

### Rule 1: Key-Value Pairs
Data is structured using colon-separated key-value pairs.
```yaml
key: value
```
> **Crucial Rule:** Always place a space **after** the colon (`:`).
> - ✅ Correct: `image: nginx:alpine`
> - ❌ Invalid: `image:nginx:alpine` (YAML parser treats this as a single string, causing syntax errors).

### Rule 2: Indentation with Spaces (Never Tabs!)
YAML strictly relies on indentation to represent parent-child relationships and scope.
- **Rule:** Use **spaces only**. **Never use tab characters (`\t`)**. Tabs cause immediate parser rejections.
- **Standard Convention:** Use **2 spaces** per indentation level.

```yaml
# ✅ Correct (2-space hierarchy)
metadata:
  name: medtrack
  labels:
    app: medtrack
    tier: backend

# ❌ Incorrect (Inconsistent indentation)
metadata:
 name: medtrack
   labels:
  app: medtrack
```

### Rule 3: Comments
Comments begin with a hash character (`#`) and continue to the end of the line.
```yaml
# Define the deployment replicas
replicas: 3  # High availability threshold
```

### Rule 4: Multiple Documents in a Single File (`---`)
Use three hyphens (`---`) to define multiple distinct Kubernetes resource definitions in a single YAML file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
---
apiVersion: v1
kind: Pod
metadata:
  name: backend
```

---

## 3. Data Types in YAML

### 3.1 String
Textual data representation. Strings can be written with single quotes, double quotes, or unquoted:
```yaml
name: medtrack
image: nginx:alpine
description: "MedTrack Backend API"
```
#### When MUST you quote strings?
1. Values containing colons, curly braces, square brackets, or asterisks:
   ```yaml
   schedule: "0 2 * * *"
   message: "Status: Success"
   ```
2. Values starting with leading zeros (which parsers might convert to octal or integers):
   ```yaml
   zipcode: "07001"
   version: "1.0"
   ```
3. Boolean-like text strings (`"true"`, `"false"`, `"yes"`, `"no"`):
   ```yaml
   environment_override: "true"
   ```

### 3.2 Integer & Float (Numbers)
Whole numbers and decimal values:
```yaml
replicas: 3
containerPort: 8080
initialDelaySeconds: 10
cpu_fraction: 0.5
percentage: 99.9
```
*Note on Kubernetes Resource Quantities:* Memory and CPU limits in Kubernetes often append units as strings:
```yaml
resources:
  requests:
    cpu: "250m"       # 250 milli-CPUs (0.25 core)
    memory: "256Mi"   # 256 Mebibytes
```

### 3.3 Boolean
Represents binary state (`true` or `false`).
```yaml
enabled: true
debug: false
privileged: false
```
> **Best Practice:** Always use lowercase `true` and `false` for consistency with JSON and Kubernetes API schemas.

### 3.4 Null
Indicates an explicitly empty or unassigned value:
```yaml
database: null
password: ~
```
Both `null` and the tilde (`~`) represent null values in YAML.

---

## 4. Lists and Arrays in YAML

A list stores an ordered collection of values under a parent key.

### Block Style (Standard in Kubernetes)
Prepend each list item with a hyphen (`-`) followed by a single space:
```yaml
ports:
  - 80
  - 443
  - 8080

environments:
  - dev
  - staging
  - prod
```

### Flow Style (Inline JSON-compatible)
Enclosed within square brackets (`[` and `]`):
```yaml
ports: [80, 443, 8080]
environments: [dev, staging, prod]
```

---

## 5. Dictionaries and Maps in YAML

A Dictionary (Map) is an unordered collection of key-value associations.

### Block Style
```yaml
database:
  host: "postgres-cluster.local"
  port: 5432
  user: "dbadmin"
```

### Flow Style (Inline)
Enclosed within curly braces (`{` and `}`):
```yaml
database: {host: "postgres-cluster.local", port: 5432, user: "dbadmin"}
```

---

## 6. Nested Dictionaries and Maps

A dictionary nested inside another dictionary establishes multi-level hierarchy:
```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```
Hierarchy Breakdown:
- Root: `resources` (Map)
  - Child 1: `requests` (Map)
    - Sub-key: `cpu: "100m"`
    - Sub-key: `memory: "128Mi"`
  - Child 2: `limits` (Map)
    - Sub-key: `cpu: "500m"`
    - Sub-key: `memory: "512Mi"`

---

## 7. Lists of Dictionaries (Crucial for Kubernetes)

This is the most common structural pattern in Kubernetes manifests (e.g., `containers`, `env`, `ports`, `volumeMounts`, `volumes`).

```yaml
containers:
  - name: frontend-container
    image: nginx:alpine
    ports:
      - containerPort: 80
        protocol: TCP

  - name: backend-container
    image: node:20-alpine
    ports:
      - containerPort: 5000
        protocol: TCP
```

### How to analyze this structure:
| YAML Element | Data Structure Representation |
| :--- | :--- |
| `containers:` | **List** container parent key |
| `- name: ...` | **First Item** in the list, containing a **Map** |
| `ports:` | **Nested List** inside the first container map |
| `- containerPort: 80` | **First Item** in the nested ports list |

### Environment Variables Pattern:
```yaml
env:
  - name: DATABASE_URL
    value: "postgresql://db.medtrack.internal:5432/meddb"
  - name: LOG_LEVEL
    value: "info"
```

---

## 8. Multiline Strings: Literal Block (`|`) vs Folded Block (`>`)

DevOps configurations frequently store shell scripts, certificates, and multi-line descriptions inside YAML files.

### A. Literal Block Scalar (`|`) — Preserves Newlines
Maintains every newline and space verbatim. Perfect for embedded scripts and configurations:
```yaml
script: |
  #!/bin/bash
  echo "Initializing MedTrack Cluster..."
  systemctl start postgresql
  nginx -g 'daemon off;'
```
**Rendered Output:**
```text
#!/bin/bash
echo "Initializing MedTrack Cluster..."
systemctl start postgresql
nginx -g 'daemon off;'
```

### B. Folded Block Scalar (`>`) — Folds Newlines into Single Line
Folds newlines into spaces, turning paragraphs into continuous single-line strings:
```yaml
description: >
  MedTrack is an enterprise-grade healthcare
  application engineered to manage real-time
  clinic schedules and patient electronic records.
```
**Rendered Output:**
```text
MedTrack is an enterprise-grade healthcare application engineered to manage real-time clinic schedules and patient electronic records.
```

### Comparison Matrix:
| Scalar Operator | Newlines Handled | Best Use Case |
| :--- | :--- | :--- |
| **`\|` (Literal)** | **Preserved exactly** | Shell scripts, ConfigMaps, SSL Certificates |
| **`>` (Folded)** | **Joined with single spaces** | Long documentation notes, git commit explanations |

---

## 9. Empty Collections (`[]` and `{}`)

When a schema requires an explicitly empty collection rather than `null`:
- **Empty List:** `[]`
  ```yaml
  ports: []
  args: []
  ```
- **Empty Map:** `{}`
  ```yaml
  labels: {}
  annotations: {}
  ```

---

## 10. Special Characters & Quoting Rules

| Symbol | Meaning | Example |
| :---: | :--- | :--- |
| `:` | Key-Value delimiter | `tier: backend` |
| `-` | List element prefix | `- name: web` |
| `#` | Single-line comment | `# Production settings` |
| `---` | Multi-document separator | `---` |
| `\|` | Literal multiline string | `script: \|` |
| `>` | Folded multiline string | `notes: >` |
| `[]` | Inline List syntax | `[dev, prod]` |
| `{}` | Inline Map syntax | `{app: web, tier: fe}` |
| `&` | Anchor definition | `&common-labels` |
| `*` | Alias reference | `*common-labels` |
| `<<` | Map merge key | `<<: *common-labels` |

---

## 11. YAML Anchors (`&`), Aliases (`*`), and Merges (`<<`)

To eliminate code duplication, YAML natively supports defining reusable blocks (Anchors) and referencing them (Aliases).

```yaml
# Step 1: Define base anchor with '&'
common-metadata: &common-labels
  app: medtrack
  project: healthcare-platform
  managed-by: terraform

# Step 2: Merge using '<<' and dereference with '*'
frontend:
  labels:
    <<: *common-labels
    tier: frontend

backend:
  labels:
    <<: *common-labels
    tier: backend
```

> **Kubernetes Note:** Kubernetes API Server processes raw JSON/YAML. Anchors work during client-side parsing (e.g., in Helm or kubectl), but standard Kubernetes manifests generally favor explicit definitions to maintain readability and GitOps diff transparency.

---

## 12. End-to-End Kubernetes Manifest Dissection

Let us analyze a real-world Kubernetes Pod manifest and inspect every single field's data structure:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: medtrack-backend
  namespace: default
  labels:
    app: medtrack
    tier: backend
    environment: dev
  annotations:
    build.commit: "f9e8d7c"
spec:
  restartPolicy: Always
  containers:
    - name: api-server
      image: nginx:alpine
      ports:
        - name: http
          containerPort: 80
      env:
        - name: APP_ENV
          value: "development"
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "512Mi"
      command:
        - /bin/sh
        - -c
      args:
        - |
          echo "Starting MedTrack API Server..."
          nginx -g 'daemon off;'
```

### Structural Dissection Table:
| Field Path | Data Structure | Explanation |
| :--- | :--- | :--- |
| `apiVersion` | String | Target Kubernetes API group and version |
| `kind` | String | Kubernetes resource type |
| `metadata` | Map (Dictionary) | Object identification and categorization |
| `metadata.name` | String | Unique identifier within the namespace |
| `metadata.labels` | Map (Key-Value) | Indexed key-value pairs for selectors |
| `metadata.annotations`| Map (Key-Value) | Non-identifying operational metadata |
| `spec` | Map (Dictionary) | Desired functional state of the resource |
| `spec.restartPolicy` | String | Container restart criteria (`Always`, `OnFailure`, `Never`) |
| `spec.containers` | **List of Maps** | Array of container specifications |
| `spec.containers[0].ports` | **List of Maps** | Network ports published by the container |
| `spec.containers[0].env` | **List of Maps** | Environment variables passed to container |
| `spec.containers[0].resources` | Nested Map | Compute constraints (`requests` and `limits`) |
| `spec.containers[0].command` | List of Strings | Container entrypoint override |
| `spec.containers[0].args` | List of Strings (`\|`) | Arguments containing a literal multiline shell script |

---

## 13. Top Common Mistakes & Best Practices

| # | Common Mistake | Why it Fails | Correct Solution |
|---|---|---|---|
| 1 | **Using Tabs instead of Spaces** | YAML specifications strictly disallow `\t`. | Configure editor to replace tabs with 2 spaces. |
| 2 | **Missing space after colon (`:`)** | `key:value` is parsed as a single string. | Always write `key: value`. |
| 3 | **Unquoted special characters** | `schedule: 0 2 * * *` breaks on asterisk. | Use double quotes: `"0 2 * * *"`. |
| 4 | **Inconsistent indentation** | Siblings at different indentation breaks hierarchy. | Ensure siblings align vertically at exact space count. |
| 5 | **Missing `-` in Lists** | Causes the parser to expect a Map instead of a List item. | Place `-` at the start of each list element. |
| 6 | **ReplicaSet / Deployment Selector Mismatch** | `spec.selector.matchLabels` differs from `spec.template.metadata.labels`. | API server rejects manifest with an immutable selector error. |

---

## 14. Hands-On Practice Exercises & Solutions

### Exercise 1: Build a Label Map
**Requirement:** Create a YAML map for an application named `medtrack`, tier `backend`, and environment `dev`.
<details>
<summary>👉 View Solution</summary>

```yaml
labels:
  app: medtrack
  tier: backend
  environment: dev
```
</details>

---

### Exercise 2: Build an Environment Tier List
**Requirement:** Create a list containing `dev`, `staging`, and `prod`.
<details>
<summary>👉 View Solution</summary>

```yaml
# Block Style
environments:
  - dev
  - staging
  - prod

# Flow Style
environments: [dev, staging, prod]
```
</details>

---

### Exercise 3: Database Connection Map
**Requirement:** Create a map defining `host`, `port` (number), `username`, and `password`.
<details>
<summary>👉 View Solution</summary>

```yaml
database:
  host: "db.medtrack.internal"
  port: 5432
  username: "admin"
  password: "SecurePassword123!"
```
</details>

---

### Exercise 4: Multi-Container List
**Requirement:** Define a list of two containers: `frontend` using `nginx:alpine` and `backend` using `node:20-alpine`.
<details>
<summary>👉 View Solution</summary>

```yaml
containers:
  - name: frontend
    image: nginx:alpine
  - name: backend
    image: node:20-alpine
```
</details>

---

### Exercise 5: Nested Resources Hierarchy
**Requirement:** Create a nested resource map with `requests` (cpu: `100m`, memory: `128Mi`) and `limits` (cpu: `500m`, memory: `512Mi`).
<details>
<summary>👉 View Solution</summary>

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```
</details>

---

## 15. Comprehensive YAML Revision Cheat Sheet

| YAML Syntax Construct | Structural Type | Practical Kubernetes Example |
| :--- | :--- | :--- |
| `key: value` | Key-Value Pair | `image: nginx:alpine` |
| `key: "quoted text"` | Explicit String | `version: "1.0"` |
| `key: 42` | Integer (Number) | `replicas: 3` |
| `key: 0.75` | Float (Decimal) | `cpu: 0.5` |
| `key: true` | Boolean | `readOnlyRootFilesystem: true` |
| `key: null` or `~` | Null / Empty | `nodeName: null` |
| `- item` | List Item (Block) | `args: ["--port=80"]` |
| `[item1, item2]` | List (Inline) | `command: ["/bin/sh", "-c"]` |
| `key: {a: 1, b: 2}` | Map (Inline) | `limits: {cpu: "1", memory: "1Gi"}` |
| `parent:`<br>&nbsp;&nbsp;`child: val` | Nested Map | `spec.template.spec` |
| `- name: a`<br>&nbsp;&nbsp;`image: b` | List of Maps | `spec.containers` |
| `\|` | Literal String (Preserve `\n`) | Multi-line shell scripts / ConfigMaps |
| `>` | Folded String (Replace `\n` with space) | Documentation, descriptions |
| `# text` | Comment | `# Audit annotation` |
| `---` | Multi-Document Delimiter | Separates Pod and Service in one file |
| `[]` | Empty List | `ports: []` |
| `{}` | Empty Dictionary / Map | `annotations: {}` |

---

## 16. Core Kubernetes Manifest Reference Files

In this folder, production-grade reference manifests are available for immediate inspection:
- 📄 [`pod.yaml`](./pod.yaml) — Standalone Pod specification with health probes, environment variables, and resources.
- 📄 [`replicaset.yaml`](./replicaset.yaml) — ReplicaSet controller definition illustrating selectors and self-healing templates.
- 📄 [`deployment.yaml`](./deployment.yaml) — Production Deployment with zero-downtime RollingUpdate strategy.
- 📄 [`service.yaml`](./service.yaml) — ClusterIP Service mapping internal port `8080` to target port `80`.
