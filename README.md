# Qmail Installation Guide

Qmail is a free and open-source Mail Server. A secure, reliable, efficient mail server

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 25/587 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 25/587 (default qmail port)
- **Dependencies**:
  - ucspi-tcp, daemontools
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install qmail
sudo dnf install -y qmail ucspi-tcp, daemontools

# Enable and start service
sudo systemctl enable --now qmail

# Configure firewall
sudo firewall-cmd --permanent --add-service=qmail
sudo firewall-cmd --reload

# Verify installation
qmail --version || systemctl status qmail
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install qmail
sudo apt install -y qmail ucspi-tcp, daemontools

# Enable and start service
sudo systemctl enable --now qmail

# Configure firewall
sudo ufw allow 25/587

# Verify installation
qmail --version || systemctl status qmail
```

### Arch Linux

```bash
# Install qmail
sudo pacman -S qmail

# Enable and start service
sudo systemctl enable --now qmail

# Verify installation
qmail --version || systemctl status qmail
```

### Alpine Linux

```bash
# Install qmail
apk add --no-cache qmail

# Enable and start service
rc-update add qmail default
rc-service qmail start

# Verify installation
qmail --version || rc-service qmail status
```

### openSUSE/SLES

```bash
# Install qmail
sudo zypper install -y qmail ucspi-tcp, daemontools

# Enable and start service
sudo systemctl enable --now qmail

# Configure firewall
sudo firewall-cmd --permanent --add-service=qmail
sudo firewall-cmd --reload

# Verify installation
qmail --version || systemctl status qmail
```

### macOS

```bash
# Using Homebrew
brew install qmail

# Start service
brew services start qmail

# Verify installation
qmail --version
```

### FreeBSD

```bash
# Using pkg
pkg install qmail

# Enable in rc.conf
echo 'qmail_enable="YES"' >> /etc/rc.conf

# Start service
service qmail start

# Verify installation
qmail --version || service qmail status
```

### Windows

```powershell
# Using Chocolatey
choco install qmail

# Or using Scoop
scoop install qmail

# Verify installation
qmail --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /var/qmail/control

# Set up basic configuration
sudo tee /var/qmail/control/qmail.conf << 'EOF'
# Qmail Configuration
concurrencyremote = 255
EOF

# Test configuration
sudo qmail -t || sudo qmail configtest

# Reload service
sudo systemctl reload qmail
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R qmail:qmail /var/qmail/control
sudo chmod 750 /var/qmail/control

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable qmail

# Start service
sudo systemctl start qmail

# Stop service
sudo systemctl stop qmail

# Restart service
sudo systemctl restart qmail

# Reload configuration
sudo systemctl reload qmail

# Check status
sudo systemctl status qmail

# View logs
sudo journalctl -u qmail -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add qmail default

# Start service
rc-service qmail start

# Stop service
rc-service qmail stop

# Restart service
rc-service qmail restart

# Check status
rc-service qmail status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'qmail_enable="YES"' >> /etc/rc.conf

# Start service
service qmail start

# Stop service
service qmail stop

# Restart service
service qmail restart

# Check status
service qmail status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start qmail
brew services stop qmail
brew services restart qmail

# Check status
brew services list | grep qmail
```

### Windows Service Manager

```powershell
# Start service
net start qmail

# Stop service
net stop qmail

# Using PowerShell
Start-Service qmail
Stop-Service qmail
Restart-Service qmail

