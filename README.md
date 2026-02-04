# Blue–Green Deployment on AWS using kOps (Kubernetes)

A production-style **Blue–Green deployment** implementation on **AWS** using **kOps-managed Kubernetes**. This project demonstrates zero-downtime releases, instant rollback, and real-world debugging of AWS LoadBalancer, Service selectors, ports, and ELB behavior.

---

## 🚀 Project Highlights

* Kubernetes cluster provisioned with **kOps** on AWS
* **Blue–Green deployment** using two Deployments and one Service
* Public exposure via **Service type: LoadBalancer (AWS ELB)**
* Instant traffic switch & rollback using **Service selectors**
* Real-world debugging: ports, selectors, ELB health, DNS, NodePort

---

## 🧱 Architecture

```
Client
  ↓
AWS ELB (Service: LoadBalancer)
  ↓
NodePort (auto-managed)
  ↓
Pods (Blue or Green)
```

**Key idea:** The **Service selector controls traffic**, not the Deployment.

---

## 🛠 Tech Stack

* **Cloud:** AWS (EC2, ELB, S3)
* **Kubernetes:** kOps
* **Container Runtime:** containerd
* **Apps:**

  * Blue: nginx (port 80)
  * Green: Tomcat-based app (port 8080)

---

## 📦 Cluster Setup (kOps)

* **Region:** ap-south-1
* **Cluster Name:** `kops.k8s.local`
* **State Store:** S3 (versioning enabled)
* **Topology:** Public
* **Nodes:** t2.micro
* **Master:** t2.medium

> kOps stores all cluster state in S3. Correct region configuration is mandatory.

---

## 🟦 Blue Deployment (Stable)

* **Labels:** `app=web, env=blue`
* **Image:** `nginx`
* **Container Port:** `80`
* **Purpose:** Current stable version

---

## 🟩 Green Deployment (New Version)

* **Labels:** `app=web, env=green`
* **Image:** Custom Tomcat app
* **Container Port:** `8080`
* **Purpose:** New release candidate

---

## 🌐 Service (Traffic Control)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb1
spec:
  type: LoadBalancer
  selector:
    app: web
    env: blue   # switch to green to route traffic
  ports:
    - port: 80
      targetPort: 80
```

### Traffic Switch

* `env=blue` → nginx (Blue)
* `env=green` → Tomcat (Green)

✔ No downtime
✔ No pod restart
✔ Instant rollback

---

## 🔁 Rollback Strategy

**Blue–Green rollback** is done by switching the Service selector back:

```yaml
selector:
  app: web
  env: blue
```

This is faster and safer than `kubectl rollout undo`.

---

## 🧪 Verification Commands

```bash
kubectl get deploy
kubectl get pods --show-labels
kubectl get svc lb1
kubectl get endpoints lb1
```

> **Endpoints** are the source of truth for where traffic is going.

---

## 🐞 Real Issues Debugged

* ❌ Immutable Deployment selectors
* ❌ Pod naming inside Deployments
* ❌ Port mismatch (80 vs 8080)
* ❌ NodePort blocked by AWS SG
* ❌ ELB with no healthy backends
* ❌ DNS typo (`.amazon` vs `.amazonaws.com`)

Each issue was fixed following production-grade debugging steps.

---

## 🧠 Key Learnings

* Service selectors control traffic
* `targetPort` must match the app’s listening port
* NodePort is internal plumbing on AWS
* Use ELB DNS for public access
* Endpoints reveal the real routing state

---

## 🧹 Cleanup

```bash
AWS_DEFAULT_REGION=ap-south-1 \
kops delete cluster --name kops.k8s.local --state s3://kops-cluster-k8s --yes
```

---

## 📈 Future Improvements

* Add readiness & liveness probes
* Use Ingress + ALB
* CI/CD automation (GitHub Actions)
* Canary deployments

---

## 📌 Author

**Sonu Kumar**
DevOps / Cloud Engineer

---

⭐ If this helped you understand real Kubernetes deployments, give it a star!
