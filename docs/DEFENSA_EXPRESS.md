# DEFENSA EXPRESS — Día de la Presentación (MV ya lista)

> Asume: MV creada, Ubuntu instalado, Docker + Minikube + kubectl instalados,
> repositorio clonado, NAT configurado. Solo hay que **arrancar y desplegar**.

**Duración total de setup: ~4 minutos**

---

## PASO 1 — Encender MV y conectar SSH (30s)

```powershell
ssh asir@localhost -p 2222
```

Contraseña: `asir`

---

## PASO 2 — Arrancar Minikube (1 min)

```bash
minikube start --driver=docker --memory=2048 --cpus=2
minikube status
```

✅ Esperado: `host: Running`, `kubelet: Running`, `apiserver: Running`

---

## PASO 3 — Desplegar todo el stack (30s)

```bash
cd ~/proyecto-intermodular-asir
kubectl apply -f manifests/ -R
```

> ⚠️ El `-R` es obligatorio por las subcarpetas.

---

## PASO 4 — Esperar pods Running (1 min)

```bash
kubectl get pods -w
```

Esperar hasta que los 4 pods estén `Running` (2 nginx-web + 1 prometheus + 1 grafana). `Ctrl+C` para salir.

---

## PASO 5 — Port-forwards (30s)

Una sola SSH con procesos en segundo plano:

```bash
kubectl port-forward svc/nginx-web 30000:80 --address 0.0.0.0 &
kubectl port-forward svc/grafana 32000:3000 --address 0.0.0.0 &
kubectl port-forward svc/prometheus 31000:9090 --address 0.0.0.0 &
```

Verificar:

```bash
ps aux | grep port-forward
```

> Deben aparecer 3 procesos.

---

## PASO 6 — Verificar en navegador (Windows)

| Servicio | URL | Esperado |
|----------|-----|----------|
| Nginx | `http://localhost:8080` | "Welcome to nginx!" |
| Prometheus | `http://localhost:9090/targets` | Targets UP (verde) |
| Grafana | `http://localhost:3000` | Login admin/admin |

> Si Prometheus o Grafana no cargan: espera 10-20s más (tardan en iniciar).

---

## DEMO DE ALTA DISPONIBILIDAD (1 min)

```bash
kubectl get pods                           # estado actual
kubectl delete pod -l app=nginx-web --force # matar un pod
kubectl get pods -w                        # ver recreación automática
```

---

## CHECKLIST 2 MINUTOS ANTES

- [ ] `minikube status` → Running
- [ ] `kubectl get pods` → 4/4 Running
- [ ] 3 port-forwards activos
- [ ] `http://localhost:8080` funciona
- [ ] `http://localhost:9090` funciona
- [ ] `http://localhost:3000` funciona (admin/admin)
- [ ] Dashboard Grafana importado y con datos

---

## RECUPERACIÓN RÁPIDA SI ALGO FALLA

| Problema | Solución |
|----------|----------|
| Pods no arrancan | `kubectl describe pod <nombre>` → Events |
| Port-forward da "address already in use" | `pkill -f "port-forward"` y repetir |
| `connection refused` en navegador | ¿NAT configurado? ¿port-forward corriendo? |
| Minikube no arranca | `docker ps` → si Docker no corre → `sudo systemctl start docker` |

---

*Tiempo total desde SSH hasta tener todo listo: ~4 minutos. Ahorra lo demás para la explicación.*
