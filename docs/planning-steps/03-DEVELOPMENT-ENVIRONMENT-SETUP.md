# Step 3: Set up Development Environment - Laravel + VoIP Stack

## Overview

This document provides a comprehensive guide to setting up the complete development environment for the VoIP platform, including Laravel backend, VoIP infrastructure (Kamailio, FreeSWITCH, RTPEngine), SMS gateway (Jasmin), and all supporting services.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│              Development Environment Stack                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Application Layer                                          │
│  ├── Laravel 11.x (PHP 8.2+)                               │
│  ├── MySQL 8.0                                             │
│  ├── Redis 7.x                                             │
│  └── Node.js 20.x (for frontend builds)                    │
│                                                             │
│  VoIP Layer                                                 │
│  ├── Kamailio 5.7.x (SIP Proxy)                           │
│  ├── FreeSWITCH 1.10.x (Media Server)                      │
│  ├── RTPEngine (Media Proxy)                              │
│  └── Coturn (STUN/TURN Server)                            │
│                                                             │
│  Messaging Layer                                            │
│  ├── Jasmin SMS Gateway                                    │
│  └── RabbitMQ 3.12.x (Message Queue)                       │
│                                                             │
│  Development Tools                                          │
│  ├── Docker & Docker Compose                              │
│  ├── Git                                                   │
│  ├── Composer                                              │
│  ├── NPM/Yarn                                              │
│  └── VS Code / PHPStorm                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

### System Requirements

**Minimum Development Machine:**
- OS: Ubuntu 22.04 LTS / macOS 13+ / Windows 11 with WSL2
- CPU: 8 cores (16 threads recommended)
- RAM: 16GB (32GB recommended)
- Storage: 100GB SSD
- Network: Stable internet connection

**Recommended:**
- CPU: 12+ cores
- RAM: 32GB+
- Storage: 256GB NVMe SSD
- Network: Gigabit ethernet

---

## Part 1: Base System Setup

### 1.1 Update System

```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# Install essential build tools
sudo apt install -y build-essential curl git wget vim \
    software-properties-common apt-transport-https ca-certificates \
    gnupg lsb-release
```

### 1.2 Install Docker & Docker Compose

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
    -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker --version
docker-compose --version

# Log out and back in for group changes to take effect
```

---

## Part 2: Laravel Application Setup

### 2.1 Install PHP 8.2+

```bash
# Add PHP repository
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP and extensions
sudo apt install -y php8.2 php8.2-fpm php8.2-cli php8.2-common \
    php8.2-mysql php8.2-zip php8.2-gd php8.2-mbstring \
    php8.2-curl php8.2-xml php8.2-bcmath php8.2-redis \
    php8.2-intl php8.2-soap php8.2-imagick

# Verify installation
php -v
```

### 2.2 Install Composer

```bash
# Download and install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer

# Verify installation
composer --version
```

### 2.3 Install Node.js & NPM

```bash
# Install Node.js 20.x LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node --version
npm --version

# Install Yarn (optional)
npm install -g yarn
```

### 2.4 Create Laravel Project

```bash
# Navigate to projects directory
cd /home/talha/projects/voIP

# Create Laravel project (if not already created)
composer create-project laravel/laravel backend "11.*"

# Navigate to project
cd backend

# Install additional packages
composer require laravel/sanctum
composer require laravel/horizon
composer require predis/predis
composer require guzzlehttp/guzzle
composer require spatie/laravel-permission
composer require barryvdh/laravel-debugbar --dev

# Publish configurations
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

# Set permissions
sudo chown -R $USER:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

### 2.5 Configure Environment

```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Edit .env file
nano .env
```

**`.env` Configuration:**

