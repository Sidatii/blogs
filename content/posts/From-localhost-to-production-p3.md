---
title: DevOps for devs - From Localhost to Production - P3.
category: DevOps
tags: Deploy, develop, VPS, HomeLab, Production.
excerpt: This blog is the third part of a DevOps for devs series. In this blog, I am taking you through an enjoyable journey of discovering the world of DevOps, CI/CD and deploying to production.
published_at: 2026-02-18
---

![Thumbnail](images/thumbnail-part-03.png)


# From Localhost to Production - Part 3: Automation, Monitoring & Going Live

**Welcome to the final part!**

Let's recap our journey so far:

**Part 1 - Foundation & Containerization:**
- ✅ Set up VPS with security
- ✅ Learned Docker and containerization
- ✅ Dockerized Java + Angular apps
- ✅ Set up Docker networking and volumes

**Part 2 - Infrastructure & Security:**
- ✅ Configured Nginx as reverse proxy
- ✅ Set up DNS and SSL certificates
- ✅ Implemented WAF for security
- ✅ Configured user management

**Now in Part 3, we're automating everything and going live!**

No more manual deployments. No more SSH-ing in every time you make a change.

We're building a professional CI/CD pipeline where:
- You push code to GitHub → Jenkins automatically builds, tests, and deploys
- Your app is monitored 24/7
- Backups run automatically
- You get alerts if something breaks

Let's finish this deployment journey strong!

## 9. Jenkins Pipeline for Automated Deployment

### 9.1 What is CI/CD?

**CI/CD (Continuous Integration/Continuous Deployment)** is like having a robot that:

1. Watches your code repository
2. When you push changes, it automatically:
   - Builds your app
   - Runs tests
   - Deploys to your server
   - Notifies you if something breaks

**Jenkins** is the robot!

### 9.2 Installing Jenkins

**Add Jenkins to docker-compose.yml:**

```yaml
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: unless-stopped
    user: root  # Needed for Docker access
    ports:
      - "8081:8080"
      - "50000:50000"
    volumes:
      - jenkins-data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - app-network
    environment:
      JAVA_OPTS: "-Djenkins.install.runSetupWizard=false"

volumes:
  jenkins-data:
```

**Start Jenkins:**

```bash
docker-compose up -d jenkins
```

**Access Jenkins:**

Open your browser to `http://your-vps-ip:8081`

**Get initial admin password:**

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 9.3 Configuring Jenkins

1. **Install suggested plugins**
2. **Create admin user**
3. **Install additional plugins:**
   - Docker Pipeline
   - Git plugin
   - Blue Ocean (nice UI)
   - NodeJS plugin

**Navigate to:** Manage Jenkins → Manage Plugins → Available → Search and install

### 9.4 Creating a Jenkins Pipeline

