Perfect 👍
You want **ONE Prometheus alert → Slack**
and **ONE Loki alert → Gmail**
➡️ **severity-based routing**.
This is **exactly how production setups are done**.

Below is a **clean, copy-paste ready, minimal & correct setup**.

---

# 🎯 GOAL

| Alert Type       | Source               | Severity | Notification  |
| ---------------- | -------------------- | -------- | ------------- |
| High CPU         | Prometheus (metrics) | warning  | Slack         |
| Error logs spike | Loki (logs)          | critical | Email (Gmail) |

---

# 1️⃣ PROMETHEUS ALERT (CPU → SLACK)

### 📄 `prom-cpu-alert.yaml`

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: prom-cpu-alert
  namespace: monitoring
spec:
  groups:
  - name: prometheus-alerts
    rules:
    - alert: HighCPUUsage
      expr: |
        sum(rate(container_cpu_usage_seconds_total{container!="",pod!=""}[1m])) > 0.01
      for: 1m
      labels:
        severity: warning
        source: prometheus
      annotations:
        summary: "High CPU usage detected"
        description: "CPU usage is above threshold for more than 1 minute"
```

Apply:

```bash
kubectl apply -f prom-cpu-alert.yaml
```

---

# 2️⃣ LOKI ALERT (ERROR LOGS → EMAIL)

### 📄 `loki-error-alert.yaml`

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: loki-error-alert
  namespace: monitoring
spec:
  groups:
  - name: loki-alerts
    rules:
    - alert: HighErrorLogs
      expr: |
        sum(
          count_over_time(
            {namespace="default"} |= "error" [2m]
          )
        ) > 5
      for: 1m
      labels:
        severity: critical
        source: loki
      annotations:
        summary: "High error logs detected"
        description: "More than 5 error logs in last 2 minutes"
```

Apply:

```bash
kubectl apply -f loki-error-alert.yaml
```

---

# 3️⃣ ALERTMANAGER ROUTING (SLACK + EMAIL)

### 📄 `alertmanager-config.yaml`

```yaml
global:
  smtp_smarthost: "smtp.gmail.com:587"
  smtp_from: "yourmail@gmail.com"
  smtp_auth_username: "${SMTP_USERNAME}"
  smtp_auth_password: "${SMTP_PASSWORD}"

route:
  receiver: "slack-receiver"
  group_by: ["alertname"]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  routes:
  # Loki alerts → Email
  - matchers:
    - source="loki"
    - severity="critical"
    receiver: "email-receiver"

  # Prometheus alerts → Slack
  - matchers:
    - source="prometheus"
    - severity="warning"
    receiver: "slack-receiver"

receivers:
- name: "slack-receiver"
  slack_configs:
  - api_url: "https://hooks.slack.com/services/T000/B000/XXXX"
    channel: "#alerts"
    title: "{{ .CommonAnnotations.summary }}"
    text: >-
      {{ range .Alerts }}
      *Description:* {{ .Annotations.description }}
      *Severity:* {{ .Labels.severity }}
      {{ end }}

- name: "email-receiver"
  email_configs:
  - to: "receiver@gmail.com"
    send_resolved: true
```

---

# 4️⃣ CREATE EMAIL SECRET (SECURE WAY ✅)

```bash
kubectl create secret generic alertmanager-email-secret \
  -n monitoring \
  --from-literal=smtp_username=yourmail@gmail.com \
  --from-literal=smtp_password=APP_PASSWORD
```

---

# 5️⃣ VALUES FILE TO INJECT SECRETS

### 📄 `values.yaml`

```yaml
alertmanager:
  extraEnv:
    - name: SMTP_USERNAME
      valueFrom:
        secretKeyRef:
          name: alertmanager-email-secret
          key: smtp_username
    - name: SMTP_PASSWORD
      valueFrom:
        secretKeyRef:
          name: alertmanager-email-secret
          key: smtp_password
```

---

# 6️⃣ APPLY ALERTMANAGER CONFIG (HELM)

```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f values.yaml \
  --set-file alertmanager.config=alertmanager-config.yaml
```

---

# 7️⃣ TEST BOTH ALERTS 🔥

### 🔹 Trigger Prometheus CPU alert (Slack)

```bash
kubectl run cpu-load --image=busybox --restart=Never -- sh -c "while true; do :; done"
```

✅ Slack message received

---

### 🔹 Trigger Loki error alert (Email)

```bash
kubectl run error-demo --image=busybox --restart=Never \
  -- sh -c 'while true; do echo "error: db failed"; sleep 10; done'
```

✅ Gmail received

---

# 🧠 HOW TO EXPLAIN THIS IN INTERVIEW ⭐

> “I configured Prometheus metric-based alerts to notify Slack and
> Loki log-based alerts to notify Email using Alertmanager routing
> based on severity and alert source.”

🔥 **This is production-grade wording**

---

# 🧾 RESUME BULLET (FINAL)

```
Implemented metric-based (Prometheus) and log-based (Loki) alerting with
Alertmanager, routing Slack and Email notifications based on alert severity.
```

---

## 🚀 NEXT (OPTIONAL BUT ADVANCED)

* Alert silences & maintenance windows
* SLO-based alerting
* PagerDuty integration
* Alert noise reduction patterns

If you want, say:
**“Next: SLO alerts”** or **“Next: Silence & maintenance”**

You now have **REAL DevOps alerting knowledge** 💪🔥
