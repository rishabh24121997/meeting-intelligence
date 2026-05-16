# API Documentation

## Webhook Endpoint

### POST /webhook

Process a meeting transcript and extract intelligence.

#### Request

```bash
POST https://[your-n8n-domain]/webhook/[webhook-id]
Content-Type: application/json
```

#### Request Body

```json
{
  "title": "Q2 Planning Meeting",
  "date": "2024-05-10",
  "participants": ["Alice", "Bob", "Sarah"],
  "transcript": "Alice: Let's discuss Q2 roadmap..."
}
```

#### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Meeting title |
| `date` | string | Yes | Meeting date (YYYY-MM-DD) |
| `participants` | array | Yes | List of attendees |
| `transcript` | string | Yes | Full meeting transcript (min 100 chars) |

#### Response (Success - 200)

```json
{
  "status": "success",
  "message": "Meeting received and validated",
  "summary": "The team discussed Q2 roadmap and agreed to launch v2.0 by June 15...",
  "action_items": [
    {
      "item": "Develop backend API",
      "owner": "Bob",
      "deadline": "June 15"
    },
    {
      "item": "Manage QA testing",
      "owner": "Sarah",
      "deadline": "June 15"
    }
  ],
  "risks": [
    {
      "blocker": "Security approval pending",
      "severity": "HIGH",
      "context": "Required before testing begins"
    }
  ],
  "meeting_title": "Q2 Planning Meeting",
  "meeting_date": "2024-05-10",
  "participants": ["Alice", "Bob", "Sarah"],
  "metrics": {
    "total_action_items": 2,
    "total_risks": 1
  }
}
```

#### Response (Validation Error - 400)

```json
{
  "status": "error",
  "message": "Invalid input",
  "required_fields": {
    "transcript": "string (required, min 100 chars)",
    "title": "string (required)",
    "date": "YYYY-MM-DD format (required)",
    "participants": "array of strings (required)"
  }
}
```

#### Response (Processing Error - 500)

```json
{
  "status": "error",
  "message": "Processing failed",
  "error_type": "openai_api_error",
  "error_details": "Rate limited by OpenAI",
  "timestamp": "2024-05-10T14:30:00Z",
  "recommendation": "Retry in 60 seconds"
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | "success" or "error" |
| `summary` | string | 2-3 sentence executive summary |
| `action_items` | array | List of tasks with owners |
| `risks` | array | Identified blockers with severity |
| `metrics` | object | Processing metrics (items count, cost) |

#### Example: Using cURL

```bash
curl -X POST https://[your-webhook-url] \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Team Standup",
    "date": "2024-05-10",
    "participants": ["Alice", "Bob"],
    "transcript": "Alice: We need to ship v2.0 by Friday. Bob: I will handle the backend."
  }'
```

#### Example: Using Python

```python
import requests
import json

webhook_url = "https://[your-webhook-url]"

data = {
    "title": "Team Standup",
    "date": "2024-05-10",
    "participants": ["Alice", "Bob"],
    "transcript": "Alice: We need to ship v2.0 by Friday. Bob: I will handle the backend."
}

response = requests.post(
    webhook_url,
    json=data,
    headers={"Content-Type": "application/json"}
)

print(response.json())
```

#### Example: Using JavaScript

```javascript
const webhookUrl = "https://[your-webhook-url]";

const data = {
  title: "Team Standup",
  date: "2024-05-10",
  participants: ["Alice", "Bob"],
  transcript: "Alice: We need to ship v2.0 by Friday. Bob: I will handle the backend."
};

fetch(webhookUrl, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(data)
})
  .then(res => res.json())
  .then(data => console.log(data));
```

#### Rate Limiting

- n8n Free: 10,000 executions/month
- No per-minute limits
- OpenAI: Standard API limits (varies by account)

#### Timeout

- Webhook response: 2 seconds
- Full processing: 8-15 seconds

#### Errors

| Code | Reason | Fix |
|------|--------|-----|
| 400 | Invalid input | Check required fields |
| 500 | OpenAI API error | Check OpenAI status page |
| 502 | n8n service error | Retry in 60 seconds |
| 503 | Rate limited | Wait 60 seconds and retry |

---

## Integration Examples

### Integrating with Slack Bot

[Add after Slack integration is built]

### Integrating with Email

[Add after email integration is built]

### Integrating with Jira

[Add after Jira integration is built]