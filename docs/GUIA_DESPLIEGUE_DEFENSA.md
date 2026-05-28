# GUÍA DE DESPLIEGUE PARA LA DEFENSA — MV desde cero

> Documento de ensayo para levantar el proyecto completo en una máquina virtual recién creada.
> Sigue estos pasos EN ORDEN. Cada paso indica **qué escribir**, **dónde** y **qué comprobar**.

---

## FASE 0 — CREACIÓN DE LA MV (VirtualBox)

### 0.1 Crear la máquina virtual

| Campo | Valor |
|-------|-------|
| Nombre | `Ubuntu-Server-ASIR` |
| Tipo | Linux |
| Versión | Ubuntu (64-bit) |
| RAM | **4096 MB** |
| Disco duro | **25 GB** (VDI, dinámico) |

**Pasos exactos en VirtualBox:**
1. Abre VirtualBox → Botón **"Nueva"**
2. Nombre: `Ubuntu-Server-ASIR` → Tipo: Linux → Versión: Ubuntu (64-bit) → Siguiente
3. RAM: `4096` → Siguiente
4. "Crear un disco duro virtual ahora" → Crear
5. Tipo de archivo: **VDI** → Siguiente
6. "Dinámicamente reservado" → Siguiente
7. Ubicación: dejar por defecto → Tamaño: `25,00 GB` → Crear

### 0.2 Configurar la ISO de instalación

1. Selecciona la MV → **Configuración** → **Almacenamiento**
2. Controlador: IDE → disco vacío → icono disco → "Elegir un archivo de disco"
3. Selecciona la ISO de **Ubuntu Server 24.04 LTS** → Aceptar

### 0.3 Configurar red en modo NAT (por defecto)

**Configuración** → **Red** → **Adaptador 1** → `NAT`

No toques nada más. El NAT se configura por defecto.

### 0.4 Configurar reenvío de puertos ANTES de arrancar

1. **Configuración** → **Red** → **Adaptador 1** → **NAT** → **"Reenvío de puertos"**
2. Añade **4 reglas** con este orden exacto:

| Nombre | Protocolo | Puerto Host | Puerto Invitado |
|--------|-----------|------------|----------------|
| SSH | TCP | `2222` | `22` |
| nginx-web | TCP | `8080` | `30000` |
| grafana | TCP | `3000` | `32000` |
| prometheus | TCP | `9090` | `31000` |

> **Importante:** Host IP y Guest IP se dejan VACÍAS.

---

## FASE 1 — INSTALACIÓN DE UBUNTU SERVER 24.04

### 1.1 Arrancar e instalar

1. Selecciona la MV → **Iniciar**
2. Espera a que cargue el menú GRUB → elige **"Try or Install Ubuntu Server"**
3. Idioma: **English** (o Español, da igual)
4. Keyboard layout: **Spanish** (o el tuyo)
5. Network connections: **dejar DHCP** (la NAT asigna IP automática)

### 1.2 Configurar el instalador

6. **Proxy address:** dejar vacío → Done
7. **Ubuntu archive mirror:** dejar por defecto → Done
8. **Storage layout:** "Use an entire disk" → disco de 25GB → Done
   - Confirmar: **Continue** (te avisa que va a borrar el disco)
9. **Profile setup:**
   - Your name: `asir`
   - Your server's name: `ubuntu-server`
   - Pick a username: `asir`
   - Password: `asir`
   - Confirm password: `asir`
10. **SSH Setup:** marca **"Install OpenSSH server"** (IMPORTANTE)
11. **Featured Server Snaps:** no marcar nada → Done
12. Esperar a que termine la instalación (~5-10 min)
13. **Reboot** (te pedirá quitar la ISO)

> ⚠️ Si no te deja hacer SSH, verifica en la MV con `ip a` que tienes IP. Con NAT la IP suele ser `10.0.2.15`.

---

## FASE 2 — CONFIGURACIÓN INICIAL DE LA VM

### 2.1 Conectar por SSH (desde tu host Windows)

Abre **PowerShell** o **cmd** en tu host:

```powershell
ssh asir@localhost -p 2222
```

Contraseña: `asir`

> Si da error de fingerprint, responde `yes`.

### 2.2 Actualizar el sistema

```bash
sudo apt update && sudo apt upgrade -y
```

> ⏱ ~2-3 minutos. Responde lo que sea necesario.

### 2.3 Instalar dependencias base

```bash
sudo apt install -y curl wget git
```

---

