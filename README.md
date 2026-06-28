# SMTA Enterprise SMTP Server (v2)

SMTA is a high-performance, enterprise-grade SMTP relay agent and injection server. Architected for high-volume transactional mailings and application integrations, it features a parallel event-driven non-blocking SMTP engine, lock-free high-throughput logging, and advanced Virtual MTA IP pool routing. 

This repository distributes pre-compiled Debian packages (`.deb`) for easy deployment on Debian/Ubuntu systems.

---

## 🚀 Key Features

* **High-Performance epoll Inbound Engine**: Asynchronous, event-driven SMTP ingestion engine capable of accepting thousands of messages per second on a single CPU core.
* **Lock-Free Logging & Webhooks**: Uses Multi-Producer Single-Consumer (MPSC) ring buffers for system access and delivery accounting logs, eliminating lock contention on the hot path.
* **Zero-Copy Spooling**: Direct kernel-space zero-copy message spooling (`sendfile`) and block pre-allocation (`fallocate`) to bypass file system metadata bottlenecks.
* **REST Transmissions API**: Inject messages securely using standard JSON payloads over HTTP/HTTPS on port `8081`.
* **Virtual MTAs (VMTAs)**: Dynamic outbound IP mapping to cycle campaigns across dedicated sending IPs, with support for domain-specific smart-host overrides.
* **Deliverability & Safety**:
  * Outbound DKIM signing.
  * Inbound SPF, DKIM, and Reverse PTR (iPrev) validation.
  * SMTP AUTH brute-force rate-limiting.
  * Adaptive outbound backoffs matching target MX SMTP responses.

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
