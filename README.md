# LogIntel 🔍

**Automated Backend Error Detection & Root-Cause Analysis System**

LogIntel is an AI-powered platform that automatically monitors your backend logs, detects errors, analyzes root causes using LLM intelligence, and creates JIRA tickets with actionable debugging insights—all without manual intervention.

## ✨ Features

- **🧠 AI-Powered Analysis**: Two-step LLM analysis using Groq's llama-3.3-70b model
  - Step 1: Identifies suspected functions from your codebase
  - Step 2: Performs detailed root-cause analysis with debugging steps and fixes

- **📑 Multi-Language Code Indexing**: Automatically extracts functions and documentation from:
  - Python
  - JavaScript/TypeScript
  - Java
  - Go

- **📡 Real-Time Log Monitoring**: Watches JSON log files for errors with:
  - Full log replay on startup (replay all historical errors)
  - Tail mode for continuous monitoring of new logs
  - Configurable error level filters (ERROR, CRITICAL, FATAL)

- **🎫 Automatic JIRA Integration**: Creates professional tickets with:
  - Root cause summary and technical explanations
  - Step-by-step debugging instructions
  - Immediate, short-term, and long-term fixes
  - Severity classification and confidence scores
  - Embedded error logs for context

- **🌐 Web UI & REST API**: 
  - Streamlit frontend for easy interaction
  - FastAPI REST API for programmatic access
  - Real-time monitoring dashboard

