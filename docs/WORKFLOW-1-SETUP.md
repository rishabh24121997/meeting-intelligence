# Workflow 1: Meeting Receiver via Webhook

## Overview
Receives meeting transcript data via HTTP POST, validates required fields, returns confirmation.

## Setup Instructions

### Webhook Node Configuration
- HTTP Method: POST
- Authentication: None
- URL: (https://rishabh-sharma.app.n8n.cloud/webhook-test/865f271e-decb-46c4-a631-e1514ba39d5c)

### Data Validation
Required fields:
- title: Meeting title (string)
- date: Meeting date (YYYY-MM-DD format)
- participants: List of attendees (array)
- transcript: Full meeting transcript (string)

### Testing
Use Postman or curl with test data provided above.

## Success Criteria
- Webhook returns 200 status
- Response includes message: "Meeting received and validated"
- Execution shows in n8n Executions tab