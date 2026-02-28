# CloudPanel + Matomo Server-Side Log Bridge

A lightweight, enterprise-grade analytics pipeline that provides 100% accurate, unblockable web analytics by directly importing CloudPanel Nginx access logs into Matomo.



## 1. Architecture Overview & Security Bypasses

Standard log importers fail on secure web servers because Cloudflare and Nginx bot-blockers intercept and block automated Python traffic. This architecture solves this through two critical bypasses:

1. **The WAF Bypass (Port 8080):** We route the Matomo API import requests through `http://127.0.0.1:8080` (or your local host equivalent). Port 8080 is CloudPanel’s direct backend PHP-FPM port. By using this local backdoor, the Python script completely bypasses the front-end Varnish cache and web application firewall (WAF), injecting thousands of rows per second directly into the Matomo database securely.
2. **The Authentication Bypass (Token-less):** Because this script runs locally on the host server, it does not use an HTTP API token. It passes the physical path of Matomo's `config.ini.php` file directly to the Python importer. The script reads the database credentials and authenticates internally as a superuser, making it highly secure.

## 2. Core Components & Directory Structure

The system relies on a single executable bash script, a state-tracking directory, and a simple configuration file.

* **Main Executable:** `/usr/local/bin/matomo-log-bridge`
* **Configuration File:** `/etc/matomo-import/sites.conf`
* **State/Offsets Directory:** `/var/lib/matomo-import/` (Stores `.offset` files so `logtail2` never double-counts a line)
* **Logs Directory:** `/var/log/matomo-import/` (Stores `import.log` and process lock files)
* **Dependencies:** `logtail2` (via `apt-get install logcheck`) and `python3`.

## 3. The Configuration File (`sites.conf`)

The bridge is entirely dynamic. It loops through `/etc/matomo-import/sites.conf` to map logs to their proper Matomo buckets.
**Format:** `domain|cloudpanel_user|matomo_site_id|site_type|log_format`

**Example:**
```text
# Sites ready for import
example.com|example_user|4|wordpress|cloudflare
anothersite.net|another_user|5|wordpress|cloudflare
