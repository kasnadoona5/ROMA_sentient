# 🚀 ROMA v0.2.0 Complete Installation Guide (Optimized to use free API)

A battle-tested guide for ROMA v0.2.0-beta with all known issues pre-solved

## ⚠️ Critical Version Notice

This guide is for **ROMA v0.2.0-beta (October 2025)**, which is a complete rewrite:

- Built on Stanford's DSPy framework
- OmegaConf profile-based configuration
- Production-ready Docker stack (MinIO, REST API)
- File-based storage only (no PostgreSQL complications)
- New module system (Atomizer, Planner, Executor, Aggregator)
- Web search capabilities with Exa integration (correct v0.2.0 format)

## 📋 Prerequisites

- VPS or local Linux server (Ubuntu 22.04+ recommended)
- Docker Compose V2 (plugin-based, installed via docker-ce)
- Root or sudo access
- OpenRouter account (free tier: https://openrouter.ai/)
- Exa.ai account (free tier: https://exa.ai/)
- 4GB+ RAM, 20GB+ disk space
- Open ports: 8000 (API), 9001 (MinIO console, optional)

## 🛠️ Part 1: System Preparation

### Step 1: Connect and Update System

```bash
ssh root@<your_vps_ip>
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Docker Compose V2

```bash
# Install essential packages
sudo apt install git ca-certificates curl -y

# Add Docker's official GPG key and repository
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker with Compose V2 plugin
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Verify installation (should show Docker Compose version v2.x.x)
docker compose version
```

### Step 3: Create docker-compose V1 Compatibility Symlink

```bash
# Create wrapper script for V1 compatibility
sudo cat > /usr/local/bin/docker-compose << 'EOF'
#!/bin/bash
docker compose "$@"
EOF

sudo chmod +x /usr/local/bin/docker-compose

# Verify both work
docker-compose version
docker compose version
```

### Step 4: Install Just Command Runner (Optional)

```bash
# Download and install Just
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin

# Verify installation
just --version
```

## 🔧 Part 2: ROMA Installation & Configuration

### Step 5: Clone and Checkout v0.2.0-beta

```bash
cd ~
git clone https://github.com/sentient-agi/ROMA.git
cd ROMA

# Checkout the stable v0.2.0-beta release
git checkout v0.2.0-beta

# Verify you're on the correct version
git describe --tags
# Should output: v0.2.0-beta
```

### Step 6: Create Required Directories with Correct Permissions

**CRITICAL FIX:** The container runs as uid=1000 (roma user), so we must pre-create directories with correct ownership:

```bash
cd ~/ROMA

# Create all required directories
mkdir -p data/executions
mkdir -p .cache/dspy
mkdir -p logs
mkdir -p config/profiles

# Set ownership to uid 1000 (container user)
chown -R 1000:1000 data
chown -R 1000:1000 .cache
chown -R 1000:1000 logs
chown -R 1000:1000 config

# Set proper permissions
chmod -R 755 data .cache logs config

# Verify permissions
ls -la | grep -E "data|cache|logs|config"
# Should show: drwxr-xr-x ... 1000 1000 ... for each directory
```

### Step 7: Configure Environment Variables

```bash
cd ~/ROMA
nano .env
```

Add your API keys:

```bash
# ===== REQUIRED API KEYS =====

# OpenRouter API Key (get free key from https://openrouter.ai/)
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Exa API Key (get free key from https://exa.ai/)
# Free tier: 1,000 searches/month
EXA_API_KEY=your_exa_api_key_here
```

**Save and exit:** Press `Ctrl+X`, then `Y`, then `Enter`

### Step 8: Create Free-Tier Profile with Correct Exa Configuration

```bash
cd ~/ROMA

cat > config/profiles/free_tier.yaml << 'EOF'
# Free-Tier Optimized Profile for ROMA v0.2.0
# Based on General.yaml but optimized for free tier with Exa search

# CRITICAL: Runtime depth control
runtime:
  max_depth: 1

# Orchestration settings
orchestration:
  max_execution_steps: 7
  max_recursion_depth: 1
  enable_checkpoints: true
  checkpoint_interval: 3
  parallel_execution: false
  max_parallel_tasks: 1

# Agent configurations
agents:
  # Atomizer: Fast decision-making
  atomizer:
    llm:
      model: "openrouter/z-ai/glm-4.5-air:free"
      temperature: 0.4              
      max_tokens: 8000              
      cache: true
    prediction_strategy: "ChainOfThought"
    enabled: true

  # Planner: Task decomposition
  planner:
    llm:
      model: "openrouter/z-ai/glm-4.5-air:free"
      temperature: 0.4              
      max_tokens: 10000              
      cache: true
    prediction_strategy: "ChainOfThought"
    enabled: true
    agent_config:
      max_subtasks: 1               # Keep: Limit recursion for free tier

  # Executor: Handles tasks with tools
  executor:
    llm:
      model: "openrouter/z-ai/glm-4.5-air:free"
      temperature: 0.6              
      max_tokens: 8000              
      cache: true
    prediction_strategy: "react"    # Keep: Needed for tool use
    enabled: true
    agent_config:
      max_executions: 1             # Keep: Single execution only
    
    toolkits:
      - class_name: FileToolkit
        enabled: true
        toolkit_config:
          max_file_size_mb: 10

      - class_name: CalculatorToolkit
        enabled: true

      - class_name: MCPToolkit
        enabled: true
        toolkit_config:
          server_name: exa
          server_type: http
          url: https://mcp.exa.ai/mcp
          headers:
            Authorization: Bearer ${oc.env:EXA_API_KEY}
          transport_type: streamable
          use_storage: false
          tool_timeout: 60

  # Aggregator: Synthesis
  aggregator:
    llm:
      model: "openrouter/z-ai/glm-4.5-air:free"
      temperature: 0.0              
      max_tokens: 10000              
      cache: true
    prediction_strategy: "ChainOfThought"
    enabled: true

  # Verifier: Validation
  verifier:
    llm:
      model: "openrouter/z-ai/glm-4.5-air:free"
      temperature: 0.0
      max_tokens: 10000              
      cache: true
    prediction_strategy: "Predict"  # CHANGE: Use Predict (just validate)
    enabled: false                  # Keep: Disabled for free tier

# Storage configuration
storage:
  base_path: "./data/executions"
  parquet_threshold_kb: 100
  enable_s3: false
  cleanup_old_executions: true
  retention_days: 30

# Observability
observability:
  enable_mlflow: false
  mlflow_tracking_uri: "http://localhost:5000"
  log_level: "INFO"
  track_token_usage: true
  track_cost: true

# Performance tuning - Match General but conservative
performance:
  enable_async: true
  request_timeout: 120
  retry_attempts: 1                
  retry_delay: 5                   
  enable_circuit_breaker: true

# Resilience - Match General but conservative
resilience:
  retry:
    enabled: true
    max_attempts: 1                
    strategy: exponential_backoff
    base_delay: 2.0
    max_delay: 10.0                

  circuit_breaker:
    enabled: true
    failure_threshold: 2           
    recovery_timeout: 120.0         
    half_open_max_calls: 1         

  checkpoint:
    enabled: true
    storage_path: ${oc.env:ROMA_CHECKPOINT_PATH,.checkpoints}
    max_checkpoints: 5            
    max_age_hours: 24.0            
    compress_checkpoints: true
    verify_integrity: true

# Cost tracking
cost_tracking:
  enabled: true
  warn_threshold_usd: 0.0
  stop_threshold_usd: 0.05
EOF

chown 1000:1000 config/profiles/free_tier.yaml
echo "✅ Improved free-tier profile created (based on General.yaml)!"


```

### Step 9: Verify Docker Compose Configuration

```bash
cd ~/ROMA

# Check if docker-compose file exists
ls -la docker-compose.y*

# View volume configuration
cat docker-compose.yaml | grep -A 20 "roma-api:" | grep -A 10 "volumes:"

# Should show volume mounts like:
#   - ./data:/app/data
#   - ./.cache:/app/.cache
#   - ./config:/app/config
```

## 🐳 Part 3: Docker Deployment

### Step 10: Build and Start ROMA Services

```bash
cd ~/ROMA

# Start services (MinIO + API only - NO PostgreSQL)
docker compose up -d --build

# Wait for services to initialize
echo "Waiting 20 seconds for services to start..."
sleep 20
```

### Step 11: Verify All Services Are Running

```bash
cd ~/ROMA

# Check container status
docker compose ps

# Expected output:
# NAME               IMAGE              STATUS
# roma-dspy-api      roma-roma-api      Up (healthy)
# roma-dspy-minio    minio/minio        Up (healthy)

# Test API health
curl http://localhost:8000/health

# Expected response: {"status":"healthy","version":"0.1.0",...}
```

### Step 12: Verify Volume Mounts Inside Container

**CRITICAL CHECK:** Ensure directories are mounted and writable:

```bash
cd ~/ROMA

# Check what user the container runs as
docker compose exec roma-api id
# Should show: uid=1000(roma) gid=1000(roma)

# Verify volumes are mounted
docker compose exec roma-api ls -la /app/ | grep -E "data|cache|config"
# Should show directories, NOT "No such file or directory"

# Test write permissions
docker compose exec roma-api touch /app/data/test_write.txt
docker compose exec roma-api ls -la /app/data/test_write.txt
# Should succeed without "Permission denied"

echo "✅ Volume mounts verified successfully"
```

If you get errors:

```bash
cd ~/ROMA

# Stop containers
docker compose down

# Fix ownership again
chown -R 1000:1000 data .cache logs config
chmod -R 755 data .cache logs config

# Restart
docker compose up -d

# Wait and test again
sleep 15
docker compose exec roma-api ls -la /app/data
```

## ✅ Part 4: Testing Your Installation

### Step 13: Create Helper Script

```bash
cd ~/ROMA

# Create quick task execution script
cat > run_task.sh << 'EOF'
#!/bin/bash
TASK="$1"
PROFILE="${2:-free_tier}"

if [ -z "$TASK" ]; then
    echo "Usage: ./run_task.sh 'Your task description' [profile]"
    echo "Example: ./run_task.sh 'What is machine learning?' free_tier"
    exit 1
fi

echo "🚀 Executing: $TASK"
docker compose exec roma-api roma-dspy solve "$TASK" --profile "$PROFILE"
EOF

chmod +x run_task.sh

echo "✅ Helper script created!"
```

### Step 14: Test Simple Task Execution

```bash
cd ~/ROMA

# Test with a simple arithmetic task
./run_task.sh "What is 2+2?" free_tier

# Expected: Quick response with answer "2 + 2 equals 4"
```

### Step 15: Test Web Search with Exa

```bash
cd ~/ROMA

# Test Exa search - this uses real web search
./run_task.sh "What are the latest developments in AI in November 2025?" free_tier

# Test another search
./run_task.sh "Find information about quantum computing breakthroughs" free_tier
```

Check logs to confirm Exa is working:

```bash
docker compose logs -f roma-api | grep -i "exa\|mcp\|search"

# You should see:
# - MCPToolkit initialized with server_name: exa
# - Exa search queries being executed
# - Search results being returned
```

### Step 16: Test via REST API

```bash
# Create an execution
curl -X POST "http://localhost:8000/api/v1/executions" \
  -H "Content-Type: application/json" \
  -d '{
    "task": "What is the latest news about AI agents?",
    "profile": "free_tier"
  }'

# Note the execution_id from response

# Check status (replace <execution_id>)
curl "http://localhost:8000/api/v1/executions/<execution_id>"
```

## 📊 Part 5: Monitoring & Management

### View Logs

```bash
cd ~/ROMA

# All services
docker compose logs -f

# Just API
docker compose logs -f roma-api

# Filter for Exa search activity
docker compose logs roma-api | grep -i "exa\|mcp\|search"
```

### Check Resource Usage

```bash
# Container resource usage
docker stats

# Disk usage
du -sh ~/ROMA/data
df -h
```

### Restart Services

```bash
cd ~/ROMA

# Restart all
docker compose restart

# Restart specific service
docker compose restart roma-api

# Full rebuild
docker compose down
docker compose up -d --build
```

### Stop Services

```bash
cd ~/ROMA

# Stop (preserves data)
docker compose down

# Stop and remove volumes (WARNING: deletes all data)
docker compose down -v
```

## 🆘 Complete Troubleshooting Guide

### Issue 1: "docker-compose: not found"

```bash
# Create V1 compatibility symlink
sudo cat > /usr/local/bin/docker-compose << 'EOF'
#!/bin/bash
docker compose "$@"
EOF
sudo chmod +x /usr/local/bin/docker-compose
```

### Issue 2: "Permission denied: 'data'"

```bash
cd ~/ROMA

# Method 1: Fix on host
chown -R 1000:1000 data .cache logs
chmod -R 755 data .cache logs
docker compose restart roma-api

# Method 2: Fix inside container
docker compose exec -u root roma-api chmod -R 777 /app/data /app/.cache
```

### Issue 3: "No such file or directory: /app/data"

```bash
cd ~/ROMA

# Volumes aren't mounted - check docker-compose.yaml
cat docker-compose.yaml | grep -A 10 "volumes:"

# If missing, edit and add them
nano docker-compose.yaml

# Then rebuild
docker compose down
docker compose up -d --build
```

### Issue 4: Exa Search Not Working

```bash
# Verify API key is set
docker compose exec roma-api bash
echo $EXA_API_KEY
exit

# Check if MCPToolkit is registered
docker compose logs roma-api | grep -i "mcp\|toolkit"

# Verify profile has MCPToolkit enabled
cat config/profiles/free_tier.yaml | grep -A 8 "MCPToolkit"
```

### Issue 5: Container Keeps Restarting

```bash
# Check container logs
docker compose logs --tail=50 roma-api

# Common causes:
# 1. Port 8000 already in use
sudo netstat -tlnp | grep 8000

# 2. Out of memory
docker stats
free -h
```

### Full Diagnostic Script

```bash
cd ~/ROMA

echo "=== ROMA Diagnostic Report ==="
echo ""
echo "1. Container Status:"
docker compose ps
echo ""
echo "2. Container User:"
docker compose exec roma-api id
echo ""
echo "3. Volume Mounts:"
docker compose exec roma-api df -h | grep app
echo ""
echo "4. Directory Permissions (Host):"
ls -la | grep -E "data|cache|logs"
echo ""
echo "5. API Health:"
curl -s http://localhost:8000/health | jq .
echo ""
echo "6. Recent API Logs:"
docker compose logs --tail=20 roma-api
echo ""
echo "=== End Diagnostic Report ==="
```

## 📈 Performance Optimization Tips

### 1. Reduce Token Usage

In your profile:

```yaml
agents:
  executor:
    llm:
      max_tokens: 6000  # Reduce from 8000
  aggregator:
    llm:
      max_tokens: 6000  # Reduce from 8000
```

### 2. Enable Aggressive Caching

```yaml
agents:
  atomizer:
    llm:
      cache: true
  planner:
    llm:
      cache: true
  executor:
    llm:
      cache: true
  aggregator:
    llm:
      cache: true
```

### 3. Limit Parallel Execution

```yaml
orchestration:
  max_parallel_tasks: 1  # Sequential only
```

### 4. Disable Verifier

```yaml
agents:
  verifier:
    enabled: false  # Saves ~20% API calls
```

## 💡 Free Tier Usage Tips

### Monitor Your Usage

```bash
# Check OpenRouter usage at https://openrouter.ai/
# Track tokens used:
docker compose logs roma-api | grep "tokens"

# Check Exa usage at https://exa.ai/
```

### Model Costs

- **GLM-4.5-Air:** Free via OpenRouter (`:free` tag)
- **Exa Search:** Free tier includes 1,000 searches/month
- **Rate Limits:** Check dashboards for current usage

### Stay Within Limits

```yaml
orchestration:
  max_execution_steps: 5      # Very conservative
  max_recursion_depth: 1
  max_parallel_tasks: 1
```

## 🎉 Success Verification Checklist

Your installation is successful when:

- ✅ `docker compose ps` shows containers as "Up (healthy)"
- ✅ `curl http://localhost:8000/health` returns `{"status":"healthy"}`
- ✅ `docker compose exec roma-api id` shows `uid=1000(roma)`
- ✅ `docker compose exec roma-api ls -la /app/data` shows directory
- ✅ `./run_task.sh "What is 2+2?"` completes without errors
- ✅ `./run_task.sh "Latest AI news"` returns search results
- ✅ Exa MCPToolkit shows in logs: `Registered toolkit: MCPToolkit`
- ✅ No PostgreSQL or database errors
- ✅ OpenRouter API key is working

---

**Last Updated:** November 2025  
**ROMA Version:** v0.2.0-beta  
**Tested On:** Ubuntu 22.04, Docker Compose v2.24.0+  
**Features:** Exa web search, Free-tier models (GLM-4.5-Air), File-based storage, Production-ready

**This guide has all issues resolved**
