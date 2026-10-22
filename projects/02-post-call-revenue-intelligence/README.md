> Public portfolio template: account-specific URLs, IDs and saved credential references were removed from this copy. Configure the `<YOUR_...>` values and attach your own credentials before use. Original development notes below describe the build, not a deployment included in this repository. Old archive ZIPs and private runtime files are deliberately excluded.

# Post-Call Revenue Intelligence System

This project archives a sales-call transcript, finds the Salesforce record, extracts structured intelligence, logs a Salesforce Task, creates a Gmail draft, and posts an internal Slack notification. It does not send email. Gmail runs before Slack so the notification can truthfully confirm that a draft exists.

## Salesforce Task mapping correction — 19 October 2026

The main JSON now uses your supplied Cloud export as its baseline. Existing credential references, node IDs, connections, channel values and other node settings are retained. Your export contained an empty Create Salesforce Task additionalFields object: the prepared subject, description and record links therefore were not being sent. Those fields are now mapped individually so they remain visible in the native node after import. The separate error workflow is unchanged.

Prepare Salesforce Task builds a readable subject ending with the stable call-key marker. Its Description maps to Salesforce Comments and includes call details, summary, action items with their owners and deadlines, objections, next steps, risk and archive references. Priority and record owner are mapped explicitly. WhoId connects the Lead or Contact; WhatId connects the Opportunity when selected. Check existing call Task recognizes both the previous and new subject formats using that marker.

Your selected status, Waiting on someone else, is preserved rather than marking unfinished work completed. Activity Date uses the earliest valid YYYY-MM-DD action-item deadline. Extraction requests a calendar date only for explicit or unambiguous relative dates anchored to the call date. Without a clear calendar date, Salesforce Due Date stays empty and Comments retain the deadline wording; no date is invented. Review extracted dates before relying on them. The mappings and date logic passed local checks, but this revision has not been run in your n8n or Salesforce. Older node descriptions below describe the original completed-call default; the imported status above is authoritative for this revision.

Import the updated main JSON in the same n8n Cloud instance to reuse its saved credential references. Verify that each existing credential is selected before testing. Existing Salesforce Tasks are not repaired retroactively. Replaying the same call is blocked by its Supabase reservation; test with a genuinely new synthetic call ID instead of clearing duplicate protection. No Salesforce, Supabase or Slack records were changed for this correction.

## Delivered and verified

The three new tables below were created in the existing Supabase project `<YOUR_SUPABASE_PROJECT_REF>`. Their columns, foreign key, unique constraint, row-level security and role access were checked. The nine pre-existing public tables were left untouched. No sample rows were inserted.

The workflow JSON files were checked locally for JSON structure, JavaScript/expression syntax, named references, connections, archive-first ordering and draft-only Gmail operations. They have **not** been imported or run in n8n, and no Salesforce, Gmail, Slack or model call has been made for this build. Node compatibility, permissions and provider responses still need your import-time tests.

Only these three files belong to this project:
- `post-call-revenue-intelligence.json`: main workflow.
- `post-call-error-handler.json`: native Error Trigger workflow.
- `README.md`: setup, SQL, node guide and recovery.

## Credentials Needed

Create credentials in n8n and select them on the matching nodes after import. Credential names are included as hints; IDs are intentionally empty. Never paste credential values into workflow parameters or exported JSON.

