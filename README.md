# k3s Installation Guide

k3s is a free and open-source lightweight Kubernetes distribution. K3s provides a production-ready Kubernetes distribution optimized for edge, IoT, and resource-constrained environments, serving as a lightweight alternative to full Kubernetes

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
  - RAM: 512MB minimum (1GB+ recommended)
  - Storage: 1GB for installation
  - Network: Cluster networking
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 6443 (default k3s port)
  - Port 10250 for kubelet
- **Dependencies**:
  - systemd or openrc
  - iptables or nftables  
  - containerd (bundled) or Docker
  - curl or wget for installation
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

# Install k3s
sudo dnf install -y k3s

# Enable and start service
sudo systemctl enable --now k3s

# Configure firewall
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --reload

# Verify installation
k3s --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install k3s
sudo apt install -y k3s

# Enable and start service
sudo systemctl enable --now k3s

# Configure firewall
sudo ufw allow 6443

# Verify installation
k3s --version
```

### Arch Linux

```bash
# Install k3s
sudo pacman -S k3s

# Enable and start service
sudo systemctl enable --now k3s

# Verify installation
k3s --version
```

### Alpine Linux

```bash
# Install k3s
apk add --no-cache k3s

# Enable and start service
rc-update add k3s default
rc-service k3s start

# Verify installation
k3s --version
```

### openSUSE/SLES

```bash
# Install k3s
sudo zypper install -y k3s

# Enable and start service
sudo systemctl enable --now k3s

# Configure firewall
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --reload

# Verify installation
k3s --version
```

### macOS

```bash
# Using Homebrew
brew install k3s

# Start service
brew services start k3s

# Verify installation
k3s --version
```

### FreeBSD

```bash
# Using pkg
pkg install k3s

# Enable in rc.conf
echo 'k3s_enable="YES"' >> /etc/rc.conf

# Start service
service k3s start

# Verify installation
k3s --version
```

### Windows

```bash
# Using Chocolatey
choco install k3s

# Or using Scoop
scoop install k3s

# Verify installation
k3s --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/k3s

# Set up basic configuration
cat > /etc/k3s/config.yaml << 'EOF'
# K3s server configuration
write-kubeconfig-mode: "0644"
tls-san:
  - "k3s.example.com"
  - "10.0.0.10"
cluster-cidr: "10.42.0.0/16"
service-cidr: "10.43.0.0/16"
cluster-dns: "10.43.0.10"
cluster-domain: "cluster.local"
log: "/var/log/k3s.log"
EOF

# Test configuration
k3s --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable k3s

# Start service
sudo systemctl start k3s

# Stop service
sudo systemctl stop k3s

# Restart service
sudo systemctl restart k3s

# Check status
sudo systemctl status k3s

# View logs
sudo journalctl -u k3s -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add k3s default

# Start service
rc-service k3s start

# Stop service
rc-service k3s stop

# Restart service
rc-service k3s restart

# Check status
rc-service k3s status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'k3s_enable="YES"' >> /etc/rc.conf

# Start service
service k3s start

# Stop service
service k3s stop

# Restart service
service k3s restart

# Check status
service k3s status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start k3s
brew services stop k3s
brew services restart k3s

# Check status
brew services list | grep k3s
```

### Windows Service Manager

```powershell
# Start service
net start k3s

# Stop service
net stop k3s

# Using PowerShell
Start-Service k3s
Stop-Service k3s
Restart-Service k3s

# Check status
Get-Service k3s
```

## Advanced Configuration

### High Availability Setup
```yaml
# /etc/k3s/config.yaml for HA masters
cluster-init: true  # First master only
server: https://10.0.0.10:6443  # Other masters join here
token: "your-secret-token"

# Embedded etcd configuration
datastore-endpoint: "etcd"
datastore-cafile: "/etc/k3s/etcd/ca.crt"
datastore-certfile: "/etc/k3s/etcd/server.crt" 
datastore-keyfile: "/etc/k3s/etcd/server.key"

# External datastore (PostgreSQL/MySQL)
datastore-endpoint: "postgres://username:password@hostname:5432/k3s"
```

### Network Configuration
```yaml
# Custom CNI configuration
flannel-backend: "vxlan"  # or host-gw, wireguard
flannel-iface: "eth0"

# Disable bundled components
flannel-backend: "none"  # Use Calico/Cilium instead
disable:
  - traefik
  - servicelb
  - metrics-server
  - local-storage

# Node IP configuration  
node-ip: "10.0.0.20"
node-external-ip: "203.0.113.20"
advertise-address: "10.0.0.20"
```

### Security Hardening
```yaml
# CIS Hardening flags
protect-kernel-defaults: true
secretsencryption: true
audit-log-path: "/var/log/k3s-audit.log"
audit-log-maxage: 30
audit-log-maxbackup: 10
audit-log-maxsize: 100

# Pod Security Standards
kube-apiserver-arg:
  - "enable-admission-plugins=NodeRestriction,PodSecurityPolicy"
  - "audit-policy-file=/etc/k3s/audit-policy.yaml"
  - "tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256"

# Kubelet security
kubelet-arg:
  - "read-only-port=0"
  - "streaming-connection-idle-timeout=5m"  
  - "make-iptables-util-chains=true"
