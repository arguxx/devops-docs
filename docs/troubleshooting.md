# Troubleshooting Guide

Common issues and solutions for DevOps workflows and tools.

## 🔍 General Debugging Approach

### 1. Gather Information

```bash
# System information
uname -a
df -h
free -m
ps aux | head -20

# Network connectivity
ping google.com
nslookup your-service.com
netstat -tulpn | grep :8080
```

### 2. Check Logs

```bash
# Application logs
tail -f /var/log/app/application.log
journalctl -u your-service -f

# System logs
tail -f /var/log/syslog
dmesg | tail -20
```

### 3. Verify Configuration

```bash
# Environment variables
env | grep -i app
cat .env

# Configuration files
cat config/production.yml
```

## 🐳 Docker Issues

### Container Won't Start

**Symptoms:**

-   Container exits immediately
-   Error messages in logs
-   Port binding failures

**Solutions:**

=== "Check Logs"
```bash # Container logs
docker logs <container-id>

    # Follow logs in real-time
    docker logs -f <container-id>

    # Last 100 lines
    docker logs --tail 100 <container-id>
    ```

=== "Debug Interactively"
```bash # Start container with shell
docker run -it --entrypoint=/bin/sh <image>

    # Execute into running container
    docker exec -it <container-id> /bin/bash

    # Check process list
    docker exec <container-id> ps aux
    ```

=== "Check Resources"
```bash # Container inspection
docker inspect <container-id>

    # Resource usage
    docker stats <container-id>

    # System resources
    docker system df
    ```

### Common Docker Errors

!!! bug "Port Already in Use"
```bash # Error: bind: address already in use

    # Find process using port
    lsof -i :8080
    sudo netstat -tulpn | grep :8080

    # Kill process or use different port
    docker run -p 8081:8080 myapp
    ```

!!! bug "Permission Denied"
```bash # Error: permission denied while trying to connect to Docker daemon

    # Add user to docker group
    sudo usermod -aG docker $USER
    newgrp docker

    # Or use sudo temporarily
    sudo docker ps
    ```

!!! bug "No Space Left on Device"
```bash # Clean up Docker resources
docker system prune -a --volumes

    # Remove unused images
    docker image prune -a

    # Remove unused containers
    docker container prune
    ```

## ☸️ Kubernetes Issues

### Pod Issues

**Pod Won't Start:**

```bash
# Check pod status
kubectl get pods -o wide

# Describe pod for events
kubectl describe pod <pod-name>

# Check pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>

# Previous container logs
kubectl logs <pod-name> --previous
```

**Common Pod Problems:**

=== "ImagePullBackOff"
```bash # Check image name and tag
kubectl describe pod <pod-name>

    # Verify image exists
    docker pull <image-name>

    # Check registry credentials
    kubectl get secrets
    kubectl describe secret <registry-secret>
    ```

=== "CrashLoopBackOff"
```bash # Check application logs
kubectl logs <pod-name> --previous

    # Debug with shell
    kubectl run debug --rm -it --image=busybox -- sh

    # Check resource limits
    kubectl describe pod <pod-name>
    ```

=== "Pending State"
```bash # Check node resources
kubectl describe nodes

    # Check PVC status
    kubectl get pvc

    # Check pod scheduling
    kubectl describe pod <pod-name>
    ```

### Service Discovery Issues

```bash
# Check service endpoints
kubectl get endpoints <service-name>

# Test service connectivity
kubectl run test --rm -it --image=busybox -- sh
# Inside pod: nslookup <service-name>

# Check service configuration
kubectl describe service <service-name>
```

### Ingress Problems

```bash
# Check ingress status
kubectl get ingress

# Describe ingress for events
kubectl describe ingress <ingress-name>

# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

## 🔧 CI/CD Pipeline Issues

### Build Failures

**Common Build Problems:**

=== "Dependency Issues"
```bash # Clear dependency cache
npm ci --cache /tmp/empty-cache
pip install --no-cache-dir -r requirements.txt

    # Lock file conflicts
    rm package-lock.json
    npm install
    ```

=== "Test Failures"
```bash # Run tests locally
npm test
pytest -v

    # Debug specific test
    npm test -- --testNamePattern="specific test"
    pytest tests/test_specific.py -v -s
    ```

=== "Build Timeouts"
`yaml
    # Increase timeout in GitHub Actions
    jobs:
      build:
        timeout-minutes: 30
        steps:
          - name: Build
            run: npm run build
            timeout-minutes: 15
    `

### Deployment Issues

**Deployment Failures:**

```bash
# Check deployment status
kubectl rollout status deployment/<deployment-name>

# View deployment history
kubectl rollout history deployment/<deployment-name>

# Rollback deployment
kubectl rollout undo deployment/<deployment-name>

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=0
kubectl scale deployment <deployment-name> --replicas=3
```

## 🌐 Networking Issues

### DNS Problems

```bash
# Test DNS resolution
nslookup google.com
dig google.com

