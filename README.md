# funkwhale Installation Guide

funkwhale is a free and open-source social music platform. Funkwhale enables sharing and listening to music within a decentralized, open network

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
  - Storage: 10GB+ for music
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5000 (default funkwhale port)
  - None
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

# Install funkwhale
sudo dnf install -y funkwhale

# Enable and start service
sudo systemctl enable --now funkwhale

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
funkwhale --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install funkwhale
sudo apt install -y funkwhale

# Enable and start service
sudo systemctl enable --now funkwhale

# Configure firewall
sudo ufw allow 5000

# Verify installation
funkwhale --version
```

### Arch Linux

```bash
# Install funkwhale
sudo pacman -S funkwhale

# Enable and start service
sudo systemctl enable --now funkwhale

# Verify installation
funkwhale --version
```

### Alpine Linux

```bash
# Install funkwhale
apk add --no-cache funkwhale

# Enable and start service
rc-update add funkwhale default
rc-service funkwhale start

# Verify installation
funkwhale --version
```

### openSUSE/SLES

```bash
# Install funkwhale
sudo zypper install -y funkwhale

# Enable and start service
sudo systemctl enable --now funkwhale

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
funkwhale --version
```

### macOS

```bash
# Using Homebrew
brew install funkwhale

# Start service
brew services start funkwhale

# Verify installation
funkwhale --version
```

### FreeBSD

```bash
# Using pkg
pkg install funkwhale

# Enable in rc.conf
echo 'funkwhale_enable="YES"' >> /etc/rc.conf

# Start service
service funkwhale start

# Verify installation
funkwhale --version
```

### Windows

```bash
# Using Chocolatey
choco install funkwhale

# Or using Scoop
scoop install funkwhale

# Verify installation
funkwhale --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/funkwhale

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
funkwhale --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable funkwhale

# Start service
sudo systemctl start funkwhale

# Stop service
sudo systemctl stop funkwhale

# Restart service
sudo systemctl restart funkwhale

# Check status
sudo systemctl status funkwhale

# View logs
sudo journalctl -u funkwhale -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add funkwhale default

# Start service
rc-service funkwhale start

# Stop service
rc-service funkwhale stop

# Restart service
rc-service funkwhale restart

# Check status
rc-service funkwhale status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'funkwhale_enable="YES"' >> /etc/rc.conf

# Start service
service funkwhale start

# Stop service
service funkwhale stop

# Restart service
service funkwhale restart

# Check status
service funkwhale status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start funkwhale
brew services stop funkwhale
brew services restart funkwhale

# Check status
brew services list | grep funkwhale
```

### Windows Service Manager

```powershell
# Start service
net start funkwhale

# Stop service
net stop funkwhale

# Using PowerShell
Start-Service funkwhale
Stop-Service funkwhale
Restart-Service funkwhale

# Check status
Get-Service funkwhale
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream funkwhale_backend {
    server 127.0.0.1:5000;
}

server {
    listen 80;
    server_name funkwhale.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name funkwhale.example.com;

    ssl_certificate /etc/ssl/certs/funkwhale.example.com.crt;
    ssl_certificate_key /etc/ssl/private/funkwhale.example.com.key;

    location / {
        proxy_pass http://funkwhale_backend;
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
    ServerName funkwhale.example.com
    Redirect permanent / https://funkwhale.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName funkwhale.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/funkwhale.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/funkwhale.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend funkwhale_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/funkwhale.pem
    redirect scheme https if !{ ssl_fc }
    default_backend funkwhale_backend

backend funkwhale_backend
    balance roundrobin
    server funkwhale1 127.0.0.1:5000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R funkwhale:funkwhale /etc/funkwhale
sudo chmod 750 /etc/funkwhale

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
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
sudo systemctl status funkwhale

# View logs
sudo journalctl -u funkwhale -f

# Monitor resource usage
top -p $(pgrep funkwhale)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/funkwhale"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/funkwhale-backup-$DATE.tar.gz" /etc/funkwhale /var/lib/funkwhale

echo "Backup completed: $BACKUP_DIR/funkwhale-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop funkwhale

# Restore from backup
tar -xzf /backup/funkwhale/funkwhale-backup-*.tar.gz -C /

# Start service
sudo systemctl start funkwhale
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u funkwhale -n 100
sudo tail -f /var/log/funkwhale/funkwhale.log

# Check configuration
funkwhale --version

# Check permissions
ls -la /etc/funkwhale
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5000

# Test connectivity
telnet localhost 5000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep funkwhale)

# Check disk I/O
iotop -p $(pgrep funkwhale)

# Check connections
ss -an | grep 5000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  funkwhale:
    image: funkwhale:latest
    ports:
      - "5000:5000"
    volumes:
      - ./config:/etc/funkwhale
      - ./data:/var/lib/funkwhale
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update funkwhale

# Debian/Ubuntu
sudo apt update && sudo apt upgrade funkwhale

# Arch Linux
sudo pacman -Syu funkwhale

# Alpine Linux
apk update && apk upgrade funkwhale

# openSUSE
sudo zypper update funkwhale

# FreeBSD
pkg update && pkg upgrade funkwhale

# Always backup before updates
tar -czf /backup/funkwhale-pre-update-$(date +%Y%m%d).tar.gz /etc/funkwhale

# Restart after updates
sudo systemctl restart funkwhale
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/funkwhale

# Clean old logs
find /var/log/funkwhale -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/funkwhale
```

## Additional Resources

- Official Documentation: https://docs.funkwhale.org/
- GitHub Repository: https://github.com/funkwhale/funkwhale
- Community Forum: https://forum.funkwhale.org/
- Best Practices Guide: https://docs.funkwhale.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
