# microk8s Installation Guide

microk8s is a free and open-source minimal, lightweight Kubernetes. Developed by Canonical, MicroK8s provides a single-package Kubernetes for developers, IoT, and edge deployments

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
  - CPU: 1 core minimum (2+ recommended)
  - RAM: 540MB minimum (4GB+ recommended)
  - Storage: 20GB recommended
  - Network: Cluster networking
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 16443 (default microk8s port)
  - Various Kubernetes ports
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

# Install microk8s
sudo dnf install -y microk8s

# Enable and start service
sudo systemctl enable --now microk8s

# Configure firewall
sudo firewall-cmd --permanent --add-port=16443/tcp
sudo firewall-cmd --reload

# Verify installation
microk8s version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install microk8s
sudo apt install -y microk8s

# Enable and start service
sudo systemctl enable --now microk8s

# Configure firewall
sudo ufw allow 16443

# Verify installation
microk8s version
```

### Arch Linux

```bash
# Install microk8s
sudo pacman -S microk8s

# Enable and start service
sudo systemctl enable --now microk8s

# Verify installation
microk8s version
```

### Alpine Linux

```bash
# Install microk8s
apk add --no-cache microk8s

# Enable and start service
rc-update add microk8s default
rc-service microk8s start

# Verify installation
microk8s version
```

### openSUSE/SLES

```bash
# Install microk8s
sudo zypper install -y microk8s

# Enable and start service
sudo systemctl enable --now microk8s

# Configure firewall
sudo firewall-cmd --permanent --add-port=16443/tcp
sudo firewall-cmd --reload

# Verify installation
microk8s version
```

### macOS

```bash
# Using Homebrew
brew install microk8s

# Start service
brew services start microk8s

# Verify installation
microk8s version
```

### FreeBSD

```bash
# Using pkg
pkg install microk8s

# Enable in rc.conf
echo 'microk8s_enable="YES"' >> /etc/rc.conf

# Start service
service microk8s start

# Verify installation
microk8s version
```

### Windows

```bash
# Using Chocolatey
choco install microk8s

# Or using Scoop
scoop install microk8s

# Verify installation
microk8s version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/microk8s

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
microk8s version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable microk8s

# Start service
sudo systemctl start microk8s

# Stop service
sudo systemctl stop microk8s

# Restart service
sudo systemctl restart microk8s

# Check status
sudo systemctl status microk8s

# View logs
sudo journalctl -u microk8s -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add microk8s default

# Start service
rc-service microk8s start

# Stop service
rc-service microk8s stop

# Restart service
rc-service microk8s restart

# Check status
rc-service microk8s status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'microk8s_enable="YES"' >> /etc/rc.conf

# Start service
service microk8s start

# Stop service
service microk8s stop

# Restart service
service microk8s restart

# Check status
service microk8s status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start microk8s
brew services stop microk8s
brew services restart microk8s

# Check status
brew services list | grep microk8s
```

### Windows Service Manager

```powershell
# Start service
net start microk8s

# Stop service
net stop microk8s

# Using PowerShell
Start-Service microk8s
Stop-Service microk8s
Restart-Service microk8s

# Check status
Get-Service microk8s
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream microk8s_backend {
    server 127.0.0.1:16443;
}

server {
    listen 80;
    server_name microk8s.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name microk8s.example.com;

    ssl_certificate /etc/ssl/certs/microk8s.example.com.crt;
    ssl_certificate_key /etc/ssl/private/microk8s.example.com.key;

    location / {
        proxy_pass http://microk8s_backend;
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
    ServerName microk8s.example.com
    Redirect permanent / https://microk8s.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName microk8s.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/microk8s.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/microk8s.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:16443/
    ProxyPassReverse / http://127.0.0.1:16443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend microk8s_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/microk8s.pem
    redirect scheme https if !{ ssl_fc }
    default_backend microk8s_backend

backend microk8s_backend
    balance roundrobin
    server microk8s1 127.0.0.1:16443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R microk8s:microk8s /etc/microk8s
sudo chmod 750 /etc/microk8s

# Configure firewall
sudo firewall-cmd --permanent --add-port=16443/tcp
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
sudo systemctl status microk8s

# View logs
sudo journalctl -u microk8s -f

# Monitor resource usage
top -p $(pgrep microk8s)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/microk8s"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/microk8s-backup-$DATE.tar.gz" /etc/microk8s /var/lib/microk8s

echo "Backup completed: $BACKUP_DIR/microk8s-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop microk8s

# Restore from backup
tar -xzf /backup/microk8s/microk8s-backup-*.tar.gz -C /

# Start service
sudo systemctl start microk8s
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u microk8s -n 100
sudo tail -f /var/log/microk8s/microk8s.log

# Check configuration
microk8s version

# Check permissions
ls -la /etc/microk8s
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 16443

# Test connectivity
telnet localhost 16443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep microk8s)

# Check disk I/O
iotop -p $(pgrep microk8s)

# Check connections
ss -an | grep 16443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  microk8s:
    image: microk8s:latest
    ports:
      - "16443:16443"
    volumes:
      - ./config:/etc/microk8s
      - ./data:/var/lib/microk8s
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update microk8s

# Debian/Ubuntu
sudo apt update && sudo apt upgrade microk8s

# Arch Linux
sudo pacman -Syu microk8s

# Alpine Linux
apk update && apk upgrade microk8s

# openSUSE
sudo zypper update microk8s

# FreeBSD
pkg update && pkg upgrade microk8s

# Always backup before updates
tar -czf /backup/microk8s-pre-update-$(date +%Y%m%d).tar.gz /etc/microk8s

# Restart after updates
sudo systemctl restart microk8s
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/microk8s

# Clean old logs
find /var/log/microk8s -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/microk8s
```

## Additional Resources

- Official Documentation: https://docs.microk8s.org/
- GitHub Repository: https://github.com/microk8s/microk8s
- Community Forum: https://forum.microk8s.org/
- Best Practices Guide: https://docs.microk8s.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
