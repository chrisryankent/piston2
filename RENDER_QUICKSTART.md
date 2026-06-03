# Render Deployment Quick Start

Deploy Piston to Render in 5 minutes!

## Step 1: Prepare Your Repository

```bash
# Ensure all files are committed
git status
git add .
git commit -m "Add Render deployment configuration"
git push origin master
```

## Step 2: Connect to Render

1. Visit https://render.com
2. Sign up or log in
3. Click **New +** → **Blueprint**
4. Connect your GitHub/GitLab repository
5. Authorize Render to access your repo

## Step 3: Configure & Deploy

1. Render auto-detects `render.yaml`
2. Review the configuration
3. Click **Deploy**
4. Wait 5-15 minutes for build to complete

## Step 4: Verify Deployment

```bash
# Replace with your service name
RENDER_URL="https://piston-api-xxxxx.onrender.com"

# Test health endpoint
curl $RENDER_URL/

# List available runtimes
curl $RENDER_URL/api/v2/runtimes

# Execute sample code
curl -X POST $RENDER_URL/api/v2/execute \
  -H "Content-Type: application/json" \
  -d '{
    "language": "python",
    "version": "*",
    "files": [{
      "name": "main.py",
      "content": "print(\"Hello from Render!\")"
    }]
  }'
```

## Step 5: (Optional) Install Language Runtimes

Use the package manager endpoint:

```bash
curl -X POST $RENDER_URL/api/v2/packages/install \
  -H "Content-Type: application/json" \
  -d '{
    "language": "python",
    "version": "3.10"
  }'
```

Or see `RENDER_DEPLOYMENT.md` for detailed setup.

## Files Created for Render Deployment

- **`render.yaml`** - Blueprint configuration (auto-deployed)
- **`RENDER_DEPLOYMENT.md`** - Complete deployment guide
- **`RENDER_CHECKLIST.md`** - Pre/post deployment checklist
- **`RENDER_TROUBLESHOOTING.md`** - Common issues & solutions
- **`.env.render.example`** - Environment variables template

## Next Steps

1. ✅ Deployed? Visit **Logs** to monitor startup
2. ❓ Issues? Check `RENDER_TROUBLESHOOTING.md`
3. 🔧 Configure? Review `RENDER_DEPLOYMENT.md`
4. 📋 Setup checklist? Use `RENDER_CHECKLIST.md`

## Key Information

| Item | Value |
|------|-------|
| Service Name | piston-api |
| Docker Image | `./api/Dockerfile` |
| Port | 2000 |
| Health Check | GET `/` |
| Build Time | 5-15 minutes |
| Runtime | Docker with cgroup v2 support |

## Monitoring Dashboard

After deployment, access your service dashboard at:
```
https://dashboard.render.com/services
```

Monitor:
- Logs (real-time output)
- Metrics (CPU, memory, network)
- Deploys (deployment history)
- Settings (environment, disk, etc.)

## Support

- 📚 Full docs: `RENDER_DEPLOYMENT.md`
- 🐛 Issues: `RENDER_TROUBLESHOOTING.md`
- 💬 Questions: https://discord.gg/engineerman
- 🔗 Render Help: https://render.com/docs
