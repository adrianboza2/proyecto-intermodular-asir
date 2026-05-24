# IMPLANTACIÓN DE UN ENTORNO KUBERNETES LOCAL CON APLICACIÓN WEB ESCALABLE Y MONITOREO

**Memoria Técnica del Proyecto Intermodular**

| Campo | Detalle |
|---|---|
| Ciclo Formativo | 2.º ASIR – Administración de Sistemas Informáticos en Red |
| Curso | 2024-2025 |
| Entrega | 2025-2026 |
| Tutora | Mercedes Rodríguez Villafafila |
| Equipo | Adrián Boza Suárez · Santiago Pérez Cano · Sergio López Pérez |
| Repositorio | https://github.com/adrianboza2/proyecto-intermodular-asir |

---

## 1. Introducción y Objetivos

### 1.1 Descripción del Proyecto

El presente proyecto consiste en la implantación de un entorno Kubernetes local completo, que incluye una aplicación web escalable basada en Nginx, un sistema de monitoreo con Prometheus y Grafana, y políticas de seguridad de red mediante NetworkPolicy. El entorno se despliega sobre una máquina virtual Ubuntu Server 24.04 LTS, ejecutada mediante VirtualBox en un equipo con Windows 10/11, utilizando Minikube como orquestador Kubernetes de un único nodo.

### 1.2 Justificación Técnica

Kubernetes se ha convertido en el estándar de facto para la orquestación de contenedores en entornos productivos. Su adopción en empresas de todos los tamaños justifica su estudio en el ciclo ASIR. La elección de Minikube como plataforma de desarrollo local permite simular un clúster real con recursos limitados, siendo ideal para entornos educativos.

Docker Engine actúa como runtime de contenedores subyacente, mientras que la virtualización mediante VirtualBox garantiza el aislamiento del entorno y su reproducibilidad en cualquier equipo Windows.

### 1.3 Objetivos

**Objetivos generales:**

- Desplegar y gestionar un clúster Kubernetes funcional en un entorno local.
- Implementar una aplicación web con alta disponibilidad y escalabilidad.
- Monitorizar el estado del clúster y de las aplicaciones en tiempo real.
- Aplicar políticas de seguridad de red para aislar el tráfico entre pods.

**Objetivos específicos:**

- Configurar una VM Ubuntu Server con Docker, kubectl y Minikube.
- Desplegar Nginx con 2 réplicas mediante Deployment y Service NodePort.
- Integrar Prometheus para la recopilación de métricas del clúster.
- Configurar Grafana con dashboards de visualización de métricas.
- Implementar NetworkPolicy para el aislamiento y control del tráfico de red.
- Documentar todo el proceso técnico y las evidencias de funcionamiento.

### 1.4 Relación con Módulos ASIR

| Módulo | Siglas | Relación con el Proyecto |
|---|---|---|
| Servicios de Red e Internet | SRI | Configuración de servicios web (Nginx), DNS interno del clúster, port forwarding y exposición de servicios. |
| Seguridad y Alta Disponibilidad | SAD | Implementación de NetworkPolicy para aislamiento de tráfico, alta disponibilidad con múltiples réplicas. |
| Administración de Sistemas Operativos | ASO | Administración de Ubuntu Server, gestión de usuarios, servicios systemd, Docker y herramientas CLI. |

---

## 2. Arquitectura del Sistema

### 2.1 Visión General

La arquitectura del sistema se organiza en tres capas principales: la capa de host Windows con VirtualBox, la capa de virtualización con Ubuntu Server y Docker, y la capa de orquestación con Kubernetes (Minikube). El acceso desde el host a los servicios del clúster se realiza mediante port forwarding de VirtualBox y `kubectl port-forward`.

