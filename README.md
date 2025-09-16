# countly Installation Guide

countly is a free and open-source product analytics. Countly provides product analytics for web, mobile, and desktop

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
  - RAM: 4GB minimum
  - Storage: 20GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 6001 (default countly port)
  - API on 3001
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

# Install countly
sudo dnf install -y countly

# Enable and start service
sudo systemctl enable --now countly

# Configure firewall
sudo firewall-cmd --permanent --add-port=6001/tcp
sudo firewall-cmd --reload

# Verify installation
countly --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install countly
sudo apt install -y countly

# Enable and start service
sudo systemctl enable --now countly

# Configure firewall
sudo ufw allow 6001

# Verify installation
countly --version
```

### Arch Linux

```bash
# Install countly
sudo pacman -S countly

# Enable and start service
sudo systemctl enable --now countly

# Verify installation
countly --version
```

### Alpine Linux

```bash
# Install countly
apk add --no-cache countly

# Enable and start service
rc-update add countly default
rc-service countly start

# Verify installation
countly --version
```

### openSUSE/SLES

```bash
# Install countly
sudo zypper install -y countly

# Enable and start service
sudo systemctl enable --now countly

# Configure firewall
sudo firewall-cmd --permanent --add-port=6001/tcp
sudo firewall-cmd --reload

# Verify installation
countly --version
```

### macOS

```bash
# Using Homebrew
brew install countly

# Start service
brew services start countly

# Verify installation
countly --version
```

### FreeBSD

```bash
# Using pkg
pkg install countly

# Enable in rc.conf
echo 'countly_enable="YES"' >> /etc/rc.conf

# Start service
service countly start

# Verify installation
countly --version
```

### Windows

```bash
# Using Chocolatey
choco install countly

# Or using Scoop
scoop install countly

# Verify installation
countly --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/countly

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
countly --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable countly

# Start service
sudo systemctl start countly

# Stop service
sudo systemctl stop countly

# Restart service
sudo systemctl restart countly

# Check status
sudo systemctl status countly

# View logs
sudo journalctl -u countly -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add countly default

# Start service
rc-service countly start

# Stop service
rc-service countly stop

# Restart service
rc-service countly restart

# Check status
rc-service countly status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'countly_enable="YES"' >> /etc/rc.conf

# Start service
service countly start

# Stop service
service countly stop

# Restart service
service countly restart

# Check status
service countly status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start countly
brew services stop countly
brew services restart countly

# Check status
brew services list | grep countly
```

### Windows Service Manager

```powershell
# Start service
net start countly

# Stop service
net stop countly

# Using PowerShell
Start-Service countly
Stop-Service countly
Restart-Service countly

# Check status
Get-Service countly
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream countly_backend {
    server 127.0.0.1:6001;
}

server {
    listen 80;
    server_name countly.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name countly.example.com;

    ssl_certificate /etc/ssl/certs/countly.example.com.crt;
    ssl_certificate_key /etc/ssl/private/countly.example.com.key;

    location / {
        proxy_pass http://countly_backend;
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
    ServerName countly.example.com
    Redirect permanent / https://countly.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName countly.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/countly.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/countly.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:6001/
    ProxyPassReverse / http://127.0.0.1:6001/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend countly_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/countly.pem
    redirect scheme https if !{ ssl_fc }
    default_backend countly_backend

backend countly_backend
    balance roundrobin
    server countly1 127.0.0.1:6001 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R countly:countly /etc/countly
sudo chmod 750 /etc/countly

# Configure firewall
sudo firewall-cmd --permanent --add-port=6001/tcp
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
sudo systemctl status countly

# View logs
sudo journalctl -u countly -f

# Monitor resource usage
top -p $(pgrep countly)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/countly"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/countly-backup-$DATE.tar.gz" /etc/countly /var/lib/countly

echo "Backup completed: $BACKUP_DIR/countly-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop countly

# Restore from backup
tar -xzf /backup/countly/countly-backup-*.tar.gz -C /

# Start service
sudo systemctl start countly
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u countly -n 100
sudo tail -f /var/log/countly/countly.log

# Check configuration
countly --version

# Check permissions
ls -la /etc/countly
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 6001

# Test connectivity
telnet localhost 6001

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep countly)

# Check disk I/O
iotop -p $(pgrep countly)

# Check connections
ss -an | grep 6001
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  countly:
    image: countly:latest
    ports:
      - "6001:6001"
    volumes:
      - ./config:/etc/countly
      - ./data:/var/lib/countly
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update countly

# Debian/Ubuntu
sudo apt update && sudo apt upgrade countly

# Arch Linux
sudo pacman -Syu countly

# Alpine Linux
apk update && apk upgrade countly

# openSUSE
sudo zypper update countly

# FreeBSD
pkg update && pkg upgrade countly

# Always backup before updates
tar -czf /backup/countly-pre-update-$(date +%Y%m%d).tar.gz /etc/countly

# Restart after updates
sudo systemctl restart countly
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/countly

# Clean old logs
find /var/log/countly -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/countly
```

## Additional Resources

- Official Documentation: https://docs.countly.org/
- GitHub Repository: https://github.com/countly/countly
- Community Forum: https://forum.countly.org/
- Best Practices Guide: https://docs.countly.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
