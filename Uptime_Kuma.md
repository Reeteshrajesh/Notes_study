# 📘 Uptime Kuma: Self-Hosted Monitoring and Alerting Tool

## 🧠 What is Uptime Kuma?

**Uptime Kuma** is an open-source self-hosted monitoring tool like "Uptime Robot." It allows you to monitor the uptime of websites, services (HTTP(s), TCP, DNS, etc.), and get instant alerts when something goes down.

---

## 🚀 Features at a Glance

* 🔍 Monitor HTTP(s), TCP, DNS, Ping, Docker, Push endpoints, and more.
* 📈 Real-time monitoring dashboard
* 🔔 Alert notifications via Telegram, Discord, Email, Slack, Webhooks, etc.
* 🔧 Easy Docker deployment
* 📊 Historical uptime metrics with graphs
* 🧩 Third-party integrations and status page generation

---

## 🐳 Docker Quick Start (Local)

```bash
docker run -d -p 3001:3001 --restart=always \
  -v uptime-kuma:/app/data \
  louislam/uptime-kuma
```

Then access UI via: `http://localhost:3001`

---

## ⚙️ How it Works

1. **Monitoring Configuration**: You add endpoints like your website, backend service, database, API, etc.
2. **Scheduled Health Checks**: Uptime Kuma pings or hits the endpoint every `x` seconds (customizable).
3. **Health Tracking**:

   * Checks for status code, response time, latency, or TCP connection.
   * Optional keyword search in HTTP body.
4. **Alerting**:

   * Sends notification if the target is down or comes back online.
5. **Logging & Metrics**:

   * Shows uptime percentage, incidents, latency chart, etc.
6. **Status Pages**:

   * You can publish custom public status pages for your services.

---

## ✅ Monitor Types

| Monitor Type | Description                                   |
| ------------ | --------------------------------------------- |
| HTTP(s)      | Website or API endpoint monitoring            |
| Ping         | ICMP ping to servers                          |
| TCP Port     | Check if port (e.g., 22, 80) is open          |
| DNS          | Validate DNS resolution                       |
| Push         | Requires external system to push "I am alive" |
| Docker       | Internal Docker container health              |
| MQTT         | MQTT broker/topic monitoring                  |

---

## 🔔 Notification Integrations

Uptime Kuma supports more than 20+ integrations for alerts:

| Integration                | Usage                                         |
| -------------------------- | --------------------------------------------- |
| Telegram                   | Fast and reliable notifications               |
| Discord                    | Ideal for dev communities                     |
| Email (SMTP)               | Standard email alerts                         |
| Slack                      | For team collaboration                        |
| Webhook                    | Integrate with anything (Zapier, custom APIs) |
| Gotify / Pushover          | Mobile push notifications                     |
| Line / Signal              | Secure messaging                              |
| Microsoft Teams            | Enterprise use                                |
| PagerDuty                  | For critical production alerting              |
| WhatsApp (via third-party) | Using gateways or Twilio                      |

➡️ You can set thresholds, retry count, delay time before alerting.

---

## 🧪 Local Testing Example

After launching Uptime Kuma locally:

1. Open `http://localhost:3001`
2. Create a monitor:

   * Name: `My Local App`
   * Type: `HTTP(s)`
   * URL: `http://localhost:8080` (your app)
   * Interval: `30 sec` (or as needed)
3. Add a notification channel (e.g., Telegram)
4. Stop your app (`Ctrl+C`) or simulate failure
5. You will get alert notification (down) and later (up)

---

## 📊 Status Pages (Public or Private)

You can create a beautiful status page:

* Add selected monitors to the status page
* Add logo, theme, and custom domain
* Share with team/stakeholders

---

## 🔒 Security Features

* Admin authentication
* Optional backup/restore of data
* Encrypted notification secrets
* Access control for status pages

---

## 🎯 Advantages of Uptime Kuma

| Advantage                          | Description                        |
| ---------------------------------- | ---------------------------------- |
| 🆓 Free and Open Source            | No licensing cost                  |
| 🖥️ Self-Hosted                    | You control your data              |
| 📱 Mobile-Responsive UI            | Monitor from phone or tablet       |
| 💬 Multi-Channel Alerts            | Choose what fits your workflow     |
| 📈 Graphs & Reports                | Helps in capacity and SLA analysis |
| 🔧 Easy Setup                      | One-liner Docker install           |
| 🔁 REST API & Webhooks             | Extend with automation             |
| 🧩 Integrates with other SRE tools | PagerDuty, Prometheus, etc.        |

---

## 🔧 Ideal Use Cases

* Production service health check
* SSL expiration monitoring
* TCP services (Redis, DB, etc.)
* Internal microservices uptime
* CI/CD pipeline health
* Custom app heartbeat

---

## 🧰 Alternatives to Uptime Kuma

| Tool           | Type                              | Notes                         |
| -------------- | --------------------------------- | ----------------------------- |
| Grafana OnCall | Alert manager                     | Best when using Grafana stack |
| PagerDuty      | Incident response                 | Enterprise-grade              |
| New Relic      | Full observability                | Includes uptime, traces       |
| Better Stack   | Uptime + Logging                  | Free and premium plans        |
| Statping       | Lightweight status page + monitor |                               |

---

## 📎 Resources

* GitHub: [https://github.com/louislam/uptime-kuma](https://github.com/louislam/uptime-kuma)
* Docs: Available in the GitHub wiki section
* Community: Active discussions and feature requests
