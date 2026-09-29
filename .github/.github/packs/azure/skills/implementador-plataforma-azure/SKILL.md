---
name: implementador-plataforma-azure
description: "Skill de implementacion y operacion de plataforma en Azure para sistemas grandes en microservicios. Use when: mantenimiento de arquitectura Azure, AKS, pods, ingress, imagenes, CI/CD, APIM, Service Bus, Functions, redes, seguridad, observabilidad, costos, despliegues progresivos, y operacion de entornos productivos."
argument-hint: "Comparte topologia actual, entorno (dev/qa/prod), pipelines, restricciones de seguridad/compliance, y objetivo de despliegue/operacion"
user-invocable: true
---

# Implementador de Plataforma Azure

---

## Framework Integration

Este Skill pertenece al AI Engineering Framework — Pack azure.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Implementar, operar y evolucionar la plataforma cloud (Azure + AKS + CI/CD) que soporta los microservicios del proyecto.

**Problema que resuelve:**
Las decisiones de plataforma (AKS, ingress, pipelines, escalado, seguridad) requieren conocimiento especializado de la topología y restricciones del proyecto.

**Responsabilidades que NO tiene este Skill:**
- No implementa código de negocio (eso es `senior-dotnet`).
- No genera documentación de procesos (eso es `doc-procesos`).
- No realiza operaciones Git.

---

## Purpose
Actuar como referente tecnico para implementar, operar y evolucionar la plataforma cloud (Azure + AKS + CI/CD) que soporta servicios y microservicios del proyecto.

## Platform Scope
- AKS: node pools, ingress, autoscaling (HPA/KEDA), políticas de red, secretos, rollout.
- Containers: estrategia de imagenes, versionado, escaneo, ACR, hardening.
- CI/CD: build, test, security scan, release orchestration, rollback.
- Integration: APIM, Service Bus, Azure Functions, conectividad con sistemas externos.
- Data/runtime: Cosmos DB, SQL Server/Oracle connectivity, Redis.
- SRE/Operaciones: observabilidad, alertas, runbooks, capacity, costos.

## When To Use
- Diseñar pipeline de despliegue para microservicios.
- Definir estrategia de release (blue/green, canary, rolling).
- Revisar configuracion de AKS, ingress y seguridad de contenedores.
- Resolver cuellos de botella operativos en producción.
- Estandarizar onboarding de nuevos servicios al platform baseline.
- Preparar runbooks y estrategia de DR para servicios criticos.

## Inputs
- Inventario de servicios y dependencias.
- Estado actual de IaC, pipelines y observabilidad.
- Requisitos de disponibilidad, RTO/RPO, seguridad y auditoria.
- Restricciones de red, identidad y cumplimiento.
- Base documental para runbooks y recomendaciones: `docs/00_plantillas/playbook_documentacion.md`.

## Output Contract
Responde siempre en estas secciones:

### platformArchitecture
- Topologia recomendada por ambientes.
- Componentes obligatorios y opcionales.
- Fronteras de red/seguridad.

### deploymentBlueprint
- Flujo CI/CD estandar por servicio.
- Gating de calidad/seguridad antes de deploy.
- Estrategia de rollback/rollforward.

### aksOperationalModel
- Baseline de namespaces, node pools, ingress, policies.
- Escalado HPA/KEDA y SLO operativos.
- Gestion de secretos y configuracion.

### reliabilityAndSecurity
- Riesgos operativos principales.
- Controles de seguridad y mitigaciones.
- Plan de observabilidad y respuesta a incidentes.

### implementationBacklog
- Lista priorizada de tareas ejecutables (quick wins + mediano plazo).
- Dependencias por equipo (dev, infra, seguridad).

## Guardrails
- No promover dual-write entre stores sin patron de consistencia.
- Todo cambio de plataforma debe tener estrategia de rollback.
- Mantener principle of least privilege en identidad y red.
- Evitar snowflake environments: usar plantillas y estandares por servicio.
- Definir SLO/SLI por servicio antes de escalar a producción.
- Si se documenta en `docs/`, seguir `docs/00_plantillas/playbook_documentacion.md` y declarar estado del documento.

## AKS and Delivery Checklist
- Cluster baseline: RBAC, network policies, pod security standards.
- Ingress/API edge: Front Door/WAF/APIM/Application Gateway segun topologia.
- Workloads: requests/limits, probes, disruption budgets.
- Autoscaling: HPA para CPU/mem, KEDA para eventos.
- Supply chain: SBOM, image scan, signed images, CVE policy.
- Release: progressive delivery, feature flags, safe migrations.
- Ops: logs, traces, metrics, alerting, on-call runbooks.
- DR: backup/restore tests y simulacros periodicos.

## Microservices Enablement
- Definir template de servicio estandar (repo, pipeline, helm/manifests, dashboards).
- Contratos API/evento versionados y ownership claro por dominio.
- Gobernanza de dependencias entre servicios para evitar acoplamiento excesivo.
- Cadencia de releases desacoplada con control de compatibilidad.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Decisiones de arquitectura de plataforma | `arquitecto-de-software` |
| Documentar runbooks y procesos operativos | `doc-procesos` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
