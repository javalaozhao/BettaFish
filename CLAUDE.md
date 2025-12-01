# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BettaFish (微舆) is a multi-agent public opinion analysis system that automates 24/7 monitoring, analysis, and reporting across 30+ domestic and international social media platforms. The system uses 5 specialized AI agents coordinated through a forum collaboration mechanism to generate comprehensive HTML/PDF reports.

**Architecture**: Multi-agent system with Flask + Streamlit interfaces, PostgreSQL/MySQL database, and modular microservices design.

## Common Development Commands

### Starting the System

```bash
# Full system (Flask web interface)
python app.py

# Individual agents via Streamlit
streamlit run SingleEngineApp/query_engine_streamlit_app.py --server.port 8503
streamlit run SingleEngineApp/media_engine_streamlit_app.py --server.port 8502
streamlit run SingleEngineApp/insight_engine_streamlit_app.py --server.port 8501

# Command-line report generation (no web UI)
python report_engine_only.py --query "Your analysis topic"
```

### Testing

```bash
# Run tests
python tests/run_tests.py

# Individual test files
python tests/test_monitor.py
python tests/test_report_engine_sanitization.py
```

### Crawler Operations (MindSpider)

```bash
cd MindSpider

# Initialize project
python main.py --setup

# Run topic extraction
python main.py --broad-topic --date 2024-01-20

# Complete crawling workflow
python main.py --complete --date 2024-01-20
```

### Dependencies & Environment

```bash
# Install dependencies
pip install -r requirements.txt
# or faster with uv
uv pip install -r requirements.txt

# Install Playwright browser (required for crawler)
playwright install chromium
```

## Architecture Overview

### Core Agent Structure

Each engine (Query, Media, Insight, Report, Forum) follows a standardized modular structure:

```
EngineName/
├── agent.py              # Main agent orchestration logic
├── llms/                 # OpenAI-compatible LLM client wrappers
├── nodes/                # Processing nodes (search, format, summarize)
├── tools/                # Specialized tools for agent's domain
├── utils/                # Utility functions and configuration
├── state/                # Agent state management
└── prompts/              # Prompt templates
```

### Multi-Agent Workflow

1. **User Query** → Flask app receives analysis request
2. **Parallel Launch** → Query, Media, and Insight agents start simultaneously
3. **Initial Analysis** → Each agent uses specialized tools for overview
4. **Strategy Formulation** → Agents develop research strategies
5. **Forum Collaboration Loop** (multi-round):
   - Deep research with reflection mechanisms
   - ForumEngine monitors and guides discussion
   - Agents exchange findings and adjust direction
6. **Report Generation** → Report Agent collects all analysis
7. **IR Representation** → Template selection and metadata binding
8. **Final Report** → HTML/PDF generation with interactive visualizations

### Key Architectural Decisions

- **OpenAI-Compatible API Standard**: All LLM integrations use OpenAI format for provider flexibility
- **Intermediate Representation (IR)**: Report Engine uses JSON-based IR for structured report generation
- **Agent-Specific LLMs**: Different agents optimized for different models (Kimi for Insight, Gemini for Media, DeepSeek for Query)
- **Async-First Design**: SQLAlchemy async engine, async HTTP clients for concurrent operations
- **Modular Tool System**: Each agent has specialized tools with clear separation of concerns

### Database Schema

The system uses two main ORM models:

- **models_bigdata.py**: Large-scale media sentiment tables (weibo_bigdata, weibo_content, xhs_bigdata, douyin_bigdata, etc.)
- **models_sa.py**: Sentiment analysis tables (DailyTopic, DailyTask, TopicToTask, AnalysisHistory)

All agents query through SQLAlchemy async session with readonly access patterns.

### Critical Configuration

**Environment variables in `.env`:**
- Database: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_DIALECT`
- LLM APIs: `*_API_KEY`, `*_BASE_URL`, `*_MODEL_NAME` for each agent (INSIGHT, MEDIA, QUERY, REPORT, FORUM, MINDSPIDER)
- Search APIs: `TAVILY_API_KEY`, `BOCHA_WEB_SEARCH_API_KEY`

**Agent-specific settings in `config.py`:**
- Search limits and timeouts
- Reflection and iteration parameters
- Content length limits
- Sentiment analysis configuration

## Important Development Notes

### When Modifying Agents

1. **Follow the established pattern**: Maintain the agent.py + llms/ + nodes/ + tools/ + utils/ + state/ + prompts/ structure
2. **Use base classes**: Inherit from base_node.py for new nodes, base.py for LLM clients
3. **State management**: Use Pydantic models for agent state consistency
4. **Logging**: Use loguru logger with appropriate levels
5. **Error handling**: Follow the retry_helper.py patterns for network operations

### When Adding New Tools

1. Place in the appropriate engine's `tools/` directory
2. Use the `@tool` decorator pattern established in existing tools
3. Implement proper error handling and logging
4. Add type hints and docstrings
5. Test with the agent's standalone Streamlit app first

### When Working with Reports

1. **Template system**: Markdown templates in `ReportEngine/report_template/`
2. **IR schema**: Validated JSON structure - see `ReportEngine/ir/schema.py`
3. **Rendering pipeline**: IR → HTML → PDF (via WeasyPrint)
4. **Output locations**: HTML in `final_reports/`, PDF in `final_reports/pdf/`

### Database Conventions

1. **Read-only access**: All agents use readonly SQLAlchemy sessions
2. **Async queries**: Use async/await with SQLAlchemy async engine
3. **Connection pooling**: Configured in config.py with appropriate timeouts
4. **Migration**: Manual SQL files in `MindSpider/schema/` - no automatic migrations

### Code Standards

- Python 3.9+ with type hints throughout
- Async/await for all I/O operations
- OpenAI-compatible API format for all LLM calls
- Comprehensive error logging with loguru
- No hardcoded credentials or API keys
- Environment-based configuration via pydantic-settings
