# 🚀 ROMA Installation Guide(Optimized to use free API)

A comprehensive guide to install and configure ROMA (Research Operations Management Agent) using **completely free APIs** with optimized settings for cost-effective usage.

## 📋 Prerequisites

- VPS or local Linux server (Ubuntu recommended)
- OpenRouter account (free tier available)
- Exa.ai account (free tier available)

## ⚡ Quick Overview

This setup uses:
- **GLM-4.5-Air** (free) for planning tasks
- **Grok-4-Fast** (free) for execution tasks
- **Exa.ai** for web search (free tier)
- **Optimized settings**: Max execution steps = 10, Max recursion depth = 1

---

## 🛠️ Installation Steps

### Part 1: Server Preparation

#### Step 1: Connect and Update System
```bash
ssh root@<your_vps_ip>
sudo apt update && sudo apt upgrade -y
```

#### Step 2: Install Docker and Git
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

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### Part 2: ROMA Installation & Configuration

#### Step 3: Clone and Setup ROMA
```bash
git clone https://github.com/sentient-agi/ROMA.git
cd ROMA
chmod +x setup.sh
./setup.sh
```
> **Note**: When prompted, select `1` for Docker Setup

#### Step 4: Configure API Keys
```bash
nano .env
```

Add your API keys to the `.env` file:
```env
# OpenRouter API Key (get free key from https://openrouter.ai/)
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Exa.ai API Key (get free key from https://exa.ai/)
EXA_API_KEY=your_exa_api_key_here
```

**Save and exit**: Press `Ctrl + X`, then `Y`, then `Enter`

#### Step 5: Apply Free-Optimized Configuration

Create and run the optimization script:

```bash
cat > optimize_for_free.sh << 'EOF'
#!/bin/bash

echo "🔧 Applying free-tier optimizations..."

# Navigate to ROMA directory
cd ~/ROMA

# Update execution limits for cost optimization
sed -i 's/max_execution_steps: int = 500/max_execution_steps: int = 10/' src/sentientresearchagent/config/config.py
sed -i "s/'max_execution_steps': 500,/'max_execution_steps': 10,/" src/sentientresearchagent/config/config.py
sed -i 's/max_recursion_depth: int = 5/max_recursion_depth: int = 1/' src/sentientresearchagent/config/config.py
sed -i "s/'max_recursion_depth': 5,/'max_recursion_depth': 1,/" src/sentientresearchagent/config/config.py

# Update frontend defaults
sed -i 's/max_execution_steps: 250,/max_execution_steps: 10,/' frontend/src/components/project/ProjectInput.tsx
sed -i 's/max_recursion_depth: 2,/max_recursion_depth: 1,/' frontend/src/components/project/ProjectInput.tsx
sed -i 's/|| 250}/|| 10}/' frontend/src/components/project/ProjectConfigPanel.tsx
sed -i 's/|| 2}/|| 1}/' frontend/src/components/project/ProjectConfigPanel.tsx
sed -i 's/max_execution_steps: 250,/max_execution_steps: 10,/' frontend/src/components/sidebar/ProjectSidebar.tsx
sed -i 's/max_recursion_depth: 2,/max_recursion_depth: 1,/' frontend/src/components/sidebar/ProjectSidebar.tsx

# Update YAML config
sed -i 's/max_execution_steps: 500/max_execution_steps: 10/' sentient.yaml

echo "✅ Free-tier optimizations applied!"
EOF

chmod +x optimize_for_free.sh
./optimize_for_free.sh
```

#### Step 6: Replace Agent Configuration with Optimized Version

Replace the default agents.yaml with our free-tier optimized configuration:

```bash
# Backup the original agents.yaml
cp src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml.backup

# Download the optimized agents.yaml configuration
cat > src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml << 'EOF'
agents:
- name: CoreResearchPlanner
  type: planner
  adapter_class: PlannerAdapter
  description: Specialized planner for research-focused tasks
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
  prompt_source: prompts.planner_prompts.PLANNER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: WRITE
    named_keys:
    - CoreResearchPlanner
    - default_planner
  enabled: false
- name: EnhancedSearchPlanner
  type: planner
  adapter_class: PlannerAdapter
  description: Specialized planner optimized for search-focused research tasks with date awareness and parallel execution
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
    temperature: 0.35
  prompt_source: prompts.planner_prompts.ENHANCED_SEARCH_PLANNER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: SEARCH
    named_keys:
    - EnhancedSearchPlanner
    - search_planner
  enabled: true
- name: EnhancedThinkPlanner
  type: planner
  adapter_class: PlannerAdapter
  description: Specialized planner optimized for search-focused research tasks with date awareness and parallel execution
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
    temperature: 0.35
  prompt_source: prompts.planner_prompts.ENHANCED_THINK_PLANNER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: THINK
    named_keys:
    - EnhancedThinkPlanner
    - think_planner
  enabled: true
- name: EnhancedWritePlanner
  type: planner
  adapter_class: PlannerAdapter
  description: Specialized planner optimized for search-focused research tasks with date awareness and parallel execution
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
    temperature: 0.35
  prompt_source: prompts.planner_prompts.ENHANCED_WRITE_PLANNER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: WRITE
    named_keys:
    - EnhancedWritePlanner
    - write_planner
  enabled: true
- name: DeepResearchPlanner
  type: planner
  adapter_class: PlannerAdapter
  description: Master planner specialized in high-level research project decomposition and strategy
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
    temperature: 0.35
  prompt_source: prompts.planner_prompts.DEEP_RESEARCH_PLANNER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: WRITE
    named_keys:
    - DeepResearchPlanner
  enabled: true
- name: GeneralTaskSolver
  type: planner
  adapter_class: PlannerAdapter
  description: Master planner specialized in high-level research project decomposition and strategy
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
    temperature: 0.3
  prompt_source: prompts.planner_prompts.GENERAL_TASK_SOLVER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: WRITE
    named_keys:
    - GeneralTaskSolver
  enabled: true
- name: SearchExecutor
  type: executor
  adapter_class: ExecutorAdapter
  description: LLM-based search executor with web search tools
  model:
    provider: openai
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.executor_prompts.SEARCH_EXECUTOR_SYSTEM_MESSAGE
  response_model: WebSearchResultsOutput
  tools:
  - DuckDuckGoTools
  - WikipediaTools
  - web_search
  registration:
    named_keys:
    - SearchExecutor
    - default_search_executor
  enabled: false
- name: BasicReasoningExecutor
  type: executor
  adapter_class: ExecutorAdapter
  description: Performs Analysis and Synthesis of information
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.executor_prompts.REASONING_EXECUTOR_SYSTEM_MESSAGE
  tools:
  - name: PythonTools
    params:
      save_and_run: false
  - ReasoningTools
  registration:
    action_keys:
    - action_verb: execute
      task_type: THINK
    named_keys:
    - BasicReasoningExecutor
  enabled: true
- name: BasicReportWriter
  type: executor
  adapter_class: ExecutorAdapter
  description: Writes detailed reports based on provided context
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.executor_prompts.BASIC_REPORT_WRITER_SYSTEM_MESSAGE
  registration:
    action_keys:
    - action_verb: execute
      task_type: WRITE
    named_keys:
    - BasicReportWriter
  enabled: true
- name: SmartWebSearcher
  type: executor
  adapter_class: ExecutorAdapter
  description: Intelligent searcher that combines AI-powered search with Wikipedia
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.executor_prompts.SMART_WEB_SEARCHER_SYSTEM_MESSAGE
  tools:
  - web_search
  - WikipediaTools
  registration:
    action_keys:
    - action_verb: execute
      task_type: SEARCH
    named_keys:
    - SmartWebSearcher
  enabled: true
- name: OpenAICustomSearcher
  type: custom_search
  adapter_class: OpenAICustomSearchAdapter
  description: Direct API-based search using OpenAI's search capabilities
  adapter_params:
    search_context_size: high
    use_openrouter: true
    model_id: openrouter/openai/gpt-oss-20b:free
  registration:
    action_keys:
    - action_verb: execute
      task_type: SEARCH
    named_keys:
    - OpenAICustomSearcher
    - default_openai_searcher
  enabled: false
- name: GeminiCustomSearcher
  type: custom_search
  adapter_class: GeminiCustomSearchAdapter
  description: Direct API-based search using Google Gemini's search capabilities
  adapter_params:
    model_id: openrouter/x-ai/grok-4-fast:free
  registration:
    action_keys:
    - action_verb: execute
      task_type: SEARCH
    named_keys:
    - GeminiCustomSearcher
    - default_gemini_searcher
    - gemini_searcher
  enabled: false
- name: ExaComprehensiveSearcher
  type: custom_search
  adapter_class: ExaCustomSearchAdapter
  description: Comprehensive search using Exa API with LiteLLM processing - prioritizes reliable sources
  adapter_params:
    model_id: openrouter/x-ai/grok-4-fast:free
    num_results: 5
  registration:
    action_keys:
    - action_verb: execute
      task_type: SEARCH
    named_keys:
    - ExaComprehensiveSearcher
    - exa_searcher
    - comprehensive_searcher
  enabled: true
- name: DefaultAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Synthesizes results from multiple child tasks
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.DEFAULT_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    action_keys:
    - action_verb: aggregate
      task_type: null
    named_keys:
    - default_aggregator
  enabled: true
- name: SearchAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Specialized aggregator for combining search results with deduplication and relevance ranking
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.SEARCH_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    action_keys:
    - action_verb: aggregate
      task_type: SEARCH
    named_keys:
    - SearchAggregator
    - search_aggregator
  enabled: true
- name: ThinkAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Specialized aggregator for synthesizing analytical outputs and drawing logical conclusions
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.THINK_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    action_keys:
    - action_verb: aggregate
      task_type: THINK
    named_keys:
    - ThinkAggregator
    - think_aggregator
  enabled: true
- name: WriteAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Specialized aggregator for combining written content with focus on narrative flow and coherence
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.WRITE_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    action_keys:
    - action_verb: aggregate
      task_type: WRITE
    named_keys:
    - WriteAggregator
    - write_aggregator
  enabled: true
- name: RootResearchAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Master aggregator for root-level research tasks with executive synthesis and strategic recommendations
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.ROOT_RESEARCH_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    named_keys:
    - RootResearchAggregator
    - root_research_aggregator
  enabled: true
- name: RootAnalysisAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Master aggregator for root-level analysis tasks with decisive conclusions and strategic recommendations
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.ROOT_ANALYSIS_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    named_keys:
    - RootAnalysisAggregator
    - root_analysis_aggregator
  enabled: true
- name: RootGeneralAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Master aggregator for root-level general tasks with comprehensive synthesis and actionable insights
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.ROOT_GENERAL_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    named_keys:
    - RootGeneralAggregator
    - root_general_aggregator
  enabled: true
- name: CryptoAnalyticsPlanner
  type: planner
  adapter_class: PlannerAdapter
  description: Specialized planner for cryptocurrency and DeFi analytics with deep domain expertise
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
    temperature: 0.35
  prompt_source: prompts.planner_prompts.CRYPTO_ANALYTICS_PLANNER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: WRITE
    named_keys:
    - CryptoAnalyticsPlanner
    - crypto_planner
  enabled: true
- name: CryptoSearchPlanner
  type: planner
  adapter_class: PlannerAdapter
  description: Specialized planner for crypto-specific search tasks optimized for blockchain data retrieval
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
    temperature: 0.35
  prompt_source: prompts.planner_prompts.CRYPTO_SEARCH_PLANNER_SYSTEM_MESSAGE
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: plan
      task_type: SEARCH
    named_keys:
    - CryptoSearchPlanner
    - crypto_search_planner
  enabled: true
- name: CryptoMarketAnalyzer
  type: executor
  adapter_class: ExecutorAdapter
  description: Advanced crypto market analyst for technical analysis and on-chain metrics
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.executor_prompts.CRYPTO_MARKET_ANALYZER_SYSTEM_MESSAGE
  tools:
  - ReasoningTools
  toolkits:
  - name: E2BTools
    params:
      timeout: 86400
      filesystem: true
      command_execution: true
      sandbox_options:
        template: sentient-e2b-s3
  - name: BinanceToolkit
    params:
      default_market_type: spot
      symbols:
      - BTCUSD
      - ETHUSD  
      - SOLUSD
      - XRPUSD
      - ADAUSD
      - DOTUSD
      - AVAXUSD
      data_dir: ./data/binance
      parquet_threshold: 10
    available_tools:
    - get_current_price
    - get_klines
    - get_order_book
    - get_symbol_ticker_change
    - get_book_ticker
  - name: CoingeckoToolkit
    params:
      data_dir: ./data/coingecko
      parquet_threshold: 10
    available_tools:
    - get_coin_info
    - get_coin_price
    - get_coin_market_chart
    - get_multiple_coins_data
    - get_historical_price
    - get_token_price_by_contract
    - search_coins_exchanges_categories
    - get_coins_list
    - get_coins_markets
    - get_coin_ohlc
    - get_global_crypto_data
  - name: DefiLlamaToolkit
    params:
      data_dir: ./data/defillama
      base_url: https://api.llama.fi
      pro_base_url: https://pro-api.llama.fi
      parquet_threshold: 10
    available_tools:
    - get_protocols
    - get_protocol_tvl
    - get_protocol_fees
    - get_chain_fees
    - get_yield_pools
    - get_active_users
    - get_chain_assets
    - get_protocol_detail
    - get_chains
    - get_chain_historical_tvl
  - name: ArkhamToolkit
    params:
      default_chain: ethereum
      data_dir: ./data/arkham
      parquet_threshold: 10
      supported_chains:
      - ethereum
      - bitcoin
      - polygon
      - avalanche
      - bsc
      include_entity_data: true
    available_tools:
    - get_top_tokens
    - get_token_holders
    - get_token_top_flow
    - get_supported_chains
    - get_transfers
    - get_token_balances
  registration:
    action_keys:
    - action_verb: execute
      task_type: THINK
    named_keys:
    - CryptoMarketAnalyzer
    - crypto_analyzer
  enabled: true
- name: CryptoResearchExecutor
  type: executor
  adapter_class: ExecutorAdapter
  description: Comprehensive crypto research executor for deep blockchain and DeFi analysis
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.executor_prompts.CRYPTO_RESEARCH_EXECUTOR_SYSTEM_MESSAGE
  registration:
    action_keys:
    - action_verb: execute
      task_type: WRITE
    named_keys:
    - CryptoResearchExecutor
    - crypto_writer
  enabled: true
- name: CryptoAnalyticsAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Specialized aggregator for crypto data synthesis and market insights
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.CRYPTO_ANALYTICS_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    named_keys:
    - CryptoAnalyticsAggregator
    - crypto_aggregator
  enabled: true
- name: CryptoRootAggregator
  type: aggregator
  adapter_class: AggregatorAdapter
  description: Master aggregator for executive-level crypto research synthesis and strategic insights
  model:
    provider: litellm
    model_id: openrouter/x-ai/grok-4-fast:free
  prompt_source: prompts.aggregator_prompts.CRYPTO_ROOT_AGGREGATOR_SYSTEM_MESSAGE
  registration:
    named_keys:
    - CryptoRootAggregator
    - crypto_root_aggregator
  enabled: true
- name: DefaultAtomizer
  type: atomizer
  adapter_class: AtomizerAdapter
  description: Determines if tasks are atomic or need further planning
  model:
    provider: litellm
    model_id: openrouter/z-ai/glm-4.5-air:free
  prompt_source: prompts.atomizer_prompts.ATOMIZER_SYSTEM_MESSAGE
  response_model: AtomizerOutput
  registration:
    action_keys:
    - action_verb: atomize
      task_type: null
    named_keys:
    - default_atomizer
  enabled: true
- name: PlanModifier
  type: plan_modifier
  adapter_class: PlanModifierAdapter
  description: Modifies existing plans based on user feedback
  model:
    provider: litellm
    model_id: openrouter/qwen/qwen3-235b-a22b:free
  prompt_source: prompts.plan_modifier_prompts.PLAN_MODIFIER_SYSTEM_PROMPT
  response_model: PlanOutput
  registration:
    action_keys:
    - action_verb: modify_plan
      task_type: null
    named_keys:
    - PlanModifier
  enabled: true
metadata:
  version: 1.0.0
  description: Production agent configuration for Sentient Research Agent
  last_updated: '2024-01-01'
  total_agents: 29
  migration_status: integrated_with_legacy
  features:
  - Structured output support
  - Tool integration
  - Dynamic prompt resolution
  - Flexible model providers
  - LLM parameter configuration
  - Legacy system compatibility
EOF

echo "✅ Optimized agents.yaml configuration applied!"
```

