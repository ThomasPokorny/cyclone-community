# Cyclone 🌪️ - AI Code Review Tool

A lightweight, self-hosted Go tool that integrates with GitHub to provide AI-powered code reviews on pull requests using Claude AI.

## Why Cyclone?

LLMs offer real potential for code reviews, particularly for semantic analysis that goes beyond what static analysis tools can catch. However, the current landscape of AI code review tools suffers from:

- **Excessive pricing** for what is essentially a webhook + LLM integration
- **Vendor lock-in** with inflexible, closed systems
- **Lack of control** over prompts, behavior, and hosting

**Cyclone is:**
- ✅ **Completely free and open source** - run it anywhere you want
- ✅ **Full control** - modify the system prompt, codebase, and behavior
- ✅ **Budget-friendly** - bring your own API key, pay only for what you use
- ✅ **Transparent** - no per-user pricing schemes or hidden costs

While Cyclone currently integrates with Anthropic's Claude, I'm working towards full AI provider agnosticism, giving you the freedom to choose any LLM you prefer.

## ✨ Features

- **🤖 AI-Powered Reviews**: Uses Claude 3.5 Sonnet for code analysis
- **📍 Line-Specific Comments**: Comments appear directly on relevant lines in the "Files changed" tab
- **📋 Comprehensive Summaries**: Overall PR analysis with structured feedback
- **🏷️ Categorized Feedback**: Issues tagged by type (nit, suggestion, issue, blocking) and focus area (security, performance, style, etc.)
- **⚙️ Repository-Specific Configuration**: Custom review precision and prompts per repository
- **📄 Smart Review Triggers**: Reviews on PR open and ready-for-review events
- **⚡ Real-time Processing**: Responds to PR events via GitHub webhooks
- **🛡️ Repository Filtering**: Only reviews configured repositories, ignores others

## 🚀 Setup

### 1. Prerequisites
- **Go 1.21+** installed
- **GitHub Personal Access Token** with `repo` and `pull_requests:write` permissions
- **Anthropic API Key** for Claude integration
- **Webhook accessibility** - Either:
    - A publicly accessible endpoint (e.g., Railway, Heroku, VPS)
    - A tunnel service like ngrok for local development (see step 6 below)

### 2. Installation
```bash
git clone https://github.com/ThomasPokorny/cyclone-community.git
cd cyclone-community
go mod tidy
```

### 3. Configuration
Create a `.env` file in the project root:
```bash
GITHUB_TOKEN=ghp_your_github_token_here
ANTHROPIC_API_KEY=sk-ant-api03-your_anthropic_key_here
PORT=8080
WEBHOOK_SECRET=optional_webhook_secret
```

