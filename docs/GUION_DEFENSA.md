# GUION_DEFENSA.md — Guion de Exposición Oral

> **Duración total estimada:** 10-15 minutos
> **Formato:** Presentación + Demo en vivo
> **Orden de intervención:** Adrián → Santiago → Sergio → (todos en demo)

---

## 0. PREPARACIÓN (5 min antes de empezar)

- [ ] VM encendida y SSH conectado
- [ ] `minikube status` → Running
- [ ] `kubectl get pods` → todos Running (1/1)
- [ ] NAT VirtualBox configurado (8080→30000, 9090→31000, 3000→32000)
- [ ] Navegador con pestañas: localhost:8080, localhost:9090/targets, localhost:3000
- [ ] Terminal con `kubectl get pods -w` preparado para demo de alta disponibilidad

---

## 1. INTRODUCCIÓN — ¿Qué es esto? (1 min) — *Cualquiera*

**Texto sugerido:**

> "Buenos días, somos Adrián Boza, Santiago Pérez y Sergio López, alumnos de 2º de ASIR. Os presentamos nuestro proyecto intermodular: **Implantación de un Entorno Kubernetes Local con Aplicación Web Escalable y Monitoreo en Tiempo Real**.
>
> El objetivo era simular, a pequeña escala, cómo una empresa real despliega y opera una aplicación web usando Kubernetes, pero ejecutándolo todo en un portátil con una máquina virtual de Ubuntu Server."

---

## 2. PROBLEMA Y SOLUCIÓN (1 min) — *Cualquiera*

**Texto sugerido:**

> "Tradicionalmente, desplegar una web en un servidor implica:
> - Configuración manual → errores humanos
> - Si el servidor se cae, la web deja de funcionar
> - Para actualizar Nginx, hay que parar el servicio
> - La monitorización es reactiva
>
> Nosotros resolvemos esto con Kubernetes: declaramos el estado deseado en archivos YAML y Kubernetes se encarga de mantenerlo. Si un pod falla, lo recrea automáticamente. Si queremos escalar, cambiamos un número en el YAML."

---

## 3. ARQUITECTURA DEL PROYECTO (2 min) — *Adrián*

**Apoyarse en el diagrama de arquitectura (diapositiva).**

**Texto sugerido:**

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

## 4. FASE 1 — INFRAESTRUCTURA Y APP WEB (2 min) — *Adrián*

**Texto sugerido:**

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

## 5. FASE 2 — MONITOREO (2 min) — *Santiago*

**Texto sugerido:**

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

## 6. FASE 3 — SEGURIDAD (2 min) — *Sergio*

**Texto sugerido:**

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

## 7. DEMO EN VIVO (3 min) — *Quien maneje el teclado*

**Preparar:** Terminal dividida en 3 paneles (pods, servicios, navegador).

### 7.1 Mostrar estado del cluster (30s)

```bash
kubectl get pods
kubectl get svc
kubectl get networkpolicies
```

**Texto:** "Aquí vemos los 4 pods en ejecución: 2 de Nginx, 1 de Prometheus, 1 de Grafana. Todos en estado Running. Los servicios están expuestos en sus NodePorts correspondientes. Y las 5 políticas de red aplicadas."

### 7.2 Mostrar Nginx en navegador (30s)

Abrir `http://localhost:8080`.

**Texto:** "Nuestra web funcionando. Es el Nginx por defecto, pero podría ser cualquier aplicación."

### 7.3 Mostrar Prometheus targets UP (30s)

Abrir `http://localhost:9090/targets`.

**Texto:** "Aquí vemos que Prometheus está scrapeando correctamente. Los targets aparecen en verde (UP)."

### 7.4 Mostrar Grafana (30s)

Abrir `http://localhost:3000` → login admin/admin → Dashboard.

**Texto:** "Grafana con el dashboard importado mostrando métricas en tiempo real de nuestro cluster."

