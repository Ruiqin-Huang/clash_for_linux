# Clash for Linux

This repository contains the binary files and configuration templates for running **Clash** on Linux systems.

## Quick Start

### 1. Prerequisites
Ensure you are in the project directory:

```bash
cd /path/to/clash_for_linux
```

### 2. Grant Execution Permissions
Before running the binary, you need to make it executable:

```bash
# For AMD64 (x86_64) systems
chmod +x clash-linux-amd64-v1.10.0

# For i386 (32-bit) systems
chmod +x clash-linux-386-v1.10.0
```

### 3. Start Clash
Run the following command to start Clash with your specific configuration file:

```bash
# Syntax: ./clash-linux-amd64-v1.10.0 -f {config_filename}.yaml -d .
./clash-linux-amd64-v1.10.0 -f config.yaml -d .
```

- `-f`: Specifies the configuration file path.
- `-d`: Specifies the working directory (usually `.` for current directory) where the database `Country.mmdb` is located.

---

## Configuration Guide (`config.yaml`)

This section explains the common parameters usually found in a Clash configuration file.

### Basic Configuration
Typically, you only need to focus on `mixed-port`, `mode`, and `external-controller`.

```yaml
# HTTP(S) and SOCKS5 server on the same port (Recommended)
mixed-port: 20000

# Allow other devices on the LAN to connect
allow-lan: false 

# Clash running mode: rule (based on rules), global (all traffic via proxy), or direct
mode: rule

# Output log level: info, warning, error, debug, silent
log-level: silent

# API for external controllers (Dashboard)
external-controller: '0.0.0.0:9090'

# Secret for the external controller (Optional)
secret: ''
```

*   **Tip**: It is recommended to use `mixed-port` instead of separate `port` (HTTP) and `socks-port` (SOCKS5) configurations for simplicity.

### Advanced DNS Configuration

```yaml
dns:
  enable: true        # Enable DNS module
  ipv6: true          # Enable IPv6 support
  enhanced-mode: fake-ip  # or 'redir-host'
  fake-ip-range: 198.18.0.1/16
  # Default DNS servers for resolving non-proxy domains
  default-nameserver: 
    - 223.5.5.5
    - 119.29.29.29
  # DNS servers used by Clash to resolve domain names
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  # Fallback servers for foreign domains
  fallback:
    - https://dns.cloudflare.com/dns-query
    - tls://8.8.4.4:853
```

---

## External Controller & Dashboard

The `external-controller` allows you to manage Clash (switch proxies, view logs) via a web interface or other tools.

### `0.0.0.0` vs `127.0.0.1`

- **`127.0.0.1:9090`**:
    - **Meaning**: The controller only listens on the `localhost` interface.
    - **Effect**: You can **ONLY** access the dashboard from the **same machine** where Clash is running.
    - **Security**: Safer, prevents unauthorized access from the network.

- **`0.0.0.0:9090`**:
    - **Meaning**: The controller listens on **ALL** available network interfaces.
    - **Effect**: You can access the dashboard from **other devices** (e.g., your phone or another laptop) on the same local network using the Linux machine's LAN IP (e.g., `http://192.168.1.5:9090`).
    - **Security**: Less secure if exposed to public networks.

### Web Dashboard (Visual Interface)

Since Clash Core effectively runs as a backend service, you can use a web-based frontend to control it, similar to the Clash for Windows experience.

**Recommended Dashboard**: [metacubexd](https://metacubexd.pages.dev/)

1.  Open [https://metacubexd.pages.dev/](https://metacubexd.pages.dev/) in your browser.
2.  Click the "Settings" or "Add" button to add your controller.
3.  Enter the API address:
    - If opening on the **same machine**: `http://127.0.0.1:9090`
    - If opening on a **different device**: `http://<Linux-IP>:9090` (Requires `external-controller: '0.0.0.0:9090'`)
4.  Enter the `secret` if you configured one in your yaml file.
5.  Click "Add" and select it.

You can now switch proxy groups, view connections, and logs visually!

## System Service (Auto-start)

To keep Clash running in the background and start automatically on boot, you can set it up as a systemd service.

### Setup Steps

1.  **Configure Service File**:
    Modify the provided `clash-linux.service` (or create one) with your paths:
    -   `User`: Your Linux username
    -   `WorkingDirectory`: Absolute path to this project
    -   `ExecStart`: Absolute path to the binary and config file

2.  **Rename Config File**:
    Ensure your configuration file is named `config.yaml` to match the service configuration (or update the service file).

3.  **Install Service**:
    ```bash
    # Copy service file to system directory (force overwrite if exists)
    sudo cp -f /path/to/clash_for_linux/clash-linux.service /etc/systemd/system/
    
    # Reload systemd
    sudo systemctl daemon-reload
    ```

4.  **Enable and Start**:
    ```bash
    # Enable auto-start on boot
    sudo systemctl enable clash-linux.service
    
    # Start immediately
    sudo systemctl start clash-linux.service
    ```

5.  **Check Status**:
    ```bash
    systemctl status clash-linux.service
    ```

### Stop and Remove Service

If you want to stop the Clash service or remove it completely:

1. **Stop the service**:
   ```bash
   sudo systemctl stop clash-linux.service
   ```

2. **Disable auto-start**:
   ```bash
   sudo systemctl disable clash-linux.service
   ```

3. **Remove service file** (Optional, for complete uninstallation):
   ```bash
   sudo rm /etc/systemd/system/clash-linux.service
   sudo systemctl daemon-reload
   ```