```env
APP_NAME="VoIP Platform"
APP_ENV=local
APP_KEY=base64:GENERATED_KEY
APP_DEBUG=true
APP_URL=http://localhost:8000

LOG_CHANNEL=stack
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=voip_platform
DB_USERNAME=voip_user
DB_PASSWORD=secure_password

BROADCAST_DRIVER=redis
CACHE_DRIVER=redis
FILESYSTEM_DISK=local
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

# VoIP Configuration
KAMAILIO_HOST=127.0.0.1
KAMAILIO_PORT=5060
FREESWITCH_HOST=127.0.0.1
FREESWITCH_ESL_PORT=8021
FREESWITCH_ESL_PASSWORD=ClueCon

# SMS Configuration
JASMIN_API_URL=http://127.0.0.1:1401
JASMIN_USERNAME=admin
JASMIN_PASSWORD=admin

# SIP Trunk (Telnyx example)
TELNYX_API_KEY=your_api_key
TELNYX_SIP_USERNAME=your_username
TELNYX_SIP_PASSWORD=your_password

# Payment Gateways
STRIPE_KEY=your_stripe_key
STRIPE_SECRET=your_stripe_secret
```

---

## Part 3: Database Setup

### 3.1 Install MySQL

```bash
# Install MySQL 8.0
sudo apt install -y mysql-server

# Secure MySQL installation
sudo mysql_secure_installation

# Start MySQL service
sudo systemctl start mysql
sudo systemctl enable mysql
```

### 3.2 Create Database and User

```bash
# Login to MySQL
sudo mysql

# Create database and user
CREATE DATABASE voip_platform CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'voip_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON voip_platform.* TO 'voip_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3.3 Run Migrations

```bash
# Run Laravel migrations
cd /home/talha/projects/voIP/backend
php artisan migrate

# Seed database (if seeders exist)
php artisan db:seed
```

---

## Part 4: Redis Setup

### 4.1 Install Redis

```bash
# Install Redis
sudo apt install -y redis-server

# Configure Redis
sudo nano /etc/redis/redis.conf

# Set supervised to systemd
# supervised systemd

# Restart Redis
sudo systemctl restart redis
sudo systemctl enable redis

# Test Redis
redis-cli ping
# Should return: PONG
```

---

## Part 5: VoIP Infrastructure Setup

### 5.1 Install Kamailio (SIP Proxy)

```bash
# Add Kamailio repository
sudo add-apt-repository ppa:kamailio/kamailio-5.7-builds -y
sudo apt update

# Install Kamailio with MySQL module
sudo apt install -y kamailio kamailio-mysql-modules \
    kamailio-tls-modules kamailio-websocket-modules \
    kamailio-json-modules kamailio-utils-modules

# Create Kamailio database
sudo kamdbctl create

# Configure Kamailio
sudo nano /etc/kamailio/kamailio.cfg
```

**Basic Kamailio Configuration:**

```cfg
#!KAMAILIO

####### Global Parameters #########
debug=2
log_stderror=no
log_facility=LOG_LOCAL0
fork=yes
children=4

listen=udp:0.0.0.0:5060
listen=tcp:0.0.0.0:5060
listen=tls:0.0.0.0:5061

####### Modules Section ########
loadmodule "tm.so"
loadmodule "sl.so"
loadmodule "rr.so"
loadmodule "pv.so"
loadmodule "maxfwd.so"
loadmodule "usrloc.so"
loadmodule "registrar.so"
loadmodule "textops.so"
loadmodule "siputils.so"
loadmodule "xlog.so"
loadmodule "sanity.so"
loadmodule "ctl.so"
loadmodule "mi_fifo.so"
loadmodule "db_mysql.so"
loadmodule "auth.so"
loadmodule "auth_db.so"
loadmodule "tls.so"
loadmodule "websocket.so"

####### Routing Logic ########
request_route {
    # Per request initial checks
    route(REQINIT);
    
    # Handle requests within SIP dialogs
    route(WITHINDLG);
    
    # Handle registrations
    route(REGISTRAR);
    
    # Handle calls
    route(INVITE);
}

# Request initial checks
route[REQINIT] {
    if (!mf_process_maxfwd_header("10")) {
        sl_send_reply("483","Too Many Hops");
        exit;
    }
    
    if(!sanity_check("1511", "7")) {
        xlog("Malformed SIP message from $si:$sp\n");
        exit;
    }
}