---

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- Groq API key (get it from [console.groq.com](https://console.groq.com))
- JIRA instance with API access (optional)
- FastAPI, Streamlit installed

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Shreyas1830/LogIntel.git
   cd LogIntel
   ```

2. **Create a virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Create environment configuration**
   ```bash
   cp app/.env.example app/.env
   ```

5. **Configure your settings** in `app/.env`:
   ```env
   # Groq LLM Configuration
   GROQ_API_KEY=your_groq_api_key
   GROQ_MODEL=llama-3.3-70b-versatile
   
   # JIRA Configuration
   JIRA_BASE_URL=https://your-jira-instance.atlassian.net
   JIRA_USERNAME=your-email@company.com
   JIRA_PASSWORD=your-api-token
   JIRA_PROJECT_KEY=YOUR_PROJECT
   JIRA_ASSIGNEE=user-key
   
   # Monitoring Configuration
   ERROR_LEVEL_FILTER=ERROR,CRITICAL,FATAL
   ```

### Running the Application

**Backend API (FastAPI)**
```bash
uvicorn app.main:app --reload --port 8000
```

**Frontend (Streamlit)**
```bash
streamlit run frontend/streamlit_app.py
```

The Streamlit UI will be available at `http://localhost:8501`

---

## 📖 How It Works

### Architecture Overview

```
                    Streamlit UI
                   (Web Interface)
                         │
                        HTTP
                         │
        ┌────────────────▼────────────────┐
        │     FastAPI Backend             │
        │                                 │
        │  Indexer  Monitor  Analyzer     │
        │  JIRA     State               │
        └────────────────┬────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
    Code Index      Log Files        Groq LLM
    Tickets         JSON Format      JIRA API
```

### Data Flow Example

1. **Upload Code**: Developer uploads backend codebase (ZIP or local path)
2. **Index**: Indexing Service scans all files and extracts functions with descriptions
3. **Monitor**: LogWatcher begins watching JSON log files
4. **Detect**: Error detected in log (ERROR, CRITICAL, or FATAL level)
5. **Analyze - Step 1**: LLM identifies which functions are likely involved
6. **Analyze - Step 2**: LLM analyzes source code of suspected functions → root cause
7. **Create Ticket**: JIRA client creates ticket with full analysis embedded
8. **Track**: Event stored in UI for review and historical reference

---

## 🎯 Core Components

### 1. **Indexer** (`app/indexer/`)
Scans backend code and extracts function metadata:
- Function names, signatures, descriptions
- Source code snippets with line numbers
- Class definitions and API routes
- Imports and dependencies

**Language Parsers:**
- `python_parser.py` - Python source code
- `js_ts_parser.py` - JavaScript/TypeScript
- `java_parser.py` - Java code
- `go_parser.py` - Go source code

### 2. **Two-Step LLM Analyzer** (`app/analyzer/`)
AI-powered root-cause analysis with Groq LLM:

**Step 1**: Identify Suspected Functions
- Input: Error message + function index
- Output: Top 1-5 functions with confidence scores

**Step 2**: Detailed Analysis
- Input: Error message + source code of suspected functions
- Output:
  - Root cause (1-sentence summary)
  - Technical explanation (2-4 paragraphs)
  - Debugging steps (numbered, actionable)
  - Possible fixes (immediate, short-term, long-term)
  - Severity (low/medium/high/critical)
  - Confidence score (0.0-1.0)

### 3. **Log Watcher** (`app/monitor/`)
Real-time JSON log file monitoring:
- **Replay Mode**: Processes entire log file on startup (catches historical errors)
- **Tail Mode**: Monitors new lines appended in real-time
- **Flexible Format**: Supports multiple JSON log formats:
  ```json
  {"timestamp": "...", "level": "ERROR", "message": "..."}
  {"time": "...", "severity": "FATAL", "msg": "..."}
  {"@timestamp": "...", "log.level": "ERROR", "message": "..."}
  ```

### 4. **JIRA Integration** (`app/jira/`)
Creates professional tickets with rich formatting:
- Error details (timestamp, service, message)
- Root cause analysis
- Technical explanation
- Debugging steps (formatted as numbered list)
- Possible fixes
- Severity and confidence metrics
- Full error log as code block

### 5. **REST API Routers** (`app/routers/`)

**Index Router** (`index_router.py`):
```
POST   /api/v1/index/upload     - Upload and index ZIP file
POST   /api/v1/index/path       - Index local directory
GET    /api/v1/index/status     - Get index status & statistics
DELETE /api/v1/index            - Clear current index
```

**Monitor Router** (`monitor_router.py`):
```
POST   /api/v1/monitor/start    - Start log monitoring
POST   /api/v1/monitor/stop     - Stop monitoring
GET    /api/v1/monitor/status   - Get monitoring status
GET    /api/v1/monitor/events   - Retrieve analyzed events
```

---

## 🖥️ Streamlit Frontend Pages

### 1. **⚙️ Setup Index**
- Upload backend code as ZIP or specify directory path
- View indexing progress and statistics
- See language breakdown of indexed code
- Save index for later use

### 2. **📡 Live Monitor**
- Start/stop log file monitoring
- Configure error level filters
- Enable/disable JIRA ticket creation
- Set monitoring duration and replay options
- Real-time event counter

### 3. **📋 Event History**
- View all detected and analyzed errors
- Filter by severity, date range, service
- View full analysis for each error:
  - Root cause
  - Technical explanation
  - Debugging steps
  - Possible fixes
- Copy error details for manual review

### 4. **🔧 JIRA Test**
- Verify JIRA API credentials
- Test ticket creation
- Check project and assignee settings
- View connection status

### 5. **🧪 Log Generator**
- Generate sample logs with different error patterns
- Create test scenarios for verification
- Export test logs for manual testing

---

## 📊 Project Structure

```
LogIntel/
├── app/
│   ├── __init__.py
│   ├── config.py              # Configuration management (Pydantic BaseSettings)
│   ├── main.py                # FastAPI application core
│   ├── models.py              # Pydantic data models
│   ├── state.py               # Global application state
│   ├── analyzer/
│   │   ├── __init__.py
│   │   └── two_step_analyzer.py    # LLM-powered analysis logic
│   ├── indexer/
│   │   ├── __init__.py
│   │   ├── service.py              # Main indexing orchestration
│   │   └── parsers/
│   │       ├── __init__.py
│   │       ├── python_parser.py    # Python code parser
│   │       ├── js_ts_parser.py     # JavaScript/TypeScript parser
│   │       ├── java_parser.py      # Java code parser
│   │       └── go_parser.py        # Go code parser
│   ├── jira/
│   │   ├── __init__.py
│   │   └── client.py               # JIRA ticket creation
│   ├── monitor/
│   │   ├── __init__.py
│   │   └── log_watcher.py          # Log file monitoring
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── index_router.py         # Index API endpoints
│   │   └── monitor_router.py       # Monitor API endpoints
│   └── utils/
│       ├── __init__.py
│       ├── language_detector.py    # Language detection
│       └── logger.py               # Logging utilities
├── frontend/
│   └── streamlit_app.py        # Streamlit web UI
├── log_generator/
│   ├── __init__.py
│   └── fake_log_generator.py   # Generate test logs
├── sample_logs/                # Sample log files for testing
├── requirements.txt            # Python dependencies
├── README.md                   # This file
└── .env.example               # Environment variables template
```

---

## 🔧 Configuration

### Environment Variables (`app/.env`)

```env
# Groq LLM Settings
GROQ_API_KEY=your_api_key_here
GROQ_MODEL=llama-3.3-70b-versatile
GROQ_TEMPERATURE=0.3
GROQ_MAX_TOKENS=4096

# JIRA Configuration
JIRA_BASE_URL=https://your-domain.atlassian.net
JIRA_USERNAME=your-email@company.com
JIRA_PASSWORD=your-api-token
JIRA_PROJECT_KEY=PROJ
JIRA_ASSIGNEE=your-user-key
JIRA_ISSUE_TYPE=Bug

# Monitoring Settings
ERROR_LEVEL_FILTER=ERROR,CRITICAL,FATAL
LOG_REPLAY_DELAY=0.5

# Application
DEBUG=false
```

---

## 📦 Dependencies

- **FastAPI 0.115.0+** - Web framework
- **Uvicorn 0.29.0+** - ASGI server
- **Streamlit 1.35.0+** - Web UI framework
- **Pydantic 2.10.0+** - Data validation
- **httpx 0.27.0+** - Async HTTP client
- **requests 2.32.0+** - HTTP library
- **python-dotenv 1.0.1+** - Environment variables
- **python-multipart 0.0.9+** - File uploads

---

## 🎮 Usage Examples

### Example 1: Index Python Backend

```bash
# Terminal 1: Start API
uvicorn app.main:app --reload

# Terminal 2: Start Streamlit UI
streamlit run frontend/streamlit_app.py

# In UI: Go to "Setup Index" → Upload your backend.zip → Process
```

### Example 2: Monitor Logs via REST API

```bash
# Index a local directory
curl -X POST "http://localhost:8000/api/v1/index/path" \
  -H "Content-Type: application/json" \
  -d '{"path": "/path/to/backend"}'

# Start monitoring logs
curl -X POST "http://localhost:8000/api/v1/monitor/start" \
  -H "Content-Type: application/json" \
  -d '{
    "log_path": "/path/to/logs.json",
    "create_jira_tickets": true
  }'

# Get analyzed events
curl "http://localhost:8000/api/v1/monitor/events"
```

### Example 3: Generate Test Logs

```bash
python log_generator/fake_log_generator.py --output test_logs/sample.json
```

---

## 🔄 Workflow

1. **Setup Phase**
   - Upload backend codebase (Python, JS, Java, Go)
   - LogIntel indexes all functions and saves `backend_index.json`

2. **Monitoring Phase**
   - Point LogIntel to your JSON log file
   - LogWatcher replays historical errors (optional)
   - Begins tailing for new errors in real-time

3. **Analysis Phase**
   - Each error triggers two-step LLM analysis
   - Step 1: Identify suspected functions
   - Step 2: Analyze source code for root cause

4. **Action Phase**
   - JIRA ticket created automatically with full analysis
   - Event stored in UI for review
   - Developer receives email with ticket and insights

5. **Review Phase**
   - Check event history in Streamlit UI
   - Review root cause analysis and suggested fixes
   - Take action based on insights

---

## 🧪 Testing

### Generate Sample Logs
```bash
python log_generator/fake_log_generator.py \
  --output sample_logs/test.json \
  --count 10
```

### Test JIRA Connection
Use the "JIRA Test" page in Streamlit UI to verify credentials and create a test ticket.

### Monitor Sample Logs
1. Generate test logs as above
2. In Streamlit: Go to "Live Monitor"
3. Point to `sample_logs/test.json`
4. Enable JIRA ticket creation
5. Watch events appear in real-time

---

## 🐛 Troubleshooting

### "Invalid Groq API Key"
- Verify `GROQ_API_KEY` in `.env`
- Check API key is valid at [console.groq.com](https://console.groq.com)

### "JIRA connection failed"
- Verify JIRA URL, username, password/token in `.env`
- Ensure API token is enabled for your JIRA user
- Check firewall/VPN doesn't block JIRA API

### "Index not loading"
- Ensure `backend_index.json` exists in project root
- Verify uploaded code was valid (no syntax errors)
- Try re-indexing the codebase

### "Log file not found"
- Verify log file path is absolute or relative to project root
- Check file format is valid JSON
- Ensure log file is readable

---

## 📄 License

This project is licensed under the MIT License - see LICENSE file for details.

---

## 🤝 Contributing

Contributions are welcome! Here's how:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📧 Support

For issues, questions, or suggestions:
- Open a GitHub Issue
- Check existing documentation
- Review sample logs and test cases

---

## 🚀 Roadmap

- [ ] Support for additional log formats (CloudWatch, ELK Stack)
- [ ] Custom LLM model support
- [ ] Slack integration for notifications
- [ ] Error grouping and deduplication
- [ ] Performance metrics and dashboard
- [ ] Database persistence for long-term storage
- [ ] Docker containerization
- [ ] Kubernetes deployment guide

---

## 📝 Changelog

### v1.0.0
- Initial release
- Multi-language code indexing (Python, JS, Java, Go)
- Two-step LLM analysis with Groq
- Real-time log monitoring
- JIRA ticket creation
- Streamlit web UI
- REST API with FastAPI

---

**Happy debugging! 🎉**

For more information, visit [GitHub Repository](https://github.com/Shreyas1830/LogIntel)
