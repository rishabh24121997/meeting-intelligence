# System Architecture

## Overview

The AI Meeting Intelligence Engine is a **serverless, event-driven workflow automation system** designed to process meeting transcripts at scale.

## Data Flow

INPUT LAYER
│
├─ Webhook Receiver
│  ├─ Accepts POST requests with meeting data
│  ├─ Validates JSON schema
│  └─ Returns 200/400 status
│
↓
PROCESSING LAYER
│
├─ Data Validation
│  ├─ Check required fields
│  ├─ Validate date format
│  └─ Verify transcript length
│
├─ Parallel OpenAI Calls
│  ├─ Summarization (500 max tokens)
│  ├─ Action Items (1000 max tokens)
│  └─ Risk Analysis (800 max tokens)
│
├─ JSON Parsing
│  └─ Convert OpenAI responses to structured data
│
├─ Data Enrichment
│  ├─ Add timestamps
│  ├─ Calculate metrics
│  └─ Assign priorities
│
↓
OUTPUT LAYER
│
├─ Notion Storage
│  ├─ Create/update meeting records
│  ├─ Index for full-text search
│  └─ Link action items to owners
│
├─ Slack Distribution
│  ├─ Send summary to #meetings channel
│  ├─ Mention owners of action items
│  └─ Highlight high-priority risks
│
├─ Email Notifications
│  ├─ Send executive summary
│  ├─ Attach action items list
│  └─ Flag blockers
│
├─ Jira Integration
│  ├─ Create tickets for action items
│  ├─ Assign to owners
│  └─ Set due dates
│
↓
LOGGING & MONITORING
│
├─ Webhook.site Logging
│  ├─ Track all requests
│  ├─ Monitor processing times
│  └─ Alert on failures
│
├─ Cost Tracking
│  ├─ Calculate API costs
│  ├─ Monitor usage trends
│  └─ Optimize token usage
│
└─ Execution Logs
├─ n8n native execution history
├─ Error tracking
└─ Performance metrics

## Component Breakdown

### 1. Input Layer (Webhook)

**Purpose**: Accept meeting data from external sources

**Input Schema**:
```json
{
  "title": "string (required)",
  "date": "YYYY-MM-DD (required)",
  "participants": ["string array (required)"],
  "transcript": "string (required)"
}
```

**Validation**:
- Title: Non-empty string
- Date: Valid YYYY-MM-DD format
- Participants: Array of 1+ items
- Transcript: Min 100 characters

**Output**: Validated JSON or 400 error

### 2. Processing Layer (OpenAI)

**Parallel Execution**: 3 independent OpenAI calls

**Call 1: Summarization**
- Model: GPT-4o
- Max tokens: 500
- Temperature: 0.3
- Purpose: Executive summary (2-3 sentences)

**Call 2: Action Items**
- Model: GPT-4o
- Max tokens: 1000
- Temperature: 0.2
- Purpose: Extract structured tasks with owners/deadlines

**Call 3: Risk Analysis**
- Model: GPT-4o
- Max tokens: 800
- Temperature: 0.2
- Purpose: Identify blockers and dependencies

**Data Transformation**:
- Parse JSON responses
- Validate structure
- Handle parsing errors

### 3. Output Layer (Distribution)

**Notion Integration**:
- Database: "Meetings" table
- Fields: Title, Date, Summary, Action Items, Risks
- Searchable by content, date, owner
- Status tracking for action items

**Slack Integration**:
- Channel: #meeting-summaries
- Message format: Blocks format
- Mentions: Task owners
- Threads: Detailed information

**Email Integration**:
- Recipients: Meeting participants
- Format: HTML email
- Attachments: CSV of action items
- Follow-up: Calendar invite for review

**Jira Integration**:
- Project: Configured per team
- Issue type: Task
- Fields: Summary, Description, Assignee, Due Date
- Labels: meeting, [meeting-title]

### 4. Monitoring & Logging

**Request Logging**:
- Webhook.site endpoint
- All requests logged with timestamp
- Enable trend analysis

**Cost Tracking**:
- Calculate tokens per call
- Estimate USD cost
- Track monthly spending

**Error Handling**:
- Catch OpenAI failures
- Log to Slack #errors channel
- Retry logic for transient failures

## Design Decisions

### Why Parallel OpenAI Calls?

Instead of one call asking for everything:
INEFFICIENT: "Summarize, extract action items, identify risks"
→ Returns one giant response
→ Hard to parse
→ Expensive (more tokens)
EFFICIENT: 3 separate calls
→ Each optimized for one task
→ Easier to debug
→ Can retry individual calls
→ Better prompt engineering

### Why Notion?

- Free tier suitable for most teams
- Rich API for programmatic access
- Good UI for browsing meetings
- Supports linked databases for cross-referencing

### Why Webhook vs Email/Slack Input?

- Webhook: Programmatic, reliable, fast
- Email: Messy parsing, slow
- Slack: Rate limited, requires interaction

Start with webhook → add email/Slack later

## Scaling Considerations

### Current Limits
- n8n Free: 10,000 executions/month (~320 meetings/month)
- OpenAI: Whatever you pay for
- Notion: Unlimited (free tier has some limits)

### To Scale to 10x

1. Upgrade n8n to Pro ($20/month)
2. Add request queuing (prevent rate limits)
3. Batch process meetings (process 5 at once)
4. Cache common patterns (reduce API calls)
5. Archive old meetings (keep Notion fast)

### To Scale to 100x

1. Self-host n8n (Docker)
2. Use OpenAI batch API
3. Archive to S3 instead of Notion
4. Add search index (Elasticsearch)

## Security Model

### Data Flow
Public Internet
↓
n8n Cloud (SSL/TLS)
↓
Internal Workflow (Encrypted)
↓
Third-party APIs (OAuth/Keys)
↓
Notion Database (n8n managed)

### Credential Management

- API keys: Stored in n8n vault (encrypted)
- No keys in workflow code
- No keys logged to external services
- Rotate keys monthly

### Access Control

- Webhook: No authentication (can add Bearer token later)
- Notion: API key managed by n8n
- OpenAI: API key managed by n8n
- Slack: OAuth token managed by n8n

## Cost Model

### Per-Meeting Costs
Summarization call:

Input: ~1000 tokens = $0.003
Output: ~150 tokens = $0.0009

Action items call:

Input: ~1000 tokens = $0.003
Output: ~300 tokens = $0.0018

Risks call:

Input: ~1000 tokens = $0.003
Output: ~200 tokens = $0.0012

TOTAL: ~$0.015-0.020 per meeting

### Monthly Estimates

- 10 meetings/day: $4.50-6/month
- 100 meetings/day: $45-60/month
- 1000 meetings/day: $450-600/month

## Future Architecture

### Version 2.0

- Real-time transcription (Zoom/Meet integration)
- Calendar trigger (auto-process meetings)
- Action item status tracking (JIRA polling)
- Recurring reminders
- Team dashboards

### Version 3.0

- Custom LLM fine-tuning
- Speaker emotion analysis
- Multi-language support
- Meeting attendance tracking
- Automated document generation
