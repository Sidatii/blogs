---
title: DevOps for devs - From Localhost to Production - P2.
category: DevOps
tags: Deploy, develop, VPS, HomeLab, Production.
excerpt: This blog is the first part of a DevOps for devs series. In this blog, I am taking you through an enjoyable journey of discovering the world of DevOps, CI/CD and deploying to production.
published_at: 2026-02-17
---

![Thumbnail](images/thumbnail-part-02.png)

***Image by Author.***

# From Localhost to Production - Part 2: Infrastructure, Security & Production Readiness

**Welcome back!**

In Part 1, we:
- ✅ Set up our VPS with proper security
- ✅ Understood Docker and containerization
- ✅ Dockerized our Java backend and Angular frontend
- ✅ Created Docker networks and volumes for data persistence

Now our apps are containerized and running. But they're still accessed via `http://your-ip:8080` and `http://your-ip:4200`.

**Not very professional, right?**

In Part 2, we're going to transform this into a production-ready deployment with:
- A single entry point through Nginx
- A real domain name (myapp.com)
- HTTPS encryption
- Security layers to protect against attacks
- Proper user management

Let's dive in!

## 5. Web Servers (Nginx as Reverse Proxy)

### 5.1 What's a Reverse Proxy?

Imagine a restaurant with one entrance. The host (Nginx) greets customers and directs them to different sections:

- "API requests? Go to the kitchen (backend)"
- "Website requests? Go to the dining room (frontend)"

That's a **reverse proxy**. It sits in front of your apps and routes traffic.

### 5.2 Why Use Nginx?

1. **Single entry point**: One port 80/443 instead of 8080, 4200, etc.
2. **SSL/TLS**: Handles HTTPS certificates
3. **Load balancing**: Distribute traffic across multiple backend instances
4. **Static file serving**: Fast delivery of images, CSS, JS
5. **Security**: Hides your internal architecture

### 5.3 Setting Up Nginx as Reverse Proxy

**Create a new directory for Nginx configuration:**

```bash
mkdir -p ~/nginx-proxy
cd ~/nginx-proxy
```

**Create `nginx.conf`:**

```nginx
events {
    worker_connections 1024;
}

http {
    # Basic settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 20M;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss;

    # Rate limiting to prevent abuse
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=general_limit:10m rate=30r/s;

    # Upstream backend servers
    upstream backend {
        server app-backend:8080;
    }

    upstream frontend {
        server app-frontend:80;
    }

    # Main server block
    server {
        listen 80;
        server_name your-domain.com www.your-domain.com;

        # Redirect to HTTPS (we'll add SSL later)
        # return 301 https://$server_name$request_uri;

        # Frontend - Angular app
        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # For WebSocket support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }

        # Backend API
        location /api/ {
            limit_req zone=api_limit burst=20 nodelay;
            
            proxy_pass http://backend/api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }

        # Health check endpoint
        location /health {
            access_log off;
            return 200 "OK";
            add_header Content-Type text/plain;
        }
    }
}
```

**Create `Dockerfile` for Nginx:**

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

**Update your `docker-compose.yml`** to include Nginx:

```yaml
  nginx:
    build:
      context: ./nginx-proxy
      dockerfile: Dockerfile
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    networks:
      - app-network
    depends_on:
      - backend
      - frontend
    volumes:
      - ./nginx-proxy/logs:/var/log/nginx
```

Now remove the port mappings from frontend and backend since Nginx handles external access:

```yaml
  backend:
    # Remove or comment out:
    # ports:
    #   - "8080:8080"
    
  frontend:
    # Remove or comment out:
    # ports:
    #   - "4200:80"
```

## 6. DNS Configuration

### 6.1 What is DNS?

**DNS (Domain Name System)** is like the internet's phonebook. Instead of remembering `45.123.45.67`, you can use `myawesomeapp.com`.

### 6.2 Setting Up DNS

