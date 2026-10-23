# telegram-content-engine

An n8n workflow that turns a text or voice idea sent to a Telegram bot into two LinkedIn posts and a short video script. Drafts are saved to Google Sheets and sent back to you on Telegram.

I built it to turn my own build notes into content, and to practice running n8n on my own machine.

<img width="1693" height="499" alt="canvas png" src="https://github.com/user-attachments/assets/7e896c04-c841-49f1-8722-3ac5d2149eb5" />
<img width="538" height="911" alt="telegram-drafts png" src="https://github.com/user-attachments/assets/3b5b07fc-777e-4f2e-876b-6a4a6446ffea" />

## What it does
 
1. You send a **text message or a voice note** to your Telegram bot.
2. The workflow checks that the message comes from **your chat only**.
3. If it is a voice note, Gemini turns it into text.
4. An AI Agent (Gemini Flash) writes **2 LinkedIn posts and a video script**, using only the facts in your idea.
5. The drafts are saved to a **Google Sheet** with the status `Draft`.
6. The drafts are sent back to you on **Telegram**.
## Tools
 
- n8n, self-hosted with Docker
- Cloudflare Tunnel, so Telegram can reach n8n on my laptop
- Telegram Bot API
- Google Gemini (voice transcription and the AI Agent)
- Google Sheets
## Design choices
 
- **Chat ID lock.** Only my own Telegram chat can use the bot. This protects my Gemini quota.
- **Strict prompt.** The AI may only use facts from my idea. It must not add stories, numbers, results, or claims. If the idea is empty, it returns an error instead of making something up.
- **Low temperature (0.3).** This keeps the writing close to the idea.
- **Reply cleaning.** A Code node removes extra text around the AI's JSON reply and fixes line breaks, so a small formatting mistake does not stop the workflow.
- **Retry on fail.** The AI Agent retries when Gemini is busy.
- **Human review.** Every draft is saved with the status `Draft`. I read and edit before I post.
## Set it up
 
### 1. Import the workflow
 
In n8n, create a new workflow and choose **Import from file**. Pick `telegram-content-engine.json`.
 
### 2. Connect your credentials
 
- **Telegram:** create a bot with @BotFather and add the token as a Telegram credential.
- **Google Gemini:** add your API key.
- **Google Sheets:** connect your Google account with OAuth.
### 3. Make your Google Sheet
 
Make a sheet with these column names in the first row:
 
`DATE`, `IDEA`, `LINKEDIN POST 1`, `LINKEDIN POST 2`, `VIDEO SCRIPT`, `STATUS`
 
Open the **Send Draft To Sheet** node and pick your sheet.
 
### 4. Add your chat ID
 
Replace `YOUR_CHAT_ID` in two nodes:
 
- **My Chat Only** (a number)
- **Send Drafts To Telegram**
To find your chat ID, run the Telegram Trigger once and read `message.chat.id`, or message a bot such as @userinfobot.
 
### 5. Give Telegram a public address
 
Telegram can only send messages to a public HTTPS link. If n8n runs on your own computer, use a tunnel such as Cloudflare Tunnel and set these environment variables when you start n8n:
 
```
WEBHOOK_URL=https://your-tunnel-link/
N8N_EDITOR_BASE_URL=http://localhost:5678
```
 
Keep your data in a Docker volume so you do not lose your workflows when you recreate the container.
 
### 6. Publish
 
Publish the workflow, then message your bot. Do not click **Execute workflow** after you publish. Test runs move the bot to a test address, and you would need to unpublish and publish again.
 
## Things I learned
 
- A **pinned** trigger replays old test data and can send a text message down the voice path.
- If the AI writes about things you never said, check what the AI actually receives. A wrong field name can leave the idea empty.
- Telegram keeps one webhook address per bot, so test runs and the published workflow can conflict.
- A free tunnel link changes each time it restarts, so the setup needs a restart routine. For client work, use a fixed address or a cloud server.
- Free-tier AI quotas are low. Use a paid API key for real or client use.
- Check the quota of each model in Google AI Studio before you rely on it.
## Results
 
> Fill these in with real numbers from your own tests. Do not estimate.
 
- Time to write one LinkedIn post by hand: **[YOUR_NUMBER]**
- Time from sending an idea to receiving drafts: **[YOUR_NUMBER]**
## Limits
 
- It runs only while the computer, Docker, and the tunnel are on.
- It does not post to LinkedIn. It writes drafts for me to review.
- It writes from the idea you send. A short idea gives a short post.
## Files
 
- `telegram-content-engine.json`: the workflow, with private details removed
- `images/`: screenshots of the canvas and the Telegram drafts
