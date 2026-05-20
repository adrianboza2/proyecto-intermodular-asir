# 🔄 WORKFLOW.md — Flujo de Trabajo y Validación

## 📦 Git Workflow
1. `git pull origin main` antes de cualquier cambio
2. Trabajar SOLO en la carpeta asignada:
   - Adrián: `manifests/app-web/`
   - Santiago: `manifests/monitoring/`
   - Sergio: `manifests/network/` + `docs/`
3. Commit descriptivo: `feat: add prometheus config` / `fix: correct nodeport`
4. `git push origin main` → Verificar en GitHub

## 🚀 Despliegue Estándar
```bash
minikube start --driver=docker --memory=2048 --cpus=2
kubectl apply -f manifests/<fase>/
kubectl get pods -l app=<label> -w
kubectl port-forward svc/<servicio> <puerto>:<target> --address 0.0.0.0 &

| ✅ Verificación Post-Despliegue | |
|---|---|
| **Comando** | **Output Esperado** |
| `kubectl get pods` | READY 1/1, STATUS Running |
| `kubectl get svc` | TYPE NodePort, PORT 80:XXXXX/TCP |
| `curl http://127.0.0.1:<puerto>` | HTML o respuesta 200 |
| `kubectl logs -l app=<label> --tail=10` | Sin errores FATAL/panic |
🛑 Resolución de Conflictos
Si git pull falla → Editar manualmente marcadores <<<<<<< → git add → git commit
Nunca usar git push --force
Si un YAML rompe el cluster → kubectl delete -f <archivo> → Corregir → Reaplicar