**In your Git repository, create `Jenkinsfile` at the root:**

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        BACKEND_DIR = './backend'
        FRONTEND_DIR = './frontend'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling latest code...'
                git branch: 'main',
                    url: 'https://github.com/yourusername/your-repo.git'
            }
        }
        
        stage('Build Backend') {
            steps {
                echo 'Building Java backend...'
                dir("${BACKEND_DIR}") {
                    sh '''
                        ./mvnw clean package -DskipTests
                        docker build -t app-backend:latest .
                    '''
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                echo 'Building Angular frontend...'
                dir("${FRONTEND_DIR}") {
                    sh '''
                        npm ci
                        npm run build -- --configuration production
                        docker build -t app-frontend:latest .
                    '''
                }
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Backend Tests') {
                    steps {
                        dir("${BACKEND_DIR}") {
                            sh './mvnw test'
                        }
                    }
                }
                stage('Frontend Tests') {
                    steps {
                        dir("${FRONTEND_DIR}") {
                            sh 'npm run test -- --watch=false --browsers=ChromeHeadless'
                        }
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying with docker-compose...'
                sh '''
                    docker-compose down
                    docker-compose up -d --build
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                echo 'Checking if services are healthy...'
                sh '''
                    sleep 30
                    curl -f http://localhost/health || exit 1
                '''
            }
        }
    }
    
    post {
        success {
            echo '🎉 Deployment successful!'
            // Optional: Send notification
        }
        failure {
            echo '❌ Deployment failed!'
            // Optional: Send notification
        }
        always {
            echo 'Cleaning up...'
            sh 'docker system prune -f'
        }
    }
}
```

### 9.5 Setting Up a Jenkins Job

1. **Click "New Item"**
2. **Enter name:** "Java-Angular-App"
3. **Select:** "Pipeline"
4. **Click OK**

**In Pipeline configuration:**

1. **Definition:** Pipeline script from SCM
2. **SCM:** Git
3. **Repository URL:** https://github.com/yourusername/your-repo.git
4. **Credentials:** Add your GitHub credentials
5. **Branch:** main
6. **Script Path:** Jenkinsfile

**Build Triggers:**
- Check "GitHub hook trigger for GITScm polling"
- Or check "Poll SCM" with schedule: `H/5 * * * *` (every 5 minutes)

**Click Save**

### 9.6 Webhook for Auto-Deployment

**In your GitHub repository:**

1. Go to Settings → Webhooks
2. Click "Add webhook"
3. **Payload URL:** `http://your-vps-ip:8081/github-webhook/`
4. **Content type:** application/json
5. **Events:** Just the push event
6. Click "Add webhook"

Now every time you push to GitHub, Jenkins automatically deploys!

---

## 10. Going Live!

### 10.1 Pre-Launch Checklist

Before you share your app with the world:

```bash
# ✅ Check all services are running
docker-compose ps

# ✅ Verify database connection
docker-compose logs database | grep "ready to accept connections"

# ✅ Test backend health
curl http://localhost:8080/api/health

# ✅ Test frontend
curl http://localhost/

# ✅ Check SSL certificate
curl -I https://your-domain.com

# ✅ Verify WAF is active
docker-compose logs nginx | grep modsecurity

# ✅ Check disk space
df -h

# ✅ Check memory usage
free -h
```

### 10.2 Monitoring Setup

**Install basic monitoring:**

```bash
# Install htop for process monitoring
sudo apt install htop -y

# Install netdata for real-time monitoring
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

Access Netdata at `http://your-vps-ip:19999`

**Add to docker-compose.yml for application monitoring:**

```yaml
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - app-network

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - app-network
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}

volumes:
  prometheus-data:
  grafana-data:
```

### 10.3 Backup Strategy

**Create a backup script (`backup.sh`):**

```bash
#!/bin/bash

BACKUP_DIR="/home/deploy/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
docker exec app-database pg_dump -U appuser myappdb > \
    $BACKUP_DIR/db_backup_$DATE.sql

# Backup Docker volumes
docker run --rm \
    -v db-data:/data \
    -v $BACKUP_DIR:/backup \
    alpine tar czf /backup/db_volume_$DATE.tar.gz /data

# Keep only last 7 days of backups
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

**Make it executable and schedule:**

```bash
chmod +x backup.sh

# Add to crontab (run daily at 2 AM)
crontab -e

# Add this line:
0 2 * * * /home/deploy/backup.sh >> /home/deploy/backup.log 2>&1
```

### 10.4 Going Live!

**Final deployment:**

```bash
# Pull latest code
cd ~/your-app
git pull origin main

# Build and deploy
docker-compose down
docker-compose up -d --build

# Watch logs
docker-compose logs -f
```

**Announce to the world! 🎉**

Share your app:
- `https://your-domain.com`
- Test on different devices
- Share with friends and users

---

## 11. Troubleshooting Common Issues

### Issue: Can't connect to database

**Solution:**
```bash
# Check database logs
docker-compose logs database

# Check if it's running
docker-compose ps database

# Restart it
docker-compose restart database

# Check environment variables
docker-compose exec backend env | grep SPRING_DATASOURCE
```

### Issue: Nginx returns 502 Bad Gateway

**Solution:**
```bash
# Check backend is running
docker-compose ps backend

# Check backend logs
docker-compose logs backend

# Check if backend is reachable from Nginx
docker-compose exec nginx ping app-backend

# Check Nginx error logs
docker-compose logs nginx | grep error
```

### Issue: Jenkins can't build

**Solution:**
```bash
# Check Jenkins logs
docker-compose logs jenkins

# Ensure Jenkins has Docker access
docker-compose exec jenkins docker ps

# Check disk space
df -h
```

### Issue: SSL certificate not working

**Solution:**
```bash
# Renew certificate
sudo certbot renew

# Check certificate expiry
sudo certbot certificates

# Reload Nginx
docker-compose restart nginx
```

### Issue: Out of disk space

**Solution:**
```bash
# Check space
df -h

# Clean Docker
docker system prune -a

# Clean old images
docker images | grep "months ago" | awk '{print $3}' | xargs docker rmi

# Clean logs
sudo journalctl --vacuum-time=7d
```

### Issue: High memory usage

**Solution:**
```bash
# Check memory
free -h
htop

# Restart heavy services
docker-compose restart backend

# Add swap space
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

## 12. Next Steps

Congratulations! You now have a production-ready deployment. But the journey doesn't end here:

### 1. Performance Optimization
- Add Redis for caching
- Implement CDN for static assets
- Database indexing and query optimization
- Enable HTTP/2 and compression

### 2. Advanced Security
- Implement rate limiting per user (not just IP)
- Add fail2ban for SSH protection
- Set up log monitoring and alerting
- Regular security audits

### 3. Scaling
- Add load balancer for multiple backend instances
- Database replication for high availability
- Container orchestration with Kubernetes
- Horizontal scaling with auto-scaling groups

### 4. Observability
- Centralized logging with ELK stack
- Application Performance Monitoring (APM)
- Error tracking with Sentry
- Uptime monitoring with UptimeRobot

### 5. Professional DevOps
- Infrastructure as Code with Terraform
- Configuration management with Ansible
- Blue-green deployments
- Feature flags for controlled rollouts


## Final Thoughts

Deploying an application is like launching a rocket 🚀. There are many moving parts, but each piece serves a purpose:

- **Docker** packages everything consistently
- **Nginx** routes and protects your traffic
- **WAF** blocks malicious requests
- **DNS** makes your app accessible by name
- **Jenkins** automates deployments
- **Volumes** preserve your data
- **Networks** allow secure communication

You've built all of this! Take a moment to appreciate how far you've come.

## Series Complete!

You made it! You've gone from "it works on localhost" to "it's live in production with automated deployments."

**What we built together:**

**Part 1:** Foundation with Docker and containerization  
**Part 2:** Production infrastructure with Nginx, DNS, and security  
**Part 3:** Automated CI/CD with Jenkins and monitoring  

This is the same deployment stack used by startups and companies worldwide.

### Your Deployment Checklist:
- [x] VPS secured and configured
- [x] Applications containerized
- [x] Nginx reverse proxy
- [x] Custom domain with HTTPS
- [x] Security layers (WAF, rate limiting)
- [x] Automated CI/CD pipeline
- [x] Monitoring and backups
- [x] App is LIVE!

### What's Next?
You're not done learning—you're just getting started:
- Scale horizontally with load balancers
- Explore Kubernetes for orchestration
- Implement advanced monitoring with ELK stack
- Learn Infrastructure as Code with Terraform

### Share Your Success!
Deployed your app? I'd love to hear about it! 

**The best developers aren't those who never face problems—they're the ones who know how to debug and ask for help. Keep building, keep learning!**

*Questions? Stuck somewhere? Ask or share your ideas with me:*
- LinkedIn: [Sidati-nouhi](https://www.linkedin.com/in/sidati-nouhi/)
- X: [SidatiNouhi](https://x.com/SidatiNouhi)
- Contact Page: [Go to page](https://outofbounds.qzz.io/contact)
- Or just leave a comment here!

> [!NOTE] **About the Author:** A developer who has deployed apps, broken things, fixed them, and lived to tell the tale. Currently enjoying the sweet spot between "it works on my machine" and "it works in production."

### The Complete Series:
- **Part 1:** Foundation & Containerization [-> HERE](https://outofbounds.qzz.io/blog/devops-for-devs-from-localhost-to-production-p1)
- **Part 2:** Infrastructure & Security [->HERE](https://outofbounds.qzz.io/blog/devops-for-devs-from-localhost-to-production-p2)
- **Part 3:** Automation & Going Live [->HERE](https://outofbounds.qzz.io/blog/devops-for-devs-from-localhost-to-production-p3)

> [!IMPORTANT] If you like my content, you can *Subscribe* to my newsletter to get notified whenever new blogs are published!
---

### Resources for Continued Learning

- [Docker Documentation](https://docs.docker.com/)
- [Nginx Admin Guide](https://www.nginx.com/resources/admin-guide/)
- [Spring Boot Deployment Guide](https://spring.io/guides/gs/spring-boot-docker/)
- [Angular Deployment Guide](https://angular.io/guide/deployment)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
