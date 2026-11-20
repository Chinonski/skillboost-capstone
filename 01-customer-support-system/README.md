# AI-Powered Customer Support & Order Management System

An automated, multi-branch customer support triage and order fulfillment workflow built in **n8n**, integrating **Tally Forms**, **OpenAI**, **Google Sheets**, and **Gmail**.

---

## Architecture & Workflow Flow

```text
[Tally Support Form Trigger]
        │
        ▼
[Normalize Ticket Fields (Edit Fields)]
        │
        ▼
[Triage & Urgency Classifier (AI Agent + OpenAI + Structured JSON Parser)]
        │
        ▼
[Log Ticket to Enquiries_Log (Google Sheets: Append Row)]
        │
        ▼
[Route by Urgency & Category (Switch)]
   ├── Output 0: Human Escalation ──► Customer Holding Email ──► Internal Support Alert ──► Mark Escalated in Log
   ├── Output 1: Order Tracking ────► Lookup Orders_DB ────────► Draft Grounded Reply ──────► Send Email ──► Log Sent
   ├── Output 2: Product Enquiry ───► Draft Product Reply ────► Send Email ──────────────► Log Sent
   └── Output 3 + Fallback: General ► Draft General Reply ────► Send Email ──────────────► Log Sent

```

---

## Core Components

### 1. Intake & Data Normalization

* **Tally Support Form Trigger:** Captures incoming customer submissions (`Customer Name`, `Customer Email`, `Order ID`, and `Enquiry Message`).
* **Normalize Ticket Fields (`Edit Fields`):** Standardizes incoming payloads into flat variables, normalizes `order_id` formatting, and generates a unique `Ticket_ID` (`TKT-YYYYMMDD-HHmmss`) alongside an ISO `Timestamp`.

### 2. AI Triage & Structured Classification

* **Triage & Urgency Classifier (`AI Agent`):** Powered by an OpenAI Chat Model and paired with a **Structured Output Parser** to enforce a strict JSON schema:
* **`category`:** `Order Tracking`, `Product Enquiry`, `Refund Request`, `Complaint`, or `General Enquiry`.
* **`urgency`:** `Low`, `Medium`, or `High`.
* **`summary`:** A concise 1-sentence internal summary for support agents.
* **`requires_human`:** Boolean flag set to `true` whenever a ticket is a `Complaint`, `Refund Request`, or `High` urgency.



### 3. Conditional Routing (`Switch` Node)

Evaluates rules in strict priority order so urgent issues never receive generic auto-replies:

1. **Output 0 — Human Escalation (`requires_human == true`):** Catches complaints, refund requests, and high-urgency tickets. Sends an immediate holding email to the customer, dispatches an internal escalation alert with the AI summary to the human support inbox, and updates the log status.
2. **Output 1 — Order Tracking (`category == "Order Tracking"`):** Queries the `Orders_DB` sheet by `Order_ID` (with **Always Output Data** enabled to handle invalid/missing IDs gracefully) and uses a grounded **Basic LLM Chain** to draft an email strictly from verified database records—preventing AI hallucinations.
3. **Output 2 — Product Enquiry (`category == "Product Enquiry"`):** Uses a product-specialist **Basic LLM Chain** to answer product compatibility, setup, and feature questions via Gmail.
4. **Output 3 & Fallback — General Enquiry (`category == "General Enquiry"` + Extra Output):** Answers store policy, hours, and general questions using verified store reference rules, with the **Extra Output** fallback ensuring uncategorized edge cases never drop.

---

## Database Schema (`Customer_Support_System` Google Sheet)

### Tab 1: `Enquiries_Log` (Audit & Ticket Lifecycle Log)

| Column | Description |
| --- | --- |
| `Timestamp` | ISO timestamp of form submission |
| `Ticket_ID` | Auto-generated unique support identifier (`TKT-...`) |
| `Customer_Name` | Customer's full name |
| `Customer_Email` | Customer's email address |
| `Order_ID` | Submitted order reference (if applicable) |
| `Enquiry_Message` | Raw customer support message |
| `AI_Category` | Classified intent category |
| `AI_Urgency` | Assigned priority (`Low`, `Medium`, `High`) |
| `Status` | Real-time ticket state (e.g., `Auto-Replied (Order Tracking)`, `Escalated - Awaiting Human`) |
| `AI_Response_Sent` | Exact email body dispatched or escalation note |

### Tab 2: `Orders_DB` (Live Order Lookup Source)

| Column | Description |
| --- | --- |
| `Order_ID` | Primary lookup key (e.g., `ORD-1001`) |
| `Customer_Email` | Email associated with the order |
| `Product_Name` | Purchased item name |
| `Order_Status` | Current fulfillment state (`Processing`, `Shipped`, `Delivered`) |
| `Tracking_Number` | Carrier tracking code (`TRK-...` or `N/A`) |
| `Estimated_Delivery` | Scheduled delivery date (`YYYY-MM-DD`) |

---

## Setup & Deployment

1. **Google Sheets:** Create a spreadsheet named `Customer_Support_System` containing `Enquiries_Log` and `Orders_DB` with the headers above.
2. **Tally Form:** Create a support intake form with fields for Name, Email, Order ID (optional), and Enquiry Message, and connect it to the **Tally Trigger** node.
3. **Credentials in n8n:** Configure credentials for **Tally**, **OpenAI API**, **Google Sheets OAuth2**, and **Gmail OAuth2**.
4. **Configure Escalation Inbox:** Update the `To` field in the **Alert Human Support Queue** Gmail node with your internal support team's email address.
5. **Activate:** Toggle the workflow to **Active** and test all four routing paths (`Order Tracking`, `Product Enquiry`, `General Enquiry`, and `Human Escalation`).

```

```