```
┌─────────────────────────────────────────────────────────┐
│                    HOST WINDOWS 10/11                   │
│                                                         │
│  Navegador → 127.0.0.1:8082 / 9092 / 3002              │
│  SSH        → 127.0.0.1:2224                            │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │           VIRTUALBOX (NAT + Port Forwarding)    │    │
│  │                                                 │    │
│  │  ┌──────────────────────────────────────────┐   │    │
│  │  │        UBUNTU SERVER 24.04 LTS           │   │    │
│  │  │        usuario: sergio                   │   │    │
│  │  │                                          │   │    │
│  │  │  ┌────────────────────────────────────┐  │   │    │
│  │  │  │    MINIKUBE (Docker driver)        │  │   │    │
│  │  │  │    Kubernetes v1.35.1              │  │   │    │
│  │  │  │                                    │  │   │    │
│  │  │  │  [nginx-web x2] [prometheus]       │  │   │    │
│  │  │  │  [grafana]      [NetworkPolicy x6] │  │   │    │
│  │  │  └────────────────────────────────────┘  │   │    │
│  │  └──────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Componentes del Sistema

**Host Windows + VirtualBox**
El equipo anfitrión ejecuta Windows 10/11 con VirtualBox como hipervisor de tipo 2. El port forwarding de VirtualBox redirige los puertos del host a la VM, permitiendo el acceso a los servicios desde el navegador del host.

**VM Ubuntu Server 24.04 LTS**
Máquina virtual con 2 GB de RAM y 2 vCPU, configurada con OpenSSH para acceso remoto. Actúa como nodo único del clúster Kubernetes. El usuario `sergio` tiene permisos de Docker y sudo.

**Docker Engine**
Motor de contenedores que actúa como driver de Minikube. Versión 29.5.1. Gestiona los contenedores del clúster Kubernetes de forma transparente.

**Minikube (Clúster Kubernetes)**
Orquestador Kubernetes de un único nodo (control-plane + worker). Versión v1.38.1 con Kubernetes v1.35.1. Proporciona todas las funcionalidades de Kubernetes en un entorno local.

**Aplicación Web Nginx**
Servidor web desplegado con 2 réplicas mediante un Deployment de Kubernetes. Expuesto mediante un Service de tipo NodePort en el puerto 30000. Labels: `app=nginx-web`.

**Stack de Monitoreo (Prometheus + Grafana)**
Prometheus recopila métricas del clúster y de los pods mediante scraping automático. Grafana visualiza las métricas en dashboards interactivos. Expuestos mediante NodePort en los puertos 31000 y 32000 respectivamente.

**NetworkPolicy de Seguridad**
Conjunto de 6 políticas de red Kubernetes que implementan el principio de mínimo privilegio: denegación por defecto de todo el tráfico, con permisos explícitos para cada flujo de comunicación necesario.

### 2.3 Tabla de Port Forwarding

| Servicio | Puerto Host (Windows) | Puerto VM (Ubuntu) | Puerto Kubernetes |
|---|---|---|---|
| SSH | 2224 | 22 | - |
| Nginx Web | 8082 | 30000 | 30000 (NodePort) |
| Prometheus | 9092 | 31000 | 31000 (NodePort) |
| Grafana | 3002 | 32000 | 32000 (NodePort) |

---

## 3. Implementación Técnica

### 3.1 Infraestructura y Aplicación Web (Adrián Boza Suárez)

#### 3.1.1 Instalación de la Máquina Virtual

Se creó una máquina virtual en VirtualBox con Ubuntu Server 24.04 LTS, asignando 4 GB de RAM, 2 núcleos de CPU y 20 GB de almacenamiento dinámico. El adaptador de red se configuró en modo NAT con las reglas de port forwarding necesarias para acceder a los servicios.

#### 3.1.2 Instalación de Docker y Minikube

Se instaló Docker Engine siguiendo el procedimiento oficial, añadiendo el repositorio de Docker y configurando el usuario en el grupo docker. Minikube se instaló como binario independiente y se inició con el driver docker:

```bash
minikube start --driver=docker --memory=2048 --cpus=2
```

#### 3.1.3 Deployment y Service de Nginx

La aplicación web se desplegó mediante dos manifests YAML en la carpeta `manifests/app-web/`:

- `deployment.yaml`: Define el Deployment de Nginx con 2 réplicas, imagen `nginx:latest` y label `app=nginx-web`.
- `service.yaml`: Define un Service NodePort que expone el puerto 80 del pod en el puerto 30000 del nodo.

#### 3.1.4 Alta Disponibilidad

Con 2 réplicas configuradas, Kubernetes garantiza que siempre haya al menos un pod de Nginx disponible. Si un pod falla, el ReplicaSet lo recrea automáticamente, asegurando la continuidad del servicio.

> **Evidencia:** Capturas 01-04 en la carpeta `img/` del repositorio.

---

### 3.2 Monitoreo con Prometheus y Grafana (Santiago Pérez Cano)

#### 3.2.1 Configuración de Prometheus

Prometheus se desplegó mediante `manifests/monitoring/prometheus.yaml`, que incluye:

- Un ServiceAccount con permisos RBAC (ClusterRole y ClusterRoleBinding) para acceder a la API de Kubernetes.
- Un ConfigMap con la configuración de scraping.
- Un Deployment con Service NodePort en el puerto 31000.

#### 3.2.2 Configuración de Grafana

Grafana se desplegó mediante `manifests/monitoring/grafana.yaml` con un Deployment y Service NodePort en el puerto 32000. Se configuró con Prometheus como fuente de datos para visualizar las métricas del clúster.

#### 3.2.3 Métricas Monitorizadas

- Uso de CPU por pod y nodo.
- Consumo de memoria RAM.
- Estado de los pods (Running, Pending, Failed).
- Tráfico de red entrante y saliente.
- Estado de los targets de scraping en Prometheus.

#### 3.2.4 Limitación: Sin Persistencia

Al no configurar volúmenes persistentes (PersistentVolume), los dashboards de Grafana se pierden al reiniciar el pod. Esta es una limitación conocida del entorno de desarrollo que se documenta como mejora futura.

> **Evidencia:** Capturas 05-08 en la carpeta `img/` del repositorio.

---

### 3.3 Seguridad de Red con NetworkPolicy (Sergio López Pérez)

#### 3.3.1 Justificación de NetworkPolicy

Por defecto, Kubernetes permite la comunicación libre entre todos los pods del clúster. Esto representa un riesgo de seguridad en entornos productivos. Las NetworkPolicy permiten implementar el principio de mínimo privilegio, denegando todo el tráfico por defecto y permitiendo únicamente las comunicaciones necesarias.

#### 3.3.2 Descripción de las Políticas Implementadas

**Política 1: `default-deny-all`**
Deniega todo el tráfico Ingress y Egress por defecto en el namespace default. Se aplica a todos los pods (podSelector vacío). Es la base del modelo de seguridad.

**Política 2: `allow-nginx-web`**
Permite el tráfico entrante al puerto 80 de los pods con label `app=nginx-web` desde cualquier fuente, permitiendo el acceso a la aplicación web desde el exterior.

**Política 3: `allow-monitoring-ingress`**
Permite el tráfico entrante al puerto 9090 de los pods con label `app=prometheus`, habilitando el acceso a la interfaz web de Prometheus.

**Política 4: `allow-grafana-ingress`**
Permite el tráfico entrante al puerto 3000 de los pods con label `app=grafana`, habilitando el acceso al dashboard de Grafana.

**Política 5: `allow-prometheus-scrape`**
Permite el tráfico saliente desde los pods de Prometheus hacia los pods con label `app=nginx-web` en el puerto 80, habilitando el scraping de métricas.

**Política 6: `allow-dns`**
Permite el tráfico UDP/TCP en el puerto 53 para todos los pods, garantizando la resolución de nombres DNS dentro del clúster.

#### 3.3.3 Archivo networkpolicy.yaml

El archivo se encuentra en `manifests/network/networkpolicy.yaml` del repositorio. Contiene los 6 documentos YAML separados por `---`, cada uno con labels `app=network-policy` y `role=security` para su identificación.

```yaml
# Política 1 - Denegar todo por defecto
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
  labels:
    app: network-policy
    role: security
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

