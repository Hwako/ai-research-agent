# Email Research Assistant

Daily AI research digest, sent to your inbox. Searches Google for a set of topics, filters for relevance,
scrapes and summarizes the relevant pages, then uses two LLM agents (a summariser and a reviewer) in a
LangGraph loop to draft and refine a single email before sending it.

Rebuilt locally from [whitew1994WW/email_research_assistant](https://github.com/whitew1994WW/email_research_assistant).

## How it works

1. `search_serper` — searches Google (via Serper) for each term in `SEARCH_TERMS`, restricted to the last 24 hours
2. `check_search_relevance` — an LLM call filters each batch down to the 5 most relevant results
3. `scrape_and_save_markdown` — scrapes each relevant link (via a scraping API) and converts the HTML to markdown
4. `generate_summaries` — an LLM call summarizes each scraped page individually
5. A LangGraph loop with two nodes:
   - `summariser` — drafts the email digest from all the page summaries, following `email_template.md`
   - `reviewer` — checks the draft, either approves it or sends it back with feedback
   - loops until approved
6. `send_email` — sends the final digest via Brevo/Sendinblue's transactional email API

## Required API keys

- `SERPER_API_KEY` — Google search, https://serper.dev/ (free tier)
- `SCRAPING_API_KEY` — HTML scraping with JS rendering, https://scrapingfish.com/ (pay-per-use, a couple dollars goes a long way)
- `SENDINGBLUE_API_KEY` — sending email, https://app.brevo.com/ (free tier, up to 300 emails/month)
- `OPENAI_API_KEY` — https://platform.openai.com (used for relevance checks, page summaries, and the summariser/reviewer agents)
- `DESTINATION_EMAIL` — the email address you want the digest sent to

## Local setup

```bash
python3 -m venv venv
source venv/bin/activate        # on Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env            # then fill in your real keys
python email_script.py
```

## Customizing

- **Search terms**: edit `SEARCH_TERMS` at the top of `email_script.py`
- **What counts as relevant**: edit `prompts/relevance_check.md`
- **Email structure**: edit `email_template.md`
- **Digest tone/behavior**: edit `prompts/summariser.md` and `prompts/reviewer.md`

## Deploying (e.g. on Render as a daily cron job)

1. Push this repo to your own GitHub
2. Point a Render Cron Job at it
3. Set the entry command to `python email_script.py`
4. Add the 5 environment variables above in the Render dashboard
5. Set the schedule (e.g. `0 0 * * *` for daily at midnight)

## Note on this rebuild

The `prompts/` files here (`relevance_check.md`, `summarise_markdown_page.md`, `summariser.md`, `reviewer.md`)
are reconstructed to match exactly what `email_script.py` expects (same template variables, same structured
output fields), based on how the original author described them — the actual prompt wording in his repo
wasn't accessible to fetch directly. Feel free to tune them once you see real output.

Also fixed a small bug from the original: the fallback `.env` loader now actually opens the file in read mode
instead of write mode (which would have wiped it).
