# 🗺️ ROADMAP.md — Estado y Próximos Pasos

## 📅 Fechas Clave
| Evento | Fecha | Estado |
|--------|-------|--------|
| Envío versión avance al tutor | Hoy | ✅ |
| Exposición oral | 28 o 29 mayo |  |
| Entrega final | Por confirmar | ⏳ |

## 🔄 Estado por Fases
| Fase | Responsable | Archivos | Estado | Próximo Hito |
|------|-------------|----------|--------|--------------|
| 1. Infra + App Web | Adrián | `app-web/` | ✅ Completado | Mantenimiento pre-defensa |
| 2. Monitoring | Santiago | `monitoring/` | 🔄 En progreso | Prometheus scrapeando Nginx |
| 3. Security + Docs | Sergio | `network/` + `docs/` | ✅ Completado | NetworkPolicy aplicada y testeada |

##  Tareas Inmediatas (Próximas 48h)
- [ ] Santiago: Validar `prometheus.yaml` con Qwen → Aplicar → Verificar targets UP
- [ ] Santiago: Configurar datasource Grafana → Importar dashboard básico
- [x] Sergio: Crear `networkpolicy.yaml` → Aplicar → Testear conectividad
- [ ] Santiago: Validar que Prometheus scrapea correctamente con la NetworkPolicy activa
- [ ] Todos: `git pull` → Ensayo de defensa con demo en vivo
- [ ] Adrián: Actualizar memoria técnica en `docs/` con estado real
- [ ] Todos: Subir evidencias faltantes a `img/`

## 🚀 Mejoras Futuras (Post-Defensa)
- HPA para escalado automático
- PVC para persistencia de Grafana
- Ingress Controller + HTTPS
- Pipeline CI/CD con GitHub Actions