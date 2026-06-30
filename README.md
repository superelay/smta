# SMTA Enterprise SMTP Server (v2)

SMTA is a high-performance, enterprise-grade SMTP relay agent and injection server. Architected for high-volume transactional mailings and application integrations, it features a parallel event-driven non-blocking SMTP engine, lock-free high-throughput logging, and advanced Virtual MTA IP pool routing. 

This repository distributes pre-compiled Debian packages (`.deb`) for easy deployment on Debian/Ubuntu systems.

---

# 🚀 Key Features

* **High-Performance epoll Inbound Engine** – Asynchronous, event-driven SMTP ingestion engine capable of accepting thousands of messages per second on a single CPU core.

* **Lock-Free Logging & Webhooks** – Multi-Producer Single-Consumer (MPSC) ring buffers provide high-throughput access logging and delivery accounting without lock contention.

* **Zero-Copy Spooling** – Direct kernel-space message spooling using `sendfile()` and `fallocate()` for maximum throughput and reduced filesystem overhead.

* **REST Transmissions API** – Secure JSON-based HTTP/HTTPS API for high-speed application integrations.

* **Virtual MTAs (VMTAs)** – Advanced outbound IP pool management with domain-based routing, smart-host overrides, and dedicated IP allocation.

* **AI Mode (Intelligent Deliverability Engine)** – AI-powered automation that continuously monitors infrastructure health, optimizes routing decisions, performs automatic IP warming, and protects sender reputation.

* **AI Copilot** – Built-in operational assistant that analyzes delivery statistics, detects anomalies, recommends configuration improvements, and assists administrators with troubleshooting, deliverability optimization, and infrastructure management.

* **Automatic IP Warming** – Safely ramps traffic on new IP addresses using adaptive learning algorithms and reputation-aware sending schedules.

* **Intelligent Traffic Routing** – Dynamically routes outbound mail through the healthiest Virtual MTAs based on reputation, provider feedback, queue load, and real-time delivery performance.

* **Sender IP Health Monitoring** – Continuously evaluates every sending IP using SMTP responses, bounce rates, reputation metrics, blacklist status, throttling events, and delivery success rates.

* **Sender Domain Health Monitoring** – Monitors sender domains for authentication status, reputation trends, DNS configuration, engagement quality, and overall deliverability health.

* **Automatic Blacklist Detection** – Continuously checks outbound IPs against major DNSBL/RBL providers and automatically removes unhealthy IPs from active rotation.

* **Automatic Suppression Engine** – Protects sender reputation by automatically suppressing recipients, domains, or campaigns when configurable bounce, complaint, or failure thresholds are exceeded.

* **Threshold-Based Reputation Protection** – Configurable suppression rules based on bounce percentage, spam complaints, invalid recipients, hard failures, soft failures, engagement metrics, and provider-specific policies.

* **Advanced Reporting & Analytics**

  * Daily delivery reports
  * Monthly volume reports
  * Delivery success trends
  * Domain-wise delivery statistics

* **Bounce Analytics**

  * Hard bounce reports
  * Soft bounce reports
  * SMTP response analysis
  * Bounce categorization
  * Top failing recipient domains
  * Historical bounce trends

* **Web-Based Monitoring Portal**

  * Real-time SMTP activity dashboard
  * Live queue monitoring
  * Active connection monitoring
  * Delivery rate graphs
  * Queue throughput visualization
  * Virtual MTA health dashboard
  * Sender IP health dashboard
  * Sender domain health dashboard

* **System Resource Monitoring**

  * CPU utilization
  * Memory consumption
  * Disk usage
  * Disk I/O performance
  * Network throughput
  * Open SMTP connections
  * Service uptime
  * Real-time performance metrics

* **Deliverability & Security**

  * Outbound DKIM signing
  * Inbound SPF validation
  * Inbound DKIM verification
  * Reverse PTR (iPrev) validation
  * SMTP AUTH brute-force protection
  * Adaptive outbound backoff algorithms
  * TLS support
  * Authentication controls
  * Reputation-aware outbound scheduling

* **Enterprise Administration**

  * Web-based administration portal
  * Role-based user management
  * Queue management
  * Live log viewer
  * REST management APIs
  * Configuration reload without downtime
  * Multi-server deployment support
  * High-performance operational dashboards
---

## 📦 Installation (Debian/Ubuntu)

