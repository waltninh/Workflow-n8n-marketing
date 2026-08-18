# Email

Two workflows for outbound product emails. `send-email` is the delivery step on
its own; `email` is the full pipeline that generates artwork first and asks for
approval before anything goes out.

| File | Name | Nodes | Role |
|---|---|---:|---|
| [`send-email.json`](send-email.json) | send email | 14 | Builds the HTML email and sends it |
| [`email.json`](email.json) | email | 80 | Generates images, requests approval, then hands off to delivery |

Both read the **Email** tab of the Marketing plan spreadsheet, using the
`status` column as a queue.

## send-email.json — build and deliver

**Triggers:** webhook (`POST /sendemail`) and a manual trigger for testing

1. Reads rows from the **Email** tab where `status = Send`
2. Splits into two parallel branches:
   - **Inline images** — lists the Drive folder in `images in email`, takes the
     first three by name, and rewrites each to a Drive thumbnail URL
     (`sz=w1200`) so recipients load a compressed image rather than the
     multi-megabyte original
   - **Attachments** — lists the Drive folder in `attachments`, downloads each
     file and packs them into `attachment_1..N` binary fields
3. Assembles a responsive HTML email: branded header, banner, three body
   paragraphs from the sheet, two inline product shots, a catalogue CTA and a
   footer. Paragraphs are split on blank lines and HTML-escaped
4. Sends via Gmail with the attachments, then sets `status = Sent`

Body copy comes from `paragraph1`, `paragraph2` and `paragraph3`; the subject
line comes from `subject email`.

## email.json — generation and approval

**Trigger:** webhook (POST, expects `row_number` in the body)

1. Marks the row `Processing`, reads it, and collects the source images from
   Drive
2. Submits image-generation jobs to kie.ai (`/jobs/createTask`), then polls
   `/jobs/recordInfo` behind Wait nodes until each job finishes
3. Branches on content type through a series of Switch nodes, with AI agents
   (backed by Tavily search and Google Docs tools) producing the copy
4. Sends previews and status updates to Telegram for approval
5. Uploads the finished artwork back to Drive and updates the sheet row

**Model:** `openai/gpt-5-mini` via OpenRouter.

## Setup

Replace these placeholders after importing:

| Placeholder | Where to get it | File |
|---|---|---|
| `YOUR_GOOGLE_SHEET_ID` | Marketing plan spreadsheet ID, from its URL | both |
| `YOUR_GOOGLE_FILE_ID` | Drive folder and file IDs — product images, catalogue, upload target | both |
| `your-email@example.com` | Sender and recipient addresses | both |
| `YOUR_KIE_AI_API_KEY` | kie.ai dashboard; sent as `Authorization: Bearer` | `email.json` |
| `YOUR_TELEGRAM_CHAT_ID` | The chat that receives approval messages | `email.json` |

Credentials to attach: Google Sheets, Google Drive, Gmail, Telegram, OpenRouter,
Tavily.

The kie.ai key is currently a plain header value on the HTTP Request nodes.
Moving it into a Header Auth credential is worth doing — n8n then redacts it
automatically on export.

Unlike the other folders, these two kept their descriptive webhook paths —
`/email` and `/sendemail` — so the intended endpoint name stays readable. Only
the internal `webhookId` was zeroed, and since it forms part of the production
URL, the full address still changes on import.

Both webhooks are unauthenticated as exported — enable Header Auth before
exposing them. `POST /sendemail` in particular is an easy path to guess.
