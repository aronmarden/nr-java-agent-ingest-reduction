# New Relic Java Agent - Ingest Reduction Configuration Guide

## Overview

This directory contains an optimized New Relic Java agent configuration file (`ingest-reduction-config.yml`) designed to significantly reduce data ingest volumes while maintaining critical application visibility. This configuration is ideal for:

- **Cost optimization**: Reduce New Relic data ingest costs by 40-80%
- **High-volume applications**: Applications generating excessive telemetry data
- **Production environments**: Where cost control is prioritized over deep debugging visibility
- **Multi-environment rollouts**: Staged implementation across dev, staging, and production

## What's Included

The `ingest-reduction-config.yml` file provides:

- **Comprehensive ingest reduction settings** organized by impact (highest to lowest)
- **Detailed comments** explaining each setting's purpose and impact
- **Recommended values** for maximum reduction while maintaining visibility
- **Estimated reduction percentages** for each configuration option

### Key Reduction Areas

1. **Span Events** (30-50% reduction) - Limits distributed tracing data volume
2. **Transaction Events** (20-30% reduction) - Controls APM transaction data
3. **Application Logging** (10-30% reduction) - Disables log forwarding
4. **Code-Level Metrics** (15-25% reduction) - Removes method-level instrumentation
5. **Custom Events** (5-15% reduction) - Limits custom event volume
6. **JFR & Other Features** (10-20% reduction) - Disables optional features

**Total Expected Reduction: 40-80% of overall ingest**

---

## Implementation Methods

You can implement these settings using three methods:

1. **YAML Configuration File** (Recommended for most environments)
2. **Environment Variables** (Best for containerized/cloud environments)
3. **System Properties** (Alternative for traditional deployments)

---

## Method 1: YAML Configuration File

### Step 1: Backup Your Existing Configuration

```bash
# Backup your current newrelic.yml
cp /path/to/newrelic.yml /path/to/newrelic.yml.backup
```

### Step 2: Implement the Configuration

**Option A: Replace Entire File (Clean Installation)**

```bash
# Copy the optimized config to your New Relic directory
cp ingest-reduction-config.yml /path/to/newrelic.yml
```

**Option B: Merge Settings (Existing Installation)**

Manually merge the settings from `ingest-reduction-config.yml` into your existing `newrelic.yml` file. Focus on these critical sections:

```yaml
# Add/update these sections in your existing newrelic.yml
transaction_events:
  max_samples_stored: 2000

span_events:
  max_samples_stored: 2000

application_logging:
  enabled: false

code_level_metrics:
  enabled: false
```

### Step 3: Update Required Fields

Edit the file and replace placeholder values:

```yaml
license_key: 'YOUR_ACTUAL_LICENSE_KEY'
app_name: "Your Actual Application Name"
labels:
  env: prod
  team: your-actual-team
  region: your-actual-region
```

### Step 4: Deploy and Restart

```bash
# Restart your application to apply changes
# The exact command depends on your deployment method
systemctl restart your-app
# or
docker restart your-container
# or
kubectl rollout restart deployment/your-deployment
```

### Step 5: Verify Configuration

Check the New Relic agent log to confirm settings are applied:

```bash
# Look for log messages confirming the configuration
tail -f /path/to/logs/newrelic_agent.log | grep -i "configuration"
```

---

## Method 2: Environment Variables (Containerized Environments)

Environment variables are ideal for Docker, Kubernetes, and cloud platforms. They override both the YAML file and system properties.

### Environment Variable Naming Convention

Convert YAML settings to environment variables using this pattern:

```
NEW_RELIC_<SECTION>_<SETTING>=value
```

- Replace dots (`.`) with underscores (`_`)
- Replace dashes (`-`) with underscores (`_`)
- Prefix with `NEW_RELIC_`
- Use uppercase

### Critical Environment Variables for Ingest Reduction