# Check /etc/resolv.conf
cat /etc/resolv.conf

# Test internal DNS (in Kubernetes)
kubectl run test --rm -it --image=busybox -- nslookup kubernetes.default
```

### Load Balancer Issues

```bash
# Check load balancer status
kubectl get svc -o wide

# Test endpoint connectivity
curl -v http://<external-ip>:<port>/health

# Check backend pod health
kubectl get pods -l app=<your-app>
kubectl logs <pod-name>
```

### SSL/TLS Problems

```bash
# Check certificate
openssl s_client -connect your-domain.com:443 -servername your-domain.com

# Verify certificate chain
curl -vI https://your-domain.com

# Check certificate expiration
echo | openssl s_client -connect your-domain.com:443 2>/dev/null | openssl x509 -noout -dates
```

## 💾 Database Issues

### Connection Problems

```bash
# Test database connection
psql -h localhost -U username -d database_name
mysql -h localhost -u username -p database_name

# Check connection limits
SELECT * FROM pg_stat_activity;  -- PostgreSQL
SHOW PROCESSLIST;                -- MySQL
```

### Performance Issues

```sql
-- PostgreSQL slow queries
SELECT query, mean_time, calls, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- MySQL slow queries
SELECT * FROM mysql.slow_log
ORDER BY start_time DESC
LIMIT 10;
```

### Storage Issues

```bash
# Check database size
SELECT pg_size_pretty(pg_database_size('database_name'));  -- PostgreSQL

# Check table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(tablename::text))
FROM pg_tables
ORDER BY pg_total_relation_size(tablename::text) DESC;
```

## 📊 Monitoring & Alerting

### Metrics Not Showing

```bash
# Check Prometheus targets
curl http://prometheus:9090/api/v1/targets

# Verify metrics endpoint
curl http://your-app:8080/metrics

# Check Grafana data source
curl -X GET http://admin:admin@grafana:3000/api/datasources
```

### Alert Manager Issues

```bash
# Check alert rules
curl http://prometheus:9090/api/v1/rules

# Test webhook
curl -X POST http://alertmanager:9093/api/v1/alerts \
  -H "Content-Type: application/json" \
  -d '[{"labels":{"alertname":"TestAlert"}}]'
```

## 🛡️ Security Issues

### Certificate Problems

```bash
# Check certificate validity
openssl x509 -in certificate.crt -text -noout

# Verify certificate chain
openssl verify -CAfile ca-bundle.crt certificate.crt

# Check private key matches certificate
openssl x509 -noout -modulus -in certificate.crt | openssl md5
openssl rsa -noout -modulus -in private.key | openssl md5
```

### Secret Management

```bash
# Check Kubernetes secrets
kubectl get secrets
kubectl describe secret <secret-name>

# Decode secret (base64)
kubectl get secret <secret-name> -o jsonpath='{.data.password}' | base64 -d
```

## 🚨 Emergency Procedures

### System Outage

1. **Assess Impact**

    ```bash
    # Check service status
    kubectl get pods -A
    docker ps
    systemctl status <service-name>
    ```

2. **Immediate Actions**

    ```bash
    # Restart services
    kubectl rollout restart deployment/<deployment-name>
    docker-compose restart
    systemctl restart <service-name>
    ```

3. **Scale Resources**

    ```bash
    # Scale up replicas
    kubectl scale deployment <deployment-name> --replicas=5

    # Add more nodes (if using cloud provider)
    # Follow your cloud provider's scaling procedures
    ```

### Data Recovery

```bash
# Restore from backup
pg_restore -h localhost -U username -d database_name backup.sql

# Verify data integrity
SELECT COUNT(*) FROM critical_table;

# Check recent changes
SELECT * FROM audit_log WHERE created_at > NOW() - INTERVAL '1 hour';
```

## 📞 Getting Help

### Information to Gather

When seeking help, provide:

-   **Error messages** (full stack traces)
-   **Environment details** (OS, versions, configurations)
-   **Recent changes** (deployments, config updates)
-   **Logs** (application and system logs)
-   **Timeline** (when did the issue start?)

### Useful Commands for Support

```bash
# Generate system report
kubectl cluster-info dump > cluster-info.txt
docker system info > docker-info.txt
journalctl --since "1 hour ago" > system-logs.txt

# Package logs
tar -czf support-bundle.tar.gz *.txt *.log
```

### Contact Information

-   **Internal Team**: #devops-support Slack channel
-   **On-call Engineer**: Use PagerDuty escalation
-   **Vendor Support**: Check service documentation
-   **Community**: Stack Overflow, GitHub Issues

---

!!! tip "Prevention is Better Than Cure" - Implement comprehensive monitoring - Use infrastructure as code - Maintain up-to-date documentation - Practice disaster recovery procedures - Keep systems updated and patched
