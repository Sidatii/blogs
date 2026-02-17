---
title: DevOps for devs - From Localhost to Production - P1.
category: DevOps
tags: Deploy, develop, VPS, HomeLab, Production.
excerpt: This blog is the first part of a DevOps for devs series. In this blog, I am taking you through an enjoyable journey of discovering the world of DevOps, CI/CD and deploying to production.
published_at: 2026-02-16
---

![Thumbnail](images/thumbnail-part-01.png)

***Image by Author.***

# From Localhost to Production - Part 2: From Localhost to Containers

Hey guys!

Let me guess — you developed an application and it works perfectly on `localhost`. Now you want to deploy it to a real server where actual people can use it.

You Google "deploying (java, python...) app" and get hit with: Docker, Kubernetes, Jenkins, CI/CD, reverse proxy, SSL certificates...

**Your head spins. You close the laptop. Maybe tomorrow.**

I've been there. That's why I created this series.

## What Makes This Different?

Most tutorials are either:
1. **The "Copy-Paste Special"** - Run these commands and pray
2. **The "Enterprise Encyclopedia"** - 200 pages assuming you know everything

**This series is different.** We'll build your deployment from the ground up, explaining every concept in plain English. No magic commands. No assumptions.

Think of it like building a house:
- **Part 1:** Lay the foundation (Docker & containers)
- **Part 2:** Build the structure (Nginx, DNS, security)
- **Part 3:** Add smart features (Jenkins CI/CD, monitoring)

## The 3-Part Journey

### **Part 1: Foundation & Containerization** (You are here!)
By the end: Your apps will be containerized and running in Docker. You'll understand *why* containers matter and *how* they work.

**Covers:** VPS setup, Docker basics, containerizing Java + Angular, networking & volumes

### **Part 2: Infrastructure & Security**  [Read here](https://outofbounds.qzz.io/blog/devops-for-devs-from-localhost-to-production-p2)
By the end: Your app will be accessible via a custom domain with HTTPS, protected by security layers.

**Covers:** Nginx reverse proxy, DNS, SSL certificates, WAF, user management

### **Part 3: Automation & Going Live** [Read here](https://outofbounds.qzz.io/blog/devops-for-devs-from-localhost-to-production-p3)
By the end: Fully automated deployment pipeline that professionals use.

**Covers:** Jenkins CI/CD, monitoring, backups, troubleshooting, launch checklist

## Who Is This For?

**Perfect for you if:**
- ✅ You have a working software development experience.
- ✅ You've never deployed to production or not fully engaged in the deployment process due to team responsibility.
- ✅ You want to understand the *why*, not just the *how*

**You'll need:**
- A VPS (DigitalOcean, Linode, Vultr, Contabo or others) - $5-10/month
- Basic terminal knowledge
- 2-4 hours for Part 1 implementation

## My Promise

I'll explain, not just instruct. You'll understand *why* we're doing things, not just *what* commands to run. Real-world analogies, actual code, common errors solved.

We're not just making your app "work on the internet." We're making it **production-ready**.

**Ready to start?**

By the end of Part 1, you'll have a secure VPS running your containerized applications. 

Grab your coffee, fire up your terminal, and let's turn that localhost app into something the world can see.

> ***One final disclaimer***: In order to be more practical, we are using Java and angular for demonstration. However, every piece of knowledge presented here remain valid for all technologies with some minor differences. I count on you to solve them on your own since it is your specialty.

**Let's dive in!**

## Understanding the Big Picture

Before we start typing commands, let's understand what we're building. Imagine your deployment as a house:

- **VPS/HomeLab**: Your plot of land
- **Docker Containers**: Individual rooms (backend, frontend, database, each in its own space)
- **Docker Volumes**: Storage closets that persist even when you redecorate rooms
- **Docker Networks**: Hallways connecting the rooms
- **Nginx**: Your front door and security guard (routes visitors to the right room)
- **WAF**: Your home security system (blocks suspicious visitors)
- **DNS**: Your mailing address (myawesomeapp.com instead of 192.168.1.100)
- **Jenkins**: Your personal assistant who rebuilds and redecorates whenever you make changes

Got it? Awesome! Let's build this house.

## What You'll Need

