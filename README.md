# Autonomous AI SDR 

An autonomous AI-powered sales development workflow built in n8n that continuously sources ICP leads, enriches them with company context, scores them against the ideal customer profile, sends personalized cold emails, and handles replies automatically.

This workflow is designed to act like a junior SDR operating around the clock: it imports leads from Apollo, creates or updates HubSpot records, researches the prospect/company, writes tailored outreach, sends emails through Gmail, monitors replies, routes conversations by intent, books meetings on Google Calendar, and updates the CRM automatically.

---

## Overview

The repository contains a single n8n workflow export:

- `Autonomous AI SDR/Autonomous AI SDR.json`
- `Autonomous AI SDR/Autonomous AI SDR.png`

The workflow is a full outbound sales automation system with the following lifecycle:

1. Pull ICP leads from Apollo.io
2. Validate and create HubSpot contacts
3. Revisit existing contacts each morning
4. Enrich prospects with LeadIQ and live company research
5. Use an AI agent to score fit and draft personalized outreach
6. Send cold emails through Gmail
7. Monitor replies in Gmail
8. Classify reply intent
9. Automatically send meeting booking responses to interested prospects
10. Book meetings on Google Calendar
11. Update HubSpot and Slack with outcomes

This is not just a simple email workflow — it is a practical, operational SDR engine for outbound prospecting.

---

## What This Workflow Does

This automation does the following:

- Finds targets based on job title and company filters
- Pulls only verified email prospects from Apollo
- Creates new leads in HubSpot with initial CRM fields
- Runs a daily outreach loop for contacts that still need engagement
- Enriches each lead with additional context from LeadIQ
- Uses company news and research via SerpAPI to personalize outreach
- Uses an AI SDR agent to:
  - Score each lead against ICP
  - Identify the top pain points and business context
  - Draft a high-converting cold email subject + body
- Sends only high-fit leads
- Monitors inbound replies
- Classifies responses into:
  - INTERESTED
  - NOT_INTERESTED
  - QUESTION
  - OOO
- Sends a meeting-booking email to interested prospects
- Creates Google Calendar meetings
- Updates HubSpot lifecycle stages, lead status, and meeting data
- Posts daily activity updates to Slack

---

## Architecture at a Glance

The workflow is organized into several stages:

### 1. Lead Ingestion
- `Apollo Daily 7am Trigger`
- `Apollo Pull ICP Leads`
- `Parse Apollo Leads`
- `Has Valid Email?`
- `HubSpot Create Apollo Lead`
- `Count Apollo Leads`
- `Slack Apollo Import Summary`

### 2. Daily Outreach Cycle
- `Daily 8am Trigger`
- `Get HubSpot Contacts`
- `Filter Contacts for Outreach`
- `LeadIQ Enrich Lead`
- `AI SDR Email Writer Agent`
- `Parse AI Email Response`
- `ICP Score 6+?`
- `Gmail Send Email`
- `HubSpot Update Contact`
- `HubSpot Create Deal`
- `Slack Daily Summary`

### 3. Reply Handling
- `Gmail Check Replies`
- `Reply Classifier Agent`
- `Parse Classification`
- `Route by Reply Type`

### 4. Interested Prospect Follow-up
- `Interested — Meeting Booking Agent`
- `Parse Interested Reply`
- `Gmail Send Interested Reply`
- `Google Calendar Book Meeting`
- `HubSpot Mark Meeting Booked`
- `Slack Meeting Booked Alert`

### 5. Low-ICP Handling
- `HubSpot Mark Low ICP`

---

## Key Features

- Apollo-powered lead sourcing
- ICP-based qualification
- AI score + personalized outreach generation
- CRM-driven contact updates
- Automated deal creation in HubSpot
- Gmail outreach automation
- Automated reply classification
- AI-driven meeting booking follow-up
- Google Calendar booking integration
- Slack notifications
- Daily SDR loop without manual intervention

---

## Workflow Logic

### Lead import flow
At 7:00 AM daily, the workflow triggers and fetches targeted leads from Apollo using search criteria such as:

- job titles:
  - CEO
  - Founder
  - Co-Founder
  - VP Sales
  - Head of Sales
  - Director of Sales