```bash
# Required - Basic Configuration
export NEW_RELIC_LICENSE_KEY='your_license_key'
export NEW_RELIC_APP_NAME='Your Application Name'
export NEW_RELIC_AGENT_ENABLED=true

# High Impact - Event Sampling
export NEW_RELIC_TRANSACTION_EVENTS_MAX_SAMPLES_STORED=2000
export NEW_RELIC_SPAN_EVENTS_MAX_SAMPLES_STORED=2000
export NEW_RELIC_CUSTOM_INSIGHTS_EVENTS_MAX_SAMPLES_STORED=5000

# High Impact - Feature Toggles
export NEW_RELIC_APPLICATION_LOGGING_ENABLED=false
export NEW_RELIC_APPLICATION_LOGGING_FORWARDING_ENABLED=false
export NEW_RELIC_APPLICATION_LOGGING_METRICS_ENABLED=false
export NEW_RELIC_CODE_LEVEL_METRICS_ENABLED=false
export NEW_RELIC_JFR_ENABLED=false
export NEW_RELIC_BROWSER_MONITORING_AUTO_INSTRUMENT=false

# High Impact - Transaction Limits
export NEW_RELIC_TRANSACTION_TRACER_SEGMENT_LIMIT=3000
export NEW_RELIC_TRANSACTION_SIZE_LIMIT=2000

# Medium Impact - Other Features
export NEW_RELIC_THREAD_PROFILER_ENABLED=false
export NEW_RELIC_CROSS_APPLICATION_TRACER_ENABLED=false
export NEW_RELIC_DISTRIBUTED_TRACING_ENABLED=true

# Low-Medium Impact - Attributes
export NEW_RELIC_ATTRIBUTES_ENABLED=true
export NEW_RELIC_ATTRIBUTES_HTTP_ATTRIBUTE_MODE=standard
export NEW_RELIC_ATTRIBUTES_EXCLUDE='request.parameters.*,message.parameters.*'

# Medium Impact - Agent Limits
export NEW_RELIC_AGENT_LIMITS_MAX_METRIC_NAMES=5000
```

### Docker Implementation

**Dockerfile:**

```dockerfile
FROM openjdk:11-jre-slim

# Copy application and New Relic agent
COPY target/your-app.jar /app/app.jar
COPY newrelic/newrelic.jar /app/newrelic.jar

# Set environment variables
ENV NEW_RELIC_LICENSE_KEY=${NEW_RELIC_LICENSE_KEY}
ENV NEW_RELIC_APP_NAME="Your Application"
ENV NEW_RELIC_TRANSACTION_EVENTS_MAX_SAMPLES_STORED=2000
ENV NEW_RELIC_SPAN_EVENTS_MAX_SAMPLES_STORED=2000
ENV NEW_RELIC_APPLICATION_LOGGING_ENABLED=false
ENV NEW_RELIC_CODE_LEVEL_METRICS_ENABLED=false

CMD ["java", "-javaagent:/app/newrelic.jar", "-jar", "/app/app.jar"]
```

**docker-compose.yml:**

```yaml
version: '3.8'
services:
  app:
    image: your-app:latest
    environment:
      NEW_RELIC_LICENSE_KEY: ${NEW_RELIC_LICENSE_KEY}
      NEW_RELIC_APP_NAME: "Your Application"
      NEW_RELIC_TRANSACTION_EVENTS_MAX_SAMPLES_STORED: 2000
      NEW_RELIC_SPAN_EVENTS_MAX_SAMPLES_STORED: 2000
      NEW_RELIC_APPLICATION_LOGGING_ENABLED: false
      NEW_RELIC_CODE_LEVEL_METRICS_ENABLED: false
      NEW_RELIC_JFR_ENABLED: false
      NEW_RELIC_THREAD_PROFILER_ENABLED: false
      NEW_RELIC_ATTRIBUTES_HTTP_ATTRIBUTE_MODE: standard
```

### Kubernetes Implementation

**ConfigMap:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: newrelic-config
data:
  NEW_RELIC_APP_NAME: "Your Application"
  NEW_RELIC_TRANSACTION_EVENTS_MAX_SAMPLES_STORED: "2000"
  NEW_RELIC_SPAN_EVENTS_MAX_SAMPLES_STORED: "2000"
  NEW_RELIC_CUSTOM_INSIGHTS_EVENTS_MAX_SAMPLES_STORED: "5000"
  NEW_RELIC_APPLICATION_LOGGING_ENABLED: "false"
  NEW_RELIC_APPLICATION_LOGGING_FORWARDING_ENABLED: "false"
  NEW_RELIC_APPLICATION_LOGGING_METRICS_ENABLED: "false"
  NEW_RELIC_CODE_LEVEL_METRICS_ENABLED: "false"
  NEW_RELIC_JFR_ENABLED: "false"
  NEW_RELIC_BROWSER_MONITORING_AUTO_INSTRUMENT: "false"
  NEW_RELIC_THREAD_PROFILER_ENABLED: "false"
  NEW_RELIC_CROSS_APPLICATION_TRACER_ENABLED: "false"
  NEW_RELIC_DISTRIBUTED_TRACING_ENABLED: "true"
  NEW_RELIC_TRANSACTION_TRACER_SEGMENT_LIMIT: "3000"
  NEW_RELIC_TRANSACTION_SIZE_LIMIT: "2000"
  NEW_RELIC_AGENT_LIMITS_MAX_METRIC_NAMES: "5000"
  NEW_RELIC_ATTRIBUTES_HTTP_ATTRIBUTE_MODE: "standard"
