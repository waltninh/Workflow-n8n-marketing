# Workflow-n8n-marketing

Marketing automation workflows for TMICORP — a Vietnamese manufacturer and
exporter of processed agricultural products.

Five n8n workflows, grouped into three areas. Each folder has its own README
explaining how those workflows run and what to configure.

| Folder | Workflows | Nodes | What it does |
|---|---|---:|---|
| [`seo/`](seo/) | `topic`, `seo` | 74 | Discovers and clusters keywords, then writes fact-checked articles and exports them as HTML |
| [`email/`](email/) | `email`, `send-email` | 94 | Generates artwork, gets approval, and sends product emails through Gmail |
| [`social-media/`](social-media/) | `social-media` | 89 | Publishes an approved post to Facebook, Instagram and LinkedIn |

All five are orchestrated through one Google Sheet — the "Marketing plan"
spreadsheet — where a `status` column acts as the work queue.

| Tab | Used by | Columns |
|---|---|---|
| **KEY WORD SEO** | `seo/` | `KW_ID`, `Pillar Topic`, `Seed`, `Geolocation`, `Tavily`, `Search Intent`, `Buyer`, `Title`, `primary`, `secondary`, `Meta`, `link`, `status` |
| **Topic** | `seo/` | `Topic`, `List_keyword`, `Pillar Topic`, `Seed` |
| **Email** | `email/` | `day`, `subject`, `subject email`, `paragraph1..3`, `product images`, `prompt`, `images in email`, `attachments`, `status` |
| **Social media** | `social-media/` | `day`, `subject`, `content`, `product images`, `prompt`, `images`, `status` |

## Importing

1. In n8n, go to **Workflows** → **Import from File**
2. Select the `.json` file you want
3. Attach your own credentials on each node
4. Replace the `YOUR_*` placeholders — the folder README lists which ones apply

## External services

Google Sheets · Google Drive · Gmail · Facebook Graph API · LinkedIn ·
Instagram Graph · Telegram · OpenRouter · Tavily · kie.ai ·
USDA FoodData Central · Google Autosuggest

## A note on secrets

All credentials, API keys, Google document IDs, social account IDs, email
addresses and webhook IDs were replaced with `YOUR_*` placeholders before
publishing. Credential blocks were stripped from every node.

Two things to be aware of if you adapt these workflows:

- Several API keys were originally typed directly into HTTP Request headers
  rather than stored as n8n credentials. n8n redacts credentials on export but
  not plain header values, so those had to be scrubbed by hand. Use Header Auth
  credentials instead.
- Every webhook trigger here is unauthenticated. Anyone with the URL can start
  the workflow. Enable Header Auth before exposing them.
