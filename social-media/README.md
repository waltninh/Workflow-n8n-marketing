# Social media

One workflow that takes a row from a spreadsheet, writes the caption, waits for
approval in Telegram, and then publishes the same post to Facebook, Instagram
and LinkedIn.

| File | Name | Nodes | Role |
|---|---|---:|---|
| [`social-media.json`](social-media.json) | social media | 89 | Multi-channel publishing with an approval gate |

It is driven by the **Social media** tab of the Marketing plan spreadsheet,
using the `status` column as a queue.

## How it runs

**Trigger:** webhook (POST, expects `row_number` in the body)

1. Marks the row `Processing` and reads it from the **Social media** tab
2. Routes on post type, then an AI agent drafts the caption from the `content`
   and `prompt` columns
3. Sends the draft to Telegram and waits — nothing publishes until it passes
   this gate
4. Collects the images from the Drive folder named in the `images` column,
   sorted by filename
5. Publishes to each channel, looping in batches:
   - **Facebook** — Graph API `v22.0`, uploading to the page's `photos` edge
   - **Instagram** — creates a carousel container, waits for it to process,
     then calls `media_publish`
   - **LinkedIn** — registers an upload against `/v2/assets`, uploads the
     binary, then creates the post through `/v2/ugcPosts`
6. Writes the resulting post links and status back to the sheet

Image generation runs through kie.ai (`/jobs/createTask` then polling
`/jobs/recordInfo`) for rows that need new artwork rather than existing photos.

**Models:** `anthropic/claude-sonnet-4.6` and `openai/gpt-5-mini` via OpenRouter.

## Sheet columns

`day`, `subject`, `content`, `product images`, `prompt`, `images`, `status`

## Setup

Replace these placeholders after importing:

| Placeholder | Where to get it |
|---|---|
| `YOUR_GOOGLE_SHEET_ID` | Marketing plan spreadsheet ID, from its URL |
| `YOUR_GOOGLE_FILE_ID` | Drive folder IDs holding the post images |
| `YOUR_FACEBOOK_PAGE_ID` | Page ID from Meta Business Suite |
| `YOUR_INSTAGRAM_ACCOUNT_ID` | Instagram Business Account ID, linked to the same page |
| `YOUR_LINKEDIN_PERSON_ID` | The `urn:li:person:` identifier that owns the posts |
| `YOUR_LINKEDIN_ACCESS_TOKEN` | OAuth token with `w_member_social` scope |
| `YOUR_KIE_AI_API_KEY` | kie.ai dashboard; sent as `Authorization: Bearer` |
| `YOUR_TELEGRAM_CHAT_ID` | The chat that receives approval messages |
| `YOUR_ALIBABA_SHOP` | Alibaba trustpass storefront subdomain, used in link-outs |

Credentials to attach: Google Sheets, Google Drive, Facebook Graph API,
Telegram, OpenRouter.

Two things worth changing before production use:

- The LinkedIn token and the kie.ai key are plain header values on HTTP Request
  nodes. Move them into Header Auth credentials so n8n redacts them on export.
- LinkedIn access tokens expire roughly every 60 days, so a hardcoded one will
  fail silently partway through a campaign.

The webhook is unauthenticated as exported — enable Header Auth before exposing
it. Since this workflow publishes to live social accounts, that matters more
here than anywhere else in this repository.