```

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: your-app:latest
        env:
        - name: NEW_RELIC_LICENSE_KEY
          valueFrom:
            secretKeyRef:
              name: newrelic-secret
              key: license-key
        envFrom:
        - configMapRef:
            name: newrelic-config
```

---

## Method 3: System Properties (Traditional Deployments)

System properties can override YAML settings but are overridden by environment variables.

### Naming Convention

Convert YAML settings to system properties using this pattern:

```
-Dnewrelic.config.<section>.<setting>=value
```

### Example Startup Command

```bash
java -javaagent:/path/to/newrelic.jar \
  -Dnewrelic.config.license_key='your_license_key' \
  -Dnewrelic.config.app_name='Your Application' \
  -Dnewrelic.config.transaction_events.max_samples_stored=2000 \
  -Dnewrelic.config.span_events.max_samples_stored=2000 \
  -Dnewrelic.config.custom_insights_events.max_samples_stored=5000 \
  -Dnewrelic.config.application_logging.enabled=false \
  -Dnewrelic.config.application_logging.forwarding.enabled=false \
  -Dnewrelic.config.application_logging.metrics.enabled=false \
  -Dnewrelic.config.code_level_metrics.enabled=false \
  -Dnewrelic.config.jfr.enabled=false \
  -Dnewrelic.config.browser_monitoring.auto_instrument=false \
  -Dnewrelic.config.thread_profiler.enabled=false \
  -Dnewrelic.config.transaction_tracer.segment_limit=3000 \
  -Dnewrelic.config.transaction_size_limit=2000 \
  -Dnewrelic.config.attributes.http_attribute_mode=standard \
  -jar your-application.jar
```

### Tomcat (catalina.sh)

```bash
export CATALINA_OPTS="$CATALINA_OPTS \
  -javaagent:/path/to/newrelic.jar \
  -Dnewrelic.config.transaction_events.max_samples_stored=2000 \
  -Dnewrelic.config.span_events.max_samples_stored=2000 \
  -Dnewrelic.config.application_logging.enabled=false \
  -Dnewrelic.config.code_level_metrics.enabled=false"
```

---

## Configuration Precedence

Settings are applied in this order (highest priority first):

1. **Environment Variables** (`NEW_RELIC_*`)
2. **System Properties** (`-Dnewrelic.config.*`)
3. **YAML Configuration File** (`newrelic.yml`)
4. **Agent Defaults**

**Best Practice:** Use environment variables for environment-specific settings (like license keys) and YAML files for shared configuration across environments.

---

## Staged Rollout Strategy

### Phase 1: Development/Test (Week 1)

Deploy to non-production environments first:

```yaml
# Conservative settings for initial testing
transaction_events:
  max_samples_stored: 5000  # 50% reduction
span_events:
  max_samples_stored: 2000
application_logging:
  enabled: true  # Keep enabled initially
  forwarding:
    max_samples_stored: 5000  # Reduce volume but keep enabled
code_level_metrics:
  enabled: true  # Keep enabled initially
```

**Monitor:**
- Data ingest reduction percentage
- Application performance
- Alert coverage
- Dashboard functionality

### Phase 2: Staging (Week 2)

Apply more aggressive settings:

```yaml
# Moderate reduction settings
transaction_events:
  max_samples_stored: 3000  # 70% reduction
span_events:
  max_samples_stored: 2000
application_logging:
  enabled: false  # Disable if not critical
code_level_metrics:
  enabled: false  # Disable if not actively used
```

**Monitor:**
- Confirm no critical visibility gaps
- Verify error detection still works
- Check distributed tracing adequacy

### Phase 3: Production (Week 3-4)

Apply full optimization:

```yaml
# Aggressive reduction settings
transaction_events:
  max_samples_stored: 2000  # 80% reduction
span_events:
  max_samples_stored: 2000
application_logging:
  enabled: false
code_level_metrics:
  enabled: false
jfr:
  enabled: false
thread_profiler:
  enabled: false
```

---

## Environment-Specific Configurations

### Development Environment

**Focus:** Full visibility for debugging

```yaml
# dev-newrelic.yml
transaction_events:
  max_samples_stored: 10000  # Default - no reduction
span_events:
  max_samples_stored: 2000
application_logging:
  enabled: true
  forwarding:
    enabled: true
code_level_metrics:
  enabled: true
thread_profiler:
  enabled: true
```

