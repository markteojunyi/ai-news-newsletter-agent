# 📰 AI News Newsletter Agent

An AI agent that reads the day's AI and business news, picks the 5 stories that matter most to a transformation leader, and emails a short HBR-style briefing every morning at 6:30am (Singapore time). It runs on a schedule with no server.

## The problem

Keeping up with AI news is a full-time job. Most feeds are written for engineers or for hype, not for leaders deciding *what this means for my organisation*. I wanted 5 minutes of signal each morning instead of 30 minutes of scrolling.

## What it does

Every morning, without me doing anything:

1. **Fetch:** pulls recent items (last 3 days) from 7 RSS feeds: TechCrunch AI, MIT Sloan Review, HBR, VentureBeat AI, the OpenAI blog, and two Google News searches (LLM releases, enterprise AI adoption).
2. **De-duplicate:** skips anything it has already sent, using a small SQLite database.
3. **Write:** Claude selects the top 5 for the target audience and writes each as a summary, then *"What this means for you"*, then *"Actionable insight"*, in under 500 words.
4. **Send:** formats a mobile-friendly HTML email and sends it via Gmail.

## How it works

```
GitHub Actions (cron, 06:30 SGT)
        │
        ▼
fetcher.py ──► database.py ──► writer.py ──► sender.py
 RSS feeds     skip seen       Claude API     Gmail SMTP
                stories        top 5 + brief  HTML email
        │
        ▼
Commit updated articles.db back to the repo (so tomorrow knows what was sent)
```

Design choices worth noting:

- **No server to maintain:** GitHub Actions runs the job on a schedule (a *cron* job) and stores the API keys as encrypted **secrets**.
- **De-duplicate by title, not URL:** Google News links are redirects that change every fetch, so the agent fingerprints the cleaned-up headline instead.
- **Fail safe:** stories are marked "seen" only *after* the email sends, so a failed send doesn't silently lose a day's news.
- **Configurable audience:** the reader profile is an environment variable, so the same agent can brief a different audience without code changes.

## Tech stack

Python · Anthropic Claude API · feedparser · SQLite · Gmail SMTP · GitHub Actions

## Run it yourself

```bash
pip install -r requirements.txt
# set these in a .env file (or as GitHub Actions secrets/variables):
# ANTHROPIC_API_KEY, GMAIL_ADDRESS, GMAIL_APP_PASSWORD, RECIPIENT_EMAIL, AUDIENCE
python main.py
```

Use a Gmail **App Password**, not your real password.

## Related

[youtube-digest](https://github.com/markteojunyi/youtube-digest) applies the same pattern to YouTube channels.
