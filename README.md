# n8n Automation and AI Agent Portfolio

Seven practical workflow projects by [Mustapha Musdhaliph Mofe](https://github.com/musdhy), covering revenue operations, customer support, finance, marketing and logistics.

These are public, configurable copies of portfolio builds. Each project includes importable n8n workflow JSON and its implementation README, including setup, data structures, prompts, controls and verification notes. They are not preconnected services and do not include credentials or access to a live database.

## Projects

| Project | What it demonstrates | Documentation |
|---|---|---|
| NovaCore Lead Handling | Website enquiry intake, company enrichment, guarded AI assessment, human review and Salesforce routing. | [View project](projects/01-novacore-lead-handling/README.md) |
| Post-Call Revenue Intelligence | Call extraction, CRM follow-up context, Gmail drafts and internal sales notifications. | [View project](projects/02-post-call-revenue-intelligence/README.md) |
| Customer Support AI Agent | Knowledge-base answers, conversation context and confirmed support-ticket creation. | [View project](projects/03-customer-support-ai/README.md) |
| Meeting Prep AI Agent | Read-only CRM and call-history research with a guarded meeting briefing. | [View project](projects/04-meeting-prep-ai/README.md) |
| AP Invoice Investigation | Invoice PDF intake, deterministic reference checks, AI investigation and internal exception reporting. | [View project](projects/05-ap-invoice-investigation/README.md) |
| Marketing Data Watchtower | Investigates discrepancies between advertising, website enquiries and CRM receipts. | [View project](projects/06-marketing-data-watchtower/README.md) |
| Shipment Exception Agent | Tracking-event deduplication, rule-based severity and AI-assisted exception drafts for a human. | [View project](projects/07-shipment-exception-agent/README.md) |

## Explore the work

Open a project's README for the architecture and setup, then its workflow JSON to inspect node parameters, agent tools, deterministic checks and error handling. The thumbnail artwork at the repository root illustrates the projects; it is not execution evidence. Original project numbers are not preserved in the directory sequence because this repository contains seven selected builds, including their later agent-based versions.

## Setup before running

1. Import the desired main workflow and its paired error workflow into your own n8n workspace.
2. Leave the main workflow unpublished while configuring it. Attach your own saved credentials; exported credential IDs are empty.
3. Replace every <YOUR_...> value in that project, including placeholders inside Code-node configuration, expressions and SQL. Set your Supabase URL, Slack channel/member IDs, any Drive folder or Salesforce instance/record IDs, webhook sender settings and supported LLM model as needed.
4. Review SQL before applying it to a separate database project. SQL may create functions, triggers, seed rows, grants and policies. Shared dependencies such as postcall_errors or earlier project's tables may need to exist; see the individual README. Existing build verification notes do not imply your database already has those objects.
5. Select the imported error workflow in the main workflow's settings. Configure webhooks, event subscriptions and OAuth callbacks for your own instance.
6. Use fictional input and verify every integration and side effect before publishing. An imported file or a local code check does not establish end-to-end success. Follow the project-specific manual approval boundaries.

## Design approach

The projects combine official n8n integration nodes and AI agents with deterministic validation, constrained tools, output guards and internal reporting. AI generates assessments and drafts; code controls identities, permissions, deduplication and approval conditions. Exact safeguards differ by project. Read each README rather than assuming every workflow has the same behavior or action permissions.

## Data, privacy and validation

Credentials, environment files, old archive ZIPs, execution exports and private runtime material are not included. Account-specific Supabase/n8n endpoints, Slack routing IDs, Drive folder IDs and credential references have been replaced or removed. The original working folders were not edited. This publication does not deploy workflows or change any external application.

Illustrative records and company/carrier names in the documentation are fictional portfolio scenarios, not claims of a commercial relationship. In particular, logistics master data and invoice scenarios are not real customer transactions. Do not submit or email addresses from a scenario as though they identify an actual contact.

Validation status is documented per project. The public copies were checked for valid JSON and obvious secret patterns; they have not been re-executed against newly configured services. Some projects have only local design/guard checks, and the repository does not claim all seven are production-tested. Supabase or n8n account access is never granted by a GitHub link.

The AP project documents synthetic PDF fixtures from the working build. The PDF fixtures are omitted from this public copy because they contain realistic remittance details. Create your own fictional invoices for end-to-end tests. Archived versions and generator scratch files are also omitted.

## Security

Use n8n Credentials for secrets. Give each service a separate credential record even when both use an X-Api-Key header. Never commit service-role keys, webhook secrets, OAuth client secrets or tokens. Review message destinations and approval rules before running an imported workflow. Real operational use requires your own access controls, monitoring and integration verification.
