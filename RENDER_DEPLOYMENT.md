# Render Deployment Guide

This guide explains how to deploy Piston to Render.

## Prerequisites

- A [Render account](https://render.com)
- Your Piston repository connected to GitHub/GitLab
- Render Blueprint features enabled

## Deployment Steps

### 1. Create a New Service on Render

1. Go to [Render Dashboard](https://dashboard.render.com)
2. Click **New +** → **Blueprint**
3. Connect your GitHub/GitLab repository containing this Piston project
4. Authorize Render to access your repository

### 2. Configure the Deployment

The project includes a `render.yaml` blueprint file that automatically configures:

- **Service Type**: Web Service
- **Runtime**: Docker
- **Docker Image**: Built from `./api/Dockerfile`
- **Port**: 2000
- **Health Check**: / endpoint
- **Region**: Oregon (configurable)

### 3. Environment Variables

Customize these environment variables in the Render dashboard under Service Settings → Environment:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | 2000 | HTTP server port |
| `PISTON_LOG_LEVEL` | INFO | Logging level (DEBUG, INFO, WARN, ERROR) |
| `PISTON_DATA_DIRECTORY` | /piston | Data directory for packages |
| `PISTON_DISABLE_NETWORKING` | true | Sandbox networking isolation |
| `PISTON_MAX_CONCURRENT_JOBS` | 64 | Concurrent execution limit |
| `PISTON_RUN_TIMEOUT` | 3000 | Max runtime in milliseconds |
| `PISTON_COMPILE_TIMEOUT` | 10000 | Max compile time in milliseconds |

For a full list of configuration options, see `api/src/config.js`.

### 4. Important Notes

#### Persistent Storage

Piston stores language packages in `/piston/packages`. For persistent data across deployments:

1. In Render dashboard, go to Service Settings
2. Add a **Disk** with:
   - **Mountpoint**: `/piston`
   - **Size**: At least 20GB (depends on language count)
3. This preserves installed packages between redeployments

#### Privileged Container Access

Piston requires privileged container mode to use cgroup v2 for sandboxing. The Docker deployment on Render supports this automatically.

#### First Run Setup

On first deployment, you'll need to install language runtimes. You can:

1. Use the admin CLI on your deployed instance:
   ```bash
   render exec piston-api -- piston install python 3.10
   ```

2. Or use the package management API endpoints (see `docs/api-v2.md`)

#### Build Time

Initial build may take 5-10 minutes due to:
- Installing language dependencies (libxml2, build-essential, etc.)
- Building and installing isolate sandbox tool
- npm dependency installation

## Accessing Your Deployment

Once deployed, your Piston API will be available at:

```
https://your-service-name.onrender.com/api/v2
```

### Test the Deployment

```bash
curl https://your-service-name.onrender.com/
# Response: {"message":"Piston v3.1.1"}

curl https://your-service-name.onrender.com/api/v2/runtimes
# Lists all installed language runtimes
```

## Monitoring

### Logs

View live logs in Render dashboard:
- Service → Logs

### Health Checks

Render automatically monitors the `/` health check endpoint every 30 seconds.

### Metrics

Monitor CPU, memory, and network usage in Render dashboard under Service Analytics.

## Troubleshooting

### Build Failures

- **Docker build timeout**: Increase timeout in `render.yaml` or use Render's Pro plan
- **Out of memory during build**: Language dependencies are memory-intensive
- Check build logs in Render dashboard for detailed errors

### Runtime Issues

- **Port binding error**: Verify `PORT` environment variable matches the service port (2000)
- **Cgroup errors**: This indicates the container runtime doesn't support cgroup v2
- **Missing language runtimes**: Install via package manager endpoint or rebuild with pre-installed runtimes

### Disk Full

- Monitor disk usage in Analytics
- Increase disk size in Service Settings
- Consider which language packages are essential

## Advanced Configuration

### Custom Domain

1. In Render dashboard: Service → Settings
2. Add custom domain under "Custom Domains"
3. Update DNS records as directed

### Environment-Specific Settings

Create separate `render.yaml` files for development/production:

```yaml
# render-prod.yaml
services:
  - name: piston-api
    plan: starter
    region: oregon
    # ... other config
```

Deploy with: **Blueprint → Select file → render-prod.yaml**

### Autoscaling

Note: Render's starter plan doesn't support autoscaling. Upgrade to Standard or Pro for:
- Concurrent instance scaling
- Load balancing
- Better resource allocation

## Rolling Back

To rollback to a previous deployment:

1. Render dashboard → Service → Deploys
2. Click the previous deployment
3. Click **Redeploy**

## Additional Resources

- [Piston Documentation](https://piston.readthedocs.io)
- [Piston API Reference](./docs/api-v2.md)
- [Render Documentation](https://render.com/docs)
- [GitHub Repository](https://github.com/engineer-man/piston)