**Get your API keys:**
- **GitHub Token**: Settings → Developer settings → Personal access tokens
- **Anthropic API Key**: [console.anthropic.com](https://console.anthropic.com) → API Keys

### 4. Create Review Configuration (Optional)

Create a `review-config.json` file in the project root to customize review behavior per repository. If no configuration is provided for a specific repository, Cyclone will use a default configuration with `"medium"` precision.

```json
{
  "organizations": [
    {
      "name": "your-github-org",
      "repositories": [
        {
          "name": "critical-service",
          "precision": "strict",
          "custom_prompt": "This is a critical production service. Pay special attention to error handling, performance, and security."
        },
        {
          "name": "frontend-app", 
          "precision": "medium",
          "custom_prompt": "Focus on React best practices, accessibility, and user experience."
        },
        {
          "name": "*",
          "precision": "medium",
          "custom_prompt": "Default configuration for all other repositories."
        }
      ]
    }
  ]
}
```

**Precision levels:**
- `"minor"`: Only critical issues and bugs
- `"medium"`: Balanced review (default)
- `"strict"`: Thorough review including style and best practices

### 5. Run Cyclone
```bash
go run main.go
```

### 6. Expose with ngrok (optional for local development)
If running locally, you'll need to expose your webhook endpoint using ngrok, or any other tool of your choice:
```bash
# Install ngrok: https://ngrok.com/download
ngrok http 8080
# Note the https URL (e.g., https://abc123.ngrok.io)
```

For production, consider hosting on platforms like Railway, Heroku, or your own VPS.

### 7. Configure GitHub Webhook
1. Go to your repository → **Settings** → **Webhooks** → **Add webhook**
2. **Payload URL**: `https://your-domain.com/webhook` (or your ngrok URL for testing)
3. **Content type**: `application/json`
4. **Events**: Select "Pull requests"
5. **Active**: ✅ Checked
6. Click **Add webhook**

## 🌪️ How It Works

1. **PR Created/Updated** → GitHub sends webhook to Cyclone
2. **Repository Check** → Cyclone verifies if repository is configured for review
3. **Smart Filtering** → Only reviews on `opened` and `ready_for_review` events
4. **Cyclone Fetches** → Gets PR diff and metadata
5. **Claude Analyzes** → AI reviews code using repository-specific configuration
6. **Structured Feedback** → Posts both overall summary and line-specific comments
7. **Categorized Comments** → Each comment tagged by type and priority

## 📝 Review Categories

Cyclone categorizes feedback with emojis and prefixes:

### **Priority Levels:**
- 🧰 **nit**: Minor style/preference issues, non-blocking
- 💡 **suggestion**: Improvements that would be nice but aren't required
- ⚠️ **issue**: Problems that should be addressed before merging
- 🚫 **blocking**: Critical issues that must be fixed
- ❓ **question**: Seeking clarification about intent or approach

### **Focus Areas:**
- 🎨 **style**: Formatting, naming conventions
- ⚡ **perf**: Performance concerns
- 🔒 **security**: Security-related issues
- 📚 **docs**: Documentation needs
- 🧪 **test**: Testing coverage or quality
- 🔧 **refactor**: Code organization improvements

## 🛠️ API Endpoints

- `GET /health` - Health check endpoint
- `POST /webhook` - GitHub webhook receiver
- `GET /` - Basic info about Cyclone

## 🎯 Example Output

**Overall PR Review:**
```
🌪️ Cyclone AI Code Review

✨ Overview
This PR enhances the authentication system with improved security measures.

🚀 What's Working Well
- 🔧 Clean dependency injection patterns
- 🛡️ Robust error handling implementation

🎯 Key Areas for Improvement
The JWT token validation could benefit from additional security checks...
```

**Line-Specific Comments:**
```
Line 45 in auth.go:
🌪️ Cyclone: 🔒 security: ⚠️ issue: Consider using bcrypt for password hashing instead of plain text storage

Line 123 in api.js:  
🌪️ Cyclone: 💡 suggestion: 🎨 style: Consider using a more descriptive variable name like 'userCount' instead of 'cnt'
```

## 🔧 Development

### Testing Locally
```bash
# Health check
curl http://localhost:8080/health

# Test webhook (with fake payload)
curl -X POST http://localhost:8080/webhook \
  -H "Content-Type: application/json" \
  -d '{"action":"opened","pull_request":{"number":123}}'
```

### Project Structure
```
cyclone-community/
├── cmd/
│   └── cyclone/
│       └── main.go              # Application entry point
├── internal/
│   ├── bot/
│   │   ├── cyclone.go           # Core bot orchestration and setup
│   │   └── webhook.go           # GitHub webhook handling
│   ├── config/
│   │   ├── config.go            # Configuration loading and management
│   │   └── types.go             # Configuration-related types and constants
│   └── review/
│       ├── ai.go                # Claude AI integration and API calls
│       ├── github.go            # GitHub API operations (diff, reviews, comments)
│       ├── parser.go            # Claude response parsing logic
│       └── types.go             # Review-related types and structures
├── .env                         # Environment variables (local development)
├── .gitignore                   # Git ignore rules
├── review-config.json           # Repository review configuration (optional)
├── go.mod                       # Go module definition
├── go.sum                       # Go module checksums
└── README.md                    # This file
```

## Current Limitations

Cyclone Community currently:
- Runs once per PR when created (doesn't yet respond to updates or comments)
- Uses GitHub Personal Access Tokens for authentication (comments appear under the token owner's account)
- Integrates only with Anthropic's Claude API

We're actively working to expand these capabilities (see Next Steps below).

## ⚡ Next Steps

- [ ] **AI/LLM provider agnosticism** - Support for OpenAI, local models, and other providers
- [ ] **GitHub App authentication** - Support for GitHub private keys and App installation
- [ ] **Enhanced PR interactions** - Respond to PR updates, reply to review comments, and re-review on demand
- [ ] **Comprehensive testing** - Unit tests, integration tests, and CI/CD pipeline
- [ ] **Improved diff handling** - Better context awareness for large PRs
- [ ] **Configuration hot-reloading** - Update review settings without restart
- [ ] **Web dashboard** - UI for managing configurations and viewing review history

## 🤝 Contributing

Contributions are more than welcome! 🫶

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test with a real PR
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Submit a pull request (and watch Cyclone review it! 🌪️)

## 📄 License

This project is released into the public domain under the [Unlicense](https://unlicense.org/).

---

**Built by Thomas Pokorny** 🌪️