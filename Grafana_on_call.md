# 📘 Grafana OnCall Integration Guide

## 🎯 What is Grafana OnCall?

**Grafana OnCall** is an alert management and on-call scheduling tool tightly integrated with **Grafana**. It provides **on-call rotation**, **escalation policies**, and **alert deduplication**, built to unify alerts from tools like Prometheus, Loki, Alertmanager, and forward them via channels like Slack, phone, or **PagerDuty**.

---

## 🧩 Key Features of Grafana OnCall

| Feature                      | Description                                                    |
| ---------------------------- | -------------------------------------------------------------- |
| 🔔 Alert routing             | Receive alerts from Prometheus, Loki, Alertmanager, etc.       |
| 👨‍💻 On-call schedules      | Define rotating on-call engineers with flexible calendar views |
| 🆘 Escalation policies       | Automatically escalate incidents if not acknowledged/resolved  |
| 🔁 Alert deduplication       | Avoid alert noise by grouping similar alerts                   |
| 🧑‍🤝‍🧑 Team-based alerting | Assign alerts to teams with dedicated rotations                |
| 📲 Notifications             | Integrate with Slack, PagerDuty, Microsoft Teams, Email        |
| 📚 Alert logs                | Track incidents, responses, and SLA adherence                  |

---

## 🧰 Integrating Grafana OnCall in Production

### ✅ Step 1: Enable Grafana OnCall

If you're using **Grafana Cloud**:

* OnCall is pre-installed → Just activate from the **OnCall** tab.

For **self-hosted Grafana**:

* Requires **Grafana Enterprise** (or use **Grafana OnCall OSS** via the plugin)
* Docs: [https://grafana.com/docs/oncall/](https://grafana.com/docs/oncall/)

---

### ✅ Step 2: Create Your Team and Schedule

1. Go to **Grafana → OnCall → Teams**
2. Create a **Team** (e.g., `backend`, `infra`, `prod-support`)
3. Add members and define:

   * **Primary** and **Secondary** responders
   * **On-call schedule** (calendar-based rotation)

---

### ✅ Step 3: Connect Alert Sources

Supported sources:

* Prometheus (via Alertmanager)
* Loki
* Zabbix
* Webhook
* AWS CloudWatch (via SNS → Webhook)
* Grafana Alerts

Example: **Prometheus + Alertmanager → OnCall**

In your `alertmanager.yml`:

```yaml
receivers:
  - name: grafana-oncall
    webhook_configs:
      - url: https://oncall.example.com/api/v1/alertmanager
```

You’ll get the **Webhook URL** from OnCall’s **Alert Source** settings.

---

### ✅ Step 4: Add Notification Channels

Under **Grafana → OnCall → Notification Channels**:

* Add **PagerDuty**

  * Paste the **Routing Key** from your PagerDuty integration
* Add **Slack**, **Email**, or **Microsoft Teams** as fallback channels

---

### ✅ Step 5: Create Escalation Policies

Set rules like:

* If alert not acknowledged in 5 minutes → escalate to next engineer
* If still unresolved in 10 minutes → notify PagerDuty or backup team

Example escalation chain:

```
[Level 1] → On-call engineer (via Slack)
   ↓ 5 min
[Level 2] → PagerDuty escalation
   ↓ 10 min
[Level 3] → Manager via phone call
```

---

### ✅ Step 6: Trigger and Test Alerts

Trigger an alert from Prometheus or use a test endpoint from OnCall UI.

Example:

```bash
curl -X POST https://oncall.example.com/api/v1/alerts/test \
  -H "Authorization: Bearer <your-api-key>"
```

Check the alert:

* Was it routed to the right team?
* Did escalation work?
* Was PagerDuty triggered?

---

## 🎁 Benefits of Grafana OnCall in Production

| Benefit                     | Value                                              |
| --------------------------- | -------------------------------------------------- |
| 🧑‍💼 Team-specific alerts  | No alert goes to the wrong team                    |
| 🕐 Reduce MTTR              | Escalations, deduplication = faster resolution     |
| 🧭 Full observability stack | Integrates with Prometheus, Loki, Tempo, etc.      |
| 📱 Native mobile support    | PagerDuty integration + Slack = alerts anywhere    |
| 📚 Alert timeline           | Know who got alerted, when, and how they responded |
| 🔐 Secure access control    | RBAC support with Grafana teams and roles          |

---

## 📎 Resources

* 📘 [Grafana OnCall Documentation](https://grafana.com/docs/oncall/)
* 🔗 [PagerDuty Integration with OnCall](https://grafana.com/docs/oncall/latest/integrations/pagerduty/)
* 💬 [Grafana Community for OnCall](https://community.grafana.com/)

---

## 🧪 When to Use Grafana OnCall?

* ✅ If you use Grafana and Prometheus/Loki
* ✅ If you want Slack/PagerDuty alerts with rich incident history
* ✅ If you need **on-call rotation + alert routing** in one system
* ✅ If you want to **centralize incident response** without additional tooling