```

### Resource Management
```yaml  
# System reserved resources
kubelet-arg:
  - "system-reserved=cpu=500m,memory=1Gi"
  - "kube-reserved=cpu=500m,memory=1Gi"
  - "eviction-hard=memory.available<500Mi,nodefs.available<10%"
  - "max-pods=110"

# Etcd snapshots
etcd-snapshot: true
etcd-snapshot-schedule-cron: "0 */12 * * *"
etcd-snapshot-retention: 5
etcd-s3: true
etcd-s3-bucket: "k3s-backups"
etcd-s3-region: "us-east-1"
```

### GPU Support
```bash
# Install NVIDIA container toolkit first
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

# K3s with GPU
cat >> /etc/k3s/config.yaml << 'EOF'
kubelet-arg:
  - "feature-gates=DevicePlugins=true"
container-runtime-endpoint: "/run/containerd/containerd.sock"
EOF

# Deploy NVIDIA device plugin
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.0/nvidia-device-plugin.yml
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream k3s_backend {
    server 127.0.0.1:6443;
}

server {
    listen 80;
    server_name k3s.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name k3s.example.com;

    ssl_certificate /etc/ssl/certs/k3s.example.com.crt;
    ssl_certificate_key /etc/ssl/private/k3s.example.com.key;

    location / {
        proxy_pass http://k3s_backend;
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
    ServerName k3s.example.com
    Redirect permanent / https://k3s.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName k3s.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/k3s.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/k3s.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:6443/
    ProxyPassReverse / http://127.0.0.1:6443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend k3s_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/k3s.pem
    redirect scheme https if !{ ssl_fc }
    default_backend k3s_backend

backend k3s_backend
    balance roundrobin
    server k3s1 127.0.0.1:6443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R k3s:k3s /etc/k3s
sudo chmod 750 /etc/k3s

# Configure firewall
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

### External Database Configuration

#### PostgreSQL Setup
```sql
-- Create database for K3s
CREATE DATABASE k3s;
CREATE USER k3s WITH ENCRYPTED PASSWORD 'secure-password';
GRANT ALL PRIVILEGES ON DATABASE k3s TO k3s;

-- Required for K3s
ALTER DATABASE k3s SET log_statement TO 'all';
```

```yaml
# /etc/k3s/config.yaml
datastore-endpoint: "postgres://k3s:secure-password@postgres.example.com:5432/k3s?sslmode=require"
```

#### MySQL/MariaDB Setup  
```sql
CREATE DATABASE k3s CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'k3s'@'%' IDENTIFIED BY 'secure-password';
GRANT ALL ON k3s.* TO 'k3s'@'%';
FLUSH PRIVILEGES;
```

```yaml
# /etc/k3s/config.yaml
datastore-endpoint: "mysql://k3s:secure-password@tcp(mysql.example.com:3306)/k3s"
```

### Embedded etcd Backup
```bash
#!/bin/bash
# Backup script for embedded etcd
BACKUP_DIR="/var/backups/k3s"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Take snapshot
k3s etcd-snapshot save \
  --name="backup-${DATE}" \
  --dir="${BACKUP_DIR}"

# Upload to S3 (optional)
aws s3 cp "${BACKUP_DIR}/backup-${DATE}" \
  s3://my-bucket/k3s-backups/
  
# Clean old backups  
find $BACKUP_DIR -name "*.db" -mtime +7 -delete
```

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
sudo systemctl status k3s

# View logs
sudo journalctl -u k3s -f

# Monitor resource usage
top -p $(pgrep k3s)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/k3s"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/k3s-backup-$DATE.tar.gz" /etc/k3s /var/lib/k3s

echo "Backup completed: $BACKUP_DIR/k3s-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop k3s

# Restore from backup
tar -xzf /backup/k3s/k3s-backup-*.tar.gz -C /

# Start service
sudo systemctl start k3s
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u k3s -n 100
sudo tail -f /var/log/k3s/k3s.log

# Check configuration
k3s --version

# Check permissions
ls -la /etc/k3s
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 6443

# Test connectivity
telnet localhost 6443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep k3s)

# Check disk I/O
iotop -p $(pgrep k3s)

# Check connections
ss -an | grep 6443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  k3s:
    image: k3s:latest
    ports:
      - "6443:6443"
    volumes:
      - ./config:/etc/k3s
      - ./data:/var/lib/k3s
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update k3s

# Debian/Ubuntu
sudo apt update && sudo apt upgrade k3s

# Arch Linux
sudo pacman -Syu k3s

# Alpine Linux
apk update && apk upgrade k3s

# openSUSE
sudo zypper update k3s

# FreeBSD
pkg update && pkg upgrade k3s

# Always backup before updates
tar -czf /backup/k3s-pre-update-$(date +%Y%m%d).tar.gz /etc/k3s

# Restart after updates
sudo systemctl restart k3s
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/k3s

# Clean old logs
find /var/log/k3s -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/k3s
```

## Additional Resources

- Official Documentation: https://docs.k3s.org/
- GitHub Repository: https://github.com/k3s/k3s
- Community Forum: https://forum.k3s.org/
- Best Practices Guide: https://docs.k3s.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
