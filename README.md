# ☸️ Proyecto Intermodular ASIR: Kubernetes Local con Alta Disponibilidad y Monitoreo

Este proyecto consiste en la implantación de un entorno de **Kubernetes local** utilizando **Minikube**. Se despliega una aplicación web escalable (Nginx) y un sistema de observabilidad completo basado en **Prometheus** y **Grafana**.

---

## 👥 Equipo de Trabajo
* **Adrián Boza Suárez (adrianboza2)**: Especialista en Infraestructura y Despliegue (K8s, Docker, Nginx).
* **Santiago Pérez Cano (santiagoperez27)**: Especialista en Monitoreo y Observabilidad (Prometheus, Grafana).
* **Sergio López Pérez (SergioLopez2411)**: Especialista en Seguridad y Documentación Técnica (NetworkPolicies, Memoria).

---

## 🛠️ Stack Tecnológico
- **SO Anfitrión:** Windows 10/11.
- **Virtualización:** VirtualBox (Ubuntu Server 24.04 LTS).
- **Orquestador:** Minikube (Kubernetes).
- **Contenedores:** Docker.
- **Aplicación:** Nginx (Deployment con 2 réplicas).
- **Observabilidad:** Prometheus (Métricas) y Grafana (Dashboards).

---

## 📂 Estructura del Repositorio
* `/manifests`: Archivos YAML de configuración de Kubernetes.
  * `/app-web`: Despliegue de la aplicación Nginx.
  * `/monitoring`: Configuración de Prometheus y Grafana.
  * `/network`: Políticas de red (Security).
* `/docs`: Memoria del proyecto y manuales de usuario/instalación.
* `/img`: Capturas de pantalla de las evidencias y pruebas de fallo.

---

## 🚀 Cómo ejecutar este proyecto
1. Instalar **Docker**, **Minikube** y **kubectl** en una VM Linux.
2. Iniciar el clúster: `minikube start --driver=docker`.
3. Aplicar los manifiestos: `kubectl apply -f manifests/app-web/`.
4. Acceder al dashboard de Grafana para ver las métricas en tiempo real.

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
