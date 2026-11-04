VendorFlow — Purchase Order Follow-up Automation
A small procurement automation workflow built with n8n, JavaScript, Google Sheets, and Gmail. It checks pending purchase orders, identifies delayed deliveries, sends supplier reminders or manager escalations, and records the follow-up activity.

The goal is simple: replace the daily manual task of reviewing an order tracker and drafting follow-up emails with a repeatable, rules-based process.

Stack: n8n Cloud · JavaScript (Code node) · Google Sheets API · Gmail API
Workflow overview

Daily 9 AM Trigger
       |
       v
Get Purchase Orders (Google Sheets)
       |
       v
Evaluate Overdue POs (JavaScript)
       |
       v
Route by Action (Switch)
       |                     |
       v                     v
Supplier Reminder       Manager Escalation
       |                     |
       +----------+----------+
                  |
                  v
       Update Purchase Order
                  |
                  v
       Append Follow-up History
How it works
1. Scheduled check: n8n starts the workflow at 9:00 AM in the configured workflow timezone (Asia/Kolkata).
2. Read orders: A Google Sheets node retrieves purchase-order rows.
3. Evaluate each order: A JavaScript Code node checks the order status, delivery date, and previous follow-up information.
4. Route the action: A Switch node sends eligible records to either the supplier-reminder or manager-escalation branch.
5. Send an email: Gmail sends the appropriate message using a connected account.
6. Track the result: Google Sheets updates the original purchase order and appends an entry to the follow-up-history tab.
Follow-up rules
Condition	Action
Delivered or cancelled	Skip
Pending, delivery date not passed	Skip
Pending, 1–2 days overdue, no prior reminder	Send one supplier reminder
Pending, 3+ days overdue, not previously escalated	Send one manager escalation
Reminder or escalation already recorded	Skip that action


Important: With the current rules, an order already 3+ days late can go straight to manager escalation; a supplier reminder is not guaranteed to have been sent first. The workflow is designed for one initial reminder and one escalation, not daily repeated reminders.
Sample run
The workflow was tested with four fictional purchase orders using 5 November 2026 as the evaluation date:
PO ID	Material	Expected delivery	Status	Decision
PO-101	Steel Rods	2026-11-01	Pending	Escalate: 4 days overdue
PO-102	Machine Bearings	2026-11-03	Pending	Reminder: 2 days overdue
PO-103	Iron Sheets	2026-11-12	Pending	Skip: not overdue
PO-104	Safety Equipment	2026-10-30	Delivered	Skip: already delivered


In the n8n test, the sheet-read node returned 4 orders, the evaluation node produced 2 follow-up actions, and the Switch node routed 1 reminder and 1 escalation. Both Gmail branches subsequently completed a send operation in a test run.

The write-back and history steps need to be checked together before treating the workflow as a fully validated end-to-end process; a successful node execution alone doesn't prove that each action was logged exactly once.
Google Sheets structure
Create one Google Spreadsheet with two tabs. Keep the column names exactly as shown because the Code node reads them by name.

Tab 1: Purchase Orders
PO ID | Material | Supplier | Supplier Email | Delivery Date | Status | Follow-up Status | Reminder Count | Last Reminder Date | Escalated | Escalated Date
Tab 2: Follow-up History
Date | PO ID | Material | Supplier | Action | Days Overdue | Sent To
A fictional starter dataset is included in [`sample-purchase-orders.csv`](sample-purchase-orders.csv). Replace the sample email addresses with your own test inbox before running the email nodes. Use dates in YYYY-MM-DD format.
Run it in your own n8n instance
1. Set up an n8n Cloud instance or run the self-hosted Community Edition.
2. Import the exported workflow JSON into the n8n editor. (Add the sanitized workflow.json to this repository before sharing it.)
3. Create the Google Spreadsheet and its two tabs with the headers above.
4. Connect your own Google Sheets and Gmail OAuth credentials in n8n. Credentials are not meant to be included in the public workflow export.
5. In all Google Sheets nodes, select your spreadsheet and the correct tab:
   - Get Purchase Orders → Get Row(s) → Purchase Orders
   - Record Follow-up on PO → Update Row, match on PO ID → Purchase Orders
   - Log Follow-up History → Append Row → Follow-up History
6. In Escalate to Manager, put a real test email address in the Gmail To field. The text Manager is only a label, not a deliverable email address.
7. Set the workflow timezone to Asia/Kolkata and test one node at a time. Confirm the email recipients before executing a Gmail node.
8. After verifying that order rows are updated and one history record is added per successful action, publish/activate the workflow if you want scheduled runs.
A GitHub repository does not host or execute the automation. The workflow runs only while its n8n instance is available. The n8n Cloud trial is temporary; the exported workflow can also be imported into a self-hosted instance.
Testing checklist
- [x] Read four sample purchase orders from Google Sheets
- [x] Detect the two overdue pending orders
- [x] Route one record to Reminder and one to Escalate
- [x] Send test emails using Gmail nodes
- [ ] Verify both purchase-order rows are updated without changing Material or Supplier
- [ ] Verify both follow-up actions produce separate, accurate history rows
- [ ] Re-run on unchanged data and confirm no duplicate notifications
- [ ] Test missing email addresses and failed send/update behavior
- [ ] Verify published workflow runs at the expected scheduled time
The unchecked cases are intentionally listed as remaining validation, not completed features.
Troubleshooting notes
Issue	What to check
Invalid To header from Gmail	The recipient must be a valid email address, not a placeholder or the word Manager.
New row created instead of updating an order	Use Update Row, choose Purchase Orders, and match on PO ID.
Material or Supplier gets overwritten	Remove those fields from the Values to Update mapping; only map the follow-up columns.
History shows 2 under Sent To	Map Sent To to the recipient field, not daysOverdue.
Only one history entry after two actions	Check how the two Gmail branches feed the shared update and append nodes; verify each item reaches the history step.
Repeated reminder emails	Confirm the updated reminder count / escalation flag is saved before the next scheduled run.


Project scope
This is a learning / portfolio prototype built with fictional purchase orders. It demonstrates scheduled workflow automation, API integration, conditional routing, state updates, and basic audit history. It is not an ERP integration or a production-grade procurement system. For production use, it would need stronger idempotency, failure recovery, monitoring, access control, and tests around partial failures.
Security and sharing
- Do not commit Google OAuth credentials, tokens, live spreadsheet IDs, or personal email addresses.
- Review the exported n8n workflow JSON before publishing it. Remove or replace private values and credential references.
- Use fictional supplier data in screenshots and examples.
- Never enable a public form that sends unrestricted emails to addresses supplied by strangers.
Demo: A short screen recording can be linked here after final end-to-end validation.
