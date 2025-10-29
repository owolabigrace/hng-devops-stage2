---

## 💡 *DECISION.md*

### 🎯 Overview
This document explains my thought process and design decisions for the Blue/Green Nginx deployment task.

---

## 🧠 1. Problem Understanding
The goal was to simulate **real-world blue/green deployment** behavior — where one environment (blue) serves live traffic while another (green) stands by for failover or new releases.

I needed:
- Instant **auto-failover** when the active service becomes unhealthy.
- **Retry logic** within Nginx so the client never sees downtime.
- **Manual toggle** capability for controlled rollout.

---

## ⚙ 2. Key Design Choices

### a) **Docker Compose**
I chose Compose to keep orchestration lightweight and CI-friendly.  
Each service (nginx, app_blue, app_green) is isolated but linked via environment variables.

### b) **Template-based Configuration**
Using `envsubst` allows the Nginx config to be dynamically rendered based on:
- `ACTIVE_POOL`
- `BLUE_PORT`, `GREEN_PORT`
- `BLUE_HOST`, `GREEN_HOST`

This avoids hardcoding, so CI can simply set `ACTIVE_POOL=green` to switch traffic.

### c) **Nginx Retry Logic**
Configured:
nginx
proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
# So any failure triggers an immediate retry on the bakcup(gree)

### d) Header Forwarding
# Used
```
proxy_pass_header X-App-Pool;
proxy_pass_header X-Release-Id;
```
# to ensure metadate stays visible - required by the grader

### e) Timeouts
Low fail timeouts(max_fails=1 fail_timeout=2s) for fast detection.

⸻

## 🧩 3. Failover Verification

I simulated chaos using /chaos/start on the Blue service and confirmed:
	-	No 5xx responses.
	-	100% of subsequent requests served by Green.
	-	When chaos stopped, manual reload restored Blue.

⸻

## 🔁 4. Future Improvements

If extended:
	-	Add health-check container to trigger automatic .env switch.
	-	Integrate with CI/CD pipeline (GitHub Actions) to redeploy on tag.
	-	Support graceful traffic drain before switching back.

⸻

## 👩🏽‍💻 Author Note

-I designed this with simplicity, readability, and reliability in mind — mimicking how real teams handle zero-downtime rollouts.

-Signed:
-Grace Owolabi