#### 3.3.4 Verificación de las Políticas

Las políticas se aplicaron con:

```bash
kubectl apply -f manifests/network/networkpolicy.yaml
```

Se verificó su correcta aplicación con:

```bash
kubectl get networkpolicy
```

Tras aplicar las políticas, se confirmó que Nginx sigue accesible, Prometheus continúa scrapeando y Grafana carga correctamente.

> **Evidencia:** Capturas 09-11 en la carpeta `img/` del repositorio.

---

## 4. Resultados y Pruebas

### 4.1 Tabla de Requisitos Funcionales

| ID | Requisito | Responsable | Estado |
|---|---|---|---|
| RF-01 | Clúster Kubernetes funcional con Minikube | Adrián | ✅ Cumplido |
| RF-02 | Aplicación Nginx con 2 réplicas accesible vía NodePort | Adrián | ✅ Cumplido |
| RF-03 | Prometheus scrapeando métricas del clúster | Santiago | ✅ Cumplido |
| RF-04 | Grafana con dashboard de visualización | Santiago | ✅ Cumplido |
| RF-05 | NetworkPolicy con aislamiento de tráfico | Sergio | ✅ Cumplido |
| RF-06 | Memoria técnica completa y repositorio organizado | Sergio | ✅ Cumplido |

### 4.2 Evidencias de Funcionamiento