| n8n name | Credential type | What to connect |
| --- | --- | --- |
| PostCall Intake Header | Header Auth | A header name such as X-PostCall-Key and a strong shared value. Your transcript sender supplies that header. |
| PostCall Supabase | Supabase API | Project Host URL and service-role secret from your Supabase project settings. This is a server-side credential with broad access; never expose it to a website. |
| PostCall Salesforce | Salesforce OAuth2 API | An OAuth app with the exact callback URL shown in n8n, API scope, refresh-token access and an authorized Salesforce user. Use the correct production or sandbox login. That user needs read access to Lead, Contact, OpportunityContactRole, Opportunity and User, plus Task read/create access. |
| PostCall Gmail | Gmail OAuth2 | Authorize the work Gmail mailbox that will own the drafts. Use n8n managed OAuth where offered; otherwise configure a Google OAuth client and n8n's displayed callback URL. |
| PostCall LLM | OpenAI API | A provider API key with access to your chosen model. Select the model in Chat model. Another compatible chat-model support node can replace it. Model usage may cost money. |
| PostCall Slack | Slack API | Bot token with chat:write, users:read and users:read.email. If the credential test requests profile access, add users.profile:read. Reinstall the Slack app after changing scopes and invite the bot to all target channels. |
| PostCall n8n API | n8n API | An n8n public API key and API base URL, normally https://YOUR-INSTANCE.app.n8n.cloud/api/v1. Grant execution read access where scoped keys are offered. |

The HTTP Request nodes reuse the Supabase, Slack or n8n predefined credentials; they need no additional plaintext header keys. No community nodes are required. Supabase HTTP nodes are used for atomic insert-on-conflict and conditional updates; the native Supabase node handles the initial raw archive and error insert.

**n8n Cloud free trial limitation:** the public API is unavailable during the trial. Error Trigger supplies error metadata but not the failed node's input. Without API access, the error workflow still attempts logging and alerting, but records `failed_input.available=false` and an explanation. Full failed-input capture requires API access and a saved failed execution. This limitation is not presented as tested or solved by substituting trigger data.

## Import and setup

1. Import the error-handler JSON, then the main JSON into your n8n Cloud instance. Leave both unpublished while configuring them.
2. Attach the named credentials. In every Supabase HTTP Request URL replace `<YOUR_SUPABASE_URL>` with the project Host URL, with no trailing slash. The credential Host must point to that same project.
3. In **Chat model**, replace `<YOUR_LLM_MODEL>` with a model available to your connected account.
4. In **Prepare Slack message**, replace `<POSTCALL_SALES_CHANNEL_ID>` and `<POSTCALL_FALLBACK_CHANNEL_ID>` with internal Slack channel IDs. In **Alert Slack admin**, replace `<POSTCALL_ADMIN_CHANNEL_ID>`. Invite the connected bot to each channel.
5. In **Read failed execution**, replace `<YOUR_N8N_CLOUD_URL>` with your n8n instance origin, without a trailing slash. Attach the n8n API credential if API access is available.
6. In main workflow Settings, select the imported **Post-Call - Global Errors** as Error workflow. Keep failed production execution data saved. Progress saving is off; successful execution data and manual execution saving are disabled in the export to reduce unnecessary transcript copies.
7. Inspect each node for missing settings after import. Test with invented call data before publishing. Use the test Webhook URL only while listening; use the production URL after publication.

In production, each SDR should connect their own work Gmail or use an explicitly designed mailbox-routing arrangement. This build uses one Gmail credential: every draft lands in that connected mailbox, even when the Salesforce owner is someone else.

## Input and response

Send a JSON POST to the URL displayed by **Receive transcript**, with the configured authentication header. This is a fake example, not a live request:

```json
{
  "call_id": "demo-call-001",
  "call_date": "2026-10-19T10:00:00+01:00",
  "attendee_name": "Demo Customer",
  "attendee_email": "test+001@example.com",
  "transcript": "Demo Customer: We spend six hours each week copying enquiries into our CRM. Rep: I will prepare a workflow proposal by Friday. Demo Customer: Please include pricing. Our main concern is implementation time."
}
```

The first database node saves the exact transcript string without trimming it, the submitted metadata as text, the execution ID and a database-generated received timestamp. Missing or malformed metadata remains visible in that raw archive. JSON or authentication failures rejected by n8n before the Webhook executes cannot be archived by a later node. Database outages can also prevent archival; no workflow can promise persistence when its first database write fails.

