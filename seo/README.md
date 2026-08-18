# SEO

Two workflows that turn a seed keyword into a published, fact-checked article.
They are meant to run in order: `topic` discovers and groups keywords, then
`seo` writes the article for a chosen keyword.

| File | Name | Nodes | Role |
|---|---|---:|---|
| [`topic.json`](topic.json) | TOPIC | 16 | Keyword discovery and clustering |
| [`seo.json`](seo.json) | SEO | 58 | Article generation, fact-checking and HTML export |

Both are driven by the **KEY WORD SEO** tab of the Marketing plan spreadsheet,
using the `status` column as a queue.

## topic.json — keyword discovery

**Trigger:** webhook (POST, expects `row_number` in the body)

1. Marks the sheet row as `Processing`, then reads it back
2. Expands the `Seed` into hundreds of query variants:
   - *alphabet soup* — seed + `a`–`z` and `0`–`9`, which surfaces pack-size
     variants like "frozen avocado 1kg"
   - *B2B modifiers* — `supplier`, `exporter`, `wholesale`, `moq`, `fob`,
     `hs code`, `from vietnam`… applied both before and after the seed
   - *B2C patterns* — question openers (`is`, `how to`, `what is`) before the
     seed, attributes (`recipe`, `calories`, `nutrition`, `organic`) after it
3. Queries each variant against Google Autosuggest, throttled to one request
   per 400 ms
4. Deduplicates suggestions and ranks them by how many variants surfaced them
5. Two LLM passes: the first groups keywords into topic clusters, the second is
   a quality-control pass that drops retailer names, out-of-scope markets,
   novelty queries and near-duplicates
6. Appends each cluster to the **Topic** tab, then marks the row `Finished`

The `Geolocation` column accepts one or more two-letter market codes
(`us` or `us, de, ae`); each market is queried separately.

## seo.json — article generation

**Trigger:** webhook (POST, expects `row_number` in the body)

1. Marks the row `Processing` and reads it from **KEY WORD SEO**
2. Normalizes `Search Intent` and routes on it:
   - **Informational** — resolves the product against USDA FoodData Central for
     real nutrition figures, researches with Tavily, then drafts, rewrites and
     generates meta description through a chain of AI agents
   - **Commercial** — plans and expands its own search queries before drafting
3. Cleans typography, converts Markdown to HTML
4. Writes the HTML file to Google Drive and records the link back in the sheet

A fact-checking agent is constrained to a fixed set of verified company facts
(registration number, certifications, product lines) and instructed to invent
nothing beyond them.

**Models:** `openai/gpt-5` and `openai/gpt-5-mini` via OpenRouter.

## Setup

Replace these placeholders after importing:

| Placeholder | Where to get it |
|---|---|
| `YOUR_GOOGLE_SHEET_ID` | The Marketing plan spreadsheet ID, from its URL |
| `YOUR_USDA_API_KEY` | Free key from <https://fdc.nal.usda.gov/api-key-signup.html> (`seo.json` only) |

Credentials to attach: Google Sheets, Google Drive, OpenRouter, Tavily.

Both webhook paths were zeroed to `00000000-0000-0000-0000-000000000000`.
n8n assigns a new path on import, so read the real URL off the webhook node
afterwards rather than assuming the one in the file.

Both webhooks are unauthenticated as exported — enable Header Auth before
exposing them.
