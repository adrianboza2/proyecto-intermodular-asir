# 📖 PROJECT_CONTEXT.md — Estado Actual del Proyecto

## 🏗️ Stack Técnico
| Componente | Versión/Config | Notas |
|------------|----------------|-------|
| Host | Windows 10/11 | VirtualBox 7.x |
| VM | Ubuntu Server 24.04 LTS | 4GB RAM, 2 CPU, 25GB disco |
| Driver | Docker Engine | `minikube start --driver=docker` |
| Kubernetes | Minikube v1.38.1 | K8s v1.35.1, 1 nodo |
| App Web | Nginx latest | 2 réplicas, NodePort 30000 |
| Monitoring | Prometheus + Grafana latest | En desarrollo |
| Seguridad | NetworkPolicy | networking.k8s.io/v1 |

## 🌐 Mapeo de Puertos (VirtualBox NAT)
| Servicio | Puerto Host | Puerto Guest (NodePort) |
|----------|-------------|------------------------|
| SSH | 2222 | 22 |
| Nginx | 8080 | 30000 |
| Grafana | 3000 | 32000 |
| Prometheus | 9090 | 31000 |

## ️ Convención de Labels
| Componente | Label |
|------------|-------|
| App Web | `app: nginx-web` |
| Prometheus | `app: prometheus` |
| Grafana | `app: grafana` |

## 👥 Equipo y Fases
| Responsable | GitHub | Fase | Estado |
|-------------|--------|------|--------|
| Adrián Boza | @adrianboza2 | Infra + App Web | ✅ Completado |
| Santiago Pérez | @santiagoperez27 | Monitoring | 🔄 En progreso |
| Sergio López | @SergioLopez2411 | Security + Docs | ⏳ Pendiente |

## ⚠️ Restricciones del Entorno
- VM limitada a 2GB RAM / 2 CPU para Minikube
- Sin volúmenes persistentes (PVC) en fase educativa
- Single-node cluster (no tolera fallos de nodo)
- Todo debe ser reproducible con `kubectl apply -f manifests/`