After that insert, the workflow responds HTTP 202 with `status: archived` and `transcript_id`. This acknowledges archival only, not completed Salesforce processing. Validation and the remaining work continue afterwards; later failures go to the error workflow rather than changing that already-sent response.

Supply a stable, globally unique `call_id` from the call source, and reuse it only for retries of that same call. Its SHA-256 hash becomes `call_key`. Without it, the key is a hash of normalized email, ISO call date and trimmed transcript; changing any of these can create a different call. The exact original transcript remains separate.

The AI receives the transcript, date and attendee name. Its schema requires summary, action_items with task/owner/due_date, objections, next_steps, risk level/reason and an email subject/body. Unstated owners or dates are null, and empty lists mean no stated items. The parser rejects invalid structure. One chain retry is enabled, for two total attempts; model retries are disabled. A second failure stops before Task or draft creation. Accuracy of a schema-valid summary still needs human judgment.

## Matching, duplicates and recovery

Salesforce matching uses exact attendee email. One unconverted Lead wins. If there is no Lead, one Contact is selected, then its related open Opportunities are checked. One open Opportunity is selected; multiple open Opportunities fall back to the Contact rather than guessing a deal. Multiple Leads or Contacts are treated as ambiguous and unmatched. An unmatched call still gets intelligence and a draft, then a no-tag fallback message stating that Salesforce was not updated.

Validated intelligence is inserted with a database UNIQUE constraint on `(call_key, matched_record_id)` and `ON CONFLICT DO NOTHING`. Only the execution receiving a newly inserted row proceeds to business writes. This reservation prevents simultaneous executions from both creating Tasks for the same call and record. A read-only Salesforce Task query also checks the stable Subject marker before creation. Task and Gmail create nodes have no blind automatic retry.

Every webhook delivery is still raw-archived, and extraction happens before reservation, so duplicate deliveries can incur model usage. The duplicate route stops after the reservation attempt. It does not overwrite the original intelligence, create a second Task, draft or notification. Changing which Salesforce record matches changes the reservation key and may permit processing for that different record.

The reservation is deliberately conservative. If processing fails after it is inserted, a repeated webhook will stop as a duplicate rather than repeat uncertain external writes. Inspect the failed execution, the intelligence status, Salesforce Task Subject and Gmail Drafts before deciding how to recover. Do not simply delete a reservation and rerun: a Task or draft may have succeeded even when the network response or following database update failed. The stored IDs help when those updates succeeded. Resolve manually or implement a separately reviewed recovery path. There is no automatic lease takeover in this build.

Successful states progress from `processing` through `task_logged` (matched calls), `draft_created`, then `completed` or `unmatched_completed`. Error handling attempts `failed` for a recoverable transcript ID without changing completed rows. A database/API outage can prevent that status update.

Slack resolves the matched Salesforce User owner by email. Queue owners or missing Slack members go to the fallback channel without a guessed tag. A Slack API error other than users_not_found stops to error handling. Slack notifications include the summary, risk, record reference, draft ID and call key, not the raw transcript.

## Supabase tables and setup SQL

Open the existing Supabase project → Table Editor → public → select `postcall_transcripts`, `postcall_intelligence` or `postcall_errors`. Filter by execution_id, call_key or transcript_id as appropriate. The dashboard is for authorized project administrators; anonymous/authenticated API roles have no access to these tables. RLS is enabled and only the server service_role has the granted SELECT/INSERT/UPDATE access used by this workflow. No public policies are added.

- **postcall_transcripts** is the permanent raw archive. Its UUID identifies one received delivery; received_at is generated by Postgres.
- **postcall_intelligence** holds validated JSON arrays/objects and useful scalar fields, links to a raw archive UUID, reserves a call/CRM-record pair and records Task/draft IDs and status.
- **postcall_errors** records workflow_name, failed_node, scrubbed error_message, failed_input JSON, execution_id and created_at.

