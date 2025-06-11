# Web Load Balancer Monitoring Project - Kelompok 8

## 📋 Deskripsi Project

Project ini merupakan implementasi sistem **Load Balancing** dan **Monitoring** untuk web service menggunakan teknologi Docker, Nginx, Prometheus, dan Grafana. Sistem ini dirancang untuk mendistribusikan beban traffic secara otomatis ke beberapa server backend dan melakukan monitoring real-time terhadap performa sistem.

## 🏗️ Arsitektur Sistem

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Client/User   │    │   Load Balancer  │    │  Backend Servers│
│                 │───▶│   (Nginx LB)     │───▶│   Web1 & Web2   │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                                ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Grafana       │◄───│   Prometheus     │◄───│  Nginx Exporters│
│  (Dashboard)    │    │  (Metrics Store) │    │  (Metrics Gen)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

![Load Balancer Architecture](https://img.shields.io/badge/Architecture-Load%20Balancer-blue?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue?style=for-the-badge&logo=docker)
![Nginx](https://img.shields.io/badge/Nginx-Load%20Balancer-green?style=for-the-badge&logo=nginx)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange?style=for-the-badge&logo=prometheus)
![Grafana](https://img.shields.io/badge/Grafana-Dashboard-red?style=for-the-badge&logo=grafana)

## 🛠️ Tech Stack

- **Docker & Docker Compose** - Containerization
- **Nginx** - Load Balancer & Web Server
- **Prometheus** - Metrics Collection & Storage
- **Grafana** - Monitoring Dashboard
- **Nginx Prometheus Exporter** - Metrics Generator
- **JMeter** - Load Testing Tool

## 📁 Struktur Project

```
sister_project_update/
├── docker-compose.yml          # Konfigurasi container
├── nginx/
│   └── nginx.conf              # Konfigurasi load balancer
├── prometheus/
│   └── prometheus.yml          # Konfigurasi metrics collection
├── web/
│   ├── index.html              # Dashboard web interface
│   ├── styles.css              # Styling dashboard
│   └── jmeter.log              # Log testing
├── web1-config/
│   └── default.conf            # Konfigurasi web server 1
├── web2-config/
│   └── default.conf            # Konfigurasi web server 2
```

## ⚙️ Tahapan Konfigurasi

### 1. Prerequisites

Pastikan sistem Anda memiliki:
- **Docker** (versi 20.10+)
- **Docker Compose** (versi 2.0+)
- **Git** untuk clone repository
- Port tersedia: `3000`, `8080`, `9090`, `9113`, `9114`, `9115`

### 2. Setup Project

```bash
# Clone atau download project
git clone <repository-url>
cd sister_project_update

# Pastikan struktur file sudah sesuai
tree .
```

### 3. Konfigurasi Environment

#### a. Network Configuration
Project menggunakan custom Docker network `monitoring` untuk komunikasi antar container.

#### b. Volume Configuration
```yaml
volumes:
  grafana-storage:  # Persistent storage untuk Grafana
```

### 4. Deployment

#### a. Start All Services
```bash
# Jalankan semua container
docker-compose up -d

# Verifikasi status container
docker-compose ps
```

#### b. Verify Deployment
```bash
# Cek log container
docker-compose logs -f

# Cek network connectivity
docker network ls
docker inspect sister_project_update_monitoring
```

### 5. Akses Services

| Service | URL | Credentials |
|---------|-----|-------------|
| **Load Balancer Dashboard** | http://localhost:8080 | - |
| **Prometheus** | http://localhost:9090 | - |
| **Grafana** | http://localhost:3000 | admin/admin |
| **Nginx Exporter** | http://localhost:9113/metrics | - |
| **Web1 Exporter** | http://localhost:9114/metrics | - |
| **Web2 Exporter** | http://localhost:9115/metrics | - |

### 6. Konfigurasi Grafana Dashboard

```bash
# Login ke Grafana (admin/admin)
# 1. Add Prometheus as Data Source
#    - URL: http://prometheus:9090
#    - Access: Server (default)

# 2. Import Dashboard
#    - Use Grafana ID: 12708 (Nginx Prometheus Exporter)
#    - Or create custom dashboard
```

### 7. Load Testing dengan JMeter

```bash
# Jalankan script testing
chmod +x test.sh
./test.sh

# Atau manual testing
curl -X GET http://localhost:8080
curl -X GET http://localhost:8080/status
```

## 🔧 Konfigurasi Detail

### Load Balancer (Nginx)

- **Algorithm**: Round Robin (default)
- **Backend Servers**: web1:80, web2:80
- **Health Check**: Via `/status` endpoint
- **Logging**: Custom upstream log format

### Monitoring Targets

- **nginx-load-balancer**: Port 9113
- **web1-server**: Port 9114  
- **web2-server**: Port 9115
- **prometheus**: Port 9090

### Metrics Collection

- **Scrape Interval**: 5 seconds (exporters), 15 seconds (prometheus)
- **Retention**: Default Prometheus retention
- **Metrics**: HTTP requests, response times, server status

## 📊 Monitoring Dashboard

Dashboard web interface menyediakan:

- **Real-time Metrics**: Request count per server
- **Server Status**: Health check hasil
- **Active Servers**: Jumlah server aktif
- **Quick Links**: Akses ke Prometheus & Grafana
- **Auto-refresh**: Update setiap 5 detik

## 🚨 Troubleshooting

### Common Issues

1. **Port Already in Use**
   ```bash
   # Cek port yang digunakan
   netstat -tulpn | grep :8080
   # Atau ubah port di docker-compose.yml
   ```

2. **Container Startup Failed**
   ```bash
   # Cek log error
   docker-compose logs <service-name>
   # Restart specific service
   docker-compose restart <service-name>
   ```

3. **Metrics Not Showing**
   ```bash
   # Verifikasi exporter connectivity
   curl http://localhost:9113/metrics
   # Cek Prometheus targets
   # Visit: http://localhost:9090/targets
   ```

4. **Load Balancer Not Working**
   ```bash
   # Test backend servers directly
   curl -H "Host: web1" http://localhost:8080
   curl -H "Host: web2" http://localhost:8080
   ```

### Log Locations

```bash
# Container logs
docker-compose logs nginx
docker-compose logs prometheus
docker-compose logs grafana

# Nginx access logs (inside container)
docker exec nginx-lb tail -f /var/log/nginx/access.log
```

## 📈 Performance Testing

### Load Testing Commands

```bash
# Simple load test
for i in {1..100}; do curl http://localhost:8080; done

# Concurrent testing
ab -n 1000 -c 10 http://localhost:8080/

# JMeter testing
jmeter -n -t loadtest.jmx -l results.jtl
```

### Monitoring During Testing

1. Watch dashboard: http://localhost:8080
2. Check Prometheus metrics: http://localhost:9090
3. View Grafana dashboards: http://localhost:3000

## 🔒 Security Notes

- Metrics endpoints dibatasi untuk private networks
- Grafana menggunakan default credentials (ubah di production)
- Nginx status endpoint hanya dapat diakses dari internal network

## 🛑 Shutdown

```bash
# Stop semua services
docker-compose down

# Stop dan hapus volumes
docker-compose down -v

# Cleanup images (optional)
docker-compose down --rmi all
```

## ✅ Health Checks

Sistem dilengkapi dengan health checks:
- **Load Balancer**: `/status` endpoint
- **Backend Servers**: `/nginx_status` endpoint  
- **Prometheus**: Built-in health endpoint
- **Grafana**: Built-in health endpoint

## 🎯 Next Steps

1. **Custom Grafana Dashboards**: Buat dashboard khusus sesuai kebutuhan
2. **Alert Manager**: Setup alerting untuk monitoring proaktif
3. **SSL/TLS**: Implementasi HTTPS untuk production
4. **Auto Scaling**: Implementasi auto scaling berdasarkan metrics
5. **Backup Strategy**: Setup backup untuk Grafana dashboards dan Prometheus data

---

## 👥 Tim Pengembang

**Kelompok 8** - Web Load Balancer Monitoring Project
- Novaldi Bilqi Akbar Santoso 		(2423600038)
- Farhan Fawwaz Saputra 			    (2423600048)
- Angga Abdul Majid	 			        (2423600050)
- R. Gusti Aryakusuma Dewa Wijaya (2423600058)


## 📞 Support

Jika mengalami issues atau butuh bantuan:
1. Cek troubleshooting guide di atas
2. Review log files untuk error messages
3. Verifikasi semua prerequisites terpenuhi

---

*Project ini dibuat untuk keperluan pembelajaran dan demonstrasi sistem load balancing dengan monitoring real-time.*