### Part 3: Launch & Access

#### Step 7: Start ROMA with Optimized Settings
```bash
cd ~/ROMA/docker/
sudo docker compose down
sudo docker compose up -d --build
```

#### Step 8: Configure Firewall
```bash
sudo ufw allow 3000/tcp
sudo ufw allow 5000/tcp
sudo ufw reload
```

#### Step 9: Access Your ROMA Instance
Open your browser and navigate to:
```
http://<your_vps_ip>:3000
```

---

## 🔍 Monitoring & Troubleshooting

### View Live Logs
```bash
cd ~/ROMA/docker/
sudo docker compose logs -f
```

### Check Container Status
```bash
sudo docker compose ps
```

### Restart Services
```bash
sudo docker compose restart
```

---

## 💡 Free Tier Optimization Details

### Model Configuration
- **Planning Tasks**: `z-ai/glm-4.5-air:free` - Efficient for task decomposition
- **Execution Tasks**: `x-ai/grok-4-fast:free` - Fast and capable for most operations
- **Search Engine**: Exa.ai (free tier) - Advanced web search capabilities

### Performance Settings
- **Max Execution Steps**: 10 (down from 500) - Reduces API calls
- **Max Recursion Depth**: 1 (down from 5) - Prevents deep recursive operations
- **Optimized Agent Selection**: Disabled expensive agents, enabled free alternatives