The exact SQL below was already applied and verified in your existing project. It creates only the three new tables and their own indexes, constraints and access controls. It intentionally aborts if any target table already exists; do not rerun it expecting an upgrade or repair. To reproduce elsewhere, open that project's SQL Editor → New query → paste → Run. No deletion or alteration of existing tables is included.

```sql
BEGIN;
DO $guard$
BEGIN
  IF to_regclass('public.postcall_transcripts') IS NOT NULL
     OR to_regclass('public.postcall_intelligence') IS NOT NULL
     OR to_regclass('public.postcall_errors') IS NOT NULL THEN
    RAISE EXCEPTION 'A postcall table already exists. Stopped without changing any existing table.';
  END IF;
END
$guard$;
CREATE TABLE public.postcall_transcripts (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  transcript text NOT NULL,
  call_date text,
  attendee_email text,
  attendee_name text,
  received_at timestamptz NOT NULL DEFAULT now(),
  execution_id text NOT NULL
);
CREATE INDEX postcall_transcripts_execution_idx ON public.postcall_transcripts(execution_id);
CREATE TABLE public.postcall_intelligence (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  transcript_id uuid NOT NULL REFERENCES public.postcall_transcripts(id),
  call_key text NOT NULL,
  matched_record_id text NOT NULL DEFAULT 'unmatched',
  record_type text NOT NULL CHECK (record_type IN ('Lead','Contact','Opportunity','unmatched')),
  summary text NOT NULL,
  action_items jsonb NOT NULL CHECK (jsonb_typeof(action_items)='array'),
  objections jsonb NOT NULL CHECK (jsonb_typeof(objections)='array'),
  next_steps jsonb NOT NULL CHECK (jsonb_typeof(next_steps)='array'),
  deal_risk_flag text NOT NULL CHECK (deal_risk_flag IN ('none','low','medium','high')),
  risk_reason text NOT NULL,
  suggested_follow_up_email jsonb NOT NULL CHECK (jsonb_typeof(suggested_follow_up_email)='object'),
  salesforce_task_id text,
  gmail_draft_id text,
  status text NOT NULL DEFAULT 'processing' CHECK (status IN ('processing','task_logged','draft_created','completed','unmatched_completed','failed')),
  created_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT postcall_call_record_key UNIQUE(call_key,matched_record_id)
);
CREATE INDEX postcall_intelligence_transcript_idx ON public.postcall_intelligence(transcript_id);
CREATE TABLE public.postcall_errors (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_name text NOT NULL,
  failed_node text NOT NULL,
  error_message text NOT NULL,
  failed_input jsonb NOT NULL,
  execution_id text,
  created_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX postcall_errors_execution_idx ON public.postcall_errors(execution_id);
ALTER TABLE public.postcall_transcripts ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.postcall_intelligence ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.postcall_errors ENABLE ROW LEVEL SECURITY;
REVOKE ALL ON TABLE public.postcall_transcripts,public.postcall_intelligence,public.postcall_errors FROM PUBLIC,anon,authenticated;
GRANT SELECT,INSERT,UPDATE ON TABLE public.postcall_transcripts,public.postcall_intelligence,public.postcall_errors TO service_role;
COMMIT;
```

## Main workflow: every node in order

The main path runs left to right. IF nodes describe their true and false routes. Chat model and Validate extraction are support connections to the extraction chain, rather than separate sequential main-path steps.

### 1. Receive transcript

Input: transcript, call_date, attendee_email/name and optional stable call_id. Attach PostCall Intake Header (Header Auth). Output: webhook body; keep its value only in the credential and trusted sender.

It passes its result to Archive raw transcript.

### 2. Archive raw transcript

First processing node. Input: original body. Output: archive row with generated id/received_at. Transcript text is never trimmed; invalid metadata remains archived as text. No blind write retry. Credential: PostCall Supabase (Supabase API). Public schema, named table only.

