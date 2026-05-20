# TASKS.md — Checklist Técnico por Fase

> Archivo de control para seguir el paso a paso (máx 3 acciones por turno).

---

## Fase 1 — Infra + App Web (Adrián) ✅ Completada

| # | Tarea | Estado |
|---|-------|--------|
| 1.1 | `deployment.yaml` — Nginx 2 réplicas + probes + resources | ✅ |
| 1.2 | `service.yaml` — NodePort 30000 | ✅ |
| 1.3 | `kubectl apply -f manifests/app-web/` y verificar pods | ✅ |
| 1.4 | `curl http://localhost:8080` responde HTML | ✅ |

---

## Fase 2 — Monitoring (Santiago) 🔄 En progreso

| # | Tarea | Estado |
|---|-------|--------|
| 2.1 | Validar `prometheus.yaml` (ConfigMap, Deployment, Service 31000) | ⏳ |
| 2.2 | `kubectl apply -f manifests/monitoring/prometheus.yaml` | ⏳ |
| 2.3 | Verificar targets UP en Prometheus (scrapea Nginx) | ⏳ |
| 2.4 | Validar `grafana.yaml` (Deployment, Service 32000, env vars) | ⏳ |
| 2.5 | `kubectl apply -f manifests/monitoring/grafana.yaml` | ⏳ |
| 2.6 | Configurar datasource Prometheus en Grafana | ⏳ |
| 2.7 | Importar dashboard básico en Grafana | ⏳ |
| 2.8 | Verificar `curl http://localhost:9090` y `curl http://localhost:3000` | ⏳ |

---

## Fase 3 — Security + Docs (Sergio) ⏳ Pendiente

| # | Tarea | Estado |
|---|-------|--------|
| 3.1 | Crear `networkpolicy.yaml` (default-deny + allow rules) | ✅ |
| 3.2 | `kubectl apply -f manifests/network/networkpolicy.yaml` | ✅ |
| 3.3 | Testear conectividad (pods app-web → monitoring, external → app-web) | ✅ |
| 3.4 | Redactar memoria técnica en `docs/` | ⏳ |
| 3.5 | Preparar manual de despliegue / defensa | ⏳ |

---

## Fase Transversal — Todos

| # | Tarea | Estado |
|---|-------|--------|
| T.1 | `git pull origin main` antes de cada sesión | ⏳ |
| T.2 | Commit descriptivo (`feat:` / `fix:`) por cambio | ⏳ |
| T.3 | `git push origin main` y verificar en GitHub | ⏳ |
| T.4 | Ensayo de defensa con demo en vivo | ⏳ |
