# Railway Deployment Guide

This document describes how to deploy this code-server instance on Railway and manage the deployment.

## Overview

This project is configured to deploy code-server (VS Code in the browser) on Railway using Docker. The setup is optimized for Railway's infrastructure and follows deployment best practices.

## How Railway Deployment Works

### Port Binding
Railway automatically provides a `PORT` environment variable that your application must bind to. This project:
- Uses Railway's `$PORT` environment variable automatically
- Falls back to port 8080 for local development
- Binds to `0.0.0.0` (all interfaces) as required by Railway

### Container Configuration
- Uses the official `codercom/code-server:latest` base image
- Runs as non-root user (`coder`) for security
- Includes a simple `start.sh` script for clarity and portability
- Copies project files to `/home/coder/workspace`

### Railway Configuration
The `railway.toml` file is minimal and only specifies:
- Docker builder usage
- No custom healthchecks (Railway handles this automatically)
- No custom environment variables (uses Railway defaults)

## Deployment Steps

### Initial Deployment
1. Connect your GitHub repository to Railway
2. Railway will automatically detect the `Dockerfile` and `railway.toml`
3. The deployment will build and start automatically
4. Railway will provide a public URL once deployment is complete

### Updating the Service Domain
You can customize your service domain through the Railway dashboard:
1. Go to your Railway project dashboard
2. Select your service
3. Go to the "Settings" tab
4. Under "Domains", you can:
   - Use the auto-generated Railway domain
   - Add a custom domain
   - Configure custom domain settings

## Environment Variables

### Automatic Variables
Railway automatically provides:
- `PORT` - The port your application should bind to
- `RAILWAY_SERVICE_NAME` - Name of your Railway service
- `RAILWAY_ENVIRONMENT` - Current environment (production/staging)

### Custom Variables (Optional)
You can add environment variables through the Railway dashboard:
1. Go to service settings
2. Navigate to "Variables" tab
3. Add any custom environment variables needed

## Local Development

To run locally and test the Railway configuration:

```bash
# Set PORT for testing (optional, defaults to 8080)
export PORT=3000

# Run with Docker
docker build -t gide-local .
docker run -p 3000:3000 -e PORT=3000 gide-local

# Or run the start script directly (if code-server is installed)
./start.sh
```

## Security Considerations

- Container runs as non-root user (`coder`)
- Authentication is enabled with password protection
- Only necessary files are copied to the container
- Uses official, maintained base image

## Troubleshooting

### Common Issues

1. **Port binding errors**: Ensure the application binds to `0.0.0.0:$PORT`
2. **Startup failures**: Check Railway logs for specific error messages
3. **File permission issues**: Ensure proper ownership is set in Dockerfile

### Checking Logs
View deployment logs in Railway:
1. Go to your service in Railway dashboard
2. Click on "Deployments" tab
3. Click on a specific deployment to view logs

### Local Testing
Test the Docker configuration locally:
```bash
# Build the image
docker build -t gide-test .

# Run with Railway-like environment
docker run -p 8080:8080 -e PORT=8080 gide-test
```

## Gide Coding Agent Extension

This deployment includes the Gide Coding Agent extension which requires additional configuration.

### Environment Variables

The extension requires the following environment variables to function properly:

#### Required Variables
- `GIDE_AGENT_ENDPOINT`: URL of your Railway-hosted coding agent service
  - Example: `https://your-agent-service.railway.app/api/chat`

#### Optional Variables
- `GIDE_API_KEY`: API key for agent authentication
- `GIDE_MODEL_PROVIDER`: AI model provider (`openai`, `anthropic`, etc.)
- `GIDE_MODEL_NAME`: Specific model name (`gpt-4`, `claude-3-sonnet`, etc.)
- `GIDE_REQUEST_TIMEOUT`: Request timeout in milliseconds (default: 30000)

### Railway Configuration

Set these variables in your Railway service:

```bash
# Using Railway CLI
railway variables set GIDE_AGENT_ENDPOINT="https://your-agent-service.railway.app/api/chat"
railway variables set GIDE_API_KEY="your_api_key_here"
railway variables set GIDE_MODEL_PROVIDER="openai"
railway variables set GIDE_MODEL_NAME="gpt-4"
```

Or through the Railway Dashboard:
1. Go to your service in Railway dashboard
2. Click on "Variables" tab
3. Add each environment variable

### Building the Extension

The extension is built automatically during deployment, but you can build it manually:

```bash
# Build the Gide extension
npm run build-gide-extension

# Or build directly
cd extensions/gide-coding-agent
yarn install
yarn build
```

### Extension Security

- Never hardcode API keys in VSCode settings
- Use environment variables for all sensitive configuration
- The extension prioritizes environment variables over VSCode settings
- All configuration is validated on startup with clear error messages

### Troubleshooting Extension Issues

1. **"Missing required environment variables" error**:
   - Ensure `GIDE_AGENT_ENDPOINT` is set in Railway variables
   - Check that the URL is properly formatted with protocol

2. **"Connection Failed" error**:
   - Verify your agent service is running and accessible
   - Check that `GIDE_API_KEY` is correct (if required)
   - Test the endpoint directly with curl

3. **Extension not loading**:
   - Check Railway deployment logs for build errors
   - Ensure the extension was built successfully
   - Verify all dependencies were installed with yarn

## Best Practices

1. **Port Configuration**: Always use Railway's `$PORT` variable
2. **Interface Binding**: Bind to `0.0.0.0`, not `localhost` or `127.0.0.1`
3. **Minimal Configuration**: Let Railway handle healthchecks and process management
4. **Security**: Run containers as non-root users when possible
5. **Resource Optimization**: Keep Docker images lean and start times fast
6. **Environment Variables**: Use Railway's environment variables for all configuration
7. **Extension Security**: Never commit API keys or secrets to version control
8. **Extension Build**: Ensure extensions are built before deployment
9. **Dependency Management**: Use yarn for extension dependencies as specified

## Links and References

- [Railway Docs on Docker](https://docs.railway.app/deploy/docker)
- [code-server documentation](https://github.com/coder/code-server/blob/main/docs/using-code-server.md)
- [Railway Environment Variables](https://docs.railway.app/develop/variables)
- [Railway Custom Domains](https://docs.railway.app/deploy/railway-app#custom-domains)