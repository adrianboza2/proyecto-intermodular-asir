# ️ STRUCTURE.md — Layout del Repositorio
proyecto-intermodular-asir/
├── manifests/
│ ├── app-web/ # ✅ Adrián
│ │ ├── deployment.yaml # Nginx 2 réplicas + probes + resources
│ │ └── service.yaml # NodePort 30000
│ ├── monitoring/ # 🔄 Santiago
│ │ ├── prometheus.yaml # ConfigMap + Deployment + Service 31000
│ │ └── grafana.yaml # Deployment + Service 32000 + env vars
│ └── network/ # ⏳ Sergio
│ └── networkpolicy.yaml # Default-deny + allow rules
├── docs/ # Memoria técnica, manuales, defensa
├── img/ # Evidencias: 03-.png, 04-.png, etc.
├── .gitignore
├── LICENSE
├── README.md
├── AGENTS.md
├── PROJECT_CONTEXT.md
├── WORKFLOW.md
── STRUCTURE.md
└── ROADMAP.md

## 📏 Convenciones Técnicas
- Indentación YAML: 2 espacios
- Nombres de recursos: kebab-case (`nginx-web`, `prometheus-svc`)
- Labels obligatorios: `app: <nombre>` en todos los pods
- Resources limits: máximo 128Mi RAM / 250m CPU por pod
- Puertos NodePort: rango 30000-32767
- Probes: siempre en puerto de la app, `initialDelaySeconds` ≥ 5