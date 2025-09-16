# open-edx Installation Guide

open-edx is a free and open-source online learning platform. Open edX powers online learning at scale

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
  - CPU: 4+ cores
  - RAM: 8GB minimum
  - Storage: 50GB for courses
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default open-edx port)
  - Various services
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

# Install open-edx
sudo dnf install -y open_edx

# Enable and start service
sudo systemctl enable --now open-edx

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
open-edx --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install open-edx
sudo apt install -y open_edx

# Enable and start service
sudo systemctl enable --now open-edx

# Configure firewall
sudo ufw allow 80

# Verify installation
open-edx --version
```

### Arch Linux

```bash
# Install open-edx
sudo pacman -S open_edx

# Enable and start service
sudo systemctl enable --now open-edx

# Verify installation
open-edx --version
```

### Alpine Linux

```bash
# Install open-edx
apk add --no-cache open_edx

# Enable and start service
rc-update add open-edx default
rc-service open-edx start

# Verify installation
open-edx --version
```

### openSUSE/SLES

```bash
# Install open-edx
sudo zypper install -y open_edx

# Enable and start service
sudo systemctl enable --now open-edx

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
open-edx --version
```

### macOS

```bash
# Using Homebrew
brew install open_edx

# Start service
brew services start open_edx

# Verify installation
open-edx --version
```

### FreeBSD

```bash
# Using pkg
pkg install open_edx

# Enable in rc.conf
echo 'open-edx_enable="YES"' >> /etc/rc.conf

# Start service
service open-edx start

# Verify installation
open-edx --version
```

### Windows

```bash
# Using Chocolatey
choco install open_edx

# Or using Scoop
scoop install open_edx

# Verify installation
open-edx --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/open_edx

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
open-edx --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable open-edx

# Start service
sudo systemctl start open-edx

# Stop service
sudo systemctl stop open-edx

# Restart service
sudo systemctl restart open-edx

# Check status
sudo systemctl status open-edx

# View logs
sudo journalctl -u open-edx -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add open-edx default

# Start service
rc-service open-edx start

# Stop service
rc-service open-edx stop

# Restart service
rc-service open-edx restart

# Check status
rc-service open-edx status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'open-edx_enable="YES"' >> /etc/rc.conf

# Start service
service open-edx start

# Stop service
service open-edx stop

# Restart service
service open-edx restart

# Check status
service open-edx status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start open_edx
brew services stop open_edx
brew services restart open_edx

# Check status
brew services list | grep open_edx
```

### Windows Service Manager

```powershell
# Start service
net start open-edx

# Stop service
net stop open-edx

# Using PowerShell
Start-Service open-edx
Stop-Service open-edx
Restart-Service open-edx

# Check status
Get-Service open-edx
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream open_edx_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name open_edx.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name open_edx.example.com;

    ssl_certificate /etc/ssl/certs/open_edx.example.com.crt;
    ssl_certificate_key /etc/ssl/private/open_edx.example.com.key;

    location / {
        proxy_pass http://open_edx_backend;
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
    ServerName open_edx.example.com
    Redirect permanent / https://open_edx.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName open_edx.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/open_edx.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/open_edx.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend open_edx_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/open_edx.pem
    redirect scheme https if !{ ssl_fc }
    default_backend open_edx_backend

backend open_edx_backend
    balance roundrobin
    server open_edx1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R open_edx:open_edx /etc/open_edx
sudo chmod 750 /etc/open_edx

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
sudo systemctl status open-edx

# View logs
sudo journalctl -u open-edx -f

# Monitor resource usage
top -p $(pgrep open_edx)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/open_edx"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/open_edx-backup-$DATE.tar.gz" /etc/open_edx /var/lib/open_edx

echo "Backup completed: $BACKUP_DIR/open_edx-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop open-edx

# Restore from backup
tar -xzf /backup/open_edx/open_edx-backup-*.tar.gz -C /

# Start service
sudo systemctl start open-edx
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u open-edx -n 100
sudo tail -f /var/log/open_edx/open_edx.log

# Check configuration
open-edx --version

# Check permissions
ls -la /etc/open_edx
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
top -p $(pgrep open_edx)

# Check disk I/O
iotop -p $(pgrep open_edx)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  open_edx:
    image: open_edx:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/open_edx
      - ./data:/var/lib/open_edx
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update open_edx

# Debian/Ubuntu
sudo apt update && sudo apt upgrade open_edx

# Arch Linux
sudo pacman -Syu open_edx

# Alpine Linux
apk update && apk upgrade open_edx

# openSUSE
sudo zypper update open_edx

# FreeBSD
pkg update && pkg upgrade open_edx

# Always backup before updates
tar -czf /backup/open_edx-pre-update-$(date +%Y%m%d).tar.gz /etc/open_edx

# Restart after updates
sudo systemctl restart open-edx
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/open_edx

# Clean old logs
find /var/log/open_edx -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/open_edx
```

## Additional Resources

- Official Documentation: https://docs.open_edx.org/
- GitHub Repository: https://github.com/open_edx/open_edx
- Community Forum: https://forum.open_edx.org/
- Best Practices Guide: https://docs.open_edx.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
