# 🎤 DEFENSA.md — Guía Paso a Paso para la Exposición Oral

> Fechas defensa: 28 o 29 de mayo
> Portátil del compañero (sin Minikube ni proyecto clonado)

---

## 0. Prerrequisitos (antes de empezar)

- [ ] VirtualBox 7.x instalado
- [ ] VM Ubuntu Server 24.04 LTS preparada
- [ ] Docker Engine instalado en la VM
- [ ] Minikube y kubectl instalados en la VM
- [ ] Reenvío de puertos VirtualBox configurado (ver sección 5)

---

## 1. Clonar el repositorio

```bash
git clone https://github.com/adrianboza2/proyecto-intermodular-asir.git
cd proyecto-intermodular-asir
```

> Si ya existe: `git pull origin main`

---

## 2. Arrancar Minikube

```bash
minikube start --driver=docker --memory=2048 --cpus=2
```

**Verificar:**
```bash
minikube status
# → host: Running, kubelet: Running, apiserver: Running
```

---

## 3. Desplegar todo el stack

```bash
kubectl apply -f manifests/ -R
```

**Explicación:** El flag `-R` es necesario porque hay subcarpetas (app-web, monitoring, network).

**Error típico:** `error: recognized file extensions are [.json .yaml .yml]` — solución: usar `-R`.

---

## 4. Verificar que todo funciona

### 4.1 Pods

```bash
kubectl get pods -w
# Esperar hasta que todos los pods muestren STATUS Running y READY 1/1
# Pulsar Ctrl+C para salir del modo watch
```

### 4.2 Servicios

```bash
kubectl get svc
```

Salida esperada:
| Nombre | Type | Port(s) |
|--------|------|---------|
| nginx-web | NodePort | 80:30000/TCP |
| prometheus | NodePort | 9090:31000/TCP |
| grafana | NodePort | 3000:32000/TCP |
| kubernetes | ClusterIP | 443/TCP |

### 4.3 NetworkPolicy

```bash
kubectl get networkpolicies
```

---

## 5. Configurar NAT en VirtualBox (imprescindible)

> Sin esto, desde el navegador del portátil NO se podrá acceder.

Abrir VirtualBox → VM → Configuración → Red → NAT → Reenvío de puertos → Añadir:

| Nombre | Protocolo | Host Port | Guest Port |
|--------|-----------|-----------|------------|
| nginx-web | TCP | 8080 | 30000 |
| prometheus | TCP | 9090 | 31000 |
| grafana | TCP | 3000 | 32000 |

**Nota:** Host IP y Guest IP se dejan vacías.

**Error típico:** `ERR_CONNECTION_REFUSED` al abrir localhost — el NAT no está configurado o está mal.

---

## 6. Probar en el navegador

Abrir en el navegador del portátil (Windows):

| Servicio | URL | Esperado |
|----------|-----|----------|
| Nginx | `http://localhost:8080` | "Welcome to nginx!" |
| Prometheus | `http://localhost:9090` | UI de Prometheus |
| Grafana | `http://localhost:3000` | Login (admin / admin) |

---

## 7. Verificar Prometheus targets UP

1. Abrir `http://localhost:9090`
2. Ir a **Status** → **Targets**
3. Verificar que nginx-web aparece como `UP` (verde)

---

## 8. Verificar Grafana

1. Abrir `http://localhost:3000` → usuario: `admin`, contraseña: `admin`
2. Ir a **Connections** → **Data sources** → Prometheus
3. URL: `http://prometheus:9090` → Save & test (debe decir "Data source is working")
4. Ir a **Dashboards** → **Import** → ID `3662` (Prometheus 2.0 Overview) → Load → Seleccionar datasource Prometheus → Import
5. Si sale todo "No data", ir a **Explore** (icono brújula) → cambiar a **Code** → escribir `up` → Run query

---

## 9. Demo de Alta Disponibilidad

```bash
# Borrar un pod de Nginx
kubectl delete pod -l app=nginx-web --force

# Ver cómo Kubernetes lo recrea automáticamente
kubectl get pods -w
```

**Explicación:** El ReplicaSet de Kubernetes detecta que falta un pod y lo recrea en menos de 60 segundos. Esto demuestra el auto-healing del cluster.

---

## 10. Errores Comunes y Soluciones

| Error | Causa | Solución |
|-------|-------|----------|
| `recognized file extensions` | `kubectl apply -f manifests/` sin -R | Añadir `-R` al comando |
| `ERR_CONNECTION_REFUSED` en navegador | NAT de VirtualBox no configurado | Configurar reenvío de puertos (sección 5) |
| `Destination path already exists` | Ya existe el repo clonado | Usar `git pull origin main` en vez de clone |
| Dashboard "No data" | El dashboard 3662 usa métricas internas de Prometheus | Usar **Explore** con query `up` para ver datos reales |
| Minikube no arranca | Docker no está corriendo | `systemctl start docker` o verificar `groups $USER` |
| Pod en estado `Pending` | Falta CPU/RAM | `kubectl describe pod <nombre>` para diagnosticar |

---

## 11. Checklist Rápida Pre-Defensa

Marítalo 10 minutos antes de empezar:

- [ ] VM encendida y conectada por SSH (`ssh usuario@localhost -p 2222`)
- [ ] Minikube running (`minikube status`)
- [ ] Todos los pods `Running` (`kubectl get pods`)
- [ ] VirtualBox NAT configurado (8080→30000, 9090→31000, 3000→32000)
- [ ] Nginx responde en `http://localhost:8080`
- [ ] Prometheus targets UP en `http://localhost:9090/targets`
- [ ] Grafana login ok en `http://localhost:3000` (admin/admin)
- [ ] Dashboard importado y con datos
- [ ] Comando de HA listo: `kubectl delete pod -l app=nginx-web --force`
- [ ] Capturas de evidencia en `img/` (11 archivos)
- [ ] Repositorio GitHub abierto para mostrar commits