## FASE 3 — INSTALAR DOCKER ENGINE

### 3.1 Eliminar paquetes antiguos (si los hay)

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove -y $pkg; done
```

### 3.2 Añadir el repositorio oficial de Docker

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### 3.3 Instalar Docker Engine

```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 3.4 Añadir tu usuario al grupo docker (para no usar sudo)

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 3.5 Verificar que Docker funciona

```bash
docker --version
docker run hello-world
```

> Output esperado: `"Hello from Docker!"`

### 3.6 Habilitar Docker para que arranque solo

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## FASE 4 — INSTALAR MINIKUBE

### 4.1 Descargar e instalar Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

### 4.2 Verificar instalación

```bash
minikube version
```

> Output ejemplo: `minikube version: v1.38.1`

---

## FASE 5 — INSTALAR KUBECTL

### 5.1 Descargar e instalar kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl
```

### 5.2 Verificar instalación

```bash
kubectl version --client
```

> Output ejemplo: `Client Version: v1.35.1`

---

## FASE 6 — ARRANCAR MINIKUBE

### 6.1 Iniciar el clúster

```bash
minikube start --driver=docker --memory=2048 --cpus=2
```

> ⏱ ~2-4 minutos la primera vez (descarga imágenes).

### 6.2 Verificar estado

```bash
minikube status
```

Output esperado:
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
```

---

## FASE 7 — CLONAR EL REPOSITORIO

### 7.1 Clonar

```bash
git clone https://github.com/adrianboza2/proyecto-intermodular-asir.git
cd proyecto-intermodular-asir
```

### 7.2 Ver la estructura

```bash
ls -la
```

Debes ver: `manifests/`, `docs/`, `img/`, `README.md`, etc.

---

## FASE 8 — DESPLEGAR TODO EL STACK

### 8.1 Aplicar todos los manifiestos

```bash
kubectl apply -f manifests/ -R
```

> ⚠️ El flag `-R` es OBLIGATORIO porque `manifests/` contiene subcarpetas (`app-web/`, `monitoring/`, `network/`).

### 8.2 Verificar que los recursos se crearon

```bash
kubectl get pods
kubectl get svc
kubectl get networkpolicies
```

### 8.3 Esperar a que todos los pods estén Running

```bash
kubectl get pods -w
```

Espera hasta ver esto:

| NAME | READY | STATUS | RESTARTS |
|------|-------|--------|----------|
| nginx-web-xxx-xxx | 1/1 | Running | 0 |
| nginx-web-xxx-xxx | 1/1 | Running | 0 |
| prometheus-xxx-xxx | 1/1 | Running | 0 |
| grafana-xxx-xxx | 1/1 | Running | 0 |

Pulsa `Ctrl+C` para salir del modo watch.

> ⏱ Espera hasta ~2-3 minutos máximo. Si algún pod se queda en `Pending` o `CrashLoopBackOff`:
> ```bash
> kubectl describe pod <nombre-del-pod>
> ```
> Busca en `Events` el motivo. Generalmente es falta de recursos.

### 8.4 Outputs esperados de verificación

**`kubectl get svc`:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP                          5m
nginx-web    NodePort    10.96.xxx.xxx   <none>        80:30000/TCP                     2m
prometheus   NodePort    10.96.xxx.xxx   <none>        9090:31000/TCP                   2m
grafana      NodePort    10.96.xxx.xxx   <none>        3000:32000/TCP                   2m
```

**`kubectl get networkpolicies`:**
```
NAME                        POD-SELECTOR     AGE
allow-dns                   <none>           1m
allow-grafana-ingress       app=grafana      1m
allow-monitoring-ingress    app=prometheus   1m
allow-nginx-web             app=nginx-web    1m
allow-prometheus-scrape     app=prometheus   1m
default-deny-all            <none>           1m
```

---

## FASE 9 — ACCEDER A LOS SERVICIOS DESDE EL HOST (port-forward)

> ⚠️ **IMPORTANTE:** Con Minikube usando driver Docker, los NodePorts quedan dentro del contenedor de
> Minikube y NO son accesibles directamente desde la VM ni desde el host Windows.
> La solución es usar `kubectl port-forward` combinado con el NAT de VirtualBox.

### 9.1 Abrir 3 ventanas SSH (método recomendado para la defensa)

Abre **3 PowerShell** en Windows:

| Ventana | Comando |
|---------|---------|
| **Terminal 1** | `ssh asir@localhost -p 2222` → `kubectl port-forward svc/nginx-web 30000:80 --address 0.0.0.0` |
| **Terminal 2** | `ssh asir@localhost -p 2222` → `kubectl port-forward svc/grafana 32000:3000 --address 0.0.0.0` |
| **Terminal 3** | `ssh asir@localhost -p 2222` → `kubectl port-forward svc/prometheus 31000:9090 --address 0.0.0.0` |

> El comando `port-forward` se queda ejecutándose y bloquea la terminal.
> Deja las 3 ventanas abiertas durante toda la defensa.

### 9.2 Alternativa rápida (todo en 1 SSH con procesos en segundo plano)

```bash
kubectl port-forward svc/nginx-web 30000:80 --address 0.0.0.0 &
kubectl port-forward svc/grafana 32000:3000 --address 0.0.0.0 &
kubectl port-forward svc/prometheus 31000:9090 --address 0.0.0.0 &
```

> El `&` lanza cada comando en segundo plano. Para verificar que están corriendo:
> ```bash
> ps aux | grep port-forward
> ```

### 9.3 Probar desde el navegador (Windows)

| Servicio | URL | Esperado |
|----------|-----|----------|
| Nginx | `http://localhost:8080` | "Welcome to nginx!" |
| Prometheus | `http://localhost:9090/targets` | Targets en verde (UP) |
| Grafana | `http://localhost:3000` | Login (admin / admin) |