**Step 1: Buy a domain** (if you haven't already)
- Go to Namecheap, Google Domains, or Cloudflare
- Search for an available domain
- Purchase it ($10-15/year typically)

**Step 2: Point DNS to your VPS**

In your domain registrar's control panel:

1. Find "DNS Settings" or "Nameservers"
2. Add an **A Record**:
   - **Type**: A
   - **Name**: @ (or blank, meaning root domain)
   - **Value**: Your VPS IP address (e.g., 45.123.45.67)
   - **TTL**: 3600 (1 hour)

3. Add a **CNAME Record** for www:
   - **Type**: CNAME
   - **Name**: www
   - **Value**: @ or your-domain.com
   - **TTL**: 3600

**Example:**
```
Type    Name    Value               TTL
A       @       45.123.45.67        3600
CNAME   www     your-domain.com     3600
```

**DNS propagation** takes 15 minutes to 48 hours. You can check if it's working:

```bash
# On your local machine
nslookup your-domain.com
dig your-domain.com
```

### 6.3 Adding SSL/TLS with Let's Encrypt

HTTPS is no longer optional—it's essential for security and SEO. Let's get a free SSL certificate!

**Install Certbot:**

```bash
sudo apt install certbot python3-certbot-nginx -y
```

**Get a certificate:**

```bash
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

Follow the prompts. Certbot will:
1. Verify you own the domain
2. Generate certificates
3. Automatically configure Nginx
4. Set up auto-renewal

**Test auto-renewal:**

```bash
sudo certbot renew --dry-run
```

Now update your `nginx.conf` to use HTTPS:

```nginx
    server {
        listen 80;
        server_name your-domain.com www.your-domain.com;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name your-domain.com www.your-domain.com;

        ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
        
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # ... rest of your location blocks
    }
```

## 7. Security with WAF

### 7.1 What is a WAF?

A **Web Application Firewall (WAF)** is like a security guard with special training. While a regular firewall checks "Are you allowed in this building?", a WAF checks "Are you trying to do something suspicious inside?"

**WAF protects against:**
- SQL injection
- Cross-site scripting (XSS)
- DDoS attacks
- Bot attacks
- Malicious payloads

### 7.2 Implementing ModSecurity with Nginx

**ModSecurity** is an open-source WAF. It's like adding a smart bouncer to your Nginx setup.

**Install ModSecurity:**

```bash
sudo apt install libmodsecurity3 libmodsecurity-dev -y
```

**Update your Nginx Dockerfile:**

```dockerfile
FROM nginx:alpine

# Install ModSecurity
RUN apk add --no-cache modsecurity modsecurity-nginx

# Copy ModSecurity config
COPY modsecurity.conf /etc/nginx/modsecurity/modsecurity.conf
COPY owasp-crs/ /etc/nginx/modsecurity/owasp-crs/

# Copy Nginx config
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

**Create `modsecurity.conf`:**

```nginx
# Turn ModSecurity on
SecRuleEngine On

# Reject requests if we fail to parse them
SecRequestBodyAccess On
SecRule REQUEST_HEADERS:Content-Type "text/xml" \
     "id:'200000',phase:1,t:none,t:lowercase,pass,nolog,ctl:requestBodyProcessor=XML"

# Buffer size
SecRequestBodyLimit 13107200
SecRequestBodyNoFilesLimit 131072

# Log everything
SecAuditEngine RelevantOnly
SecAuditLogType Serial
SecAuditLog /var/log/nginx/modsec_audit.log

# Include OWASP Core Rule Set
Include /etc/nginx/modsecurity/owasp-crs/crs-setup.conf
Include /etc/nginx/modsecurity/owasp-crs/rules/*.conf
```

**Download OWASP Core Rule Set:**

```bash
cd ~/nginx-proxy
git clone https://github.com/coreruleset/coreruleset.git owasp-crs
cd owasp-crs
mv crs-setup.conf.example crs-setup.conf
```

**Update `nginx.conf` to use ModSecurity:**

```nginx
http {
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsecurity/modsecurity.conf;
    
    # ... rest of config
}
```

### 7.3 Rate Limiting (Already in our Nginx config!)

We already added rate limiting in our Nginx configuration:

```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
```

This means: "Each IP address can make 10 API requests per second. Keep track of this in 10MB of memory."

**Tuning for your needs:**
- **rate=10r/s**: 10 requests per second (adjust based on your API)
- **burst=20**: Allow temporary bursts up to 20 requests
- **nodelay**: Don't delay burst requests

## 8. User Management & Access Control

### 8.1 Linux User Management

Remember earlier when we created the `deploy` user? Let's set up proper access control.

**Create specialized users:**

```bash
# User for running apps
sudo adduser --system --group apprunner

# User for Jenkins
sudo adduser jenkins
sudo usermod -aG docker jenkins
```

**SSH Key-based authentication:**

```bash
# On your local machine, generate SSH keys
ssh-keygen -t ed25519 -C "your-email@example.com"

# Copy your public key to server
ssh-copy-id deploy@your-vps-ip

# On server, disable password authentication
sudo nano /etc/ssh/sshd_config

# Change these lines:
PasswordAuthentication no
PermitRootLogin no

# Restart SSH
sudo systemctl restart sshd
```

**Why SSH keys?** They're like having a unique, unbreakable physical key instead of a password that can be guessed.

### 8.2 Application-Level User Management

For your Java backend, you'll want authentication and authorization. Here's a quick Spring Security example:

**In your Spring Boot app:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/**").authenticated()
            )
            .oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt)
            .csrf().disable();  // Disable for REST APIs, use CORS instead
        
        return http.build();
    }
}
```

**Environment variables for secrets:**

```yaml
# In your docker-compose.yml backend service
environment:
  JWT_SECRET: ${JWT_SECRET}
  ADMIN_PASSWORD: ${ADMIN_PASSWORD}
```

**Add to your `.env` file:**

```bash
JWT_SECRET=your-super-secret-jwt-key-change-this
ADMIN_PASSWORD=secure-admin-password
```

## What's Next?

Your application is now:
- ✅ Containerized with Docker  
- ✅ Accessible via a custom domain with HTTPS  
- ✅ Protected by Nginx reverse proxy  
- ✅ Secured with WAF and rate limiting  
- ✅ Set up with proper user access control  

But here's the thing—every time you make a code change, you have to:
1. SSH into your server
2. Pull the latest code
3. Rebuild containers
4. Restart services
5. Check logs
6. Hope nothing broke

**That's not sustainable.**

In Part 3 (the final part!), we'll automate EVERYTHING:

 **Jenkins CI/CD pipeline** - Deploy on every git push  
 **Monitoring and backups** - Know when things break  
 **Going live** - Pre-launch checklist and launch day  
 **Troubleshooting guide** - Solutions to common problems  

From manual deployment to fully automated production pipeline.

**See you in Part 3!**

---

### Resources Covered in Part 2:
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt](https://letsencrypt.org/)
- [ModSecurity Documentation](https://github.com/SpiderLabs/ModSecurity)
- [OWASP Core Rule Set](https://coreruleset.org/)

### Coming Up in Part 3:
- Jenkins CI/CD pipeline setup
- Automated testing and deployment
- Monitoring with Prometheus & Grafana
- Backup strategies
- Going live checklist

---

*Part 3 coming soon. Subscribe to get updated when it is published!*
