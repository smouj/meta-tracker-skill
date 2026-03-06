---
name: "Meta Tracker"
description: "AI-powered devops task tracker with predictive analytics and automated workflow orchestration"
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

AI-powered tracker for devops tasks, providing intelligent prioritization, incident correlation, and automated remediation suggestions.

## Purpose

Meta Tracker is used in real-world scenarios:
- **Incident Management**: Track production incidents with AI-suggested severity and assignment based on historical data and system impact.
- **Deployment Monitoring**: Automatically track deployments, rollbacks, and health checks with predictive failure detection.
- **Change Risk Assessment**: Analyze pull requests and infrastructure changes to predict rollout risks and suggest mitigation strategies.
- **Capacity Forecasting**: AI-driven resource usage predictions and anomaly detection for proactive scaling decisions.
- **Compliance Auditing**: Continuous tracking of configuration changes against compliance policies with auto-generated audit trails.

## Scope

### Core Commands

- `meta-tracker init [--project PROJECT] [--config PATH]` - Initialize tracking for a project
- `meta-tracker add [--type TYPE] [--severity {low,medium,high,critical}] [--component COMP]` - Add a new devops event
- `meta-tracker status [--watch] [--filter FILTER]` - Show current tracked items
- `meta-tracker predict [--target TARGET] [--horizon {5m,15m,1h,6h,24h}]` - Get AI predictions
- `meta-tracker correlate [--incident ID]` - Find related events across systems
- `meta-tracker report [--format {json,html,markdown}] [--days N]` - Generate analytics report
- `meta-tracker alert [--channel SLACK|TEAMS|WEBHOOK] [--threshold N]` - Configure alerting
- `meta-tracker rollback [--target DEPLOYMENT_ID] [--mode {auto,manual}]` - Execute rollback
- `meta-tracker learn [--feedback FILE]` - Update AI model with new patterns

### Command Flags

- `--project`: Project identifier (required for multi-tenant setups)
- `--env`: Environment name (dev, staging, prod)
- `--component`: System component (api, db, cache, worker, etc.)
- `--region`: Geographic region for multi-region deployments
- `--dry-run`: Preview actions without execution
- `--timeout`: Operation timeout in seconds (default: 300)
- `--priority`: Override AI priority (P0-P3)
- `--exclude`: Exclude patterns from correlation

## Work Process

### 1. Initialization

```
meta-tracker init --project myapp --config .meta-tracker.yaml
```

The skill reads configuration, validates API connectivity, sets up local database, and configures webhook endpoints.

### 2. Event Collection

Events are ingested via:
- Direct CLI: `meta-tracker add --type deployment --component api --severity high`
- Webhook: POST to `/track` endpoint with JSON payload
- CI/CD Integration: Use `meta-tracker ci` command in pipelines
- System Metrics: Automatic collection from prometheus/datadog

### 3. AI Analysis

For each event:
1. Extract features (timestamp, component, patterns, text embeddings)
2. Query AI model for classification and priority
3. Correlate with historical incidents using similarity search
4. Generate suggested actions and owners

### 4. Notification & Action

- High-severity triggers immediate alerts to configured channels
- Medium/Low are batched in periodic reports
- Auto-remediation suggestions include exact CLI commands to run

### 5. Learning Loop

Weekly `meta-tracker learn` updates model with outcome data:
- False positives/negatives
- Resolution times
- User feedback

## Golden Rules

1. **Never suppress critical alerts**: Always surface AI-predicted critical incidents, even if noisy.
2. **Validate rollback targets**: Confirm rollback target exists and is healthy before execution.
3. **Secure API keys**: Never log META_TRACKER_API_KEY; use secret management.
4. **Correlate across teams**: Include all relevant teams in incident correlation, not just the reporter's team.
5. **Document false positives**: Feed all false alerts back into training data.
6. **Test in staging first**: Always validate new rules/patterns in non-production.
7. **Respect rate limits**: Implement exponential backoff on API failures.
8. **Maintain audit trail**: All actions must be logged with user identity and timestamp.

## Examples

### Track a deployment with AI priority

```bash
meta-tracker add \
  --type deployment \
  --component payments-api \
  --env prod \
  --region us-east-1 \
  --notes "Rolling out v2.3.1"
```

Output:
```
Added event #4521:
  Type: deployment
  Component: payments-api
  Environment: prod
  AI Priority: P1 (High risk - similar to 3 past incidents)
  Suggested Owner: platform-team
  Auto-remediation: monit service payments-api --check health
```