It passes its result to Acknowledge archive.

### 3. Acknowledge archive

Return one202 receipt only after storage; original item continues. No later node responds again.

It passes its result to Check call input.

### 4. Check call input

Input: archive and original body. Output: validated transcript/metadata, transcript_id, SHA256 call_key. Invalid input fails after archival. Hash stable call_id if provided; otherwise email/date/transcript.

It passes its result to Find Lead.

### 5. Find Lead

Input: attendee email. Output: up to2 exact-email unconverted Leads. Lead search always comes first. Credential: PostCall Salesforce (Salesforce OAuth2 API). Read-only query; empty results still emit one item.

It passes its result to Lead found?.

### 6. Lead found?

True: select Lead or mark duplicate matches ambiguous. False: search Contact.

It passes its result to Select Salesforce record (true); Find Contact (false).

### 7. Find Contact

Only after no Lead match. Output: up to2 exact-email Contacts. Credential: PostCall Salesforce (Salesforce OAuth2 API). Read-only query; empty results still emit one item.

It passes its result to One Contact?.

### 8. One Contact?

True: look for its open Opportunities. False: unmatched; do not guess among duplicate Contacts.

It passes its result to Find linked Opportunity (true); Select Salesforce record (false).

### 9. Find linked Opportunity

Read OpportunityContactRole for up to2 open Opportunities. Use one unique Opportunity; otherwise retain the Contact match. Credential: PostCall Salesforce (Salesforce OAuth2 API). Read-only query; empty results still emit one item.

It passes its result to Select Salesforce record.

### 10. Select Salesforce record

Output: one context with record_type/id, owner_id, contact_id and match_reason, or unmatched. No record is created.

It passes its result to Extract call intelligence.

### 11. Extract call intelligence

Input: archived transcript and metadata. Output: parser-validated JSON. Two total tries: one retry on schema failure, then stop and Error Trigger before Task or draft writes.

It passes its result to Prepare intelligence archive.

### 12. Chat model

Attach PostCall LLM (OpenAI API) and select a supported model. The chain owns the single retry; provider retries are zero. Another compatible provider support node can replace this one.

It passes its result to Extract call intelligence [ai_languageModel].

### 13. Validate extraction

No credential. Enforce all required fields, action owner/due_date types, arrays, enumerated risk/reason and email subject/body. Invalid output throws inside the retrying parent chain.

It passes its result to Extract call intelligence [ai_outputParser].

### 14. Prepare intelligence archive

Output: proposed intelligence columns only; valid extraction linked to transcript_id with shared call_key.

It passes its result to Save intelligence and reserve call.

### 15. Save intelligence and reserve call

Save structured output separately from raw text. UNIQUE call_key+record reserves only one execution. Conflicts preserve the existing row and return an empty body. Credential: PostCall Supabase. Replace <YOUR_SUPABASE_URL> with project URL, no trailing slash.

It passes its result to New call reservation?.

### 16. New call reservation?

True: process new intelligence. False: stop repeat Task/draft/Slack actions.

It passes its result to Use saved intelligence (true); Stop duplicate call (false).

### 17. Stop duplicate call

The raw delivery is archived, but business processing ends here. Inspect any existing partial/failed reservation before deliberate replay.

This is the end of this route.

### 18. Use saved intelligence

Output: saved intelligence row with ID for all later state updates.

It passes its result to Matched Salesforce record?.

### 19. Matched Salesforce record?

True: check/log Salesforce Task. False: skip CRM writes and create the draft before fallback Slack.

It passes its result to Prepare Salesforce Task (true); Create Gmail draft (false).

### 20. Prepare Salesforce Task

Build a completed Task with summary, actions, objections, next steps and risk. Input: selected CRM match and saved intelligence. Output: Salesforce Task fields. No manual values.

It passes its result to Check existing call Task.

### 21. Check existing call Task