**Aplicación Web accesible**
La aplicación Nginx responde correctamente en `http://127.0.0.1:8082` desde el navegador del host Windows, mostrando la página de bienvenida de Nginx. Verificado con curl desde la VM:
```bash
curl http://$(minikube ip):30000
```

**Auto-recuperación de pods**
Kubernetes recrea automáticamente los pods que fallan gracias al ReplicaSet. Verificado tras el reinicio de la VM, donde los pods volvieron al estado `Running` de forma autónoma.

**Métricas en tiempo real**
Prometheus está scrapeando correctamente los targets configurados (estado UP). Grafana muestra dashboards con datos de CPU, RAM y estado de pods en tiempo real, accesible en `http://127.0.0.1:3002`.

**Aislamiento de red verificado**
Las 6 NetworkPolicy están aplicadas y activas (`kubectl get networkpolicy`). Nginx sigue siendo accesible tras aplicar el aislamiento, confirmando que las políticas permisivas funcionan correctamente junto con la denegación por defecto.

---

## 5. Conclusiones y Trabajo Futuro

### 5.1 Conclusiones

Este proyecto ha permitido al equipo adquirir experiencia práctica en tecnologías clave del mercado actual: contenedores Docker, orquestación con Kubernetes, monitoreo con Prometheus/Grafana y seguridad de red con NetworkPolicy.

Se han aplicado conceptos de todos los módulos del ciclo ASIR en un proyecto integrador real, desde la administración del sistema operativo hasta la seguridad y los servicios de red. El uso de Git y GitHub ha fomentado el trabajo colaborativo y el control de versiones del código.

La principal dificultad encontrada fue la limitación de recursos de la máquina virtual (2 GB RAM), que requirió optimizar la configuración de Minikube y gestionar cuidadosamente los recursos del clúster.

### 5.2 Dificultades y Soluciones

| Dificultad | Solución Aplicada |
|---|---|
| Disco C: lleno en el host Windows | Se movió la VM al disco E: con más espacio disponible. |
| Corrupción de configuración de Minikube | Se eliminó `~/.minikube` y se reinició el clúster desde cero. |
| Acceso desde el navegador Windows | Se utilizó `kubectl port-forward` con `--address 0.0.0.0`. |
| Clave SSH cambiada tras reinstalación | Se ejecutó `ssh-keygen -R [127.0.0.1]:2224` para limpiar la clave antigua. |

### 5.3 Mejoras Posibles

**Volúmenes Persistentes**
Configurar PersistentVolume y PersistentVolumeClaim para que los dashboards de Grafana y los datos de Prometheus persistan entre reinicios.

**Clúster Multi-Nodo**
Escalar el clúster a múltiples nodos para simular un entorno productivo con distribución de carga real.

**Pipeline CI/CD**
Integrar GitHub Actions para despliegue automático mediante `kubectl apply` en cada push al repositorio.

**Ingress Controller**
Sustituir los NodePort por un Ingress Controller (nginx-ingress) para gestión centralizada del tráfico HTTP.

**TLS/HTTPS**
Añadir certificados TLS mediante cert-manager para securizar las comunicaciones.