# Handle within dialog requests
route[WITHINDLG] {
    if (has_totag()) {
        if (loose_route()) {
            route(RELAY);
        } else {
            if (is_method("ACK")) {
                if (t_check_trans()) {
                    t_relay();
                    exit;
                } else {
                    exit;
                }
            }
            sl_send_reply("404","Not here");
        }
        exit;
    }
}

# REGISTER handling
route[REGISTRAR] {
    if (is_method("REGISTER")) {
        if (!save("location")) {
            sl_reply_error();
        }
        exit;
    }
}

# INVITE handling
route[INVITE] {
    if (is_method("INVITE")) {
        # Route to FreeSWITCH
        $du = "sip:127.0.0.1:5080";
        route(RELAY);
        exit;
    }
}

# Relay route
route[RELAY] {
    if (!t_relay()) {
        sl_reply_error();
    }
    exit;
}
```

```bash
# Start Kamailio
sudo systemctl start kamailio
sudo systemctl enable kamailio

# Check status
sudo systemctl status kamailio
```

### 5.2 Install FreeSWITCH (Media Server)

```bash
# Add FreeSWITCH repository
wget -O - https://files.freeswitch.org/repo/deb/debian-release/fsstretch-archive-keyring.asc | sudo apt-key add -
echo "deb https://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" | sudo tee /etc/apt/sources.list.d/freeswitch.list

sudo apt update

# Install FreeSWITCH
sudo apt install -y freeswitch-meta-all

# Start FreeSWITCH
sudo systemctl start freeswitch
sudo systemctl enable freeswitch

# Connect to FreeSWITCH console
sudo fs_cli
```

**Basic FreeSWITCH Configuration:**

```bash
# Edit SIP profile
sudo nano /etc/freeswitch/sip_profiles/internal.xml
```

```xml
<profile name="internal">
  <settings>
    <param name="sip-port" value="5080"/>
    <param name="rtp-ip" value="$${local_ip_v4}"/>
    <param name="sip-ip" value="$${local_ip_v4}"/>
    <param name="ext-rtp-ip" value="$${external_rtp_ip}"/>
    <param name="ext-sip-ip" value="$${external_sip_ip}"/>
    
    <param name="codec-prefs" value="OPUS,PCMU,PCMA"/>
    <param name="inbound-codec-negotiation" value="generous"/>
    
    <param name="apply-inbound-acl" value="domains"/>
    <param name="local-network-acl" value="localnet.auto"/>
    
    <param name="record-path" value="$${recordings_dir}"/>
    <param name="record-template" value="${caller_id_number}.${target_domain}.${strftime(%Y-%m-%d-%H-%M-%S)}.wav"/>
  </settings>
</profile>
```

```bash
# Restart FreeSWITCH
sudo systemctl restart freeswitch
```

### 5.3 Install RTPEngine (Media Proxy)

```bash
# Add RTPEngine repository
sudo add-apt-repository ppa:sipwise/rtpengine -y
sudo apt update

# Install RTPEngine
sudo apt install -y ngcp-rtpengine-daemon ngcp-rtpengine-kernel-dkms \
    ngcp-rtpengine-iptables

# Configure RTPEngine
sudo nano /etc/rtpengine/rtpengine.conf
```

```ini
[rtpengine]
interface = 127.0.0.1
listen-ng = 127.0.0.1:2223
port-min = 30000
port-max = 40000
timeout = 60
silent-timeout = 3600
tos = 184
log-level = 6
log-facility = local1
```

```bash
# Start RTPEngine
sudo systemctl start ngcp-rtpengine-daemon
sudo systemctl enable ngcp-rtpengine-daemon

# Check status
sudo systemctl status ngcp-rtpengine-daemon
```

### 5.4 Install Coturn (STUN/TURN Server)

```bash
# Install Coturn
sudo apt install -y coturn

# Enable Coturn
sudo nano /etc/default/coturn
# Uncomment: TURNSERVER_ENABLED=1

# Configure Coturn
sudo nano /etc/turnserver.conf
```

```conf
listening-port=3478
tls-listening-port=5349

listening-ip=0.0.0.0
relay-ip=YOUR_PUBLIC_IP

external-ip=YOUR_PUBLIC_IP

realm=voip.yourdomain.com
server-name=voip.yourdomain.com

