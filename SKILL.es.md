name: "Meta Tracker"
description: "Rastreador de tareas de DevOps impulsado por IA con análisis predictivo y orquestación automatizada del flujo de trabajo"
version: "2.4.1"
author: "DevOps Team"
tags:
  - devops
  - ai
  - automation
  - monitoring
  - cicd
dependencies:
  - python >=3.9
  - docker
  - kubectl
  - jq
  - curl
  - git
  - openclaw-core >=1.2.0
  - meta-tracker-ai-model >=1.0.0
environment:
  - META_TRACKER_API_KEY
  - META_TRACKER_ENDPOINT
  - META_TRACKER_ENV
  - CI
---

# Meta Tracker

Rastreador impulsado por IA para tareas de DevOps, que proporciona priorización inteligente, correlación de incidentes y sugerencias de remediación automatizadas.

## Propósito

Meta Tracker se utiliza en escenarios del mundo real:
- **Gestión de Incidentes**: Rastrear incidentes de producción con severidad y asignación sugeridas por IA basadas en datos históricos e impacto del sistema.
- **Monitoreo de Implementaciones**: Rastrear automáticamente implementaciones, rollbacks y verificaciones de健康 con detección predictiva de fallos.
- **Evaluación de Riesgos de Cambio**: Analizar solicitudes de extracción (pull requests) y cambios en la infraestructura para predecir riesgos de implementación y sugerir estrategias de mitigación.
- **Pronóstico de Capacidad**: Predicciones de uso de recursos impulsadas por IA y detección de anomalías para decisiones de escalado proactivas.
- **Auditoría de Cumplimiento**: Seguimiento continuo de cambios de configuración contra políticas de cumplimiento con rutas de auditoría generadas automáticamente.

## Alcance

### Comandos Principales

- `meta-tracker init [--project PROJECT] [--config PATH]` - Inicializa el seguimiento para un proyecto
- `meta-tracker add [--type TYPE] [--severity {low,medium,high,critical}] [--component COMP]` - Agrega un nuevo evento de DevOps
- `meta-tracker status [--watch] [--filter FILTER]` - Muestra los elementos rastreados actualmente
- `meta-tracker predict [--target TARGET] [--horizon {5m,15m,1h,6h,24h}]` - Obtiene predicciones de IA
- `meta-tracker correlate [--incident ID]` - Encuentra eventos relacionados en todos los sistemas
- `meta-tracker report [--format {json,html,markdown}] [--days N]` - Genera un informe de análisis
- `meta-tracker alert [--channel SLACK|TEAMS|WEBHOOK] [--threshold N]` - Configura alertas
- `meta-tracker rollback [--target DEPLOYMENT_ID] [--mode {auto,manual}]` - Ejecuta rollback
- `meta-tracker learn [--feedback FILE]` - Actualiza el modelo de IA con nuevos patrones

### Banderas de Comando

- `--project`: Identificador de proyecto (requerido para configuraciones multi-inquilino)
- `--env`: Nombre del entorno (dev, staging, prod)
- `--component`: Componente del sistema (api, db, cache, worker, etc.)
- `--region`: Región geográfica para implementaciones multi-región
- `--dry-run`: Previsualiza acciones sin ejecutarlas
- `--timeout`: Tiempo de espera de la operación en segundos (predeterminado: 300)
- `--priority`: Anula la prioridad de IA (P0-P3)
- `--exclude`: Excluye patrones de la correlación

## Proceso de Trabajo

### 1. Inicialización

```
meta-tracker init --project myapp --config .meta-tracker.yaml
```

La skill lee la configuración, valida la conectividad de la API, configura la base de datos local y configura los endpoints de webhook.

### 2. Recopilación de Eventos

Los eventos se ingieren a través de:
- CLI directa: `meta-tracker add --type deployment --component api --severity high`
- Webhook: POST al endpoint `/track` con payload JSON
- Integración CI/CD: Usar el comando `meta-tracker ci` en pipelines
- Métricas del sistema: Recopilación automática desde prometheus/datadog

### 3. Análisis de IA

Para cada evento:
1. Extrae características (timestamp, componente, patrones, incrustaciones de texto)
2. Consulta el modelo de IA para clasificación y prioridad
3. Correlaciona con incidentes históricos usando búsqueda por similitud
4. Genera acciones sugeridas y responsables

### 4. Notificación y Acción

- Las de alta severidad disparan alertas inmediatas a los canales configurados
- Las medias/bajas se agrupan en informes periódicos
- Las sugerencias de remediación automática incluyen comandos CLI exactos para ejecutar