### On Your Local Machine
- Git installed
- Basic terminal/command line knowledge
- Your application application code
- An SSH client (built into Mac/Linux, use PuTTY for Windows)

### For Your VPS
- A VPS (Virtual Private Server) from providers like:
  - DigitalOcean ($5-10/month for starter)
  - Linode
  - Vultr
  - Or a HomeLab server if you're running one at home
- Ubuntu 24.04 LTS installed (we'll use this in our examples)
- At least 2GB RAM, 2 CPU cores, 50GB storage
- Root or sudo access

### Optional But Recommended
- A domain name (from Namecheap, Google Domains, etc.) - $10-15/year
- A GitHub/GitLab account for your code repository

> [!TIP] You can get a free domain name through the open source community.
> Visit [Digital Plat](https://domain.digitalplat.org/) and grab your free domain.

## 1. Setting Up Your VPS

### 1.1 Initial Server Access

First, let's connect to your VPS. When you create a VPS, you'll get an IP address (something like `192.168.1.100` or `45.123.45.67`). You'll also have a root password or SSH key.

```bash
# Connect via SSH
ssh root@your-vps-ip-address

# Example:
ssh root@45.123.45.67
```

**What is SSH?** Think of it as a secure phone line to your server. You type commands on your computer, but they execute on the server.

### 1.2 Securing Your Server

First things first—security! Let's create a non-root user and set up a firewall.

```bash
# Update system packages
apt update && apt upgrade -y

# Create a new user (replace 'deploy' with your preferred username)
adduser deploy

# Give them sudo powers
usermod -aG sudo deploy

# Switch to the new user
su - deploy
```

**Why not use root?** Using root is like driving with no seatbelt. One typo could break your entire server. A regular user with sudo is safer.

### 1.3 Setting Up Firewall

```bash
# Install UFW (Uncomplicated Firewall)
sudo apt install ufw -y

# Allow SSH (so you don't lock yourself out!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable the firewall
sudo ufw enable

# Check status
sudo ufw status
```

**What's a firewall?** Imagine your server has 65,535 doors (ports). A firewall keeps most of them locked and only opens the ones you need (like 80 for websites, 22 for SSH).


## 2. Understanding Containerization

### What is Docker?

Imagine you're a chef with a famous recipe. You could try to recreate your kitchen in every restaurant, installing the same stove, the same pots, the same ingredients. Or... you could pack everything into a food truck that works anywhere.

That's Docker!

**Docker** lets you package your application with everything it needs (Java runtime, libraries, configurations) into a **container**. This container runs the same way on your laptop, your VPS, or anywhere else.

### Key Docker Concepts

1. **Image**: The blueprint (like a recipe)
2. **Container**: A running instance of an image (like a dish made from the recipe)
3. **Dockerfile**: Instructions to build an image
4. **Docker Compose**: A way to define and run multiple containers together
5. **Volume**: Persistent storage that survives container restarts
6. **Network**: How containers talk to each other

### Installing Docker

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group (so you don't need sudo every time)
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose -y

# Verify installation
docker --version
docker-compose --version

# Log out and back in for group changes to take effect
exit
# Then SSH back in
```


## 3. Dockerizing Your Applications

### 3.1 Dockerizing the Java Backend

Let's assume you have a Spring Boot application. Here's how to containerize it.

**Create a file called `Dockerfile` in your backend project root:**

```dockerfile
# Start from a base image with Java
FROM eclipse-temurin:17-jdk-alpine AS build

# Set working directory
WORKDIR /app

# Copy Maven/Gradle files
COPY pom.xml .
COPY src ./src

# Build the application
RUN ./mvnw clean package -DskipTests

# Use a smaller runtime image
FROM eclipse-temurin:17-jre-alpine

# Set working directory
WORKDIR /app

# Copy the built jar from build stage
COPY --from=build /app/target/*.jar app.jar

# Expose port 8080
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**What's happening here?**

- **FROM**: We start with a pre-made image that has Java installed
- **WORKDIR**: Like `cd` into a directory
- **COPY**: Copy files from your computer into the container
- **RUN**: Execute commands (like building your app)
- **EXPOSE**: Document which port the app uses (doesn't actually open it)
- **ENTRYPOINT**: The command to run when the container starts

**Multi-stage build**: We use one image to build (with all dev tools), then copy just the result to a smaller runtime image. Like preparing a meal in a big kitchen, then serving it in a small dining room.

### 3.2 Dockerizing the Angular Frontend

**Create a `Dockerfile` in your Angular project root:**

```dockerfile
# Build stage
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build the app for production
RUN npm run build -- --configuration production

# Runtime stage - use Nginx to serve
FROM nginx:alpine

# Copy built files to Nginx
COPY --from=build /app/dist/your-app-name /usr/share/nginx/html

# Copy custom Nginx config (we'll create this next)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Create `nginx.conf` in your Angular project root:**

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Support Angular routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Compression for better performance
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

**Why Nginx for Angular?** Angular builds to static files (HTML, CSS, JS). Nginx is a super-fast web server perfect for serving these files. It's like using a specialized delivery service instead of a general courier.


## 4. Docker Networking & Volumes

### 4.1 Understanding Docker Networks

When you run multiple containers, they need to talk to each other. Docker networks are like creating a private neighborhood where your containers can communicate.

**Types of Networks:**

- **Bridge** (default): Containers can talk to each other using container names
- **Host**: Container uses the host's network directly
- **None**: No network access

We'll use a **bridge network** named `app-network`.

### 4.2 Understanding Docker Volumes

Containers are ephemeral—when you delete them, their data disappears. Volumes are like external hard drives that persist data.

**Why volumes?**

Imagine you have a database container. Without volumes, restarting it means losing all your data. With volumes, data is stored outside the container and survives restarts.

### 4.3 Docker Compose: Orchestrating Everything

Instead of running multiple `docker run` commands, we'll use **Docker Compose** to define all our services in one file.

**Create `docker-compose.yml` in a new directory on your VPS:**

```yaml
version: '3.8'

services:
  # PostgreSQL Database
  database:
    image: postgres:15-alpine
    container_name: app-database
    restart: unless-stopped
    environment:
      POSTGRES_DB: myappdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: ${DB_PASSWORD}  # We'll use an env file
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Java Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: app-backend
    restart: unless-stopped
    depends_on:
      database:
        condition: service_healthy
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://database:5432/myappdb
      SPRING_DATASOURCE_USERNAME: appuser
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
    ports:
      - "8080:8080"
    networks:
      - app-network
    volumes:
      - backend-logs:/app/logs

  # Angular Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: app-frontend
    restart: unless-stopped
    ports:
      - "4200:80"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
  backend-logs:
```

**Breaking this down:**

- **services**: Each container we want to run
- **depends_on**: "Don't start backend until database is healthy"
- **environment**: Configuration variables passed to containers
- **volumes**: Named volumes for persistent data
- **networks**: All containers join `app-network` and can talk using service names
- **restart: unless-stopped**: Auto-restart containers if they crash

**Create a `.env` file** (in the same directory):

```bash
DB_PASSWORD=super_secret_password_change_me
```

> [!WARNING] Never commit `.env` to Git! Add it to `.gitignore`.

## What's Next?

Congratulations! Your applications are now containerized and running in Docker. But there's a problem—they're only accessible via IP addresses and ports like `http://45.123.45.67:8080`.

In Part 2 of this series, we'll transform your setup into a production-ready deployment:

🌐 **Setting up a proper web server** (Nginx as reverse proxy)  
🔒 **Getting a domain name and SSL certificates** (myapp.com with HTTPS)  
🛡️ **Implementing security layers** (WAF, rate limiting, user management)

Your app will go from "it works" to "it's production-ready."

**Stay tuned for Part 2!**


### Resources Covered So Far:
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Spring Boot Docker Guide](https://spring.io/guides/gs/spring-boot-docker/)

### Coming Up in Part 2:
- Nginx reverse proxy configuration
- DNS and SSL setup
- Web Application Firewall (WAF)
- User management and access control


> [!IMPORTANT] If you like my content, you can *Subscribe* to my newsletter to get notified whenever new blogs are published!

- [Read Part 2](https://outofbounds.qzz.io/from-localhost-to-production-p2)
- [Read Part 3](https://outofbounds.qzz.io/from-localhost-to-production-p3)