> ✅ El NAT de VirtualBox ya tiene las reglas: Host → Guest:
> - 8080 → 30000 (Nginx)
> - 9090 → 31000 (Prometheus)
> - 3000 → 32000 (Grafana)

### 9.4 Conectar datasource Prometheus en Grafana

1. Ve a **Connections** → **Data sources** → **Add data source**
2. Elige **Prometheus**
3. En **Prometheus server URL**: `http://prometheus:9090`
4. **Save & test**
5. ✅ Esperado: `"Data source is working"`

### 9.5 Importar dashboard

1. **Dashboards** → **Import**
2. ID: `3662` (Prometheus 2.0 Overview)
3. Seleccionar datasource: Prometheus
4. **Import**

---

## FASE 10 — DEMO DE ALTA DISPONIBILIDAD (para la defensa)

### 10.1 Mostrar estado actual

```bash
kubectl get pods
kubectl get svc
kubectl get networkpolicies
```

### 10.2 Eliminar un pod de Nginx (simular fallo)

```bash
kubectl delete pod -l app=nginx-web --force
```

### 10.3 Ver la auto-recuperación en tiempo real

```bash
kubectl get pods -w
```

Verás cómo el pod eliminado desaparece y Kubernetes crea uno nuevo automáticamente en segundos.

---

## ❗ CHECKLIST RÁPIDO PRE-DEFENSA

10 minutos antes de empezar:

- [ ] `minikube status` → host Running, kubelet Running, apiserver Running
- [ ] `kubectl get pods` → todos Running (4 pods)
- [ ] `ssh asir@localhost -p 2222` funciona
- [ ] 3 port-forwards corriendo (nginx-web:30000, grafana:32000, prometheus:31000)
- [ ] `http://localhost:8080` → Welcome to nginx
- [ ] `http://localhost:9090/targets` → targets UP
- [ ] `http://localhost:3000` → login admin/admin funciona
- [ ] Dashboard de Grafana con datos visibles

---

## 🐛 SOLUCIÓN DE ERRORES COMUNES

| Error | Causa | Solución |
|-------|-------|----------|
| `kubectl apply -f manifests/` da error | Falta `-R` | Usar `kubectl apply -f manifests/ -R` |
| `ERR_CONNECTION_REFUSED` en navegador | NAT no configurado o port-forward no iniciado | VirtualBox → Reenvío de puertos + `kubectl port-forward svc/... --address 0.0.0.0` |
| Pod en `Pending` | Faltan recursos | `kubectl describe pod <nombre>` para diagnósticar |
| `Destination path already exists` al clonar | Repo ya existe | `git pull origin main` en vez de clone |
| `docker: permission denied` | Usuario no está en grupo docker | `sudo usermod -aG docker $USER && newgrp docker` |
| Minikube no arranca | Docker no está corriendo | `sudo systemctl status docker` → si no, `sudo systemctl start docker` |
| Dashboard sin datos | NetworkPolicy bloquea | Verificar que `allow-prometheus-scrape` existe |

---

*Documento generado para ensayo de defensa — Proyecto Intermodular ASIR 2025/2026*