### 1. Prerequisites (System Dependencies)
SMTA requires the following shared system libraries to run. These will be automatically installed when using `apt`, but you can also install them manually:
```bash
sudo apt-get update
sudo apt-get install -y libssl3 libjansson4 libsqlite3-0 libcurl4 zlib1g
```

### 2. Installing the Debian Package
Download the latest `.deb` package from the **GitHub Releases** page and install it using one of the following methods:

#### Method A: Using `apt` (Recommended - automatically handles dependencies)
```bash
sudo apt-get update
sudo apt-get install ./smta-enterprise_2.1.2_amd64.deb
```

#### Method B: Using `dpkg` (Manual installation)
```bash
# Install the package
sudo dpkg -i smta-enterprise_2.1.2_amd64.deb

# If there are any missing dependency errors, resolve them with:
sudo apt-get install -f
```

### 3. File and Directory Structure Created
Upon installation, the package sets up the following system files and directories:
* **Executables**: 
  * `/usr/local/bin/smta` (High-throughput MTA SMTP daemon)
  * `/usr/local/bin/smta-cli` (Control and queue monitoring utility)
* **Configuration**: Default templates placed at `/etc/smta/smta.conf`
* **Web Dashboard Assets**: Served from `/var/lib/smta/web/`
* **Log Paths**: `/var/log/smta/` (Auto-creates `smta-access.log` and `acct.json`)
* **Spool Paths**: `/var/spool/smta/` (Mail transaction queuing and shards)
* **Systemd Integration**: `/lib/systemd/system/smta.service`

---

## ⚙️ Service Management

SMTA runs as a systemd service daemon:

```bash
# Start the SMTA service
sudo systemctl start smta

# Enable SMTA to start automatically at system boot
sudo systemctl enable smta

# Check the current runtime status
sudo systemctl status smta

# Restart the service (e.g., to load configuration changes)
sudo systemctl restart smta
```

---

## 🔧 Basic Configuration

The main configuration file is located at `/etc/smta/smta.conf`. Below is a basic starter configuration:

```text
# ==============================================================================
# Basic SMTA Configuration (/etc/smta/smta.conf)
# ==============================================================================

# Set canonical hostname and unique host ID
host-id 1
host-name mail.yourdomain.com
license-file /etc/smta/smta.lic

# Inbound Performance Settings
inbound-mode epoll       # Set to 'epoll' for high performance or 'thread' for fallback
inbound-workers 8        # Number of worker threads (0 for auto-detect)

# Listener Configuration
smtp-listener 0.0.0.0:25  # Listen on port 25 across all interfaces

# Admin & API Ports
http-mgmt-port 8080      # Admin UI/Port
http-access 127.0.0.1 admin
transmissions-api-enabled yes
<transmissions-api-listener 0.0.0.0:8081>
</transmissions-api-listener>

# Allow relaying from localhost
<source 127.0.0.1>
    always-allow-relaying yes
    smtp-service yes
    require-auth true
    smtp-user your_smtp_username
    smtp-password your_smtp_password
</source>
```

For advanced settings—such as configuring DKIM keys, Domain Macros, SMTP pattern-matching backoffs, or custom virtual MTA IP pools—please refer to the comments inside the default `/etc/smta/smta.conf` template.

---

## 🛠️ CLI Management (`smta-cli`)

Use the built-in control utility to monitor the server status and control the mail queue:

```bash
# Reload the configuration file without restarting the daemon
smta-cli reload

# Show statistics and inbound/outbound active connection rates
smta-cli status

# List the current active mail queues
smta-cli queue list

# Pause or resume a specific outbound queue
smta-cli queue pause gmail.com
smta-cli queue resume gmail.com

# Delete matching queued messages
# Examples:
smta-cli delete --queue=gmail.com               # Delete all messages queued for gmail.com
smta-cli delete --rcpt=user@example.com         # Delete all messages targeting a specific recipient
smta-cli delete --older-than=2h                # Delete all messages older than 2 hours
smta-cli delete --jobId=job123 --dsn           # Delete messages under job123 and bounce them back to the sender
```

---

## 🔑 Licensing & Support

SMTA Enterprise requires a cryptographically signed license file (`smta.lic`) located under `/etc/smta/` to validate Virtual MTA bounds and enable compliance modules.

To request a commercial license, trial key, or custom deliverability feature development, please contact:

* **Email**: [info@superelay.co.in](mailto:info@superelay.co.in)
* **WhatsApp**: [+91 8887848523](https://wa.me/918887848523)
* **Website**: [superelay.co.in](https://superelay.co.in)