**Alertmanager**
Integrar Alertmanager con Prometheus para enviar notificaciones cuando las métricas superen umbrales definidos.

### 5.4 Aplicación a Entornos Productivos

Los conocimientos adquiridos en este proyecto son directamente aplicables a entornos productivos. Kubernetes es utilizado por empresas como Google, Amazon, Microsoft y miles de organizaciones en todo el mundo. Las habilidades de despliegue, monitoreo y seguridad de contenedores son altamente demandadas en el mercado laboral actual, especialmente en roles de DevOps, SRE (Site Reliability Engineer) y administración de sistemas en la nube.

---

## 6. Anexos

### 6.1 Repositorio GitHub

```
https://github.com/adrianboza2/proyecto-intermodular-asir
```

Estructura del repositorio:

```
proyecto-intermodular-asir/
├── manifests/
│   ├── app-web/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── monitoring/
│   │   ├── prometheus.yaml
│   │   └── grafana.yaml
│   └── network/
│       └── networkpolicy.yaml
├── docs/
│   └── memoria-tecnica.md
├── img/
│   ├── 09-networkpolicy-list.png
│   ├── 10-networkpolicy-describe.png
│   └── 11-conectividad-test.png
└── README.md
```

### 6.2 Comandos Útiles de Referencia

| Descripción | Comando |
|---|---|
| Ver todos los recursos del clúster | `kubectl get all -A` |
| Ver estado de los pods | `kubectl get pods` |
| Ver logs de un pod | `kubectl logs -f <pod-name>` |
| Ejecutar comando en un pod | `kubectl exec -it <pod-name> -- bash` |
| Ver NetworkPolicy aplicadas | `kubectl get networkpolicy` |
| Describir un recurso | `kubectl describe <tipo> <nombre>` |
| Aplicar manifests | `kubectl apply -f <archivo.yaml>` |
| Eliminar recurso | `kubectl delete <tipo> <nombre>` |
| Ver IP de Minikube | `minikube ip` |
| Iniciar clúster | `minikube start --driver=docker --memory=2048 --cpus=2` |
| Parar clúster | `minikube stop` |
| Port-forward Nginx | `kubectl port-forward svc/nginx-web 30000:80 --address 0.0.0.0 &` |
| Port-forward Prometheus | `kubectl port-forward svc/prometheus 31000:9090 --address 0.0.0.0 &` |
| Port-forward Grafana | `kubectl port-forward svc/grafana 32000:3000 --address 0.0.0.0 &` |

### 6.3 Glosario de Términos Kubernetes

**Pod:** Unidad mínima de despliegue en Kubernetes. Contiene uno o más contenedores que comparten red y almacenamiento.

**Deployment:** Recurso que gestiona un conjunto de pods idénticos, garantizando el número de réplicas deseado.

**Service:** Abstracción que expone pods como un servicio de red estable con IP y DNS propios.

**NodePort:** Tipo de Service que expone el puerto del pod en un puerto del nodo, accesible desde fuera del clúster.

**NetworkPolicy:** Recurso que define reglas de firewall a nivel de pod, controlando el tráfico Ingress y Egress.

**Namespace:** Mecanismo de aislamiento lógico dentro del clúster para organizar recursos.

**ConfigMap:** Recurso para almacenar configuración no sensible en forma de pares clave-valor.

**RBAC:** Role-Based Access Control. Sistema de control de acceso basado en roles para la API de Kubernetes.

**Minikube:** Herramienta que ejecuta un clúster Kubernetes de un único nodo en local para desarrollo y pruebas.

**kubectl:** CLI oficial para interactuar con la API de Kubernetes desde la línea de comandos.

**Prometheus:** Sistema de monitoreo y alerta de código abierto que recopila métricas mediante scraping HTTP.

**Grafana:** Plataforma de visualización de métricas con dashboards interactivos y configurables.

**ReplicaSet:** Recurso que garantiza que un número específico de réplicas de un pod estén corriendo en todo momento.

**PersistentVolume (PV):** Recurso de almacenamiento en el clúster que persiste independientemente del ciclo de vida de los pods.

---

*Documento generado por Sergio López Pérez — Proyecto Intermodular ASIR 2024-2025*
