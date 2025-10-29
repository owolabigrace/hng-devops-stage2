# 🚀 Blue-Green Deployment with Nginx (Auto-Failover + Manual Toggle)

### 👋 Overview
This project demonstrates a *Blue/Green deployment pattern* using *Docker Compose* and *Nginx upstreams*.  
It ensures *zero downtime, **auto-failover, and **manual environment switching* — all without rebuilding images.

Both blue and green services are *pre-built Node.js containers* that expose health and version endpoints.

---

## ⚙ Architecture
```
┌────────────┐
      │   Client   │
      └──────┬─────┘
             │
      http://localhost:8080
             │
         ┌───▼───┐
         │ NGINX │  ← Smart proxy
         └───┬───┘
    ┌────────┴────────┐
    │                 │

   ```
 
    ✅ Blue is *active* by default.  
✅ Green is *standby* (backup).  
✅ If Blue fails, Nginx *automatically switches* traffic to Green — with *no failed client requests*.  

---

## 🧩 How It Works

### Core endpoints
```
| Endpoint | Purpose |
|-----------|----------|
| /version | Returns JSON + headers: X-App-Pool, X-Release-Id |
| /healthz | Health/liveness endpoint |
| /chaos/start | Simulates downtime or 500 errors |
| /chaos/stop | Ends simulated failure |
```
### Failover logic
- Nginx defines *primary* and *backup* upstreams.  
- Tight timeouts + low max_fails = fast detection.  
- If Blue fails or times out → Nginx retries Green *in the same request*.  
- Headers like X-App-Pool and X-Release-Id are forwarded unchanged.

---

## 📦 Setup & Run

### 1️⃣ Clone the repo
```bash
git clone https://github.com/owolabigrace/hng-devops-stage2.git
cd hng-devops-stage2
```
## 2️⃣ Create .env
cp .env.example .env
 ## Then Edit as Needed
 -ACTIVE_POOL=blue
-BLUE_IMAGE=yimikaade/wonderful:devops-stage-two.
-GREEN_IMAGE=yimikaade/wonderful:devops-stage-two
## 3️⃣ Start Services
```bash
docker compose up -d
```
## 4️⃣ Verify baseline
curl -i http://localhost:8080/version
## 5️⃣ Simulate failure
```bash
curl -X POST http://localhost:8081/chaos/start?mode=error
sleep 3
curl -i http://localhost:8080/version
```
## Now
- X-App-Pool:green
- X-Release-id:release-green-001

## 6️⃣ Stop failure simulation
  ```
  cul -X POST http://localhost:8081/chaos/stop
  ```
## 7️⃣ Manual switch (optional)
  Update .env
  ``` ACTIVE_POOL:green
```
Then Reload
```
docker compose up -d --force-recreate nginx
```
## 🧠 Design Principles
-	Zero downtime via automatic retry and backup upstream.
-	Stateless switch — no image rebuilds.
-	Config-driven using .env + envsubst.
-	Forward headers — no info lost through proxy.
## Quick Test Loop
```
for i in {1..20}; do curl -s http://localhost:8080/version | jq .; sleep 0.5; done
```
## Project Structure
```
├── docker-compose.yml
├── .env.example
├── .env
├── nginx.conf.template   
├── README.md
└── DECISION.md
```
##✍ Author
-Name: Grace Owolabi
-Slack: @owolabigrace
-Stage 2 — DevOps (Blue/Green Deployment)
-GitHub: https://github.com/owolabigrace/hng-devops-stage2

