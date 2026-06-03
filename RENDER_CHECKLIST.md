# Render Deployment Checklist

## Pre-Deployment

- [ ] Ensure your Piston repository is on GitHub/GitLab
- [ ] Have a Render account created (https://render.com)
- [ ] Verify you have `render.yaml` in the repository root
- [ ] Review environment variables in `.env.render.example`

## Initial Deployment

- [ ] Connect your Git repository to Render
- [ ] Select "Blueprint" as deployment method
- [ ] Render will auto-detect `render.yaml`
- [ ] Confirm the service configuration
- [ ] Click "Deploy"
- [ ] Wait for build to complete (5-15 minutes initially)

## Post-Deployment Setup

- [ ] Verify service is running (check Logs)
- [ ] Test health endpoint: `curl https://your-service.onrender.com/`
- [ ] Install language runtimes via package manager
  - Visit `/api/v2/runtimes` to see installed runtimes
  - Use POST endpoint to install new languages
- [ ] (Optional) Add persistent storage disk for `/piston` if needed
- [ ] (Optional) Set up custom domain

## Configuration

- [ ] Review environment variables in Service Settings
- [ ] Adjust `PISTON_MAX_CONCURRENT_JOBS` based on your plan
- [ ] Set appropriate timeout values for your use case
- [ ] Enable networking if required: `PISTON_DISABLE_NETWORKING=false`

## Monitoring

- [ ] Set up alerts for deployment failures
- [ ] Monitor error logs for startup issues
- [ ] Watch disk usage (check Analytics)
- [ ] Test API endpoints under load

## Troubleshooting

- [ ] Check build logs if deployment fails
- [ ] Verify Docker image builds locally: `docker build -f api/Dockerfile .`
- [ ] Review cgroup requirements in `api/src/docker-entrypoint.sh`
- [ ] Check PORT environment variable matches service configuration

## Security Considerations

- [ ] Verify `PISTON_DISABLE_NETWORKING=true` for untrusted code
- [ ] Review UID/GID range configuration for sandbox
- [ ] Test that code execution is properly isolated
- [ ] Implement API authentication if exposing to public
- [ ] Consider rate limiting for production deployments

## Performance Tuning

- [ ] Monitor concurrent job limit effectiveness
- [ ] Adjust timeout values based on actual usage
- [ ] Consider upgrading Render plan if hitting resource limits
- [ ] Review memory limits for compiled languages

## Backup & Disaster Recovery

- [ ] Document installed language packages
- [ ] Export Render service configuration
- [ ] Keep render.yaml and environment config backed up
- [ ] Plan for disk snapshot/backup strategy

## Production Readiness

- [ ] Set up monitoring/alerting integration
- [ ] Implement CI/CD for automated deployments
- [ ] Document scaling strategy
- [ ] Plan capacity for expected load
- [ ] Set up SSL/TLS (automatic with Render)
