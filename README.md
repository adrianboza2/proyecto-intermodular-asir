<div align="center">

# ☸️ Self-healing Kubernetes lab

**A web app that survives its own pods dying — monitored, and locked down with default-deny network policies.**

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square)

<sub>🇬🇧 English · <a href="README.es.md">🇪🇸 Español</a></sub>

<img src="assets/self-healing.svg" alt="Replay: a pod is deleted, Kubernetes recreates it and the second replica keeps serving" width="820">

<sub>Replay of the self-healing test you can run yourself — <a href="#-try-it-in-5-commands">see below</a>.</sub>

</div>

## 💡 What it shows

- **High availability** — Nginx runs as 2 replicas behind a Service. Delete one and the other keeps serving while Kubernetes brings a new one up.
- **Self-healing** — liveness and readiness probes on port 80: a broken pod is restarted, and a new one gets no traffic until it is ready.
- **Zero-trust networking** — 6 NetworkPolicies: deny everything by default, then allow only web traffic, Prometheus scraping and DNS.
- **Observability** — Prometheus scrapes cAdvisor through the API server (with its own RBAC); Grafana graphs CPU and memory per pod.
- **Everything as code** — the whole stack is plain YAML in [`manifests/`](manifests/), with resource requests and limits so it fits on a small VM.

## 🏗️ Architecture

```mermaid
flowchart LR
    user([Browser]) -->|":30000"| svc[Service<br/>nginx-web]
    admin([Admin]) -->|":32000"| graf
    subgraph mk[Minikube · Ubuntu Server 24.04 VM]
        svc --> p1[nginx pod 1]
        svc --> p2[nginx pod 2]
        prom[Prometheus] -.cAdvisor metrics<br/>per pod.-> api[(API server)]
        graf[Grafana] --> prom
    end
    classDef pod fill:#326CE5,color:#fff,stroke:#326CE5
    classDef mon fill:#E6522C,color:#fff,stroke:#E6522C
    class p1,p2 pod
    class prom,graf mon
```

<details>
<summary><b>🔐 Network policies</b> — what may talk to what</summary>
<br>

| Policy | Applies to | Allows |
|---|---|---|
| `default-deny-all` | every pod | nothing in, nothing out |
| `allow-nginx-web` | nginx | inbound TCP 80 |
| `allow-monitoring-ingress` | Prometheus | inbound TCP 9090 |
| `allow-grafana-ingress` | Grafana | inbound TCP 3000 |
| `allow-prometheus-scrape` | Prometheus | outbound to nginx on TCP 80 |
| `allow-dns` | every pod | outbound DNS on 53 (TCP/UDP) |

Source: [`manifests/network/networkpolicy.yaml`](manifests/network/networkpolicy.yaml)

</details>

## 🚀 Try it in 5 commands

On any machine with Docker, Minikube and kubectl:

```bash
minikube start --driver=docker --memory=2048 --cpus=2
kubectl apply -f manifests/app-web/ -f manifests/monitoring/ -f manifests/network/
kubectl get pods -w                                   # wait for everything to be Running
kubectl delete $(kubectl get pod -l app=nginx-web -o name | head -1)   # kill one replica
kubectl get pods -l app=nginx-web -w                  # watch it heal
```

Web on NodePort `30000`, Grafana on `32000` (`minikube service nginx-web --url`), Prometheus with `kubectl port-forward svc/prometheus 9090`.

## 📸 Evidence

<table>
<tr>
<td width="50%"><img src="img/05-prometheus-targets.png" alt="Prometheus target health: kubernetes-cadvisor UP"><br><sub>Prometheus scraping cAdvisor — target <b>UP</b></sub></td>
<td width="50%"><img src="img/07.2-grafana-dashboard.png" alt="Grafana panel: CPU of the nginx pods"><br><sub>Grafana — CPU per nginx pod</sub></td>
</tr>
<tr>
<td><img src="img/03-nginx-deployment.png" alt="kubectl: nginx deployment 2/2 ready"><br><sub>Deployment with 2/2 replicas ready</sub></td>
<td><img src="img/09-networkpolicy-list.png" alt="kubectl get networkpolicies"><br><sub>Network policies applied</sub></td>
</tr>
</table>

## 👥 Team

Final project of the Network & Systems Administration degree (ASIR), 2025–2026. Team of three:

| | Role |
|---|---|
| **Adrián Boza** ([@adrianboza2](https://github.com/adrianboza2)) | Kubernetes infrastructure & deployment |
| Santiago Pérez ([@santiagoperez27](https://github.com/santiagoperez27)) | Monitoring & observability |
| Sergio López ([@SergioLopez2411](https://github.com/SergioLopez2411)) | Network security & documentation |

## 🧭 What I'd do next

`HPA` autoscaling · persistent volumes for Grafana/Prometheus · Ingress + cert-manager · a GitHub Actions job that validates the manifests with `kubeconform` · move to a multi-node cluster (k3s or AKS/EKS).

---

<sub>📄 The full write-up (in Spanish: problem/solution, step-by-step install, troubleshooting, roadmap) is in [`docs/DETALLES.md`](docs/DETALLES.md), and the project report in [`docs/`](docs/). MIT licensed.</sub>
