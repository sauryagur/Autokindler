<div align="center">
  <img src="https://github.com/user-attachments/assets/b8936abc-fdc0-4050-a88d-31e390dc0e6e" alt="Tiffinwala Banner" width="700" />
  <br />

  <h1>tiffinwala</h1>

  <p><strong>a self-hosted distributed job runner for teams who just want their code to run</strong></p>

  <br />

  <a href="https://www.go.dev/">
    <img src="https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white" alt="Go" />
  </a>
  <a href="https://containerd.io/">
    <img src="https://img.shields.io/badge/containerd-575757?style=flat-square&logo=containerd&logoColor=white" alt="containerd" />
  </a>
  <img src="https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square" alt="MIT License" />
  <img src="https://img.shields.io/badge/status-early%20development-orange?style=flat-square" alt="Status" />

  <br />
  <br />

> **Run containers across your team's machines — laptops, servers, whatever you have — without touching AWS.**

  <br />
</div>

---

**Tiffinwala** is a distributed job runner built for teams who need "Fargate-like" container execution on their own hardware. It turns a collection of Linux machines—VPS instances, old office servers, or even your laptop—into a unified compute cluster.

Throw a job definition at it, and Tiffinwala handles the rest: scheduling, container orchestration via `containerd`, log streaming, and automatic retries. No cloud lock-in, no IAM complexity, and no $400 surprise bills.

---

## how it works

1.  **Define:** You write a **recipe** (a simple YAML file describing your job).
2.  **Submit:** You push it via the CLI.
3.  **Schedule:** The **scheduler** finds a healthy **node** with available resources.
4.  **Execute:** The node agent pulls the image and runs the **dabba** (your job).
5.  **Stream:** Logs are streamed back to your terminal in real-time.

<!-- end list -->

```text
  [ CLI ]             [ SERVER ]                [ NODES ]
     │                    │                         │
     │  submit recipe     │                         │
     ├───────────────────>│    1. Validate          │
     │                    │    2. Persist           │
     │                    │    3. Match Node        │
     │                    │                         │
     │   stream logs      │<────────────────────────┤
     │<───────────────────┤      run dabba          │
     │                    │      (containerd)       │
```

---

## key features

- **Dead Simple Job Definitions** — One YAML file for images, resources, commands, and secrets.
- **Run Anywhere** — If it runs Linux and `containerd`, it’s a node. Mix and match providers and hardware.
- **Lease-Based Reliability** — Jobs are assigned via temporary leases. If a node goes dark, the lease expires and the job is automatically re-queued.
- **Secrets Management** — Credentials are encrypted at rest and injected into the container at runtime. They are never persisted to the node's disk.
- **Live Log Streaming** — Real-time `stdout/stderr` multiplexing from the node to your CLI.
- **Zero Drama Ops** — Two binaries. No Kubernetes, no JVM, no complex networking overlays.

---

## 📖 the vocabulary

We keep the essential "flavor" for the user-facing concepts while using standard industry terms for the infrastructure.

| Concept            | Name       | What it is                                                   |
| :----------------- | :--------- | :----------------------------------------------------------- |
| **Job / Task**     | **dabba**  | A single container execution request.                        |
| **Job Definition** | **recipe** | The YAML file describing the job requirements.               |
| **Worker Machine** | **node**   | Any machine running the agent binary.                        |
| **Node Agent**     | **agent**  | The daemon that talks to the server and `containerd`.        |
| **Assignment**     | **lease**  | A node's temporary, exclusive right to run a specific dabba. |
| **Environment**    | **masala** | Encrypted variables injected at runtime.                     |

---

## quick start

### 1\. Run the Server (Control Plane)

The server is stateful and acts as the brain of the cluster.

```bash
tiffinwala-server --port 8080 --db ./tiffinwala.db
```

### 2\. Connect a Node

Run this on any machine you want to contribute compute power.

```bash
tiffinwala-agent --server http://your-server:8080 --class server
```

### 3\. Submit a Recipe

Create a `test.yaml`:

```yaml
name: integration-tests
image: golang:1.22
cpu: 2
memory: 1gb
commands:
  - go test ./...
```

Submit it:

```bash
tiffinwala submit test.yaml
```

---

## architecture & guarantees

Tiffinwala is designed for **correctness in distributed environments**.

- **Leases & Heartbeats:** Agents must check in ("ping") the server every few seconds. If a node fails to heartbeat, the server revokes its active **leases** and moves those dabbas back into the global queue.
- **mTLS Security:** Communication between the server and agents is secured via mTLS. The server acts as a lightweight CA, issuing short-lived certificates to agents upon registration.
- **Node Classes:** Nodes can be labeled (e.g., `--class laptop` or `--class server`). The scheduler prioritizes high-availability "server" nodes for long-running tasks, keeping "laptop" nodes for burst capacity.

---

## what Tiffinwala is NOT

- **Not a Kubernetes Replacement:** We don't do service discovery, ingress controllers, or persistent volumes.
- **Not for Long-Running Services:** Tiffinwala is for jobs that run to completion (CI/CD, batch processing, data crunching). It is not a web server orchestrator.
- **Not a Managed Cloud:** This is strictly self-hosted. You own your data and your compute.

---

## project status

Tiffinwala is currently in **early development**. The core execution lifecycle and CLI are functional, but advanced features like Job Pipelines (DAGs) and Artifact Storage are on the [roadmap](https://www.google.com/search?q=docs/roadmap.md).

---
