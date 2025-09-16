# taiga Installation Guide

taiga is a free and open-source project management platform. Taiga provides agile project management for developers

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 5GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default taiga port)
  - API on port 8000
- **Dependencies**:
  - See official documentation for specific requirements
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

# Install taiga
sudo dnf install -y taiga

# Enable and start service
sudo systemctl enable --now taiga

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
taiga --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install taiga
sudo apt install -y taiga

# Enable and start service
sudo systemctl enable --now taiga

# Configure firewall
sudo ufw allow 80

# Verify installation
taiga --version
```

### Arch Linux

```bash
# Install taiga
sudo pacman -S taiga

# Enable and start service
sudo systemctl enable --now taiga

# Verify installation
taiga --version
```

### Alpine Linux

```bash
# Install taiga
apk add --no-cache taiga

# Enable and start service
rc-update add taiga default
rc-service taiga start

# Verify installation
taiga --version
```

### openSUSE/SLES

```bash
# Install taiga
sudo zypper install -y taiga

# Enable and start service
sudo systemctl enable --now taiga

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
taiga --version
```

### macOS

```bash
# Using Homebrew
brew install taiga

# Start service
brew services start taiga

# Verify installation
taiga --version
```

### FreeBSD

```bash
# Using pkg
pkg install taiga

# Enable in rc.conf
echo 'taiga_enable="YES"' >> /etc/rc.conf

# Start service
service taiga start

# Verify installation
taiga --version
```

### Windows

```bash
# Using Chocolatey
choco install taiga

# Or using Scoop
scoop install taiga

# Verify installation
taiga --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/taiga

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
taiga --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable taiga

# Start service
sudo systemctl start taiga

# Stop service
sudo systemctl stop taiga

# Restart service
sudo systemctl restart taiga

# Check status
sudo systemctl status taiga

# View logs
sudo journalctl -u taiga -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add taiga default

# Start service
rc-service taiga start

# Stop service
rc-service taiga stop

# Restart service
rc-service taiga restart

# Check status
rc-service taiga status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'taiga_enable="YES"' >> /etc/rc.conf

# Start service
service taiga start

# Stop service
service taiga stop

# Restart service
service taiga restart

# Check status
service taiga status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start taiga
brew services stop taiga
brew services restart taiga

# Check status
brew services list | grep taiga
```

### Windows Service Manager

```powershell
# Start service
net start taiga

# Stop service
net stop taiga

# Using PowerShell
Start-Service taiga
Stop-Service taiga
Restart-Service taiga

# Check status
Get-Service taiga
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream taiga_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name taiga.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name taiga.example.com;

    ssl_certificate /etc/ssl/certs/taiga.example.com.crt;
    ssl_certificate_key /etc/ssl/private/taiga.example.com.key;

    location / {
        proxy_pass http://taiga_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName taiga.example.com
    Redirect permanent / https://taiga.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName taiga.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/taiga.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/taiga.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend taiga_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/taiga.pem
    redirect scheme https if !{ ssl_fc }
    default_backend taiga_backend

backend taiga_backend
    balance roundrobin
    server taiga1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R taiga:taiga /etc/taiga
sudo chmod 750 /etc/taiga

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status taiga

# View logs
sudo journalctl -u taiga -f

# Monitor resource usage
top -p $(pgrep taiga)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/taiga"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/taiga-backup-$DATE.tar.gz" /etc/taiga /var/lib/taiga

echo "Backup completed: $BACKUP_DIR/taiga-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop taiga

# Restore from backup
tar -xzf /backup/taiga/taiga-backup-*.tar.gz -C /

# Start service
sudo systemctl start taiga
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u taiga -n 100
sudo tail -f /var/log/taiga/taiga.log

# Check configuration
taiga --version

# Check permissions
ls -la /etc/taiga
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep taiga)

# Check disk I/O
iotop -p $(pgrep taiga)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  taiga:
    image: taiga:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/taiga
      - ./data:/var/lib/taiga
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update taiga

# Debian/Ubuntu
sudo apt update && sudo apt upgrade taiga

# Arch Linux
sudo pacman -Syu taiga

# Alpine Linux
apk update && apk upgrade taiga

# openSUSE
sudo zypper update taiga

# FreeBSD
pkg update && pkg upgrade taiga

# Always backup before updates
tar -czf /backup/taiga-pre-update-$(date +%Y%m%d).tar.gz /etc/taiga

# Restart after updates
sudo systemctl restart taiga
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/taiga

# Clean old logs
find /var/log/taiga -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/taiga
```

## Additional Resources

- Official Documentation: https://docs.taiga.org/
- GitHub Repository: https://github.com/taiga/taiga
- Community Forum: https://forum.taiga.org/
- Best Practices Guide: https://docs.taiga.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