lt-cred-mech
user=turnuser:turnpassword

total-quota=100
stale-nonce=600

cert=/etc/letsencrypt/live/voip.yourdomain.com/cert.pem
pkey=/etc/letsencrypt/live/voip.yourdomain.com/privkey.pem

no-stdout-log
log-file=/var/log/turnserver.log
```

```bash
# Start Coturn
sudo systemctl start coturn
sudo systemctl enable coturn
```

---

## Part 6: SMS Gateway Setup (Jasmin)

### 6.1 Install Jasmin Dependencies

```bash
# Install Python and dependencies
sudo apt install -y python3 python3-pip python3-dev \
    libssl-dev libffi-dev rabbitmq-server

# Install Jasmin
sudo pip3 install jasmin

# Create Jasmin directories
sudo mkdir -p /etc/jasmin/store
sudo mkdir -p /var/log/jasmin
```

### 6.2 Configure Jasmin

```bash
# Initialize Jasmin configuration
sudo jasmin-cfg init

# Edit configuration
sudo nano /etc/jasmin/jasmin.cfg
```

```ini
[sm-listener]
host = 0.0.0.0
port = 1401

[smpp-server]
host = 0.0.0.0
port = 2775

[redis-client]
host = 127.0.0.1
port = 6379

[amqp-broker]
host = 127.0.0.1
port = 5672
vhost = /
username = guest
password = guest
```

### 6.3 Start Jasmin

```bash
# Start Jasmin services
sudo systemctl start jasmin
sudo systemctl enable jasmin

# Check status
sudo systemctl status jasmin

# Access Jasmin CLI
telnet localhost 8990
# Default credentials: jcliadmin / jclipwd
```

---

## Part 7: Docker Compose Setup (Alternative)

For easier development, you can use Docker Compose to run all services:

### 7.1 Create Docker Compose File

```bash
cd /home/talha/projects/voIP
nano docker-compose.yml
```

```yaml
version: '3.8'

services:
  # MySQL Database
  mysql:
    image: mysql:8.0
    container_name: voip_mysql
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: voip_platform
      MYSQL_USER: voip_user
      MYSQL_PASSWORD: secure_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - voip_network

  # Redis
  redis:
    image: redis:7-alpine
    container_name: voip_redis
    ports:
      - "6379:6379"
    networks:
      - voip_network

  # RabbitMQ
  rabbitmq:
    image: rabbitmq:3.12-management
    container_name: voip_rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    networks:
      - voip_network

  # Kamailio (SIP Proxy)
  kamailio:
    image: kamailio/kamailio:5.7-alpine
    container_name: voip_kamailio
    ports:
      - "5060:5060/udp"
      - "5060:5060/tcp"
      - "5061:5061/tcp"
    volumes:
      - ./kamailio/kamailio.cfg:/etc/kamailio/kamailio.cfg
    networks:
      - voip_network
    depends_on:
      - mysql

  # FreeSWITCH (Media Server)
  freeswitch:
    image: signalwire/freeswitch:latest
    container_name: voip_freeswitch
    ports:
      - "5080:5080/udp"
      - "5080:5080/tcp"
      - "8021:8021"
      - "16384-16394:16384-16394/udp"
    volumes:
      - ./freeswitch/conf:/etc/freeswitch
    networks:
      - voip_network

  # Laravel Application
  laravel:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: voip_laravel
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/var/www/html
    environment:
      - DB_HOST=mysql
      - REDIS_HOST=redis
      - QUEUE_CONNECTION=redis
    depends_on:
      - mysql
      - redis
      - rabbitmq
    networks:
      - voip_network
    command: php artisan serve --host=0.0.0.0 --port=8000

  # Laravel Horizon (Queue Worker)
  horizon:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: voip_horizon
    volumes:
      - ./backend:/var/www/html
    depends_on:
      - mysql
      - redis
    networks:
      - voip_network
    command: php artisan horizon

volumes:
  mysql_data:

networks:
  voip_network:
    driver: bridge