Check the stable call-key Subject on the matched WhoId or WhatId before creating a Task. Passes Task Id or an empty item. Credential: PostCall Salesforce (Salesforce OAuth2 API). Read-only query; empty results still emit one item.

It passes its result to Task already exists?.

### 22. Task already exists?

Existing Task goes directly to saving its ID. Otherwise create one Task. No manual settings.

It passes its result to Save Salesforce Task ID (true); Create Salesforce Task (false).

### 23. Create Salesforce Task

Create one completed Task on the matched Lead, Contact or Opportunity. Input: prepared fields; output: Task ID. Credential: PostCall Salesforce. No automatic retry: inspect Salesforce after an ambiguous network failure.

It passes its result to Save Salesforce Task ID.

### 24. Save Salesforce Task ID

Save the existing or new Task ID on the intelligence reservation. Input: Task ID; output: updated row in body. Credential: PostCall Supabase. Replace <YOUR_SUPABASE_URL> with project URL, no trailing slash.

It passes its result to Create Gmail draft.

### 25. Create Gmail draft

Create a draft only, addressed to the submitted attendee. Input: validated email subject/body; output: Gmail draft ID. Credential: PostCall Gmail (Gmail OAuth2). The connected mailbox owns the draft. No retry or send operation.

It passes its result to Save Gmail draft ID.

### 26. Save Gmail draft ID

Save the draft ID before claiming it is ready. Input: Gmail draft ID; output: updated intelligence row. Credential: PostCall Supabase. Replace <YOUR_SUPABASE_URL> with project URL, no trailing slash.

It passes its result to Find Salesforce owner.

### 27. Find Salesforce owner

Read the matched record owner User email. Unmatched records and queue owners return no User; they use the fallback Slack route. Credential: PostCall Salesforce (Salesforce OAuth2 API). Read-only query; empty results still emit one item.

It passes its result to Owner email available?.

### 28. Owner email available?

A User email allows Slack lookup. Otherwise proceed to fallback notification without a guessed tag.

It passes its result to Find Slack owner (true); Prepare Slack message (false).

### 29. Find Slack owner

Resolve the Salesforce owner email to a Slack member ID. Input: User.Email; output: Slack ok/user or users_not_found. Credential: PostCall Slack; users:read.email scope required. Other API errors stop to the error workflow.

It passes its result to Prepare Slack message.

### 30. Prepare Slack message

Build the owner-tagged sales message or no-tag fallback. Input: CRM context, saved draft and Slack lookup; output: channel/text. Replace <POSTCALL_SALES_CHANNEL_ID> and <POSTCALL_FALLBACK_CHANNEL_ID> inside this node. No secrets.

It passes its result to Notify sales or fallback.

### 31. Notify sales or fallback

Post summary, risk and truthful Task/draft status. Input: channel/text; output: Slack message response. Channel IDs are set in Prepare Slack message. Credential: PostCall Slack; invite the bot to the target internal channel.

It passes its result to Mark processing complete.

### 32. Mark processing complete

Mark complete only after draft and notification succeeded. Input: Slack response; output: updated intelligence row. Credential: PostCall Supabase. Replace <YOUR_SUPABASE_URL> with project URL, no trailing slash.

This is the end of this route.

The sticky notes **Archive first** and **Connect before use** are canvas instructions; they do not execute or pass data.

## Error workflow: every node in order

An automatic production failure invokes this separate native Error Trigger workflow only after you select it in the main workflow Settings. Clicking Execute Workflow manually does not invoke this error trigger.

### 1. Catch workflow error

Receives failed production execution metadata from the main workflow. Set this workflow as the main workflow Error Workflow. Manual Execute Workflow does not trigger it.

It passes its result to Read failed execution.

### 2. Read failed execution