### 7.5 Demo de Alta Disponibilidad (1 min)

```bash
kubectl delete pod -l app=nginx-web --force
kubectl get pods -w   # (mostrar cómo aparece uno nuevo automáticamente)
```

**Texto:** "Voy a borrar uno de los pods de Nginx simulando un fallo. Observad cómo Kubernetes lo recrea automáticamente en segundos. Esto es el auto-healing."

---

## 8. DIFICULTADES Y SOLUCIONES (1 min) — *Cada uno cuenta una*

| Integrante | Dificultad | Solución |
|---|---|---|
| Adrián | Minikube no arrancaba con 1GB RAM | Aumentar a 2GB |
| Santiago | Prometheus no scrapeaba Nginx | NetworkPolicy bloqueaba el tráfico — añadir regla allow-prometheus-scrape |
| Sergio | Error al hacer `kubectl apply -f manifests/` sin -R | Leer la documentación y añadir flag -R para subdirectorios |
| Todos | NAT mal configurado → ERR_CONNECTION_REFUSED | Verificar puertos anfitrión/invitado en VirtualBox |

---

## 9. CONCLUSIONES Y MEJORAS FUTURAS (1 min) — *Cualquiera*

**Texto sugerido:**

> "Para terminar, conclusiones:
>
> 1. Hemos demostrado que se puede montar un cluster Kubernetes funcional con monitorización y seguridad en un portátil con recursos limitados.
> 2. Las NetworkPolicies son una herramienta potente pero hay que planificarlas bien.
> 3. La infraestructura como código (YAML versionado) hace que todo sea reproducible.
>
> Como mejoras futuras: añadiríamos un Ingress Controller con HTTPS, escalado automático con HPA, persistencia con PVCs para los datos de Prometheus, y un pipeline CI/CD con GitHub Actions. Pero para el alcance del proyecto, estamos muy satisfechos con el resultado."
>

---

## 10. PREGUNTAS DEL TRIBUNAL (preparación)

Posibles preguntas y respuestas preparadas:

| Pregunta | Respuesta |
|---|---|
| ¿Por qué Minikube y no k3s o kind? | Minikube es el estándar educativo, tiene buena documentación y es el que hemos visto en clase. |
| ¿Cuánto consume el cluster? | Aprox 1.2GB RAM entre todos los pods + sistema. |
| ¿Y si un pod de Nginx falla de verdad? | Kubernetes lo reinicia automáticamente (livenessProbe) o lo recrea (ReplicaSet). |
| ¿Qué seguridad tiene Grafana? | Solo credenciales básicas admin/admin. En producción usaríamos OAuth o secrets externos. |
| ¿Por qué no usáis namespaces? | Para este proyecto usamos el namespace default por simplicidad. En producción separaríamos por entorno. |
| ¿Cómo sabéis que las NetworkPolicies funcionan? | Lo testeamos con `kubectl exec` entre pods antes y después de aplicar las políticas. |
| ¿Qué versión de Kubernetes usáis? | v1.35.1 con Minikube v1.38.1 sobre Ubuntu Server 24.04. |
| ¿Podríais desplegar esto en la nube? | Sí, los mismos YAMLs funcionan en AKS, EKS o GKE cambiando el tipo de Service a LoadBalancer. |

---

## Checklist Pre-Defensa

- [ ] Diapositivas preparadas y probadas en el proyector
- [ ] Demo grabada (por si el WiFi falla)
- [ ] Capturas de pantalla en `img/` accesibles
- [ ] GitHub abierto en el repositorio
- [ ] VM arrancada y Minikube funcionando
- [ ] NAT de VirtualBox configurado
- [ ] Todos los servicios accesibles desde el navegador
- [ ] Comando `kubectl delete pod -l app=nginx-web --force` copiado en portapapeles
- [ ] Cada miembro sabe qué va a decir y en qué orden
- [ ] Reloj/temporizador visible para controlar 15 minutos
