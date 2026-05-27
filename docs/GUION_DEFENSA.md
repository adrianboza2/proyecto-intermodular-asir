# 🎤 GUION_DEFENSA.md — Guion Completo de Exposición Oral

> **Duración total:** 10-15 minutos
> **Formato:** Presentación (PPTX) + Demo en vivo
> **Orden de intervención:** Adrián → Santiago → Sergio → (todos demo)
> **Fechas defensa:** 28 o 29 de mayo
> **Presentación:** `docs/Presentacion_Proyecto_Intermodular.pptx`

---

## Índice

1. [Prerrequisitos del Portátil](#1-prerrequisitos-del-portátil)
2. [Checklist Pre-Defensa](#2-checklist-pre-defensa)
3. [Despliegue en Vivo](#3-despliegue-en-vivo)
4. [Verificación de Servicios](#4-verificación-de-servicios)
5. [Guion de Exposición Oral](#5-guion-de-exposición-oral)
6. [Demo de Alta Disponibilidad](#6-demo-de-alta-disponibilidad)
7. [Errores Comunes y Soluciones](#7-errores-comunes-y-soluciones)
8. [Preguntas del Tribunal (Q&A)](#8-preguntas-del-tribunal-qa)

---

## 1. Prerrequisitos del Portátil

El portátil del día de la defensa debe tener instalado:

- [ ] VirtualBox 7.x
- [ ] VM Ubuntu Server 24.04 LTS con Docker Engine, Minikube y kubectl
- [ ] Proyecto clonado (`git clone https://github.com/adrianboza2/proyecto-intermodular-asir.git`)
- [ ] Reenvío de puertos NAT configurado (ver sección 3.5)

> Si el repo ya existe: `git pull origin main`

---

## 2. Checklist Pre-Defensa

**Márcatelo 10 minutos antes de empezar:**

### Estado del sistema
- [ ] VM encendida y SSH conectado (`ssh usuario@localhost -p 2222`)
- [ ] Minikube running (`minikube status`)
- [ ] Todos los pods `Running` (`kubectl get pods`)
- [ ] VirtualBox NAT configurado (8080→30000, 9090→31000, 3000→32000)
- [ ] Terminal con `kubectl get pods -w` preparado para demo de HA

### Acceso desde navegador
- [ ] Nginx responde en `http://localhost:8080`
- [ ] Prometheus targets UP en `http://localhost:9090/targets`
- [ ] Grafana login ok en `http://localhost:3000` (admin/admin)
- [ ] Dashboard importado y con datos

### Material de apoyo
- [ ] Presentación `docs/Presentacion_Proyecto_Intermodular.pptx` abierta
- [ ] Memoria final `docs/Kubernetes_AdrianBS_SantiagoPC_SergioLP.pdf` disponible
- [ ] Capturas de evidencia en `img/` (11 archivos)
- [ ] Repositorio GitHub abierto para mostrar commits
- [ ] Demo grabada (por si falla el directo)
- [ ] Reloj/temporizador visible para controlar los 15 minutos

### Reparto de intervenciones
- [ ] Cada miembro sabe qué va a decir y en qué orden
- [ ] Comando de HA copiado en portapapeles

---

## 3. Despliegue en Vivo

> Pasos para levantar el proyecto desde cero durante la defensa (si hiciese falta).

### 3.1 Clonar y arrancar Minikube

```bash
git clone https://github.com/adrianboza2/proyecto-intermodular-asir.git
cd proyecto-intermodular-asir

minikube start --driver=docker --memory=2048 --cpus=2
minikube status
# → host: Running, kubelet: Running, apiserver: Running
```

### 3.2 Desplegar todo el stack

```bash
kubectl apply -f manifests/ -R
```

> **Importante:** el flag `-R` es necesario porque hay subcarpetas (app-web, monitoring, network). Sin él da error `recognized file extensions`.

### 3.3 Esperar a que los pods arranquen

```bash
kubectl get pods -w
# Esperar hasta que todos muestren STATUS Running y READY 1/1
# Ctrl+C para salir del modo watch
```

### 3.4 Configurar NAT en VirtualBox

Abrir VirtualBox → VM → Configuración → Red → NAT → Reenvío de puertos → Añadir:

| Nombre | Protocolo | Host Port | Guest Port |
|--------|-----------|-----------|------------|
| nginx-web | TCP | 8080 | 30000 |
| prometheus | TCP | 9090 | 31000 |
| grafana | TCP | 3000 | 32000 |

> **Importante:** Host IP y Guest IP se dejan vacías.
> **Error típico:** `ERR_CONNECTION_REFUSED` → el NAT no está configurado.

---

## 4. Verificación de Servicios

### 4.1 Pods

```bash
kubectl get pods
```

Salida esperada: 4 pods en estado Running (2 nginx-web + 1 prometheus + 1 grafana).

### 4.2 Servicios

```bash
kubectl get svc
```

| Nombre | Type | Port(s) |
|--------|------|---------|
| nginx-web | NodePort | 80:30000/TCP |
| prometheus | NodePort | 9090:31000/TCP |
| grafana | NodePort | 3000:32000/TCP |
| kubernetes | ClusterIP | 443/TCP |

### 4.3 NetworkPolicies

```bash
kubectl get networkpolicies
```

Salida esperada: 5 políticas (default-deny-all, allow-nginx-web, allow-monitoring-ingress x2, allow-prometheus-scrape, allow-dns).

### 4.4 Probar en el navegador

| Servicio | URL | Esperado |
|----------|-----|----------|
| Nginx | `http://localhost:8080` | "Welcome to nginx!" |
| Prometheus | `http://localhost:9090` | UI de Prometheus |
| Grafana | `http://localhost:3000` | Login (admin / admin) |

### 4.5 Verificar Prometheus targets

1. Abrir `http://localhost:9090`
2. Ir a **Status** → **Targets**
3. Verificar que los targets aparecen como `UP` (verde)

### 4.6 Verificar Grafana

1. Abrir `http://localhost:3000` → admin / admin
2. Ir a **Connections** → **Data sources** → Prometheus
3. URL: `http://prometheus:9090` → Save & test → "Data source is working"
4. Ir a **Dashboards** → **Import** → ID `3662` (Prometheus 2.0 Overview)
5. Si no hay datos: ir a **Explore** → query `up` → Run

---

## 5. Guion de Exposición Oral

### 5.1 INTRODUCCIÓN — ¿Qué es esto? (1 min)

**Quién:** Cualquiera

**Texto sugerido:**

> "Buenos días, somos Adrián Boza, Santiago Pérez y Sergio López, alumnos de 2º de ASIR. Os presentamos nuestro proyecto intermodular: **Implantación de un Entorno Kubernetes Local con Aplicación Web Escalable y Monitoreo en Tiempo Real**.
>
> El objetivo era simular, a pequeña escala, cómo una empresa real despliega y opera una aplicación web usando Kubernetes, pero ejecutándolo todo en un portátil con una máquina virtual de Ubuntu Server."

---

### 5.2 PROBLEMA Y SOLUCIÓN (1 min)

**Quién:** Cualquiera

**Texto sugerido:**

> "Tradicionalmente, desplegar una web en un servidor implica:
> - Configuración manual → errores humanos
> - Si el servidor se cae, la web deja de funcionar
> - Para actualizar Nginx, hay que parar el servicio
> - La monitorización es reactiva
>
> Nosotros resolvemos esto con Kubernetes: declaramos el estado deseado en archivos YAML y Kubernetes se encarga de mantenerlo. Si un pod falla, lo recrea automáticamente. Si queremos escalar, cambiamos un número en el YAML."

---

### 5.3 ARQUITECTURA DEL PROYECTO (2 min)

**Quién:** Adrián

**Apoyarse en el diagrama de arquitectura (diapositiva 3).**

> "La arquitectura es en 4 capas:
>
> **Capa 1 — Hardware:** Portátil Windows con VirtualBox.
>
> **Capa 2 — VM:** Ubuntu Server 24.04 con Docker Engine. Configuramos reenvío de puertos para acceder desde el navegador del host.
>
> **Capa 3 — Kubernetes:** Minikube con 1 nodo, 2GB RAM, 2 CPUs. Aquí se ejecutan todos los componentes.
>
> **Capa 4 — Aplicaciones:** Desplegamos 3 servicios:
> - **Nginx** (2 réplicas) — nuestra aplicación web
> - **Prometheus** (1 réplica) — recopila métricas
> - **Grafana** (1 réplica) — visualiza métricas en dashboards
>
> Todo está protegido por 5 políticas de red que implementan un modelo zero-trust: nada entra ni sale a menos que esté explícitamente permitido."

---

### 5.4 FASE 1 — INFRAESTRUCTURA Y APP WEB (2 min)

**Quién:** Adrián

> "Yo me encargué de la infraestructura y el despliegue de la aplicación web. Los puntos clave:
>
> **Deployment de Nginx (`app-web/deployment.yaml`):**
> - 2 réplicas para alta disponibilidad
> - Health checks con livenessProbe (cada 10s) y readinessProbe (cada 5s)
> - Límites de recursos: 128Mi RAM y 250m CPU por pod
>
> **Service (`app-web/service.yaml`):**
> - Tipo NodePort en el puerto 30000
> - Accesible desde el host mediante NAT: `localhost:8080`
>
> **Alta disponibilidad:** Si borramos un pod, el ReplicaSet lo recrea en menos de 60 segundos. Lo veremos en la demo."

---

### 5.5 FASE 2 — MONITOREO (2 min)

**Quién:** Santiago

> "Yo configuré el stack de monitorización con Prometheus y Grafana.
>
> **Prometheus (`monitoring/prometheus.yaml`):**
> - Desplegamos Prometheus con su propia ServiceAccount, ClusterRole y ClusterRoleBinding para que pueda acceder a la API de Kubernetes.
> - Tiene un ConfigMap con la configuración de scraping: descubre automáticamente pods que tengan anotaciones `prometheus.io/scrape=true`, y también obtiene métricas del nodo con cAdvisor.
> - Lo exponemos por NodePort 31000.
>
> **Grafana (`monitoring/grafana.yaml`):**
> - Una réplica con credenciales por defecto admin/admin.
> - Lo conectamos al datasource de Prometheus usando el nombre del servicio interno: `http://prometheus:9090`.
> - Importamos el dashboard oficial **Prometheus 2.0 Overview** (ID 3662) que ya viene preconfigurado con gráficos de CPU, memoria, etc."

---

### 5.6 FASE 3 — SEGURIDAD (2 min)

**Quién:** Sergio

> "Yo implementé la seguridad de red con 5 NetworkPolicies que siguen el modelo zero-trust:
>
> **1. `default-deny-all`** — Bloquea todo el tráfico entrante y saliente de todos los pods. Es la base.
>
> **2. `allow-nginx-web`** — Permite tráfico entrante al puerto 80 de Nginx.
>
> **3. `allow-monitoring-ingress`** — Permite tráfico al puerto 9090 de Prometheus y al 3000 de Grafana.
>
> **4. `allow-prometheus-scrape`** — Permite que Prometheus (app=prometheus) pueda salir al puerto 80 de Nginx para recoger métricas.
>
> **5. `allow-dns`** — Permite tráfico DNS (UDP y TCP 53) para que los pods puedan resolver nombres.
>
> (Mostrar captura `networkpolicy-list.png`) Aquí se ve la lista de políticas aplicadas. Sin estas reglas, los pods no pueden ni comunicarse entre sí."

---

### 5.7 DIFICULTADES Y SOLUCIONES (1 min)

**Quién:** Cada uno cuenta una

| Integrante | Dificultad | Solución |
|---|---|---|
| Adrián | Minikube no arrancaba con 1GB RAM | Aumentar a 2GB |
| Santiago | Prometheus no scrapeaba Nginx | NetworkPolicy bloqueaba el tráfico — añadir regla allow-prometheus-scrape |
| Sergio | Error al hacer `kubectl apply -f manifests/` sin -R | Leer la documentación y añadir flag -R para subdirectorios |
| Todo el equipo | NAT mal configurado → ERR_CONNECTION_REFUSED | Verificar puertos anfitrión/invitado en VirtualBox |

---

### 5.8 CONCLUSIONES Y MEJORAS FUTURAS (1 min)

**Quién:** Cualquiera

> "Para terminar, conclusiones:
>
> 1. Hemos demostrado que se puede montar un cluster Kubernetes funcional con monitorización y seguridad en un portátil con recursos limitados.
> 2. Las NetworkPolicies son una herramienta potente pero hay que planificarlas bien.
> 3. La infraestructura como código (YAML versionado) hace que todo sea reproducible.
>
> Como mejoras futuras: añadiríamos un Ingress Controller con HTTPS, escalado automático con HPA, persistencia con PVCs para los datos de Prometheus, y un pipeline CI/CD con GitHub Actions. Pero para el alcance del proyecto, estamos muy satisfechos con el resultado."

---

## 6. Demo de Alta Disponibilidad

**Preparar:** Terminal dividida en 3 paneles (pods, servicios, navegador).

### 6.1 Mostrar estado del cluster (30s)

```bash
kubectl get pods
kubectl get svc
kubectl get networkpolicies
```

**Texto:** "Aquí vemos los 4 pods en ejecución: 2 de Nginx, 1 de Prometheus, 1 de Grafana. Todos en estado Running. Los servicios están expuestos en sus NodePorts correspondientes. Y las 5 políticas de red aplicadas."

### 6.2 Mostrar Nginx en navegador (30s)

Abrir `http://localhost:8080`.

**Texto:** "Nuestra web funcionando. Es el Nginx por defecto, pero podría ser cualquier aplicación."

### 6.3 Mostrar Prometheus targets UP (30s)

Abrir `http://localhost:9090/targets`.

**Texto:** "Aquí vemos que Prometheus está scrapeando correctamente. Los targets aparecen en verde (UP)."

### 6.4 Mostrar Grafana (30s)

Abrir `http://localhost:3000` → login admin/admin → Dashboard.

**Texto:** "Grafana con el dashboard importado mostrando métricas en tiempo real de nuestro cluster."

### 6.5 Demo de Alta Disponibilidad (1 min)

```bash
kubectl delete pod -l app=nginx-web --force
kubectl get pods -w   # (mostrar cómo aparece uno nuevo automáticamente)
```

**Texto:** "Voy a borrar uno de los pods de Nginx simulando un fallo. Observad cómo Kubernetes lo recrea automáticamente en segundos. Esto es el auto-healing."

---

## 7. Errores Comunes y Soluciones

| Error | Causa | Solución |
|---|---|---|
| `recognized file extensions` | `kubectl apply -f manifests/` sin -R | Añadir `-R` al comando |
| `ERR_CONNECTION_REFUSED` en navegador | NAT de VirtualBox no configurado | Configurar reenvío de puertos (8080→30000, 9090→31000, 3000→32000) |
| `Destination path already exists` | Ya existe el repo clonado | Usar `git pull origin main` en vez de clone |
| Dashboard "No data" | El dashboard 3662 usa métricas internas de Prometheus | Usar **Explore** con query `up` para ver datos reales |
| Minikube no arranca | Docker no está corriendo | `systemctl start docker` o verificar `groups $USER` |
| Pod en estado `Pending` | Falta CPU/RAM | `kubectl describe pod <nombre>` para diagnosticar |

---

## 8. Preguntas del Tribunal (Q&A)

Posibles preguntas y respuestas preparadas:

| Pregunta | Respuesta |
|---|---|
| ¿Por qué Minikube y no k3s o kind? | Minikube es el estándar educativo, tiene buena documentación y es el que hemos visto en clase. |
| ¿Cuánto consume el cluster? | Aprox 1.2 GB RAM entre todos los pods + sistema. |
| ¿Y si un pod de Nginx falla de verdad? | Kubernetes lo reinicia automáticamente (livenessProbe) o lo recrea (ReplicaSet). |
| ¿Qué seguridad tiene Grafana? | Solo credenciales básicas admin/admin. En producción usaríamos OAuth o secrets externos. |
| ¿Por qué no usáis namespaces? | Para este proyecto usamos el namespace default por simplicidad. En producción separaríamos por entorno. |
| ¿Cómo sabéis que las NetworkPolicies funcionan? | Lo testeamos con `kubectl exec` entre pods antes y después de aplicar las políticas. |
| ¿Qué versión de Kubernetes usáis? | v1.35.1 con Minikube v1.38.1 sobre Ubuntu Server 24.04. |
| ¿Podríais desplegar esto en la nube? | Sí, los mismos YAMLs funcionan en AKS, EKS o GKE cambiando el tipo de Service a LoadBalancer. |

---

*Documento generado como parte del proyecto intermodular de 2.º ASIR — Curso 2025/2026*