- company size:
  - 11–50 employees
  - 51–200 employees
- geography:
  - United States
  - United Kingdom
  - Canada
  - Australia
- contact quality:
  - verified email only
- business verticals:
  - SaaS
  - software
  - ecommerce
  - marketing

The workflow then parses the Apollo payload, filters out empty/invalid emails, and pushes valid leads into HubSpot as new contacts with fields such as:

- email
- firstname
- lastname
- company
- jobtitle
- website
- LinkedIn
- outreach status
- ICP score
- sequence day
- personalization notes
- company news

A Slack summary is sent after the import completes.

### Daily outreach flow
At 8:00 AM daily, the workflow gets active HubSpot contacts that remain eligible for outreach.

It filters contacts according to internal rules, such as:

- valid email available
- not already closed
- not a previously contacted high-priority conversation
- eligible for outreach based on sequence state and date

Then the workflow:

- enriches the lead via LeadIQ
- researches company activity with SerpAPI
- uses OpenRouter + a Nemotron-based model to score the prospect against the ICP
- generates a sales email tailored to the company and role
- sends the email with Gmail
- updates HubSpot with outreach result, score, notes, and follow-up metadata
- creates a HubSpot deal if the lead is qualified
- posts a summary to Slack

### AI email behavior
The AI agent is instructed to:

- score lead fit 1–10 versus ICP
- identify high-priority pain points
- research the latest relevant company news
- write a personalized email subject
- produce a strong outreach email body
- return structured output for downstream processing

The workflow only proceeds when:
- `icp_score >= 6`
- email exists
- email body is present

This prevents low-fit leads from being contacted.

### Reply handling logic
The system checks Gmail for new unread emails in the inbox and classifies the sender's response.

Classification categories:

- INTERESTED
- NOT_INTERESTED
- QUESTION
- OOO

These replies are then routed through the workflow:

- Interested → send a warm follow-up and book a meeting
- Not Interested → mark contact as closed / not interested
- Question → handle with a Q&A follow-up or route manually
- OOO → defer and continue later

### Meeting booking flow
If a prospect responds positively, the workflow:

- crafts a personalized follow-up message
- sends the email to the prospect
- books a meeting in Google Calendar
- updates the HubSpot contact status to `Booked`
- marks the lifecycle stage as `salesqualifiedlead`
- sends a Slack alert to the team

---

## Nodes & Tools Used

| Category | Nodes / Services |
|----------|------------------|
| **Scheduling** | Schedule Trigger (7 AM, 8 AM) |
| **Lead Sourcing** | Apollo.io API |
| **Data Processing** | Code nodes (JavaScript) |
| **CRM** | HubSpot (contacts, deals, custom properties) |
| **Lead Enrichment** | LeadIQ API |
| **Research** | SerpAPI (company research) |
| **AI/LLM** | OpenRouter (Nemotron 70B model) |
| **Agents** | LangChain Agent nodes |
| **Email** | Gmail API |
| **Calendar** | Google Calendar API |
| **Notifications** | Slack API |
| **Routing** | IF conditions, Switch statements |

---

## Prerequisites

Before importing and running this workflow, you need:

- **n8n instance** (cloud or self-hosted)
- **Apollo.io account** with API access
- **HubSpot account** with OAuth2 credentials
- **LeadIQ API** access
- **SerpAPI account** with API key
- **OpenRouter API key** for model access
- **Gmail account** with OAuth2 credentials
- **Google Calendar** access
- **Slack app** with OAuth2 credentials
- Proper permissions to create/update HubSpot records and send emails

---

## Setup & Usage

### 1. Import the workflow
Import the workflow JSON into your n8n instance:

```
Autonomous AI SDR/Autonomous AI SDR.json
```

Via: **Settings → Workflows → Import**

### 2. Configure credentials
Create or attach n8n credentials for:

- Apollo API (Header Auth)
- HubSpot OAuth2
- LeadIQ (Bearer Token)
- SerpAPI (API Key)
- OpenRouter (API Key)
- Gmail OAuth2
- Google Calendar OAuth2
- Slack OAuth2