### 5. Ciclo de Aprendizaje

Semanalmente `meta-tracker learn` actualiza el modelo con datos de resultados:
- Falsos positivos/negativos
- Tiempos de resolución
- Retroalimentación del usuario

## Reglas de Oro

1. **Nunca suprimir alertas críticas**: Siempre mostrar incidentes críticos predichos por IA, incluso si son ruidosos.
2. **Validar objetivos de rollback**: Confirmar que el objetivo de rollback existe y está saludable antes de la ejecución.
3. **Proteger claves API**: Nunca registrar META_TRACKER_API_KEY; usar gestión de secretos.
4. **Correlacionar entre equipos**: Incluir todos los equipos relevantes en la correlación de incidentes, no solo el equipo del reportador.
5. **Documentar falsos positivos**: Alimentar todas las alertas falsas de nuevo en los datos de entrenamiento.
6. **Probar primero en staging**: Siempre validar nuevas reglas/patrones en entornos no productivos.
7. **Respetar límites de tasa**: Implementar retroceso exponencial en fallos de API.
8. **Mantener rastro de auditoría**: Todas las acciones deben registrarse con identidad de usuario y marca de tiempo.

## Ejemplos

### Rastrear una implementación con prioridad de IA

```bash
meta-tracker add \
  --type deployment \
  --component payments-api \
  --env prod \
  --region us-east-1 \
  --notes "Rolling out v2.3.1"
```

Salida:
```
Added event #4521:
  Type: deployment
  Component: payments-api
  Environment: prod
  AI Priority: P1 (High risk - similar to 3 past incidents)
  Suggested Owner: platform-team
  Auto-remediation: monit service payments-api --check health
```

### Predecir problemas de capacidad

```bash
meta-tracker predict --target redis-cluster --horizon 6h
```

Salida:
```
Prediction for redis-cluster (next 6h):
  - Memory usage: 87% → 95% (High risk of OOM)
  - Connection count: 1.2k → 2.1k (Approaching limit)
  - Suggested action: redis-cli --cluster reshard 3 nodes
  - Confidence: 0.89
```

### Correlacionar un incidente

```bash
meta-tracker correlate --incident INC-7892
```

Salida:
```
Correlation for INC-7892:
  Related events:
    - #4519 (deployment payments-api v2.3.1)
    - #4520 (metric latency spike)
    - #4521 (alert cache-miss rate)
  Root cause hypothesis: Deployment caused cache invalidation storm
  Recommended: Rollback to v2.3.0 or increase cache TTL
```

### Generar informe de cumplimiento

```bash
meta-tracker report --format markdown --days 30 > compliance-audit.md
```

Salida de muestra:
```
# Compliance Audit Report (Last 30 Days)

## Configuration Changes
- Total changes: 142
- Unauthorized: 0
- Missing approvals: 3 (see events #4401, #4456, #4499)

## Drift Detection
- Production vs. IaC drift: 12 instances
  - Security group mismatch (SG-1023)
  - Missing encryption on 5 EBS volumes

## Access Reviews
- Service account usage: 156 active accounts
- Inactive >90 days: 23 accounts (recommend revocation)
```

## Comandos de Rollback

Meta Tracker rastrea todos los eventos de implementación y proporciona rollback seguro:

```
meta-tracker rollback --target dep-4521 --mode auto
```

Esto:
1. Verifica que la implementación objetivo existe y tuvo éxito
2. Encuentra la versión estable anterior del historial
3. Verifica compatibilidad (migraciones de BD, contratos de API)
4. Ejecuta el rollback vía pipeline o CLI
5. Monitorea la salud durante 5 minutos post-rollback
6. Crea un evento de correlación vinculado al incidente original

Rollback dry-run:

```
meta-tracker rollback --target dep-4521 --mode manual --dry-run
```

Salida:
```
Rollback plan for dep-4521 (payments-api v2.3.1):
  Target version: v2.3.0 (deployed 2024-01-15 10:30 UTC)
  Estimated downtime: 30s
  Pre-checks:
    - [X] Previous image exists in registry
    - [ ] DB migration rollback script exists (MISSING)
    - [X] Feature flag to disable new endpoints exists
  Commands to execute:
    1) kubectl set image deployment/payments-api payments-api=myrepo/payments:v2.3.0
    2) kubectl rollout status deployment/payments-api --timeout=5m
  To execute: meta-tracker rollback --target dep-4521 --mode auto
```

## Pasos de Verificación

Después de la instalación:

