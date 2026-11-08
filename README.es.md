# Kubernetes que se recupera solo

[English](README.md)

Una web en un clúster de Kubernetes local que sigue sirviendo cuando cae uno de sus pods. Está monitorizada con Prometheus y Grafana, y la red está cerrada por defecto. Proyecto final del ciclo ASIR (2025–26), en equipo de tres.

<img src="assets/architecture.svg" alt="Arquitectura: Service delante de dos réplicas de nginx, Prometheus y Grafana, dentro de una red con denegación por defecto" width="560">

## Cómo funciona

| | |
|---|---|
| **Disponibilidad** | Nginx con dos réplicas detrás de un Service NodePort. |
| **Recuperación** | Liveness y readiness probes en el puerto 80. Un pod que falla se reinicia, y uno nuevo no recibe tráfico hasta que está listo. |
| **Red** | Seis NetworkPolicies: se deniega todo y luego se permiten, una a una, la web, Prometheus y el DNS ([verlas](manifests/network/networkpolicy.yaml)). |
| **Monitorización** | Prometheus lee las métricas de cAdvisor por pod a través del API server, con su propia service account y RBAC. Grafana muestra CPU y memoria. |
| **Tamaño** | Requests y limits en cada contenedor, para que todo quepa en una VM de 2 GB. |

## La prueba de recuperación

<img src="assets/self-healing.svg" alt="Repetición: se borra una réplica, la otra sigue sirviendo y Kubernetes crea una nueva" width="760">

Pruébalo con Docker, Minikube y kubectl:

```bash
minikube start --driver=docker --memory=2048 --cpus=2
kubectl apply -f manifests/app-web/ -f manifests/monitoring/ -f manifests/network/
kubectl delete $(kubectl get pod -l app=nginx-web -o name | head -1)
kubectl get pods -l app=nginx-web -w
```

La web está en el NodePort `30000` y Grafana en el `32000`. Para Prometheus, `kubectl port-forward svc/prometheus 9090`.

## Capturas

<table>
<tr>
<td width="50%"><img src="img/05-prometheus-targets.png" alt="Prometheus: target kubernetes-cadvisor UP"><br><sub>Prometheus, target de cAdvisor activo</sub></td>
<td width="50%"><img src="img/07.2-grafana-dashboard.png" alt="Grafana: CPU de los pods de nginx"><br><sub>Grafana, CPU por pod de nginx</sub></td>
</tr>
<tr>
<td><img src="img/03-nginx-deployment.png" alt="deployment de nginx 2/2 listo"><br><sub>Deployment, 2/2 listo</sub></td>
<td><img src="img/09-networkpolicy-list.png" alt="lista de políticas de red"><br><sub>Políticas de red aplicadas</sub></td>
</tr>
</table>

## Equipo

- **Adrián Boza** ([@adrianboza2](https://github.com/adrianboza2)): clúster, despliegues y servicios
- Santiago Pérez ([@santiagoperez27](https://github.com/santiagoperez27)): monitorización
- Sergio López ([@SergioLopez2411](https://github.com/SergioLopez2411)): políticas de red y documentación

## Siguientes pasos

Autoescalado con HPA, volúmenes persistentes para Prometheus y Grafana, un Ingress con TLS y un job de GitHub Actions que valide los manifiestos.

<sub>Documentación completa, con instalación paso a paso y solución de problemas, en [docs/DETALLES.md](docs/DETALLES.md). Licencia MIT.</sub>