### 3. Update Apollo search criteria
Customize the Apollo search parameters in the "Apollo Pull ICP Leads" node:

- Job titles (CEO, VP Sales, etc.)
- Company size ranges
- Geographic locations
- Industry keywords

### 4. Set schedule timing
Adjust the scheduled trigger times if needed:

- 7:00 AM → Apollo lead import
- 8:00 AM → Daily outreach cycle

### 5. Configure HubSpot custom properties
Ensure your HubSpot account has these custom properties:

- `outreach_status` (string)
- `icp_score` (number)
- `sequence_day` (number)
- `personalization_notes` (text)
- `company_news` (text)
- `reply_classification` (string)

### 6. Update Slack channel
Replace the Slack channel ID with your target channel for notifications.

### 7. Activate workflow
Toggle the workflow to **Active** and verify the scheduled triggers are enabled.

---

## Use Cases

- **Sales teams** building daily outbound pipelines with AI-qualified leads
- **Startups** automating first-pass outreach without hiring SDRs
- **B2B agencies** scaling cold outreach campaigns for multiple clients
- **SaaS companies** generating consistent top-of-funnel opportunities
- **Consultancies** qualifying inbound interest and booking discovery calls
- **Solo founders** automating sales without outsourcing
- **Revenue teams** reducing manual prospecting time while increasing quality

---

## Benefits

- **Fully automated** lead generation and outreach
- **AI-powered** ICP scoring and personalization
- **24/7 operation** without manual intervention
- **Higher reply rates** through contextual, personalized emails
- **Faster sales cycles** with automated meeting booking
- **CRM sync** for visibility and handoff to sales
- **Lower cost** compared to hiring dedicated SDRs
- **Scalable** pipeline growth without proportional headcount increase

---

## Best Practices

- Keep Apollo search criteria tightly scoped to avoid low-quality volume
- Review and adjust ICP score thresholds based on conversion data
- Validate AI-generated email personalization before broad rollout
- Monitor reply patterns to continuously improve AI prompts
- Use HubSpot custom properties to track outreach progression
- Schedule the workflow during business hours in your target timezone
- Maintain opt-out and unsubscribe handling per email regulations
- Test with a small segment before scaling to high volume

---

## Security & Compliance

- Never expose API keys in exported workflow JSON
- Rotate credentials regularly and use environment variables where possible
- Restrict Gmail/HubSpot/Google Calendar scopes to minimum necessary permissions
- Ensure outbound email compliance with CAN-SPAM, GDPR, and other regulations
- Maintain proper unsubscribe and opt-out mechanisms
- Review AI-generated content for brand voice and compliance before production
- Monitor for deliverability issues and adjust sending patterns as needed

---

## Troubleshooting

**No leads imported from Apollo:**
- Verify Apollo API credentials
- Check search criteria match available leads
- Confirm per_page and page limits

**Emails not sending:**
- Verify Gmail credentials and OAuth2 scopes
- Check email validation logic passes
- Review ICP score threshold

**HubSpot updates not working:**
- Confirm OAuth2 credentials
- Verify custom property names match exactly
- Check HubSpot API limits

**Meetings not booking:**
- Verify Google Calendar OAuth2 access
- Ensure calendar ID is correct
- Check for calendar availability

**Low reply rates:**
- Review AI prompt and personalization quality
- Verify email content is engaging and relevant
- Check Gmail deliverability and spam folder
- Adjust ICP score threshold if too many poor-fit leads

---

## Summary

**Autonomous AI SDR** is a production-ready, fully-automated outbound sales system. It combines lead generation, AI-based qualification, personalized cold outreach, CRM management, reply handling, and meeting automation into a single, cohesive n8n workflow.

It is designed for sales teams, founders, and growth operators who want to scale outbound prospecting without proportional headcount growth. By leveraging AI, public data APIs, and workflow automation, this system performs the core functions of a junior SDR 24/7 — discovering prospects, researching fit, personalizing outreach, and routing qualified leads to the sales team.

---

**Built with:** n8n • LangChain • OpenRouter • Apollo.io • HubSpot • Gmail • Google Calendar • Slack
