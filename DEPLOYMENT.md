# Dokploy Deployment Guide

This guide will help you deploy the React TypeScript frontend application using Dokploy.

## Prerequisites

- A Dokploy instance running and accessible
- A domain name pointing to your server (A record configured)
- Git repository with this code

## Deployment Steps

### 1. Create a New Project in Dokploy

1. Log into your Dokploy dashboard
2. Create a new project (e.g., "dokploy-static")

### 2. Create a Docker Compose Service

1. Click "Create Service" and select **Compose**
2. Select **Docker Compose** as the Compose Type
3. Configure the following:
   - **Provider**: Select GitHub or Git
   - **Repository**: Your repository URL
   - **Branch**: `main` (or your preferred branch)
   - **Compose Path**: `./docker-compose.yml`
4. Click **Save**

### 3. Configure Domain

1. Go to the **Domains** tab in your service
2. Add your domain (e.g., `your-domain.com`)
3. The Traefik labels in `docker-compose.yml` will handle:
   - Automatic SSL certificate generation via Let's Encrypt
   - Reverse proxy routing
   - HTTPS redirection

**Important**: Update the domain in `docker-compose.yml`:
```yaml
- "traefik.http.routers.frontend-app.rule=Host(`your-domain.com`)"
```
Replace `your-domain.com` with your actual domain.

### 4. Deploy

1. Click the **Deploy** button in the General tab
2. Wait for the build process to complete (this may take a few minutes)
3. Monitor the deployment logs for any issues
4. After deployment, wait ~10 seconds for Traefik to generate SSL certificates

### 5. Access Your Application

Once deployed, your application will be accessible at:
- `https://your-domain.com`

## Docker Compose Configuration

The `docker-compose.yml` is configured according to Dokploy best practices:

### Key Configuration Elements

1. **Network**: Uses `dokploy-network` (external network managed by Dokploy)
2. **Port Exposure**: Uses `expose` instead of `ports` to limit access to the container network
3. **Traefik Labels**: Configured for automatic routing and SSL
4. **No container_name**: Omitted to ensure proper logging and metrics in Dokploy

### Traefik Labels Explained

```yaml
labels:
  - "traefik.enable=true"                                                    # Enable Traefik routing
  - "traefik.http.routers.frontend-app.rule=Host(`your-domain.com`)"       # Domain routing rule
  - "traefik.http.routers.frontend-app.entrypoints=websecure"              # Use HTTPS
  - "traefik.http.routers.frontend-app.tls.certResolver=letsencrypt"       # Auto SSL certificates
  - "traefik.http.services.frontend-app.loadbalancer.server.port=3000"     # Internal port
```

## Dockerfile Details

The production Dockerfile uses:
- **Multi-stage build**: Optimizes image size by separating build and runtime stages
- **Build Stage (Node 22 Alpine)**: Builds the React app with pnpm
- **Production Stage (nginx Alpine)**: Ultra-lightweight production server
- **nginx**: Industry-standard, high-performance web server
- **Port 3000**: Configured to match Dokploy's expected port
- **Built-in health checks**: Automatic container health monitoring at `/health` endpoint

### Why nginx over serve?

1. **Performance**: nginx is significantly faster and more efficient at serving static files
2. **Production-Ready**: Battle-tested in production environments worldwide
3. **Advanced Features**:
   - Gzip compression for faster load times
   - Security headers (X-Frame-Options, X-Content-Type-Options, etc.)
   - Aggressive caching for static assets (1 year cache with immutable flag)
   - Client-side routing support (for React Router, etc.)
4. **Resource Efficient**: Lower memory footprint than Node.js-based servers
5. **Health Monitoring**: Built-in health check endpoint at `/health`

## Monitoring and Logs

In Dokploy dashboard:
- **Monitoring Tab**: View CPU, memory, and network usage
- **Logs Tab**: Access real-time and historical logs
- **Deployments Tab**: View deployment history and status

## Troubleshooting

### Domain Not Working
- Verify DNS A record points to your server's IP
- Check that ports 80 and 443 are open on your firewall
- Wait 10 seconds after deployment for SSL certificate generation

### Build Failures
- Check deployment logs in Dokploy
- Ensure all dependencies are properly listed in `package.json`
- Verify Node version compatibility (using Node 22)

### Container Not Starting
- Review container logs in the Logs tab
- Check for port conflicts
- Verify environment variables if any were added

## Production Best Practices

1. **Resource Limits**: Consider adding resource limits in `docker-compose.yml`:
   ```yaml
   deploy:
     resources:
       limits:
         memory: 512M
         cpus: '0.5'
   ```

2. **Health Checks**: Add health check endpoint to your app and configure in Dockerfile:
   ```dockerfile
   HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
     CMD wget --no-verbose --tries=1 --spider http://localhost:3000/ || exit 1
   ```

3. **Environment Variables**: Use Dokploy's Environment tab to manage secrets and configuration

## CI/CD Integration

For automated deployments:

1. Go to **Deployments** tab in Dokploy
2. Copy the **Webhook URL**
3. Add webhook to your Git provider (GitHub/GitLab/Gitea)
4. Push to your repository to trigger automatic deployments

## Additional Resources

- [Dokploy Documentation](https://docs.dokploy.com)
- [Docker Compose Documentation](https://docs.dokploy.com/docs/core/docker-compose)
- [Traefik Labels Guide](https://docs.dokploy.com/docs/core/docker-compose/domains)