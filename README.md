# Workflow-n8n-marketing

Marketing automation workflows for TMICORP — a Vietnamese manufacturer and
exporter of processed agricultural products.

All credentials, API keys, Google document IDs, email addresses and webhook IDs
have been replaced with placeholders before publishing. See
[Configuration](#configuration).

## Workflows

| File | Name | Nodes | Purpose |
|---|---|---:|---|
| [`send-email.json`](send-email.json) | send email | 14 | Reads a Google Sheet, builds an HTML email with images from Drive, sends it via Gmail, marks the row as sent |
| [`topic.json`](topic.json) | TOPIC | 16 | Harvests keywords from Google Autosuggest, clusters them by search intent with an LLM, writes results back to the Sheet |
| [`seo.json`](seo.json) | SEO | 58 | SEO content pipeline: research (Tavily, USDA FoodData), multiple AI agents, output to Sheets/Drive |
| [`email.json`](email.json) | email | 80 | Extended email pipeline: image generation via kie.ai, branching by content type, Telegram notifications |
| [`social-media.json`](social-media.json) | social media | 89 | Multi-channel publishing: Facebook Graph API, LinkedIn, Instagram; image generation via kie.ai; orchestrated through a Sheet |

### External services used

Google Sheets · Google Drive · Gmail · Facebook Graph API · LinkedIn · Instagram
Graph · Telegram · OpenRouter · Tavily · kie.ai · USDA FoodData Central ·
Google Autosuggest

## Importing

1. In n8n, go to **Workflows** → **Import from File**
2. Select the `.json` file you want
3. Reattach credentials on each node (see below)
4. Replace the `YOUR_*` placeholders with real values

## Configuration

### Credentials

Credential references were stripped from these files. After importing, create
and attach your own in n8n for the Google Sheets, Google Drive, Gmail, Facebook
Graph API, Telegram, OpenRouter and Tavily nodes.

### Placeholders to replace

| Placeholder | Meaning | Appears in |
|---|---|---|
| `YOUR_GOOGLE_SHEET_ID` | ID of the orchestrating Google Sheet (the "Marketing plan" spreadsheet) | all |
| `YOUR_GOOGLE_FILE_ID` | Drive file/folder IDs (product images, catalogue, upload folder) | `send-email`, `email`, `social-media` |
| `YOUR_KIE_AI_API_KEY` | kie.ai API key, sent as an `Authorization: Bearer` header | `email`, `social-media` |
| `YOUR_LINKEDIN_ACCESS_TOKEN` | LinkedIn OAuth access token | `social-media` |
| `YOUR_USDA_API_KEY` | USDA FoodData Central API key | `seo` |
| `YOUR_ALIBABA_SHOP` | Alibaba trustpass storefront subdomain | `social-media` |
| `your-email@example.com` | Sender / recipient addresses | `send-email`, `email`, `social-media` |

### Webhooks

Webhook nodes have had their `webhookId` and `path` reset to a null UUID
(`00000000-0000-0000-0000-000000000000`). n8n generates fresh IDs on import.

These webhooks carry **no authentication** by default — anyone who knows the URL
can trigger the workflow. Enable Header Auth on the webhook node before running
them in production.

### Google Sheet structure

The workflows read from and write to these tabs within a single spreadsheet:

- **Email** — `day`, `subject`, `subject email`, `paragraph1..3`, `product images`, `prompt`, `images in email`, `attachments`, `status`
- **KEY WORD SEO** — `KW_ID`, `Pillar Topic`, `Seed`, `Geolocation`, `Tavily`, `Search Intent`, `Buyer`, `Title`, `primary`, `secondary`, `Meta`, `link`, `status`
- **Topic** — `Topic`, `List_keyword`, `Pillar Topic`, `Seed`

The `status` column acts as a queue: workflows filter for `Send` / `Processing`,
then update rows to `Sent` / `Finished` once processing completes.