# Check status
Get-Service qmail
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /var/qmail/control/qmail.conf << 'EOF'
concurrencyremote = 255
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart qmail
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream qmail_backend {
    server 127.0.0.1:25/587;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name qmail.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name qmail.example.com;

    ssl_certificate /etc/ssl/certs/qmail.example.com.crt;
    ssl_certificate_key /etc/ssl/private/qmail.example.com.key;

    location / {
        proxy_pass http://qmail_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName qmail.example.com
    Redirect permanent / https://qmail.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName qmail.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/qmail.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/qmail.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:25/587/
    ProxyPassReverse / http://127.0.0.1:25/587/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:25/587/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend qmail_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/qmail.pem
    redirect scheme https if !{ ssl_fc }
    default_backend qmail_backend

backend qmail_backend
    balance roundrobin
    option httpchk GET /health
    server qmail1 127.0.0.1:25/587 check
    server qmail2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R qmail:qmail /var/qmail/control
sudo chmod 750 /var/qmail/control

# Configure firewall
sudo firewall-cmd --permanent --add-service=qmail
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/qmail.conf << 'EOF'
[qmail]
enabled = true
port = 25/587
filter = qmail
logpath = /var/log/qmail/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/qmail.key \
    -out /etc/ssl/certs/qmail.crt

# Configure SSL in qmail
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE qmail_db;
CREATE USER qmail_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE qmail_db TO qmail_user;
EOF

# Configure qmail to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE qmail_db;
CREATE USER 'qmail_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON qmail_db.* TO 'qmail_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Qmail specific tuning
concurrencyremote = 255
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
qmail soft nofile 65535
qmail hard nofile 65535
qmail soft nproc 32768
qmail hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'qmail'
    static_configs:
      - targets: ['localhost:25/587']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet qmail; then
    echo "Qmail is running"
    exit 0
else
    echo "Qmail is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/qmail << 'EOF'
/var/log/qmail/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 qmail qmail
    postrotate
        systemctl reload qmail > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Qmail backup script
BACKUP_DIR="/backup/qmail"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop qmail

# Backup configuration
tar -czf "$BACKUP_DIR/qmail-config-$DATE.tar.gz" /var/qmail/control

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/qmail-data-$DATE.tar.gz" /var/lib/qmail

# Start service
systemctl start qmail

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop qmail

# Restore configuration
sudo tar -xzf /backup/qmail/qmail-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/qmail/qmail-data-*.tar.gz -C /

# Set permissions
sudo chown -R qmail:qmail /var/qmail/control
sudo chown -R qmail:qmail /var/lib/qmail

# Start service
sudo systemctl start qmail
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u qmail -n 100
sudo tail -f /var/log/qmail/*.log

# Check configuration
sudo qmail -t || sudo qmail configtest

# Check permissions
ls -la /var/qmail/control
ls -la /var/lib/qmail
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 25/587
sudo netstat -tlnp | grep 25/587

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 25/587
nc -zv localhost 25/587
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep qmail-send)
htop -p $(pgrep qmail-send)

# Check connections
ss -ant | grep :25/587 | wc -l

# Monitor I/O
iotop -p $(pgrep qmail-send)
```

### Debug Mode

```bash
# Run in debug mode
sudo qmail -d
# or
sudo qmail debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  qmail:
    image: qmail:latest
    container_name: qmail
    ports:
      - "25/587:25/587"
    volumes:
      - ./config:/var/qmail/control
      - ./data:/var/lib/qmail
    environment:
      - qmail_CONFIG=/var/qmail/control/qmail.conf
    restart: unless-stopped
    networks:
      - qmail_net

networks:
  qmail_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: qmail
spec:
  replicas: 3
  selector:
    matchLabels:
      app: qmail
  template:
    metadata:
      labels:
        app: qmail
    spec:
      containers:
      - name: qmail
        image: qmail:latest
        ports:
        - containerPort: 25/587
        volumeMounts:
        - name: config
          mountPath: /var/qmail/control
      volumes:
      - name: config
        configMap:
          name: qmail-config
---
apiVersion: v1
kind: Service
metadata:
  name: qmail
spec:
  selector:
    app: qmail
  ports:
  - port: 25/587
    targetPort: 25/587
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Qmail
  hosts: all
  become: yes
  tasks:
    - name: Install qmail
      package:
        name: qmail
        state: present
    
    - name: Configure qmail
      template:
        src: qmail.conf.j2
        dest: /var/qmail/control/qmail.conf
        owner: qmail
        group: qmail
        mode: '0640'
      notify: restart qmail
    
    - name: Start and enable qmail
      systemd:
        name: qmail
        state: started
        enabled: yes
  
  handlers:
    - name: restart qmail
      systemd:
        name: qmail
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update qmail

# Debian/Ubuntu
sudo apt update && sudo apt upgrade qmail

# Arch Linux
sudo pacman -Syu qmail

# Alpine Linux
apk update && apk upgrade qmail

# openSUSE
sudo zypper update qmail

# FreeBSD
pkg update && pkg upgrade qmail

# Always backup before updates
tar -czf /backup/qmail-pre-update-$(date +%Y%m%d).tar.gz /var/qmail/control

# Restart after updates
sudo systemctl restart qmail
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/qmail -name "*.log" -mtime +30 -delete

# Verify integrity
sudo qmail --verify || sudo qmail check

# Update databases (if applicable)
sudo qmail-update-db

# Optimize performance
sudo qmail-optimize

# Check for security updates
sudo qmail --security-check
```

## Additional Resources

- Official Documentation: https://docs.qmail.org/
- GitHub Repository: https://github.com/qmail/qmail
- Community Forum: https://forum.qmail.org/
- Wiki: https://wiki.qmail.org/
- Comparison vs Postfix, Exim, Sendmail, OpenSMTPD: https://docs.qmail.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