### Cost Benefits
- ✅ 100% free API usage within tier limits
- ✅ Reduced token consumption
- ✅ Faster execution times
- ✅ Suitable for learning and development

---

## 🔧 Configuration Files

### Key Configuration Changes Made

The setup includes a pre-optimized `agents.yaml` with:
- GLM-4.5-Air for all planning agents
- Grok-4-Fast for all execution and aggregation agents
- OpenAI Custom Searcher disabled
- Exa Comprehensive Searcher enabled
- Optimized token limits and execution parameters

---

## 🆘 Support & Resources

- **ROMA Repository**: [sentient-agi/ROMA](https://github.com/sentient-agi/ROMA)
- **OpenRouter Free Tier**: [openrouter.ai](https://openrouter.ai/)
- **Exa.ai Free Account**: [exa.ai](https://exa.ai/)
- **Docker Documentation**: [docs.docker.com](https://docs.docker.com/)

---

## 📝 Notes

- This configuration is optimized for **learning and development** purposes
- For production use, consider upgrading to paid tiers for higher limits
- Monitor your API usage to stay within free tier limits
- The optimized settings balance functionality with cost-effectiveness

---

## 🎉 Success!

Your ROMA instance is now running with:
- ✅ Free API integration
- ✅ Optimized performance settings  
- ✅ Exa.ai search capabilities
- ✅ Cost-effective operation

Start creating your first research project and explore ROMA's capabilities!
