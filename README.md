# WordPress Docker Setup with Nginx Reverse Proxy

This project provides a containerized WordPress environment using Docker Compose, optimized for deployment behind an Nginx reverse proxy.

## Architecture

- **WordPress**: PHP-FPM based WordPress image.
- **MySQL**: Database for WordPress.
- **phpMyAdmin**: Web interface for database management.
- **Nginx (Host)**: Acts as a reverse proxy to handle SSL and port 80/443 traffic.

## Prerequisites

- Docker and Docker Compose installed on your VM.
- Nginx installed on the host machine.

## Setup Instructions

### 1. Configure Environment Variables

Create a `.env` file in the root directory (or ensure they are set in your shell):

```env
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=wp_password
```

### 2. Start the Docker Containers

Run the following command to start the services in background mode:

```bash
docker-compose up -d
```

The services will be available at:

- **WordPress**: `http://localhost:8081`
- **phpMyAdmin**: `http://localhost:8080`

## Server Setup (Nginx Reverse Proxy)

Since port 80 is likely occupied by the host's Nginx, we use a reverse proxy configuration to forward traffic to the WordPress container.

### 1. Update Nginx Configuration

Edit your Nginx site configuration (e.g., `/etc/nginx/sites-available/default`):

```nginx
server {
    listen 80;
    server_name yourdomain.com; # Replace with your domain or IP

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Increase max upload size for media uploads
        client_max_body_size 64M;
    }
}
```

### 2. Apply Nginx Changes

Test the configuration and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Port Conventions Used

- `80`: Public HTTP traffic (handled by Host Nginx).
- `8081`: Internal WordPress traffic (Proxied).
- `8080`: Internal phpMyAdmin traffic.
- `3306`: Internal MySQL traffic (Not exposed to host).
