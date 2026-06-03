# Render Deployment: Common Issues & Solutions

## Build Issues

### "Docker build timed out"
- **Cause**: Build taking longer than timeout limit
- **Solution**: 
  - Use Render's Pro plan for longer timeouts
  - Pre-build Docker image and push to Docker Hub
  - Split multi-stage build differently

### "Build fails with 'permission denied'"
- **Cause**: Dockerfile trying to write to read-only filesystem
- **Solution**:
  - Ensure `WORKDIR /piston_api` exists
  - Check `RUN mkdir` commands have proper permissions
  - Verify `RUN chown` commands are correct

### "Out of memory during build"
- **Cause**: Language dependencies require significant memory
- **Solution**:
  - Upgrade Render plan to allocate more build resources
  - Increase build concurrency timeout in render.yaml
  - Consider building custom image locally and pushing to registry

### "npm install fails with package not found"
- **Cause**: Package versions outdated or network issue
- **Solution**:
  ```bash
  # Locally update and commit
  cd api
  rm package-lock.json
  npm install
  git add package-lock.json
  git commit -m "Update npm dependencies"
  git push
  ```

## Runtime Issues

### "Port already in use"
- **Cause**: PORT environment variable conflicts or dual binding
- **Solution**:
  - Check `render.yaml` port setting (should be 2000)
  - Verify `PISTON_BIND_ADDRESS` env var isn't set
  - Ensure only one service instance in render.yaml

### "Service keeps crashing with cgroup errors"
- **Cause**: Container runtime doesn't support cgroup v2
- **Solution**:
  - This is a Render platform limitation
  - Contact Render support to enable cgroup v2
  - Consider running on Render Docker plan (should support it)
  - Alternative: Self-host on Docker-compatible platform

### "Cannot find /sys/fs/cgroup"
- **Cause**: Docker not configured with cgroup v2
- **Solution**:
  - Verify Render is using Docker runtime (not just standard runtime)
  - Check that `privileged: true` permission is set
  - See error in "cgroup v2 not found" for more details

### "/piston directory not found"
- **Cause**: Missing volume/disk setup
- **Solution**:
  - Add persistent disk in Render dashboard:
    - Service → Settings → Disks
    - Add disk with mountpoint `/piston`
  - Or ensure `/piston` is created in Dockerfile

### "Language packages not installed"
- **Cause**: First deployment or missed installation step
- **Solution**:
  - Check `/api/v2/runtimes` to see installed runtimes
  - Use package manager to install:
    ```bash
    curl -X POST https://your-service.onrender.com/api/v2/packages/install \
      -H "Content-Type: application/json" \
      -d '{"language":"python","version":"3.10"}'
    ```
  - Or create custom Docker image with pre-installed packages

## Performance Issues

### "API responding slowly"
- **Cause**: Under-resourced plan or too many concurrent jobs
- **Solution**:
  - Reduce `PISTON_MAX_CONCURRENT_JOBS` if CPU-bound
  - Increase Render plan to allocate more CPU/memory
  - Monitor actual usage in Analytics dashboard
  - Check if jobs timing out due to short limits

### "High memory usage leading to OOM"
- **Cause**: Memory limits set too high or memory leak
- **Solution**:
  - Reduce `PISTON_RUN_MEMORY_LIMIT` to enforce limits
  - Check application logs for memory growth patterns
  - Upgrade Render plan for more available memory
  - Restart service to reset memory

### "Disk full errors"
- **Cause**: Too many language packages or tmp files accumulating
- **Solution**:
  - Monitor disk usage in Analytics
  - Increase disk size in Service Settings → Disks
  - Clean up unused language packages
  - Set `tmpfs` size limits if using ephemeral storage

## Deployment Issues

### "Service won't start after deployment"
- **Cause**: Configuration errors or missing dependencies
- **Solution**:
  - Check Live Logs for error messages
  - Verify all required environment variables are set
  - Test Dockerfile locally: `docker build -f api/Dockerfile . && docker run -it --name piston <image-id>`
  - Ensure `/piston` directory exists or has persistent disk

### "Redeploy not picking up changes"
- **Cause**: Cache or uncommitted changes
- **Solution**:
  - Ensure all changes are committed and pushed to Git
  - Render auto-deploys on push (if autoDeploy: true)
  - Manual redeploy in dashboard → Service → Manual Deploy
  - Check build logs to confirm new code is being used

### "Connection timeout to health check"
- **Cause**: Service not starting within health check timeout
- **Solution**:
  - Increase `healthCheckInterval` in render.yaml if needed
  - Check startup logs for slow initialization
  - Reduce initial startup work or optimize dependencies
  - Increase Render plan resources

## Data & Persistence Issues

### "Data lost after redeployment"
- **Cause**: No persistent disk configured
- **Solution**:
  - Add persistent disk in Render dashboard
  - Mountpoint: `/piston` with adequate size
  - Redeploy service
  - Data will now persist between deployments

### "Disk fills up with old package versions"
- **Cause**: Multiple versions of packages installed
- **Solution**:
  - Implement version cleanup process
  - Or increase disk size and accept space usage
  - Remove unused language package versions

## Debugging

### Enable Debug Logging
```bash
# In Render dashboard: Service Settings → Environment
PISTON_LOG_LEVEL=DEBUG
# Redeploy and check logs
```

### Access Service Shell (if available)
```bash
# Via Render CLI or dashboard
render exec piston-api -- /bin/bash

# Check disk space
df -h
# List installed packages
ls -la /piston/packages
# View logs
tail -f /var/log/render-service.log
```

### Local Testing
```bash
# Build and test locally before deploying
docker build -f api/Dockerfile -t piston-local .
docker run -it \
  -p 2000:2000 \
  --privileged \
  -e PISTON_DATA_DIRECTORY=/piston \
  piston-local
```

## Getting Help

1. **Render Support**: render.com/support
2. **Piston Issues**: https://github.com/engineer-man/piston/issues
3. **Piston Discord**: https://discord.gg/engineerman
4. **Check this guide first**: RENDER_DEPLOYMENT.md