Read saved failed execution to recover the actual previous-node output supplied to the failed node. Credential: PostCall n8n API. Replace <YOUR_N8N_CLOUD_URL> without trailing slash. Requires n8n API access; unavailable on Cloud free trial. Failure continues with input explicitly unavailable.

It passes its result to Prepare safe error.

### 3. Prepare safe error

Extract workflow name, failed node, scrubbed error and actual input when saved source data exists. Remove credential/header fields and recognizable tokens. Input: execution API response plus trigger; output: exact postcall_errors columns. Never substitute the trigger as failed input. No manual values.

It passes its result to Log error in Supabase.

### 4. Log error in Supabase

Store scrubbed error and JSON failed_input. Input: exact error columns; output: inserted error row or error item. Logging failures continue to admin alert. Credential: PostCall Supabase (Supabase API). Public schema, named table only.

It passes its result to Mark intelligence failed.

### 5. Mark intelligence failed

Mark this call reservation failed when its archived transcript ID can be recovered. Completed rows stay complete. Unknown ID performs a no-op. Failure continues to admin alert. Credential: PostCall Supabase. Replace <YOUR_SUPABASE_URL> with project URL, no trailing slash.

It passes its result to Alert Slack admin.

### 6. Alert Slack admin

Post short admin alert without transcript or failed-input contents. Replace <POSTCALL_ADMIN_CHANNEL_ID>. Input: error logging result and safe metadata; output: Slack response. A failure here must be inspected in Executions. Credential: PostCall Slack; invite the bot to the target internal channel.

This is the end of this route.

The sticky note **Error recovery requirements** explains the required settings and does not execute.

Failed-input recovery follows the failed node's recorded source connection to the actual previous-node output in the saved execution. It can be unavailable for trigger failures, missing execution data or API restrictions. The error logger removes credential/header fields and recognizable token patterns before storing input; this is a safeguard, not a reason to submit secrets in transcripts. Error rows may contain customer transcript data, so retain the restricted access controls. Admin Slack receives only the short error metadata and recovery/logging status, not failed-input contents. If admin Slack itself fails, inspect the error workflow execution directly; it does not recursively invoke itself.

## Tests to run after setup

Use synthetic data and inspect both n8n Executions and the three Supabase tables. These checks have not been run for you.

1. Submit a valid call with an email matching a test Lead. Confirm the exact raw transcript, linked intelligence, one completed Task, one Gmail draft and an owner-tagged Slack message.
2. Use a test Contact with exactly one related open Opportunity. Confirm Task.WhatId targets that Opportunity and Task.WhoId targets the Contact.
3. Submit an unmatched email. Confirm no Salesforce Task, a draft in the connected mailbox and a no-tag fallback message saying manual linking is needed.
4. Submit the same stable call_id twice, including simultaneous requests if you can. Expect two raw deliveries but only one reservation, Task, draft and notification for the same matched record.
5. Submit an invalid email. Expect the raw delivery to remain archived, followed by validation failure and error handling, with no Task or draft.
6. In a temporary copy of the workflow, make the extraction schema impossible to satisfy. Confirm two total attempts, then failure, error row and admin alert, with no business writes.
7. Test missing Slack owner mapping and a deliberate downstream permission failure. Confirm fallback for users_not_found and the error route for real permission errors. Inspect failed_input.available and the admin alert's database logging status.
8. Confirm an actual automatic production failure triggers the error workflow; manually running it alone is not that test. Check the n8n API credential and failed execution retention if input is unavailable.

Keep raw transcripts as permanent records unless you separately decide an explicit retention policy. Do not bulk-delete archive rows to recover failures.

## Reference documentation

- [n8n Error Trigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.errortrigger/)
- [n8n public API authentication and availability](https://docs.n8n.io/api/authentication/)
- [Gmail draft operations](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.gmail/draft-operations/)
- [Slack credential setup](https://docs.n8n.io/integrations/builtin/credentials/slack/)
- [Supabase API security](https://supabase.com/docs/guides/api/securing-your-api)

