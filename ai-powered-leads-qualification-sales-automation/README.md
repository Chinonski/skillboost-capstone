AI-Powered Lead Qualification & Sales Automation

An automated n8n sales pipeline that captures inbound client enquiries via Tally, evaluates budget, timeline, and fit using an AI Agent with structured JSON output, logs every submission and lead score to Google Sheets, builds custom HTML emails, and routes prospects via conditional logic to send tailored responses and internal team alerts.

Tech Stack: Tally · n8n · OpenAI · Google Sheets · Gmail

Workflow Architecture

[Tally Trigger] ──▶ [Code in JavaScript] ──▶ [AI Agent] ──▶ [Append row in sheet] ──▶ [Build HTML Email] ──▶ [If]
                                              ├── OpenAI Chat Model                                         ├── (True)  ──▶ [Send a message] ──▶ [Send a message2] ──▶ [Update row in sheet]
                                              └── Structured Output Parser                                  └── (False) ──▶ [Send a message1]


Node Breakdown

1. Intake, AI Evaluation & Logging

Tally Trigger: Listens for real-time webhook events whenever a potential client submits the Tally enquiry form.

Code in JavaScript: Parses and flattens Tally's nested data.fields array into clean key-value pairs (name, email, company, services, budget, timeline, and goals).

AI Agent: Evaluates the prospect's budget, timeline, and goals against agency qualification rules to assign a lead score (1–100), qualification status (true/false), reasoning, and tailored email copy.

OpenAI Chat Model: Provides the underlying LLM reasoning engine powering the AI Agent.

Structured Output Parser: Enforces a strict JSON schema on the AI Agent's output so downstream nodes reliably receive structured fields (is_qualified, lead_score, qualification_reason, personalized_email_subject, personalized_email_body).

Append row in sheet: Logs the prospect's normalized form responses, qualification status, lead score, and AI reasoning into the Leads tab in Google Sheets.

Build HTML Email: Formats the AI-generated response copy into a styled HTML email template ready for delivery.

If: Evaluates is_qualified == true to split the workflow into conditional routing branches.

2. Conditional Routing & Outputs

True Branch (Qualified Leads):

Send a message: Sends the personalized HTML discovery-call invitation (including a calendar booking link) to the qualified prospect via Gmail.

Send a message2: Sends an internal Gmail alert to the sales team summarizing the high-intent prospect's details, budget, and AI lead score.

Update row in sheet: Updates the prospect's record in Google Sheets to mark Email Sent = Yes, maintaining an accurate outreach audit trail.

False Branch (Unqualified Leads):

Send a message1: Sends a polite, personalized Gmail nurture response explaining minimum engagement thresholds and sharing helpful resources.