### Predict capacity issues

```bash
meta-tracker predict --target redis-cluster --horizon 6h
```

Output:
```
Prediction for redis-cluster (next 6h):
  - Memory usage: 87% → 95% (High risk of OOM)
  - Connection count: 1.2k → 2.1k (Approaching limit)
  - Suggested action: redis-cli --cluster reshard 3 nodes
  - Confidence: 0.89
```

### Correlate an incident

```bash
meta-tracker correlate --incident INC-7892
```

Output:
```
Correlation for INC-7892:
  Related events:
    - #4519 (deployment payments-api v2.3.1)
    - #4520 (metric latency spike)
    - #4521 (alert cache-miss rate)
  Root cause hypothesis: Deployment caused cache invalidation storm
  Recommended: Rollback to v2.3.0 or increase cache TTL
```

### Generate compliance report

```bash
meta-tracker report --format markdown --days 30 > compliance-audit.md
```

Output sample:
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

## Rollback Commands

Meta Tracker tracks all deployment events and provides safe rollback:

```
meta-tracker rollback --target dep-4521 --mode auto
```

This:
1. Verifies target deployment exists and succeeded
2. Finds previous stable version from history
3. Checks compatibility (DB migrations, API contracts)
4. Executes rollback via pipeline or CLI
5. Monitors health for 5 minutes post-rollback
6. Creates correlation event linking to original incident

Rollback dry-run:

```
meta-tracker rollback --target dep-4521 --mode manual --dry-run
```

Output:
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

## Verification Steps

After installation:

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

## Troubleshooting

### Issue: `meta-tracker: command not found`
**Solution**: Ensure `~/.local/bin` in PATH or use absolute path. Reinstall with `pip install meta-tracker-cli`.

### Issue: API rate limit exceeded
**Symptoms**: `429 Too Many Requests`
**Solution**: Set `META_TRACKER_RATE_LIMIT=100` or use `--throttle` flag. Contact admin for quota increase.

### Issue: AI predictions all "Unknown"
**Symptoms**: Priority shows "N/A"
**Causes**:
- Model not loaded (check `meta-tracker models`)
- Insufficient training data (< 100 events)
- Feature extraction failure
**Solution**: Run `meta-tracker learn --force` to retrain. Ensure events have proper labels.

### Issue: Correlation returns no results
**Check**:
1. Event has sufficient metadata (`--component`, `--type`)
2. Webhook events include `request_id` for tracing
3. AI model is up-to-date (last trained < 7 days ago)
4. Search window not too narrow (use `--hours 24`)

### Issue: Rollback fails with "target not found"
**Cause**: Deployment ID incorrect or expired.
**Fix**: List deployments: `meta-tracker deployments --days 7`. Use full ID like `dep-4521`.

## Dependencies

### Required
- Python 3.9+ with pip
- Docker (for isolation)
- kubectl (Kubernetes clusters)
- jq (JSON processing)
- Git (version control integration)

### Optional
- prometheus-client (metrics collection)
- datadog (external monitoring)
- awscli (AWS resource tracking)
- vault (secret management)

### AI Model
The default model (`meta-ai-v3`) requires:
- 4GB RAM
- GPU optional (CPU fallback)
- Downloads on first use (~500MB)

Set `META_TRACKER_MODEL=mock` for testing without AI.

## Configuration Files

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

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `META_TRACKER_API_KEY` | Yes (cloud) | API key from Meta Tracker Cloud |
| `META_TRACKER_ENDPOINT` | No | Custom API endpoint (default: https://api.metatracker.io) |
| `META_TRACKER_ENV` | Yes | Current environment (dev/staging/prod) |
| `META_TRACKER_MODEL` | No | AI model name (default: meta-ai-v3) |
| `META_TRACKER_DRY_RUN` | No | Set to 1 to simulate all operations |
| `CI` | No | Auto-detected CI environment (GitHub Actions, GitLab CI, Jenkins) |

## Security Considerations

- All webhook payloads validated with HMAC signature
- API keys stored in OS keyring (`keyring` library)
- Events containing passwords/keys are automatically redacted
- Network traffic uses TLS 1.3 only
- Audit logs immutable for 90 days per compliance requirement

## Support

- Documentation: https://docs.metatracker.io
- Issues: https://github.com/meta-tracker/meta-tracker-cli/issues
- Community Slack: #meta-tracker on devops-community.slack.com
```