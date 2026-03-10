# ☸️ Proyecto Intermodular ASIR — Kubernetes Local

<div align="center">

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

**Implantación de un Entorno Kubernetes Local con Aplicación Web Escalable y Monitoreo en Tiempo Real**

*2.º Curso ASIR — Curso 2025–2026*

</div>

---

## 📋 Descripción

Implementación de un clúster **Kubernetes local** usando **Minikube** sobre una VM Ubuntu Server (VirtualBox). El proyecto despliega una aplicación web Nginx con **alta disponibilidad** (2 réplicas), un stack de monitoreo completo con **Prometheus y Grafana**, y políticas de seguridad de red, todo ello de forma completamente **gratuita y reproducible** desde cualquier ordenador personal.

---

## 👥 Equipo

| Integrante | GitHub | Rol |
|---|---|---|
| Adrián Boza Suárez | [@adrianboza2](https://github.com/adrianboza2) | Infraestructura y Despliegue Kubernetes |
| Santiago Pérez Cano | [@santiagoperez27](https://github.com/santiagoperez27) | Monitoreo y Observabilidad (Prometheus/Grafana) |
| Sergio López Pérez | [@SergioLopez2411](https://github.com/SergioLopez2411) | Seguridad de Red y Documentación Técnica |

---

## 🏗️ Arquitectura

```
Windows 10/11 (Host)
└── VirtualBox
    └── Ubuntu Server 24.04 LTS (NAT + Port Forwarding)
        └── Docker (driver de Minikube)
            └── Minikube (Kubernetes 1 nodo)
                ├── Namespace: default
                │   ├── Deployment: nginx (2 réplicas)
                │   ├── Service: nginx-svc (NodePort)
                │   └── NetworkPolicy: restrict-ingress
                └── Namespace: monitoring
                    ├── Prometheus (scraping de métricas)
                    └── Grafana (dashboards)
```

---

## 🛠️ Stack Tecnológico

| Componente | Tecnología | Versión |
|---|---|---|
| Orquestador | Minikube + Kubernetes | Latest |
| Contenedores | Docker Engine | Latest |
| App web | Nginx | Latest |
| Métricas | Prometheus | Latest |
| Dashboards | Grafana | Latest |
| SO Invitado | Ubuntu Server | 24.04 LTS |
| Virtualización | VirtualBox | 7.x |

---

## 📂 Estructura del Repositorio

```
proyecto-intermodular-asir/
├── manifests/
│   ├── app-web/
│   │   ├── deployment.yaml      # Nginx con 2 réplicas
│   │   └── service.yaml         # NodePort para acceso exterior
│   ├── monitoring/
│   │   ├── prometheus.yaml      # Stack de métricas
│   │   └── grafana.yaml         # Dashboard y visualización
│   └── network/
│       └── networkpolicy.yaml   # Aislamiento de tráfico
├── docs/
│   └── memoria.docx             # Memoria técnica del proyecto
├── img/
│   └── ...                      # Capturas de evidencia
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🚀 Instalación Rápida

### Prerrequisitos

- VirtualBox + Ubuntu Server 24.04 LTS en VM
- Docker, Minikube y kubectl instalados en la VM
- Reenvío de puertos VirtualBox: `2222→22` (SSH), `8080→30000` (web), `3000→32000` (Grafana)

### 1. Clonar el repositorio

```bash
git clone https://github.com/adrianboza2/proyecto-intermodular-asir.git
cd proyecto-intermodular-asir
```

### 2. Iniciar el clúster

```bash
minikube start --driver=docker --memory=2048 --cpus=2
minikube status
```

### 3. Desplegar la aplicación web

```bash
kubectl apply -f manifests/app-web/
kubectl get pods -w
# Esperar hasta ver 2/2 pods en estado Running
```

### 4. Desplegar el stack de monitoreo

```bash
kubectl apply -f manifests/monitoring/
kubectl get pods -n monitoring -w
```

### 5. Aplicar políticas de red

```bash
kubectl apply -f manifests/network/
kubectl get networkpolicies
```

### 6. Acceder a los servicios

| Servicio | URL | Credenciales |
|---|---|---|
| App web (Nginx) | http://localhost:8080 | — |
| Grafana | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |

---

## 🧪 Prueba de Alta Disponibilidad

```bash
# Simular fallo de un pod
kubectl delete pod $(kubectl get pods -l app=nginx -o jsonpath='{.items[0].metadata.name}')

# Observar la recuperación automática
kubectl get pods -w

# El sistema debe recrear el pod en < 60 segundos
```

---

## 📚 Módulos ASIR Relacionados

| Módulo | RA | Concepto aplicado |
|---|---|---|
| **SRI** | RA1, RA2, RA4 | Services K8s, Nginx, Réplicas |
| **SAD** | RA1, RA3, RA5 | NetworkPolicies, Self-healing |
| **ASO** | RA2, RA4, RA6 | Prometheus/Grafana, Docker, YAML |

---

## ⚠️ Solución de Problemas Comunes

**Minikube no arranca con driver Docker:**
```bash
# Verificar que el usuario está en el grupo docker
groups $USER
# Si no aparece 'docker', ejecutar:
sudo usermod -aG docker $USER && newgrp docker
```

**Pods en estado Pending:**
```bash
kubectl describe pod <nombre-pod>
# Revisar la sección Events para ver el motivo
```

**No se puede acceder al servicio desde el host:**
```bash
# Verificar que el port-forwarding de VirtualBox está configurado
# Puerto anfitrión 8080 → Puerto invitado 30000
```

---

## 📄 Licencia

MIT License — Ver [LICENSE](LICENSE) para más detalles.

## 📚 Módulos ASIR Relacionados
Este proyecto integra conocimientos transversales de los siguientes módulos:

### 🌐 Servicios de Red e Internet (SRI)
* **RA1 (Servicios de Red):** Configuración de `Services` en Kubernetes para exponer la app web al exterior.
* **RA2 (Servicios Web):** Implantación de un servidor web Nginx contenedorizado.
* **RA4 (Alta Disponibilidad):** Uso de réplicas de pods para garantizar la disponibilidad del servicio.

### 🛡️ Seguridad y Alta Disponibilidad (SAD)
* **RA1 (Seguridad en Redes):** Configuración de `Network Policies` para el aislamiento de tráfico entre pods.
* **RA3 (Continuidad del Negocio):** Implementación de resiliencia mediante la recuperación automática de pods (Self-healing).
* **RA5 (Recuperación):** Gestión de la persistencia y disponibilidad en entornos de contenedores.

### 🐧 Administración de Sistemas Operativos (ASO)
* **RA2 (Gestión de Recursos):** Monitoreo de rendimiento (CPU/RAM) mediante el stack Prometheus + Grafana.
* **RA4 (Virtualización):** Despliegue de infraestructura sobre Docker y orquestación con Minikube.
* **RA6 (Automatización):** Gestión de la infraestructura mediante código (Manifiestos YAML declarativos).
