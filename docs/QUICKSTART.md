# Quick Start Guide (5 minutes)

## Step 1: Get Your Webhook URL (2 min)

1. Log in to n8n.cloud
2. Open workflow: "1-webhook-receiver-and-openai"
3. Click Webhook node
4. Copy the "Webhook URL"
5. Save it somewhere safe

## Step 2: Test With Postman (2 min)

1. Download Postman: https://www.postman.com/downloads/
2. Create new POST request
3. Paste your webhook URL
4. Go to Body → Raw → JSON
5. Paste this:

```json
{
  "title": "Team Standup",
  "date": "2024-05-10",
  "participants": ["Alice", "Bob", "Sarah"],
  "transcript": "Alice: Good morning. We need to launch v2.0 by June 15. Bob: I'll handle the backend. Sarah: I'll manage QA testing."
}
```

6. Click Send
7. See success response with action items ✅

## Step 3: Check Results (1 min)

Go to n8n → Executions tab → Click latest execution

You should see:
- Summary extracted
- Action items identified
- Risks flagged

**That's it! Your system is working.**

## Next Steps

- Set up Notion database (see docs/SETUP.md)
- Add Slack notifications (see docs/SETUP.md)
- Configure email distribution (see docs/SETUP.md)

## Troubleshooting

**Webhook URL not working?**
- Check workflow is Activated (blue toggle)
- Verify URL is copied correctly

**OpenAI error?**
- Check API key is added in n8n credentials
- Verify OpenAI account has payment method
- Check API usage limits in OpenAI dashboard

**Bad action items?**
- Review actual transcript in execution logs
- Prompts may need adjustment for your meeting style

See `docs/TROUBLESHOOTING.md` for more.