```

### 7.2 Create Laravel Dockerfile

```bash
cd /home/talha/projects/voIP/backend
nano Dockerfile
```

```dockerfile
FROM php:8.2-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy application files
COPY . .

# Install dependencies
RUN composer install --no-interaction --optimize-autoloader

# Set permissions
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache

EXPOSE 8000

CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
```

### 7.3 Start Docker Environment

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Rebuild services
docker-compose up -d --build
```

---

## Part 8: Development Tools Setup

### 8.1 Install VS Code Extensions

**Recommended Extensions:**
- PHP Intelephense
- Laravel Extension Pack
- Docker
- GitLens
- ESLint
- Prettier
- REST Client
- Database Client

### 8.2 Install Postman

```bash
# Install Postman for API testing
sudo snap install postman
```

### 8.3 Install SIP Testing Tools

```bash
# Install SIPp (SIP testing tool)
sudo apt install -y sipp

# Install Linphone (SIP client for testing)
sudo apt install -y linphone
```

---

## Part 9: Testing the Setup

### 9.1 Test Laravel Application

```bash
cd /home/talha/projects/voIP/backend

# Run development server
php artisan serve

# In another terminal, test
curl http://localhost:8000
```

### 9.2 Test Database Connection

```bash
# Test database connection
php artisan tinker

# In tinker:
DB::connection()->getPdo();
# Should not throw errors
```

### 9.3 Test Redis Connection

```bash
# Test Redis
php artisan tinker

# In tinker:
Redis::set('test', 'value');
Redis::get('test');
# Should return 'value'
```

### 9.4 Test Kamailio

```bash
# Check if Kamailio is listening
sudo netstat -tulpn | grep kamailio

# Test SIP registration (using SIPp or Linphone)
```

### 9.5 Test FreeSWITCH

```bash
# Connect to FreeSWITCH console
sudo fs_cli

# Check status
fs_cli> status
fs_cli> sofia status
```

### 9.6 Test Jasmin

```bash
# Test Jasmin HTTP API
curl -X POST http://localhost:1401/send \
  -d "username=admin&password=admin&to=+1234567890&from=1234&content=Test"
```

---

## Part 10: Environment Verification Checklist

- [ ] **PHP 8.2+** installed and working
- [ ] **Composer** installed
- [ ] **Node.js & NPM** installed
- [ ] **MySQL 8.0** running and accessible
- [ ] **Redis** running and accessible
- [ ] **Laravel application** running
- [ ] **Kamailio** running on port 5060
- [ ] **FreeSWITCH** running on port 5080
- [ ] **RTPEngine** running
- [ ] **Coturn** running on port 3478
- [ ] **Jasmin** running on port 1401
- [ ] **RabbitMQ** running on port 5672
- [ ] **Docker & Docker Compose** installed (if using)
- [ ] **All services** can communicate with each other
- [ ] **Firewall rules** configured (if applicable)

---

## Troubleshooting

### Common Issues

**Issue: Kamailio won't start**
```bash
# Check logs
sudo tail -f /var/log/syslog | grep kamailio

# Check configuration syntax
sudo kamailio -c
```

**Issue: FreeSWITCH permission errors**
```bash
# Fix permissions
sudo chown -R freeswitch:freeswitch /var/lib/freeswitch
sudo chown -R freeswitch:freeswitch /var/log/freeswitch
```

**Issue: Laravel database connection failed**
```bash
# Verify MySQL is running
sudo systemctl status mysql

# Test connection
mysql -u voip_user -p -h 127.0.0.1 voip_platform
```

**Issue: Redis connection refused**
```bash
# Check Redis status
sudo systemctl status redis

# Test Redis
redis-cli ping
```

---

## Next Steps

1. ✅ Verify all services are running
2. ✅ Run Laravel migrations
3. ✅ Seed test data
4. ✅ Configure SIP trunk provider credentials
5. ✅ Configure SMS provider credentials
6. ✅ Test end-to-end call flow
7. ✅ Test SMS sending/receiving
8. ✅ Set up Git repository
9. ✅ Configure CI/CD pipeline
10. ✅ Document any custom configurations

---

*Document Version: 1.0*  
*Created: February 2026*  
*Status: Planning Document*