### Staging Environment

**Focus:** Production-like with moderate reduction

```yaml
# staging-newrelic.yml
transaction_events:
  max_samples_stored: 5000  # 50% reduction
span_events:
  max_samples_stored: 2000
application_logging:
  enabled: true
  forwarding:
    max_samples_stored: 5000
code_level_metrics:
  enabled: false
```

### Production Environment

**Focus:** Maximum cost optimization

```yaml
# prod-newrelic.yml (use ingest-reduction-config.yml)
transaction_events:
  max_samples_stored: 2000  # 80% reduction
span_events:
  max_samples_stored: 2000
application_logging:
  enabled: false
code_level_metrics:
  enabled: false
```

---

## Monitoring and Validation

### Verify Configuration is Applied

**Check Agent Log:**

```bash
tail -f /path/to/logs/newrelic_agent.log | grep -E "(transaction_events|span_events|application_logging)"
```

Look for log entries confirming your settings:

```
INFO: Using transaction_events.max_samples_stored: 2000
INFO: Using span_events.max_samples_stored: 2000
INFO: application_logging.enabled: false
```

### Monitor Data Ingest in New Relic UI

1. Navigate to **Account Settings** → **Data Management** → **Data Ingest**
2. Filter by application name
3. Compare data volumes before and after implementation
4. Track reduction percentage over time

### Key Metrics to Monitor

**Data Ingest Metrics:**
- Total data ingest (GB/day)
- APM events (transaction events, span events)
- Log data volume
- Custom events volume

**Application Health Metrics:**
- Transaction throughput (should be unchanged)
- Error rate detection (should be unchanged)
- Alert sensitivity (verify alerts still fire)
- Trace sampling coverage (should see representative samples)

---

## Troubleshooting

### Issue: Configuration Not Taking Effect

**Solution:**
1. Verify the config file location:
   ```bash
   ps aux | grep newrelic.jar
   ```
2. Check for system property override:
   ```bash
   ps aux | grep "newrelic.config"
   ```
3. Verify environment variables:
   ```bash
   printenv | grep NEW_RELIC
   ```

### Issue: Too Much Data Loss

**Solution:**
Incrementally adjust settings:

```yaml
# Increase sampling rates
transaction_events:
  max_samples_stored: 5000  # Increase from 2000
span_events:
  max_samples_stored: 5000  # Increase from 2000
```

### Issue: Missing Critical Traces

**Solution:**
Enable application logging for critical services:

```yaml
application_logging:
  enabled: true
  forwarding:
    enabled: true
    max_samples_stored: 2000  # Keep reduced but enabled
```

---

## Customization Guide

### Adjusting for Your Workload

**High-Volume Applications (>1000 req/min):**
- Use aggressive settings (as provided)
- Consider dropping to 1000 samples for transaction_events

**Medium-Volume Applications (100-1000 req/min):**
- Use moderate settings
- transaction_events: 3000-5000

**Low-Volume Applications (<100 req/min):**
- Consider less aggressive reduction
- transaction_events: 5000-10000

### Critical Services vs. Non-Critical Services

**Critical Production Services:**
```yaml
transaction_events:
  max_samples_stored: 5000  # More visibility
span_events:
  max_samples_stored: 5000
application_logging:
  enabled: true  # Keep for critical debugging
```

**Non-Critical/Internal Services:**
```yaml
transaction_events:
  max_samples_stored: 1000  # Maximum reduction
span_events:
  max_samples_stored: 1000
application_logging:
  enabled: false  # Disable completely
```

---

## Additional Resources

- [Official Java Agent Configuration Documentation](https://docs.newrelic.com/docs/apm/agents/java-agent/configuration/java-agent-configuration-config-file)
- [Java Agent Installation Guide](https://docs.newrelic.com/docs/apm/agents/java-agent/installation/install-java-agent)
- [Data Ingest Management](https://docs.newrelic.com/docs/data-apis/manage-data/manage-data-coming-new-relic)
- [Java Agent on GitHub](https://github.com/newrelic/newrelic-java-agent)

---

## Support

For questions or issues with this configuration:

1. **Review the Comments:** Each setting in `ingest-reduction-config.yml` has detailed explanations
2. **Check New Relic Documentation:** Links provided above
3. **Monitor Your Results:** Use New Relic's Data Management UI to track impact
4. **Adjust as Needed:** Start conservative and tune based on your specific needs

---

## Change Log

| Date | Change | Impact |
|------|--------|--------|
| 2025-10-31 | Initial configuration created | 40-80% ingest reduction |

---

**Last Updated:** October 31, 2025
