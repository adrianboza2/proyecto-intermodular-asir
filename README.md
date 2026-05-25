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
## 🎯 Problema y Solución

### ❌ Situación Tradicional (Sin Orquestación)

En un despliegue clásico de una aplicación web:

| Problema | Consecuencia |
|----------|-------------|
| Despliegue manual en un servidor | Errores humanos, inconsistencia entre entornos |
| Si el servidor falla → la web se cae | Downtime hasta intervención manual |
| Actualizar Nginx requiere parar el servicio | Tiempo de inactividad obligatorio |
| ¿Cómo sé si va lento? | Monitorización reactiva, logs manuales |
| ¿Quién puede acceder? | Firewall básico del SO, sin aislamiento fino |

### ✅ Nuestra Solución con Kubernetes

Este proyecto simula un entorno **cloud-native** a escala educativa, aplicando prácticas de la industria:

| Capacidad | Implementación en el Proyecto | Beneficio |
|-----------|-------------------------------|-----------|
| **Alta Disponibilidad** | Deployment con `replicas: 2` + selector de etiquetas | Si un pod falla, el otro sigue sirviendo tráfico |
| **Auto-recuperación** | `livenessProbe` + `readinessProbe` en puerto 80 | Kubernetes detecta y reinicia pods fallidos automáticamente (< 60s) |
| **Actualizaciones sin downtime** | Rolling updates nativos de Kubernetes | Nueva versión de Nginx sin interrumpir el servicio |
| **Observabilidad proactiva** | Prometheus (métricas) + Grafana (dashboards) | Detectamos cuellos de botella antes de que los usuarios se quejen |
| **Seguridad zero-trust** | NetworkPolicy con reglas ingress/egress explícitas | Solo el tráfico autorizado puede comunicarse entre servicios |
| **Infraestructura como código** | Manifiestos YAML versionados en Git | Reproducible, auditable y reversible en cualquier momento |

### 🌍 ¿Qué Escenario Real Simula?

    Este proyecto replica, a pequeña escala, el flujo de trabajo de un equipo DevOps en una empresa tech:

    1. Desarrollo → Código en Git
    2. CI/CD → Validación automática de YAMLs (futuro)
    3. Despliegue → kubectl apply en cluster Kubernetes
    4. Operación → Monitoring con alertas visuales
    5. Seguridad → Políticas de red granulares por servicio

    La diferencia: lo ejecutamos en un portátil con 2GB RAM, no en un cluster de producción de 100 nodos.
    Pero los conceptos, manifiestos y mentalidad son 100% transferibles.

> 💡 **En esencia**: No gestionamos servidores, definimos el estado deseado y dejamos que Kubernetes lo mantenga.

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
                └── Namespace: default
                    ├── Deployment: nginx-web (2 réplicas)
                    ├── Service: nginx-web (NodePort 30000)
                    ├── Deployment: prometheus (scraping de métricas)
                    ├── Service: prometheus (NodePort 31000)
                    ├── Deployment: grafana (dashboards)
                    ├── Service: grafana (NodePort 32000)
                    └── 5 NetworkPolicies (modelo zero-trust)
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
│   └── MEMORIA_TECNICA.md        # Memoria técnica del proyecto
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
kubectl get pods -w
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
kubectl delete pod -l app=nginx-web --force

# Observar la recuperación automática
kubectl get pods -w

# El sistema debe recrear el pod en < 60 segundos
```

---

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

## 🚀 Mejoras Futuras (Roadmap)

Este proyecto es una base sólida que puede evolucionar en múltiples direcciones. Aquí proponemos mejoras organizadas por complejidad:

### 🔧 Corto Plazo (1-2 meses)

| Mejora | Impacto | Implementación Sugerida |
|--------|---------|------------------------|
| **Annotations para Prometheus** | Alta | Añadir `prometheus.io/scrape: "true"` en el deployment de Nginx para discovery automático |
| **Namespace dedicado** | Media | Mover todos los recursos a `namespace: asir-project` para aislamiento lógico |
| **ResourceQuotas** | Media | Limitar CPU/RAM del namespace para evitar que un pod sature la VM |
| **Validación automática de YAML** | Media | Script `scripts/lint-yaml.sh` con `yamllint` o `kubeconform` en pre-commit |
| **Plantillas GitHub** | Baja | `.github/PULL_REQUEST_TEMPLATE.md` para estandarizar contribuciones |

### ⚙️ Medio Plazo (3-6 meses)

| Mejora | Impacto | Implementación Sugerida |
|--------|---------|------------------------|
| **Horizontal Pod Autoscaler (HPA)** | Alta | Escalar automáticamente de 2 → 5 réplicas si CPU > 70% |
| **Volúmenes Persistentes (PVC)** | Alta | Retener dashboards de Grafana y datos de Prometheus tras reinicios |
| **Ingress Controller** | Media | Reemplazar NodePort por Nginx Ingress + reglas de host/path |
| **Cert-Manager + Let's Encrypt** | Media | HTTPS automático con certificados válidos para demo pública |
| **Pipeline CI/CD básico** | Media | GitHub Actions que valide YAMLs y despliegue en Minikube tras push a main |

### 🌐 Largo Plazo (6-12 meses)

| Mejora | Impacto | Implementación Sugerida |
|--------|---------|------------------------|
| **Migrar a cluster multi-nodo** | Crítico para HA real | Minikube no tolera fallos de nodo; usar k3s o cloud (AKS/EKS/GKE) |
| **GitOps con ArgoCD o Flux** | Transformador | Sincronización automática cluster ↔ Git; cero kubectl manual |
| **Service Mesh (Istio/Linkerd)** | Avanzado | Observabilidad avanzada, mTLS, tráfico canary, retries automáticos |
| **Logging centralizado (ELK/Loki)** | Operativo | Agregar logs de todos los pods en un dashboard único de búsqueda |
| **Preparar certificación CKA** | Profesional | Usar este proyecto como base de estudio para Certified Kubernetes Administrator |

### 🧭 ¿Por dónde empezar?

Recomendamos este orden de priorización:

    Fase 1: Base actual → Annotations + Namespace + ResourceQuotas
    Fase 2: HPA + PVC + Ingress + HTTPS
    Fase 3: CI/CD básico con GitHub Actions
    Fase 4: GitOps + Service Mesh (avanzado)

Cada paso añade valor tangible y prepara para el siguiente. No es necesario implementar todo: **elige según tus objetivos de aprendizaje**.

---

## 📄 Licencia

MIT License — Ver [LICENSE](LICENSE) para más detalles.


