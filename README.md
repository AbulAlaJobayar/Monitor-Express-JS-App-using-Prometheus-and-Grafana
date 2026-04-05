# 🚀 Monitor Express JS App using Prometheus & Grafana


## 📌 Prerequisites
Before starting, make sure you have:
- **Node.js** and **npm** installed
- **Prometheus** installed and running
- **Grafana** installed and running

---

## ⚡ Step 1: Initialize Node.js Project
Open your terminal and run:

```bash
mkdir express-prometheus-grafana
cd express-prometheus-grafana
npm init -y
npm install express prom-client
```

---

## ⚡ Step 2: Create Express Server with Prometheus Metrics
Create a file named **`server.js`** and add the following code:
```js
const express = require("express");
const client = require("prom-client");

const app = express();
const PORT = 3001;

// Registry
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metric: total requests counter
const httpRequestCount = new client.Counter({
  name: "http_request_count",
  help: "Total number of HTTP requests",
});
register.registerMetric(httpRequestCount);

// Middleware: increase counter on each request
app.use((req, res, next) => {
  httpRequestCount.inc();
  next();
});

// Example route
app.get("/", (req, res) => {
  res.send("Hello Prometheus + Grafana!");
});

// Metrics endpoint for Prometheus
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```
```javascript
const express = require("express");
const client = require("prom-client");

const app = express();
const PORT = 3001;

// Register metrics
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metric (HTTP request duration)
const httpRequestDuration = new client.Histogram({
  name: "http_request_duration_seconds",
  help: "Duration of HTTP requests in seconds",
  labelNames: ["method", "route", "status_code"],
});
register.registerMetric(httpRequestDuration);

// Middleware for measuring request duration
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on("finish", () => {
    end({ method: req.method, route: req.path, status_code: res.statusCode });
  });
  next();
});

// Routes
app.get("/", (req, res) => {
  res.send("Hello, Express with Prometheus!");
});

// Prometheus metrics endpoint
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});

// Start server
app.listen(PORT, "0.0.0.0", () => {
  console.log(`✅ Server running on http://localhost:${PORT}`);
});
```

Run the server:

```bash
npm start
```

👉 Open in browser: [http://localhost:3001/metrics](http://localhost:3001/metrics) to see Prometheus metrics.

---

## ⚡ Step 3: Configure Prometheus
Open Prometheus configuration file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add the following job:

```yaml
scrape_configs:
  - job_name: "express-app"
    static_configs:
      - targets: ["<your-system-ip>:3001"]
```

👉 Replace `<your-system-ip>` with your actual IP (e.g., `192.168.1.100:3001`).

Save file (**CTRL + O, Enter, CTRL + X**).

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

Check targets:

👉 Open [http://localhost:9090/classic/targets](http://localhost:9090/classic/targets) and verify **express-app** status is **UP** ✅

---

## ⚡ Step 4: Visualize Metrics in Grafana
1. Open Grafana: [http://localhost:3000](http://localhost:3000)
2. Go to **Dashboards > New Dashboard > Add Query**
3. Select **Prometheus** as data source
4. Run this query:

```promql
sum by (instance, method) (rate(http_request_duration_seconds_count[5m]))
```

5. Set **Legend** as:
```
{{instance}} - {{method}}
```
6. Under **Standard Options > Unit**, select `requests/sec (rps)`

🎉 Now you can see **real-time Express app metrics** in Grafana!

---

## 📊 Example Dashboard
- HTTP Requests per second (RPS)
- Average Request Duration
- Success vs Failed Requests

---


---



## ⚡ Step 4: Configure SMTP in Grafana (Email Settings)

Edit Grafana config:

```bash
sudo nano /etc/grafana/grafana.ini
```

Uncomment + set:

```ini
[smtp]
enabled = true
host = smtp.gmail.com:587
user = abulalajobayar@gmail.com
password = "bdof axxf khxi hzkq"   ; (App Password from Gmail)
from_address = abulalajobayar@gmail.com
from_name = Grafana
startTLS_policy = OpportunisticStartTLS
```

Save + restart Grafana:

```bash
sudo systemctl restart grafana-server
sudo systemctl status grafana-server
```

---

## ⚡ Step 5: Create Contact Point in Grafana

1. Open Grafana UI → **Alerting → Contact Points → New Contact Point**  
2. Type: **Email**  
3. Name: `My Email`  
4. Recipients: `your@domain.com`  
5. Save  
6. (Optional) Send Test Email → check inbox ✅  

---

## ⚡ Step 6: Create Dashboard in Grafana

1. Open Grafana UI → **Dashboards → New Dashboard → Add new panel**  
2. In query box, write:

   ```promql
   rate(http_request_count[1m])
   ```

3. Panel will show **per-minute request rate**.  
4. Go to **Alert tab** → Enable Alert.  
5. Condition: `A > 0` → means request received.  
6. Notification → Select Contact Point: `My Email`.  
7. Save Dashboard ✅  

---

## ✅ Final Result

- Express app will expose metrics.
- Prometheus will scrape those metrics.
- Grafana dashboard will show request rate graph (`rate(http_request_count[1m])`).
- If a request comes in → condition `A > 0` triggers.
- Grafana will immediately send you an **email alert** 🚨📧

👉 **Result Example:**

- Open [http://localhost:3001/](http://localhost:3001/) → request count increases.
- Grafana dashboard shows a spike 📈
- Your email inbox receives:  
  *"Alert firing: Express App Request Alert – at least 1 request detected!"*

---

## 🚀 Conclusion
You have successfully:
- Built an **Express.js app** with Prometheus metrics
- Configured **Prometheus** to scrape metrics
- Created **Grafana dashboard** for visualization

Now your Express app is fully **monitored with Grafana + Prometheus** 🎯
