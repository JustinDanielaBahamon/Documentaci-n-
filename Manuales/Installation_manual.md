# Manual de Instalación - AlertaMujer

**Versión:** 1.0  
**Fecha:** 5 de octubre de 2026  
**Audiencia:** Administradores de Sistemas, DevOps, Personal de TI  
**Entorno:** Producción

---

## Tabla de Contenidos

1. [Requisitos del Sistema](#1-requisitos-del-sistema)
2. [Instalación del Backend](#2-instalación-del-backend)
3. [Instalación del Frontend Web](#3-instalación-del-frontend-web)
4. [Configuración de la Base de Datos](#4-configuración-de-la-base-de-datos)
5. [Despliegue en Producción](#5-despliegue-en-producción)
6. [Configuración de SSL/HTTPS](#6-configuración-de-sslhttps)
7. [Troubleshooting](#7-troubleshooting)
8. [Mantenimiento](#8-mantenimiento)

---

## 1. Requisitos del Sistema

### 1.1 Requisitos de Hardware

**Servidor Backend (Mínimo):**
- CPU: 2 núcleos
- RAM: 4 GB
- Disco: 50 GB SSD
- Red: 1 Gbps

**Servidor Backend (Recomendado):**
- CPU: 4 núcleos
- RAM: 8 GB
- Disco: 100 GB SSD
- Red: 1 Gbps

**Servidor Frontend Web (Mínimo):**
- CPU: 1 núcleo
- RAM: 2 GB
- Disco: 20 GB SSD
- Red: 1 Gbps

**Servidor Frontend Web (Recomendado):**
- CPU: 2 núcleos
- RAM: 4 GB
- Disco: 50 GB SSD
- Red: 1 Gbps

### 1.2 Requisitos de Software

**Sistema Operativo:**
- Ubuntu 20.04 LTS o superior (recomendado)
- CentOS 8 o superior
- Debian 11 o superior
- Windows Server 2019 o superior

**Software Requerido:**
- Docker 20.10+
- Docker Compose 2.0+
- Nginx 1.18+ (para frontend web)
- PostgreSQL 16+ (si no se usa Docker)
- Java 17+ (si no se usa Docker)
- Node.js 18+ (para build del frontend)
- Git

**Software Opcional:**
- Certbot (para SSL gratuito con Let's Encrypt)
- PM2 (para gestión de procesos Node.js)
- Fail2ban (para seguridad)
- UFW (firewall)

### 1.3 Requisitos de Red

**Puertos Requeridos:**

| Servicio | Puerto | Protocolo | Descripción |
|---------|--------|-----------|-------------|
| Backend API | 8080 | TCP | API REST |
| Frontend Web | 80 | TCP | HTTP |
| Frontend Web | 443 | TCP | HTTPS |
| PostgreSQL | 5432 | TCP | Base de datos (interno) |
| Docker Registry | 5000 | TCP | (opcional) |

**Requisitos de DNS:**
- Dominio para el frontend web (ej: alertamujer.com)
- Subdominio para la API (ej: api.alertamujer.com)
- Registros A apuntando a la IP del servidor

### 1.4 Requisitos de Seguridad

**Certificados SSL:**
- Certificado SSL válido para el dominio
- Certificado intermedio de la CA
- Clave privada

**Firewall:**
- Solo puertos necesarios abiertos (80, 443)
- Acceso SSH restringido por IP
- Reglas de seguridad configuradas

**Autenticación:**
- Contraseñas fuertes para todos los servicios
- Autenticación de dos factores para acceso SSH (recomendado)
- Llaves SSH para autenticación

---

## 2. Instalación del Backend

### 2.1 Preparación del Servidor

#### Paso 1: Actualizar el Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

#### Paso 2: Instalar Docker y Docker Compose

```bash
# Instalar dependencias
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Agregar repositorio de Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Habilitar Docker al inicio
sudo systemctl enable docker
sudo systemctl start docker

# Verificar instalación
docker --version
docker compose version
```

#### Paso 3: Crear Usuario para Docker (Opcional)

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 2.2 Obtener el Código Fuente

#### Paso 1: Clonar el Repositorio

```bash
cd /opt
sudo git clone <repository-url> alertamujer
cd alertamujer/Backend-ALERTA-MUJER
```

#### Paso 2: Configurar Permisos

```bash
sudo chown -R $USER:$USER /opt/alertamujer
```

### 2.3 Configurar Variables de Entorno

#### Paso 1: Crear Archivo .env

```bash
cd /opt/alertamujer/Backend-ALERTA-MUJER
cp .env.example .env
```

#### Paso 2: Editar .env con Valores de Producción

```env
# Base de Datos
POSTGRES_USER=alerta_mujer_prod
POSTGRES_PASSWORD=<contraseña_fuerte_32_caracteres>
POSTGRES_DB=alerta_mujer_prod

# JWT
JWT_SECRET=<secreto_aleatorio_64_caracteres>
JWT_EXPIRES_IN=24h

# CORS
CORS_ORIGIN=https://alertamujer.com

# Servidor
SERVER_PORT=8080
SPRING_PROFILES_ACTIVE=prod
```

**Generar Contraseña Fuerte:**
```bash
openssl rand -base64 32
```

**Generar Secreto JWT:**
```bash
openssl rand -base64 64
```

### 2.4 Configurar Docker Compose para Producción

#### Paso 1: Editar docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: alerta-mujer-db
    restart: always
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - alertamujer-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend-monolith
    container_name: alerta-mujer-backend
    restart: always
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: ${JWT_EXPIRES_IN}
      CORS_ORIGIN: ${CORS_ORIGIN}
      SPRING_PROFILES_ACTIVE: prod
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - alertamujer-network
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  postgres_data:

networks:
  alertamujer-network:
    driver: bridge
```

### 2.5 Construir e Iniciar el Backend

#### Paso 1: Construir Imágenes Docker

```bash
cd /opt/alertamujer/Backend-ALERTA-MUJER
docker compose build
```

#### Paso 2: Iniciar Servicios

```bash
docker compose up -d
```

#### Paso 3: Verificar Estado

```bash
docker compose ps
docker compose logs backend
```

#### Paso 4: Verificar Health Check

```bash
curl http://localhost:8080/health
```

Respuesta esperada:
```json
{
  "status": "ok",
  "service": "alerta-mujer-backend",
  "type": "monolithic"
}
```

### 2.6 Configurar Firewall

```bash
# Permitir SSH
sudo ufw allow 22/tcp

# Permitir HTTP
sudo ufw allow 80/tcp

# Permitir HTTPS
sudo ufw allow 443/tcp

# Habilitar firewall
sudo ufw enable

# Verificar estado
sudo ufw status
```

---

## 3. Instalación del Frontend Web

### 3.1 Instalar Node.js y npm

```bash
# Instalar Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Verificar instalación
node --version
npm --version
```

### 3.2 Obtener y Construir el Frontend

#### Paso 1: Navegar al Directorio

```bash
cd /opt/alertamujer/Frond-end-Web/alerta-mujer-web
```

#### Paso 2: Instalar Dependencias

```bash
npm install --production
```

#### Paso 3:	Configurar Variables de Entorno

```bash
cp .env.example .env
```

Editar `.env`:
```env
NG_APP_API_URL=https://api.alertamujer.com
```

#### Paso 4: Construir para Producción

```bash
npm run build
```

El resultado estará en `dist/alerta-mujer-web/`

### 3.3 Instalar y Configurar Nginx

#### Paso 1: Instalar Nginx

```bash
sudo apt install -y nginx
```

#### Paso 2: Configurar Nginx para el Frontend

Crear archivo de configuración:
```bash
sudo nano /etc/nginx/sites-available/alertamujer
```

Contenido:
```nginx
server {
    listen 80;
    server_name alertamujer.com www.alertamujer.com;

    root /opt/alertamujer/Frond-end-Web/alerta-mujer-web/dist/alerta-mujer-web;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml+rss application/javascript application/json;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Angular routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
}
```

#### Paso 3: Habilitar Sitio

```bash
sudo ln -s /etc/nginx/sites-available/alertamujer /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```

#### Paso 4: Verificar Configuración

```bash
sudo nginx -t
```

#### Paso 5: Reiniciar Nginx

```bash
sudo systemctl restart nginx
sudo systemctl enable nginx
```

### 3.4 Configurar Proxy para API

Editar configuración de Nginx para incluir proxy a la API:

```nginx
server {
    listen 80;
    server_name api.alertamujer.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

---

## 4. Configuración de la Base de Datos

### 4.1 Configuración Inicial con Docker

La base de datos se inicializa automáticamente con el script `db/init.sql` al iniciar el contenedor PostgreSQL por primera vez.

### 4.2 Configuración Manual (Sin Docker)

#### Paso 1: Instalar PostgreSQL

```bash
sudo apt install -y postgresql-16 postgresql-contrib
```

#### Paso 2: Iniciar PostgreSQL

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

#### Paso 3: Crear Base de Datos y Usuario

```bash
sudo -u postgres psql
```

```sql
-- Crear usuario
CREATE USER alerta_mujer_prod WITH PASSWORD 'contraseña_fuerte';

-- Crear base de datos
CREATE DATABASE alerta_mujer_prod OWNER alerta_mujer_prod;

-- Conceder privilegios
GRANT ALL PRIVILEGES ON DATABASE alerta_mujer_prod TO alerta_mujer_prod;

-- Salir
\q
```

#### Paso 4: Ejecutar Script de Inicialización

```bash
sudo -u alerta_mujer_prod psql -d alerta_mujer_prod -f /opt/alertamujer/Backend-ALERTA-MUJER/db/init.sql
```

### 4.3 Configurar Replicación (Opcional, para Alta Disponibilidad)

#### Configuración de Master

Editar `postgresql.conf`:
```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

Agregar:
```conf
wal_level = replica
max_wal_senders = 3
wal_keep_size = 100MB
```

Editar `pg_hba.conf`:
```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

Agregar:
```conf
host    replication     replicator      <slave-ip>/32            scram-sha-256
```

### 4.4 Configurar Backups Automáticos

#### Paso 1: Crear Script de Backup

```bash
sudo nano /usr/local/bin/backup-alertamujer.sh
```

Contenido:
```bash
#!/bin/bash
BACKUP_DIR="/var/backups/alertamujer"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/alertamujer_$DATE.sql"

mkdir -p $BACKUP_DIR

docker exec alerta-mujer-db pg_dump -U alerta_mujer_prod alerta_mujer_prod > $BACKUP_FILE

# Comprimir backup
gzip $BACKUP_FILE

# Eliminar backups antiguos (mantener últimos 7 días)
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete
```

#### Paso 2: Dar Permisos

```bash
sudo chmod +x /usr/local/bin/backup-alertamujer.sh
```

#### Paso 3: Configurar Cron

```bash
sudo crontab -e
```

Agregar:
```cron
0 2 * * * /usr/local/bin/backup-alertamujer.sh
```

Esto ejecutará el backup diariamente a las 2 AM.

### 4.5 Configurar Monitoreo de Base de Datos

#### Instalar pgAdmin (Opcional)

```bash
sudo apt install -y pgadmin4
```

#### Configurar Alertas

Monitorear:
- Uso de CPU
- Uso de memoria
- Espacio en disco
- Conexiones activas
- Tiempo de respuesta de queries

---

## 5. Despliegue en Producción

### 5.1 Verificación Pre-Despliegue

#### Checklist

- [ ] Todos los requisitos de hardware cumplidos
- [ ] Docker y Docker Compose instalados
- [ ] Variables de entorno configuradas
- [ ] Firewall configurado
- [ ] DNS configurado correctamente
- [ ] Certificados SSL listos
- [ ] Backups configurados
- [ ] Monitoreo configurado

### 5.2 Despliegue Inicial

#### Paso 1: Iniciar Servicios

```bash
cd /opt/alertamujer/Backend-ALERTA-MUJER
docker compose up -d
```

#### Paso 2: Verificar Servicios

```bash
# Verificar contenedores
docker compose ps

# Verificar logs
docker compose logs -f

# Verificar health check
curl http://localhost:8080/health
```

#### Paso 3: Verificar Frontend

```bash
# Verificar Nginx
sudo systemctl status nginx

# Verificar acceso
curl http://localhost
```

### 5.3 Configurar SSL con Let's Encrypt

#### Paso 1: Instalar Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

#### Paso 2: Obtener Certificado

```bash
sudo certbot --nginx -d alertamujer.com -d www.alertamujer.com -d api.alertamujer.com
```

Seguir las instrucciones en pantalla.

#### Paso 3: Configurar Renovación Automática

```bash
sudo certbot renew --dry-run
```

Certbot configura automáticamente la renovación.

### 5.4 Configurar Balanceo de Carga (Opcional)

#### Usar Nginx como Load Balancer

```nginx
upstream backend {
    server backend1:8080;
    server backend2:8080;
    server backend3:8080;
}

server {
    listen 443 ssl;
    server_name api.alertamujer.com;

    ssl_certificate /etc/letsencrypt/live/api.alertamujer.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.alertamujer.com/privkey.pem;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 5.5 Configurar Escalado Horizontal

#### Usar Docker Swarm

```bash
# Inicializar Swarm
docker swarm init

# Desplegar stack
docker stack deploy -c docker-compose.yml alertamujer

# Escalar servicio
docker service scale alertamujer_backend=3
```

---

## 6. Configuración de SSL/HTTPS

### 6.1 Configuración Manual de SSL

Si no se usa Let's Encrypt, configurar SSL manualmente:

#### Paso 1: Colocar Certificados

```bash
sudo mkdir -p /etc/nginx/ssl
sudo cp your-cert.crt /etc/nginx/ssl/
sudo cp your-key.key /etc/nginx/ssl/
sudo cp your-ca-bundle.crt /etc/nginx/ssl/
```

#### Paso 2: Configurar Nginx

```nginx
server {
    listen 443 ssl http2;
    server_name alertamujer.com www.alertamujer.com;

    ssl_certificate /etc/nginx/ssl/your-cert.crt;
    ssl_certificate_key /etc/nginx/ssl/your-key.key;
    ssl_trusted_certificate /etc/nginx/ssl/your-ca-bundle.crt;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Resto de configuración...
}

server {
    listen 80;
    server_name alertamujer.com www.alertamujer.com;
    return 301 https://$server_name$request_uri;
}
```

#### Paso 3: Verificar Configuración

```bash
sudo nginx -t
sudo systemctl restart nginx
```

### 6.2 Verificar SSL

```bash
openssl s_client -connect alertamujer.com:443 -servername alertamujer.com
```

### 6.3 Test de Seguridad SSL

Usar herramientas como:
- SSL Labs Test: https://www.ssllabs.com/ssltest/
- Qualys SSL Test

---

## 7. Troubleshooting

### 7.1 Problemas Comunes del Backend

#### El Backend No Inicia

**Síntomas:**
- Contenedor no arranca
- Error en logs

**Soluciones:**

1. Verificar logs:
```bash
docker compose logs backend
```

2. Verificar que PostgreSQL esté listo:
```bash
docker compose logs postgres
```

3. Verificar variables de entorno:
```bash
docker compose config
```

4. Verificar puertos disponibles:
```bash
sudo netstat -tulpn | grep 8080
```

5. Reconstruir imagen:
```bash
docker compose down
docker compose build --no-cache
docker compose up -d
```

#### Error de Conexión a Base de Datos

**Síntomas:**
- Error "Connection refused"
- Error "Connection timeout"

**Soluciones:**

1. Verificar que PostgreSQL esté corriendo:
```bash
docker compose ps postgres
```

2. Verificar credenciales en `.env`:
```bash
cat .env | grep POSTGRES
```

3. Verificar conectividad:
```bash
docker exec -it alerta-mujer-backend ping postgres
```

4. Verificar health check de PostgreSQL:
```bash
docker exec alerta-mujer-db pg_isready -U alerta_mujer_prod
```

#### Error de JWT

**Síntomas:**
- Error "Invalid token"
- Error "Token expired"

**Soluciones:**

1. Verificar que `JWT_SECRET` esté configurado:
```bash
cat .env | grep JWT_SECRET
```

2. Verificar que el secreto sea el mismo en todos los entornos

3. Regenerar token si es necesario:
```bash
# Generar nuevo secreto
openssl rand -base64 64
```

#### Error de CORS

**Síntomas:**
- Error "CORS policy" en navegador
- Bloqueo de requests

**Soluciones:**

1. Verificar configuración de CORS en `.env`:
```bash
cat .env | grep CORS_ORIGIN
```

2. Verificar que el origen esté incluido en la lista permitida

3. Para desarrollo, usar `*` (no recomendado en producción)

### 7.2 Problemas Comunes del Frontend

#### Nginx No Sirve el Sitio

**Síntomas:**
- Error 502 Bad Gateway
- Página en blanco

**Soluciones:**

1. Verificar que Nginx esté corriendo:
```bash
sudo systemctl status nginx
```

2. Verificar configuración:
```bash
sudo nginx -t
```

3. Verificar que el frontend esté construido:
```bash
ls -la /opt/alertamujer/Frond-end-Web/alerta-mujer-web/dist/alerta-mujer-web/
```

4. Verificar permisos:
```bash
sudo chown -R www-data:www-data /opt/alertamujer/Frond-end-Web/alerta-mujer-web/dist/
```

5. Reiniciar Nginx:
```bash
sudo systemctl restart nginx
```

#### Error de Proxy a API

**Síntomas:**
- Error 502 al llamar API
- Timeout

**Soluciones:**

1. Verificar que el backend esté corriendo:
```bash
curl http://localhost:8080/health
```

2. Verificar configuración de proxy en Nginx:
```bash
sudo cat /etc/nginx/sites-available/alertamujer
```

3. Verificar que el puerto 8080 esté accesible:
```bash
sudo netstat -tulpn | grep 8080
```

#### Build Falla

**Síntomas:**
- Error durante `npm run build`
- Error de memoria

**Soluciones:**

1. Aumentar memoria de Node:
```bash
export NODE_OPTIONS="--max-old-space-size=4096"
npm run build
```

2. Limpiar cache:
```bash
rm -rf node_modules package-lock.json
npm install
npm run build
```

3. Verificar versión de Node:
```bash
node --version
npm --version
```

### 7.3 Problemas de Base de Datos

#### PostgreSQL No Inicia

**Síntomas:**
- Contenedor no arranca
- Error de conexión

**Soluciones:**

1. Verificar logs:
```bash
docker compose logs postgres
```

2. Verificar espacio en disco:
```bash
df -h
```

3. Verificar permisos de volumen:
```bash
docker volume inspect alertamujer_postgres_data
```

4. Reiniciar contenedor:
```bash
docker compose restart postgres
```

#### Error de Permiso en Base de Datos

**Síntomas:**
- Error "permission denied"
- No se pueden crear tablas

**Soluciones:**

1. Conectar a la base de datos:
```bash
docker exec -it alerta-mujer-db psql -U alerta_mujer_prod -d alerta_mujer_prod
```

2. Verificar privilegios:
```sql
\du
```

3. Conceder privilegios:
```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO alerta_mujer_prod;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO alerta_mujer_prod;
```

#### Base de Datos Lenta

**Síntomas:**
- Queries lentos
- Timeout en requests

**Soluciones:**

1. Verificar uso de recursos:
```bash
docker stats alerta-mujer-db
```

2. Analizar queries lentos:
```bash
docker exec -it alerta-mujer-db psql -U alerta_mujer_prod -d alerta_mujer_prod
```

```sql
SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
```

3. Crear índices faltantes:
```sql
-- Ejemplo
CREATE INDEX idx_users_email ON identity.users(email);
```

4. Aumentar recursos del contenedor:
```yaml
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
```

### 7.4 Problemas de SSL

#### Certificado Expirado

**Síntomas:**
- Error de seguridad en navegador
- Certificado no válido

**Soluciones:**

1. Renovar certificado:
```bash
sudo certbot renew
```

2. Verificar fecha de expiración:
```bash
sudo certbot certificates
```

#### Error de Configuración SSL

**Síntomas:**
- Error 502 después de configurar SSL
- Certificado no cargado

**Soluciones:**

1. Verificar que los archivos existan:
```bash
ls -la /etc/nginx/ssl/
```

2. Verificar permisos:
```bash
sudo chmod 600 /etc/nginx/ssl/your-key.key
```

3. Verificar configuración:
```bash
sudo nginx -t
```

4. Reiniciar Nginx:
```bash
sudo systemctl restart nginx
```

### 7.5 Problemas de Red

#### Firewall Bloqueando Conexiones

**Síntomas:**
- No se puede acceder desde fuera
- Timeout

**Soluciones:**

1. Verificar estado de firewall:
```bash
sudo ufw status
```

2. Abrir puertos necesarios:
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

3. Verificar reglas:
```bash
sudo ufw show added
```

#### DNS No Resuelve

**Síntomas:**
- Dominio no accesible
- Error NXDOMAIN

**Soluciones:**

1. Verificar configuración DNS:
```bash
nslookup alertamujer.com
```

2. Verificar registros DNS en el proveedor

3. Verificar propagación:
```bash
dig alertamujer.com
```

4. Configurar DNS local:
```bash
sudo nano /etc/resolv.conf
```

### 7.6 Problemas de Docker

#### Contenedor No Arranca

**Síntomas:**
- Status "exited"
- Error en logs

**Soluciones:**

1. Verificar logs:
```bash
docker logs <container-name>
```

2. Verificar recursos:
```bash
docker system df
```

3. Limpiar recursos no usados:
```bash
docker system prune -a
```

4. Reconstruir imagen:
```bash
docker compose down
docker compose build --no-cache
docker compose up -d
```

#### Error de Espacio en Disco

**Síntomas:**
- "No space left on device"
- Contenedores no arrancan

**Soluciones:**

1. Verificar espacio:
```bash
df -h
```

2. Limpiar imágenes no usadas:
```bash
docker image prune -a
```

3. Limpiar volúmenes no usados:
```bash
docker volume prune
```

4. Limpiar build cache:
```bash
docker builder prune
```

---

## 8. Mantenimiento

### 8.1 Actualizaciones

#### Actualizar Backend

```bash
cd /opt/alertamujer/Backend-ALERTA-MUJER
git pull
docker compose down
docker compose build
docker compose up -d
```

#### Actualizar Frontend

```bash
cd /opt/alertamujer/Frond-end-Web/alerta-mujer-web
git pull
npm install
npm run build
sudo systemctl reload nginx
```

### 8.2 Monitoreo

#### Monitorear Contenedores

```bash
docker stats
```

#### Monitorear Logs

```bash
# Backend
docker compose logs -f backend

# PostgreSQL
docker compose logs -f postgres

# Nginx
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

#### Monitorear Recursos del Sistema

```bash
# CPU y Memoria
htop

# Disco
df -h

# Red
iftop
```

### 8.3 Backups

#### Backup Manual

```bash
# Backup de base de datos
docker exec alerta-mujer-db pg_dump -U alerta_mujer_prod alerta_mujer_prod > backup.sql

# Backup de volúmenes
docker run --rm -v alertamujer_postgres_data:/data -v $(pwd):/backup ubuntu tar czf /backup/postgres_backup.tar.gz /data
```

#### Restaurar Backup

```bash
# Restaurar base de datos
cat backup.sql | docker exec -i alerta-mujer-db psql -U alerta_mujer_prod alerta_mujer_prod
```

### 8.4 Rotación de Logs

#### Configurar Logrotate

Crear archivo:
```bash
sudo nano /etc/logrotate.d/alertamujer
```

Contenido:
```
/var/log/alertamujer/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data www-data
}
```

### 8.5 Seguridad

#### Actualizar Sistema Regularmente

```bash
sudo apt update
sudo apt upgrade -y
```

#### Escanear Vulnerabilidades

```bash
# Escanear imágenes Docker
docker scan alerta-mujer-backend

# Escanear sistema
sudo apt install lynis
sudo lynis audit system
```

#### Configurar Fail2ban

```bash
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 8.6 Escalado

#### Escalar Backend

```bash
docker compose up -d --scale backend=3
```

#### Configurar Auto-scaling (con Kubernetes)

Crear archivo `deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alertamujer-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: alertamujer-backend
  template:
    metadata:
      labels:
        app: alertamujer-backend
    spec:
      containers:
      - name: backend
        image: alertamujer-backend:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

---

## 9. Análisis Multiplataforma de Despliegue

Esta sección proporciona un análisis exhaustivo para desplegar AlertaMujer en Linux, Windows y macOS, garantizando compatibilidad total en entornos de desarrollo y producción.

### 9.1 Matriz de Compatibilidad de Entornos

#### Tabla Comparativa de Dependencias

| Componente | Linux (Ubuntu/Debian) | Windows (Server/10/11) | macOS (Intel/Apple Silicon) |
|------------|----------------------|------------------------|----------------------------|
| **Gestor de Paquetes** | `apt` / `apt-get` | `winget` / `choco` | `brew` (Homebrew) |
| **Docker** | `apt install docker.io` | `winget install Docker.DockerDesktop` | `brew install --cask docker` |
| **Docker Compose** | `apt install docker-compose-plugin` | Incluido en Docker Desktop | Incluido en Docker Desktop |
| **Java 17+** | `apt install openjdk-17-jdk` | `winget install Oracle.JDK.17` | `brew install openjdk@17` |
| **Node.js 18+** | `apt install nodejs npm` | `winget install OpenJS.NodeJS` | `brew install node` |
| **PostgreSQL 16** | `apt install postgresql-16` | `winget install PostgreSQL.PostgreSQL` | `brew install postgresql@16` |
| **Nginx** | `apt install nginx` | `winget install nginx.nginx` | `brew install nginx` |
| **Git** | `apt install git` | `winget install Git.Git` | `brew install git` |
| **Curl** | `apt install curl` | Incluido en Windows 10+ | Incluido en macOS |
| **OpenSSL** | `apt install openssl` | Incluido en Git for Windows | Incluido en macOS |
| **Editor de Texto** | `nano` / `vim` | `notepad` / `code` (VS Code) | `nano` / `vim` / `code` |

#### Requisitos Mínimos por Sistema Operativo

**Linux (Ubuntu Server 20.04+ / Debian 11+)**
- Kernel: 5.4+
- Sistema de archivos: ext4, xfs, btrfs
- Arquitectura: amd64, arm64
- Docker Engine: 20.10+
- Compose: 2.0+

**Windows Server 2019+ / Windows 10/11**
- Sistema: Windows Server 2019/2022 o Windows 10/11 Pro/Enterprise
- Virtualización: Hyper-V habilitado (para Docker Desktop)
- WSL2: Windows Subsystem for Linux 2 (recomendado para Docker)
- Arquitectura: amd64
- Docker Desktop: 4.15+

**macOS 11+ (Big Sur)**
- Sistema: macOS 11+ (Big Sur, Monterey, Ventura, Sonoma)
- Arquitectura: Intel (x86_64) o Apple Silicon (arm64)
- Docker Desktop: 4.15+
- Homebrew: 4.0+

### 9.2 Estrategia de Contenerización Homogénea (Docker)

#### docker-compose.yml Optimizado Multiplataforma

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: alerta-mujer-db
    restart: always
    platform: linux/amd64
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_INITDB_ARGS: "-E UTF8 --locale=C"
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - alertamujer-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    extra_hosts:
      - "host.docker.internal:host-gateway"

  backend:
    build:
      context: ./backend-monolith
      dockerfile: Dockerfile
      args:
        BUILDKIT: 1
    container_name: alerta-mujer-backend
    restart: always
    platform: linux/amd64
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: ${JWT_EXPIRES_IN}
      CORS_ORIGIN: ${CORS_ORIGIN}
      SPRING_PROFILES_ACTIVE: prod
      TZ: ${TZ:-UTC}
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - alertamujer-network
    volumes:
      - backend_logs:/app/logs
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  postgres_data:
    driver: local
  backend_logs:
    driver: local

networks:
  alertamujer-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

#### Gestión de Volúmenes y Permisos por Sistema Operativo

**Linux (ext4/xfs)**
```bash
sudo mkdir -p /opt/alertamujer/data/postgres
sudo chown -R 1000:1000 /opt/alertamujer/data/postgres
sudo chmod -R 755 /opt/alertamujer/data/postgres
docker volume create alertamujer_postgres_data
```

**Windows (NTFS)**
```powershell
New-Item -ItemType Directory -Path "C:\alertamujer\data\postgres" -Force
# Configurar Docker Desktop para usar WSL2 backend
docker volume create alertamujer_postgres_data
```

**macOS (APFS)**
```bash
mkdir -p ~/alertamujer/data/postgres
# Configurar Docker Desktop para usar Linux containers
docker volume create alertamujer_postgres_data
```

#### Manejo de Conectividad de Red

**Linux:** Usar bridge network (default). Acceder desde host: `localhost:puerto`

**Windows:** Usar `host.docker.internal`. Configurar en `docker-compose.yml`:
```yaml
extra_hosts:
  - "host.docker.internal:host-gateway"
```

**macOS:** Similar a Windows, usar `host.docker.internal`

### 9.3 Servidor Web y Proxy Inverso Multiplataforma

#### Configuración de Nginx por Sistema Operativo

**Linux (/etc/nginx/)**
```bash
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/alertamujer
sudo ln -s /etc/nginx/sites-available/alertamujer /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

**Windows (C:\nginx\)**
```powershell
winget install nginx.nginx
notepad C:\nginx\conf\nginx.conf
cd C:\nginx
start nginx
```

**macOS (/usr/local/etc/nginx/)**
```bash
brew install nginx
nano /usr/local/etc/nginx/servers/alertamujer.conf
brew services start nginx
```

#### Alternativa: Caddy Server (HTTPS Automático)

**Instalación:**
- Linux: `sudo apt install caddy`
- Windows: `winget install caddy`
- macOS: `brew install caddy`

**Caddyfile:**
```
alertamujer.com {
    root * /path/to/frontend/dist
    file_server
    try_files {path} /index.html
    encode gzip
}

api.alertamujer.com {
    reverse_proxy localhost:8080
}
```

#### Generación de Certificados SSL

**Linux (Certbot):**
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d alertamujer.com -d www.alertamujer.com -d api.alertamujer.com
```

**Windows (Win-ACME):**
Descargar desde https://www.win-acme.com/ y ejecutar `wacs.exe`

**macOS (Certbot):**
```bash
brew install certbot
sudo certbot certonly --nginx -d alertamujer.com
```

### 9.4 Automatización de Scripts (Backups y Mantenimiento)

#### Script de Backup - Shell Script (.sh) para Linux/macOS

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/alertamujer"
DATE=$(date +%Y%m%d_%H%M%S)
POSTGRES_USER="${POSTGRES_USER:-alerta_mujer_prod}"
POSTGRES_DB="${POSTGRES_DB:-alerta_mujer_prod}"
CONTAINER_NAME="${CONTAINER_NAME:-alerta-mujer-db}"

mkdir -p "$BACKUP_DIR"
docker exec "$CONTAINER_NAME" pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB" > "$BACKUP_DIR/alertamujer_$DATE.sql"
gzip "$BACKUP_DIR/alertamujer_$DATE.sql"
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete
```

#### Script de Backup - PowerShell (.ps1) para Windows

```powershell
$BackupDir = "C:\Backups\AlertaMujer"
$Date = Get-Date -Format "yyyyMMdd_HHmmss"
$PostgresUser = $env:POSTGRES_USER
$PostgresDb = $env:POSTGRES_DB
$ContainerName = $env:CONTAINER_NAME

if (-not $PostgresUser) { $PostgresUser = "alerta_mujer_prod" }
if (-not $PostgresDb) { $PostgresDb = "alerta_mujer_prod" }
if (-not $ContainerName) { $ContainerName = "alerta-mujer-db" }

New-Item -ItemType Directory -Path $BackupDir -Force
docker exec $ContainerName pg_dump -U $PostgresUser $PostgresDb | Out-File -FilePath "$BackupDir\alertamujer_$Date.sql" -Encoding UTF8
Get-ChildItem $BackupDir -Filter "*.sql" | Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-7) } | Remove-Item -Force
```

### 9.5 Guía Paso a Paso de Despliegue Comparativo

#### Bloque A: Linux (Ubuntu Server / Debian)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git docker.io docker-compose-plugin nodejs nginx
sudo systemctl enable docker nginx
cd /opt && sudo git clone <repo> alertamujer
cd alertamujer/Backend-ALERTA-MUJER && docker compose up -d --build
cd ../Frond-end-Web/alerta-mujer-web && npm install && npm run build
sudo nano /etc/nginx/sites-available/alertamujer
sudo ufw allow 80,443/tcp && sudo ufw enable
```

#### Bloque B: Windows (PowerShell)

```powershell
winget install Docker.DockerDesktop OpenJS.NodeJS.LTS Git.Git nginx.nginx
git clone <repo> C:\alertamujer
cd C:\alertamujer\Backend-ALERTA-MUJER
docker compose up -d --build
cd ..\Frond-end-Web\alerta-mujer-web
npm install && npm run build
New-NetFirewallRule -DisplayName "AlertaMujer HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
New-NetFirewallRule -DisplayName "AlertaMujer HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
```

#### Bloque C: macOS

```bash
brew install docker node nginx git
git clone <repo> ~/alertamujer
cd ~/alertamujer/Backend-ALERTA-MUJER && docker compose up -d --build
cd ../Frond-end-Web/alerta-mujer-web && npm install && npm run build
nano /usr/local/etc/nginx/servers/alertamujer.conf
brew services restart nginx
```

### 9.6 Checklist de Seguridad y Firewall por Sistema Operativo

#### Linux (UFW)
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22,80,443/tcp
sudo ufw enable
```

#### Windows (PowerShell)
```powershell
New-NetFirewallRule -DisplayName "AlertaMujer HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
New-NetFirewallRule -DisplayName "AlertaMujer HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
```

#### macOS (pf)
```bash
sudo nano /etc/pf.conf
# Agregar: pass in proto tcp from any to any port 80
# Agregar: pass in proto tcp from any to any port 443
sudo pfctl -f /etc/pf.conf && sudo pfctl -e
```

### 9.7 Análisis de Riesgos y Troubleshooting Multiplataforma

#### Top 5 Errores Comunes y Soluciones

**Error 1: Saltos de Línea (CRLF vs LF)**

**Síntoma:** Scripts de shell fallan en Windows, errores de sintaxis en archivos de configuración.

**Causa:** Windows usa CRLF (`\r\n`) mientras que Linux/macOS usan LF (`\n`).

**Solución:**
- En VS Code: Configurar `endOfLine: lf` en settings.json
- En Git: `git config --global core.autocrlf input` (Linux/macOS) o `true` (Windows)
- Convertir archivos: `dos2unix archivo.sh` (Linux) o usar editor con conversión

**Error 2: Rutas Absolutas vs Relativas**

**Síntoma:** Archivos no encontrados, volúmenes Docker no montan.

**Causa:** Diferencias en separadores de ruta (`/` vs `\`) y estructura de directorios.

**Solución:**
- Usar siempre rutas relativas en docker-compose.yml
- Para scripts, usar variables de entorno:
  ```bash
  # Linux/macOS
  SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
  ```
  ```powershell
  # Windows
  $ScriptDir = Split-Path -Parent $MyInvocation.MyCommand.Path
  ```
- En docker-compose, usar `${PWD}` o `.` para directorio actual

**Error 3: Permisos de Puertos en IIS/Nginx**

**Síntoma:** Error "Address already in use", puertos bloqueados.

**Causa:** Otro servicio usando el puerto (IIS en Windows, Apache en Linux).

**Solución:**
- Linux: `sudo netstat -tulpn | grep :80` para identificar proceso
- Windows: `netstat -ano | findstr :80` y `taskkill /PID <pid> /F`
- Detener servicio conflictivo o cambiar puerto en configuración

**Error 4: Virtualización Hyper-V/WSL2**

**Síntoma:** Docker Desktop no inicia en Windows, error de virtualización.

**Causa:** Hyper-V no habilitado, WSL2 no instalado, virtualización deshabilitada en BIOS.

**Solución:**
- Habilitar Hyper-V: `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All`
- Habilitar WSL2: `wsl --install`
- Habilitar virtualización en BIOS (VT-x/AMD-V)
- Reiniciar sistema

**Error 5: Permisos de Sistema de Archivos en Volúmenes Docker**

**Síntoma:** Contenedor no puede escribir en volumen, errores de permiso denegado.

**Causa:** Diferencias entre sistemas de archivos (ext4 vs NTFS vs APFS) y UID/GID.

**Solución:**
- Usar named volumes en lugar de bind mounts
- Configurar UID/GID en docker-compose:
  ```yaml
  user: "1000:1000"
  ```
- En Windows, usar WSL2 backend para mejor compatibilidad
- En macOS, configurar Docker Desktop para compartir directorios correctamente

---

## 10. Apéndice

### 9.1 Comandos Útiles

#### Docker

```bash
# Ver contenedores corriendo
docker ps

# Ver todos los contenedores
docker ps -a

# Ver logs
docker logs <container>

# Entrar en contenedor
docker exec -it <container> bash

# Detener contenedor
docker stop <container>

# Iniciar contenedor
docker start <container>

# Eliminar contenedor
docker rm <container>

# Ver imágenes
docker images

# Eliminar imágenes no usadas
docker image prune -a
```

#### Nginx

```bash
# Verificar configuración
sudo nginx -t

# Recargar configuración
sudo nginx -s reload

# Reiniciar
sudo systemctl restart nginx

# Ver estado
sudo systemctl status nginx

# Ver logs de acceso
sudo tail -f /var/log/nginx/access.log

# Ver logs de error
sudo tail -f /var/log/nginx/error.log
```

#### PostgreSQL

```bash
# Conectar a base de datos
docker exec -it alerta-mujer-db psql -U alerta_mujer_prod -d alerta_mujer_prod

# Backup
docker exec alerta-mujer-db pg_dump -U alerta_mujer_prod alerta_mujer_prod > backup.sql

# Restaurar
cat backup.sql | docker exec -i alerta-mujer-db psql -U alerta_mujer_prod alerta_mujer_prod

# Ver tamaño de base de datos
docker exec -it alerta-mujer-db psql -U alerta_mujer_prod -d alerta_mujer_prod -c "SELECT pg_size_pretty(pg_database_size('alerta_mujer_prod'));"
```

### 9.2 Contacto de Soporte

**Soporte Técnico:**
- Email: soporte@alertamujer.com
- Teléfono: +57 1 123 4567
- Horario: Lunes a viernes, 8:00 AM - 6:00 PM

**Emergencias:**
- Línea 24/7: +57 1 800 123 4567

### 9.3 Recursos Adicionales

- [Documentación de Docker](https://docs.docker.com)
- [Documentación de Nginx](https://nginx.org/en/docs/)
- [Documentación de PostgreSQL](https://www.postgresql.org/docs)
- [Documentación de Let's Encrypt](https://letsencrypt.org/docs)

---

**Fin del Manual de Instalación**

*Este manual está diseñado para administradores de sistemas y personal de TI responsables del despliegue en producción de AlertaMujer.*
