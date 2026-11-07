# AI Job Match Agent (n8n + Gemini)

An automated workflow built in n8n that fetches job postings every day, scores each one against my profile using an LLM, logs everything to Google Sheets, and sends the best matches to me on Telegram.

## Problem

Searching job boards by hand takes time, and most postings are not a good fit for a fresher. Keyword alerts can't tell whether a role really matches my skills or asks for too much experience. This workflow does that first screening automatically.

## What it does

1. Runs on a daily schedule (or manually for testing).
2. Fetches remote job postings from the Remotive API.
3. Cleans the data into one standard format (job id, title, company, location, URL, description) and removes duplicates.
4. Sends each job and my profile to Google Gemini, which returns structured JSON:
   score (0-100), a short reason, matched skills, missing skills, required skills, experience level and work mode.
5. Logs every scored job to a Google Sheet.
6. Keeps only jobs scoring 70 or above and sends a ranked digest to Telegram.

## Architecture

```
Schedule Trigger
      |
 My Profile (Edit Fields)
      |
 Remotive Jobs (HTTP Request)
      |
 Split Out  ->  Normalize Jobs (Code)  ->  Remove Duplicates  ->  Limit
      |
 Score Job (Basic LLM Chain + Google Gemini + Structured Output Parser)
      |
 Flatten Result (Edit Fields)
      |-----------------------------|
 Google Sheets (log all jobs)   Good Match? (IF score >= 70)
                                      |
                                Build Digest (Code)
                                      |
                                Send Digest (Telegram)
```

## Tech stack

- **n8n** (self-hosted with Docker)
- **Google Gemini** for job scoring (via the n8n Gemini chat model node)
- **Remotive API** for job postings (no key needed)
- **Google Sheets** for the job log
- **Telegram Bot API** for the daily digest
- **JavaScript** in n8n Code nodes for normalizing data and building the digest

## Key design choices

- **Structured output:** a JSON schema (Structured Output Parser) keeps the LLM response in a predictable format that later nodes can use.
- **Grounded scoring:** the prompt tells the model to score only from skills listed in the profile and to penalize roles that need 3+ years of experience.
- **Cheap filtering after scoring:** every job is logged, but only strong matches are sent to Telegram.
- **Deduplication:** jobs already processed in earlier runs are skipped, so the same posting is not scored twice.
- **No secrets in the workflow:** API keys and the bot token are stored in n8n credentials, not in the workflow file.

## Setup

1. Install Docker Desktop and start n8n:
   ```bash
   docker volume create n8n_data
   docker run -d --name n8n --restart unless-stopped -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
   ```
2. Open `http://localhost:5678` and create an account.
3. Import `workflow.json` (paste it onto the canvas with Ctrl+V).
4. Create these credentials in n8n:
   - Google Gemini API key (from Google AI Studio)
   - Google Sheets (sign in with Google)
   - Telegram bot token (from @BotFather)
5. Create a Google Sheet named `Job Tracker` with these columns:
   `job_id, date, title, company, location, url, score, reason, matched_skills, missing_skills, status`
6. In the **My Profile** node, replace the text with your own skills and the roles you want.
7. In the **Send Digest** node, set your own Telegram chat ID.
8. Run **Execute workflow** once to test, then publish the workflow.

## Screenshots

Add your screenshots here:

- `screenshots/workflow.png` - the workflow canvas
- `screenshots/telegram-digest.png` - the Telegram message
- `screenshots/google-sheet.png` - the job tracker sheet

## Limitations and future work

- Remotive lists remote jobs only, so postings are not specific to one city. Adding another source (for example Jooble) would widen coverage.
- Scores come from an LLM and are not yet checked against manual ratings.
- Planned: embedding-based similarity as a second scoring signal, tailored resume tips and cover letters for top matches, and deployment on a server so the schedule runs without a laptop.

## Author

Shubham
GitHub: [github.com/shubham-1716](https://github.com/shubham-1716)
LinkedIn: [linkedin.com/in/shubham-kakade](https://linkedin.com/in/shubham-kakade)
