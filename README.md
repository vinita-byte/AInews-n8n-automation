# AInews-n8n-automation
a simple automation workflow that fetches ai news, drafts an email containing snippets and sends it at the specified time
An n8n workflow that pulls the latest AI news from three tech publications every morning, summarises each story with an LLM, and emails a clean HTML digest to your inbox.

What it does
Runs daily at 5:45 AM (in your n8n instance's timezone).
Reads three RSS feeds in parallel:
TechCrunch — AI: https://techcrunch.com/category/artificial-intelligence/feed/
Ars Technica — AI: https://arstechnica.com/ai/feed/
The Verge — AI: https://www.theverge.com/rss/ai-artificial-intelligence/index.xml
Merges all articles into one list.
Sorts them by publish date (isoDate, newest first).
Keeps the top 6 stories.
Summarises each story with an LLM (Groq, openai/gpt-oss-120b), returning:
the headline in <h3> tags
a ~40-word plain-English summary in <p> tags
a "Read the full story" link
Aggregates the six summaries into a single item.
Sends one Gmail message ("AI digest") with the summaries separated by <hr> lines.
Workflow diagram
Schedule Trigger (5:45 AM)
   ├── RSS Read  (TechCrunch)
   ├── RSS Read1 (Ars Technica)
   └── RSS Read2 (The Verge)
            │
          Merge (3 inputs)
            │
          Sort (isoDate ↓)
            │
          Limit (6)
            │
   Basic LLM Chain ◄── Groq Chat Model
            │
          Aggregate (text)
            │
   Send a message (Gmail)
Nodes
Node	Type	Purpose
Schedule Trigger	Schedule	Fires once a day at 05:45
RSS Read / RSS Read1 / RSS Read2	RSS Feed Read	Fetch AI articles from each source
Merge	Merge	Combines the three feeds into one stream
Sort	Sort	Orders by isoDate, newest first
Limit	Limit	Keeps the 6 most recent items
Basic LLM Chain	LangChain LLM Chain	Turns title + snippet + link into an HTML summary
Groq Chat Model	Groq Chat	Model powering the chain (openai/gpt-oss-120b)
Aggregate	Aggregate	Collects all text outputs into one array
Send a message	Gmail	Emails the joined HTML digest
Setup
Import the workflow into n8n (Workflows → Import from file / paste JSON).
Add credentials
Groq: create an API key at console.groq.com and add it as a Groq credential on the Groq Chat Model node.
Gmail: connect a Gmail OAuth2 credential on the Send a message node.
Set the recipient in Send a message → Send To to the address that should receive the digest.
Check the timezone under Workflow Settings so 5:45 AM matches your local time (e.g. Asia/Kolkata).
Click Test workflow to run it once, confirm the email looks right, then Activate it.
Customising
Add or swap sources: duplicate an RSS Read node, change the URL, and increase the Merge node's number of inputs.
More or fewer stories: change Max Items in the Limit node.
Change the send time: edit the hour/minute in Schedule Trigger.
Tweak the summary style: edit the prompt in Basic LLM Chain (length, tone, format). Keep the HTML tags if you want the email to stay formatted.
Different model: swap the Groq model name, or replace the Groq node with another chat model (OpenAI, Anthropic, Gemini, etc.).
Subject line: consider adding the date, e.g. AI digest – {{ $now.toFormat('dd LLL yyyy') }}.
Prompt used
You are summarising an AI news story for a daily email digest. Return it in exactly this shape and nothing else:
Line 1: the headline wrapped in <h3> tags.
Line 2: one paragraph of about 40 words in plain English, wrapped in <p> tags.
Line 3: an anchor tag linking to the article, with the link text "Read the full story".

Title: {{ $json.title }}
Snippet: {{ $json.contentSnippet }}
Link: {{ $json.link }}
Known limitations
No de-duplication: if two outlets cover the same story, both can appear.
Only the RSS snippet is summarised, not the full article, so summaries depend on snippet quality.
If a feed is down, the RSS node may fail the run; enable Continue On Fail on the RSS nodes to make it more resilient.
The Gmail message is sent as HTML; check the node's email type option if formatting doesn't render.
Requirements
n8n (Cloud or self-hosted) with LangChain nodes
Groq API key
Gmail account connected via OAuth2
