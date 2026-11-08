<div align="center">

# ☸️ Kubernetes que se cura solo

**Una web que sobrevive a que se le mueran los pods — monitorizada y cerrada con políticas de red de denegación por defecto.**

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square)

<sub><a href="README.md">🇬🇧 English</a> · 🇪🇸 Español</sub>

<img src="assets/self-healing.svg" alt="Repetición: se borra un pod, Kubernetes lo recrea y la otra réplica sigue sirviendo" width="820">

<sub>Repetición de la prueba de autorrecuperación que puedes lanzar tú — <a href="#-pruébalo-en-5-comandos">mira abajo</a>.</sub>

</div>

## 💡 Qué demuestra

- **Alta disponibilidad** — Nginx con 2 réplicas detrás de un Service. Borras una y la otra sigue sirviendo mientras Kubernetes levanta otra.
- **Autorrecuperación** — liveness y readiness probes en el puerto 80: un pod roto se reinicia, y uno nuevo no recibe tráfico hasta que está listo.
- **Red zero-trust** — 6 NetworkPolicies: se deniega todo por defecto y solo se permite el tráfico web, el scraping de Prometheus y el DNS.
- **Observabilidad** — Prometheus lee cAdvisor a través del API server (con su propio RBAC); Grafana grafica CPU y memoria por pod.
- **Todo como código** — el stack entero son YAML en [`manifests/`](manifests/), con requests y limits para que quepa en una VM pequeña.

## 🏗️ Arquitectura

```mermaid
flowchart LR
    user([Navegador]) -->|":30000"| svc[Service<br/>nginx-web]
    admin([Admin]) -->|":32000"| graf
    subgraph mk[Minikube · VM Ubuntu Server 24.04]
        svc --> p1[pod nginx 1]
        svc --> p2[pod nginx 2]
        prom[Prometheus] -.métricas cAdvisor<br/>por pod.-> api[(API server)]
        graf[Grafana] --> prom
    end
    classDef pod fill:#326CE5,color:#fff,stroke:#326CE5
    classDef mon fill:#E6522C,color:#fff,stroke:#E6522C
    class p1,p2 pod
    class prom,graf mon
```

<details>
<summary><b>🔐 Políticas de red</b> — quién puede hablar con quién</summary>
<br>

| Política | Se aplica a | Permite |
|---|---|---|
| `default-deny-all` | todos los pods | nada de entrada ni de salida |
| `allow-nginx-web` | nginx | entrada TCP 80 |
| `allow-monitoring-ingress` | Prometheus | entrada TCP 9090 |
| `allow-grafana-ingress` | Grafana | entrada TCP 3000 |
| `allow-prometheus-scrape` | Prometheus | salida hacia nginx por TCP 80 |
| `allow-dns` | todos los pods | salida DNS por el 53 (TCP/UDP) |

Fuente: [`manifests/network/networkpolicy.yaml`](manifests/network/networkpolicy.yaml)

</details>

## 🚀 Pruébalo en 5 comandos

En cualquier máquina con Docker, Minikube y kubectl:

```bash
minikube start --driver=docker --memory=2048 --cpus=2
kubectl apply -f manifests/app-web/ -f manifests/monitoring/ -f manifests/network/
kubectl get pods -w                                   # espera a que todo esté Running
kubectl delete $(kubectl get pod -l app=nginx-web -o name | head -1)   # mata una réplica
kubectl get pods -l app=nginx-web -w                  # mira cómo se recupera
```

La web en el NodePort `30000`, Grafana en el `32000` (`minikube service nginx-web --url`) y Prometheus con `kubectl port-forward svc/prometheus 9090`.

## 📸 Evidencias

<table>
<tr>
<td width="50%"><img src="img/05-prometheus-targets.png" alt="Prometheus: target kubernetes-cadvisor UP"><br><sub>Prometheus leyendo cAdvisor — target <b>UP</b></sub></td>
<td width="50%"><img src="img/07.2-grafana-dashboard.png" alt="Grafana: CPU de los pods de nginx"><br><sub>Grafana — CPU por pod de nginx</sub></td>
</tr>
<tr>
<td><img src="img/03-nginx-deployment.png" alt="kubectl: deployment de nginx 2/2 listo"><br><sub>Deployment con 2/2 réplicas listas</sub></td>
<td><img src="img/09-networkpolicy-list.png" alt="kubectl get networkpolicies"><br><sub>Políticas de red aplicadas</sub></td>
</tr>
</table>

## 👥 Equipo

Proyecto intermodular de 2.º de ASIR, curso 2025–2026. Equipo de tres:

| | Rol |
|---|---|
| **Adrián Boza** ([@adrianboza2](https://github.com/adrianboza2)) | Infraestructura y despliegue en Kubernetes |
| Santiago Pérez ([@santiagoperez27](https://github.com/santiagoperez27)) | Monitorización y observabilidad |
| Sergio López ([@SergioLopez2411](https://github.com/SergioLopez2411)) | Seguridad de red y documentación |

## 🧭 Lo siguiente que haría

Autoescalado con `HPA` · volúmenes persistentes para Grafana y Prometheus · Ingress + cert-manager · un job de GitHub Actions que valide los manifiestos con `kubeconform` · pasar a un clúster multinodo (k3s o AKS/EKS).

---

<sub>📄 La documentación completa (problema y solución, instalación paso a paso, solución de problemas, roadmap) está en [`docs/DETALLES.md`](docs/DETALLES.md), y la memoria del proyecto en [`docs/`](docs/). Licencia MIT.</sub>