```bash
# 1. Check CLI
meta-tracker --version
# Expected: 2.4.1

# 2. Test API connectivity
meta-tracker ping
# Expected: {"status":"ok","model":"meta-ai-v3"}

# 3. List models
meta-tracker models
# Expected: Shows available AI models and versions

# 4. Simulate event
meta-tracker add --type test --component cli
# Expected: Event added, AI priority assigned

# 5. Check learning
meta-tracker learn --dry-run
# Expected: "Would update model with 23 recent events"
```

## Solución de Problemas

### Problema: `meta-tracker: command not found`
**Solución**: Asegurar que `~/.local/bin` esté en PATH o usar la ruta absoluta. Reinstalar con `pip install meta-tracker-cli`.

### Problema: API rate limit exceeded
**Síntomas**: `429 Too Many Requests`
**Solución**: Configurar `META_TRACKER_RATE_LIMIT=100` o usar la bandera `--throttle`. Contactar al administrador para aumentar el cupo.

### Problema: AI predictions all "Unknown"
**Síntomas**: La prioridad muestra "N/A"
**Causas**:
- Modelo no cargado (verificar `meta-tracker models`)
- Datos de entrenamiento insuficientes (< 100 eventos)
- Fallo en extracción de características
**Solución**: Ejecutar `meta-tracker learn --force` para reentrenar. Asegurar que los eventos tengan etiquetas apropiadas.

### Problema: Correlation returns no results
**Verificar**:
1. El evento tiene metadatos suficientes (`--component`, `--type`)
2. Los eventos de webhook incluyen `request_id` para trazabilidad
3. El modelo de IA está actualizado (último entrenamiento < 7 días)
4. La ventana de búsqueda no es muy estrecha (usar `--hours 24`)

### Problema: Rollback fails with "target not found"
**Causa**: ID de implementación incorrecto o expirado.
**Corrección**: Listar implementaciones: `meta-tracker deployments --days 7`. Usar el ID completo como `dep-4521`.

## Dependencias

### Requeridas
- Python 3.9+ con pip
- Docker (para aislamiento)
- kubectl (clústeres de Kubernetes)
- jq (procesamiento JSON)
- Git (integración de control de versiones)

### Opcionales
- prometheus-client (recopilación de métricas)
- datadog (monitoreo externo)
- awscli (rastreo de recursos AWS)
- vault (gestión de secretos)

### Modelo de IA
El modelo predeterminado (`meta-ai-v3`) requiere:
- 4GB RAM
- GPU opcional (fallback a CPU)
- Descarga en primer uso (~500MB)

Configurar `META_TRACKER_MODEL=mock` para pruebas sin IA.

## Archivos de Configuración

`.meta-tracker.yaml`:

```yaml
project: "myapp"
environment: "prod"
ai:
  model: "meta-ai-v3"
  confidence_threshold: 0.7
  retrain_days: 7
correlation:
  window_hours: 24
  min_similarity: 0.6
alerts:
  slack:
    webhook: "${SLACK_WEBHOOK}"
    channel: "#devops-alerts"
  teams:
    webhook: "${TEAMS_WEBHOOK}"
rollback:
  default_timeout: 300
  health_check_interval: 10
  max_parallel: 2
```

## Variables de Entorno

| Variable | Requerida | Descripción |
|----------|-----------|-------------|
| `META_TRACKER_API_KEY` | Sí (cloud) | Clave API de Meta Tracker Cloud |
| `META_TRACKER_ENDPOINT` | No | Endpoint de API personalizado (default: https://api.metatracker.io) |
| `META_TRACKER_ENV` | Sí | Entorno actual (dev/staging/prod) |
| `META_TRACKER_MODEL` | No | Nombre del modelo de IA (default: meta-ai-v3) |
| `META_TRACKER_DRY_RUN` | No | Configurar a 1 para simular todas las operaciones |
| `CI` | No | Entorno CI auto-detectado (GitHub Actions, GitLab CI, Jenkins) |

## Consideraciones de Seguridad

- Todos los payloads de webhook se validan con firma HMAC
- Claves API almacenadas en anillo de claves del SO (`keyring`)
- Eventos que contienen contraseñas/claves se redactan automáticamente
- Tráfico de red usa TLS 1.3 únicamente
- Logs de auditoría inmutables durante 90 días según requisito de cumplimiento

## Soporte

- Documentación: https://docs.metatracker.io
- Issues: https://github.com/meta-tracker/meta-tracker-cli/issues
- Slack Comunitario: #meta-tracker en devops-community.slack.com