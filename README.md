# Self-healing Kubernetes lab

[Español](README.es.md)

A web app on a local Kubernetes cluster that keeps serving when one of its pods dies. It's monitored with Prometheus and Grafana, and the network is closed by default. Final project of my ASIR degree (2025–26), built by a team of three.

<img src="assets/architecture.svg" alt="Architecture: Service in front of two nginx replicas, Prometheus and Grafana, inside a default-deny network" width="560">

## How it works

| | |
|---|---|
| **Availability** | Nginx runs as two replicas behind a NodePort Service. |
| **Recovery** | Liveness and readiness probes on port 80. A failed pod is restarted, and a new one gets no traffic until it is ready. |
| **Network** | Six NetworkPolicies: everything is denied, then web traffic, Prometheus and DNS are allowed one by one ([see them](manifests/network/networkpolicy.yaml)). |
| **Monitoring** | Prometheus reads per-pod cAdvisor metrics through the API server, with its own service account and RBAC. Grafana shows CPU and memory. |
| **Size** | Requests and limits on every container, so the whole stack fits on a 2 GB VM. |

## The self-healing test

<img src="assets/self-healing.svg" alt="Replay: one replica is deleted, the other keeps serving and Kubernetes creates a new one" width="760">

Run it yourself with Docker, Minikube and kubectl:

```bash
minikube start --driver=docker --memory=2048 --cpus=2
kubectl apply -f manifests/app-web/ -f manifests/monitoring/ -f manifests/network/
kubectl delete $(kubectl get pod -l app=nginx-web -o name | head -1)
kubectl get pods -l app=nginx-web -w
```

The web app is on NodePort `30000` and Grafana on `32000`. For Prometheus, use `kubectl port-forward svc/prometheus 9090`.

## Screenshots

<table>
<tr>
<td width="50%"><img src="img/05-prometheus-targets.png" alt="Prometheus target kubernetes-cadvisor UP"><br><sub>Prometheus, cAdvisor target up</sub></td>
<td width="50%"><img src="img/07.2-grafana-dashboard.png" alt="Grafana CPU panel for the nginx pods"><br><sub>Grafana, CPU per nginx pod</sub></td>
</tr>
<tr>
<td><img src="img/03-nginx-deployment.png" alt="nginx deployment with 2/2 ready"><br><sub>Deployment, 2/2 ready</sub></td>
<td><img src="img/09-networkpolicy-list.png" alt="list of network policies"><br><sub>Network policies applied</sub></td>
</tr>
</table>

## Team

- **Adrián Boza** ([@adrianboza2](https://github.com/adrianboza2)): cluster, deployments and services
- Santiago Pérez ([@santiagoperez27](https://github.com/santiagoperez27)): monitoring
- Sergio López ([@SergioLopez2411](https://github.com/SergioLopez2411)): network policies and documentation

## What's next

Autoscaling with HPA, persistent volumes for Prometheus and Grafana, an Ingress with TLS, and a GitHub Actions job that validates the manifests.

<sub>Full write-up in Spanish, with a step-by-step install and troubleshooting, in [docs/DETALLES.md](docs/DETALLES.md). MIT license.</sub>
