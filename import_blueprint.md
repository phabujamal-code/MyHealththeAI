# Social Media AI Automation – Make.com Import Blueprint

Use this blueprint text when creating an **Import Blueprint** file in Make.com to generate and publish unique daily content automatically. Copy the JSON below into a new file named `social_media_ai_automation.json`, then import it in Make.com via **Scenarios → Import blueprint**.

## How it works
1. A daily scheduler triggers the scenario at the configured time.
2. The scenario builds a content brief (topic, tone, hashtags, CTA) and sends it to the OpenAI **ChatGPT** module for copy generation.
3. The generated copy is enriched with emojis/hashtags, and a **Text Aggregator** formats the final caption.
4. A second OpenAI call produces a concise prompt for the **DALL·E** image generator.
5. The image URL is fetched via **HTTP Get a file** and pushed to cloud storage (e.g., Google Drive/S3) for reuse.
6. The caption and image are posted to your social channel (e.g., Instagram, Facebook, X) through the relevant Make.com connector.
7. A log row is stored (e.g., in Google Sheets/Airtable) with the post time, platform, caption, and asset URL.

## Blueprint JSON
```json
{
  "name": "Daily Social Media AI Automation",
  "flow": {
    "rootModule": 0,
    "modules": [
      {"id": 0, "name": "Scheduler", "type": "tools.schedule", "parameters": {"mode": "daily", "time": "09:00"}},
      {"id": 1, "name": "Build Brief", "type": "tools.setVariable", "parameters": {"topic": "health & habits", "tone": "friendly", "cta": "Follow for daily tips", "hashtags": "#Wellness #DailyTips"}},
      {"id": 2, "name": "Generate Caption", "type": "openai.chatgpt.createCompletion", "parameters": {"model": "gpt-4o-mini", "prompt": "Write a 80-120 word social caption about {{topic}} in a {{tone}} tone. Include emojis and hashtags: {{hashtags}}. CTA: {{cta}}."}},
      {"id": 3, "name": "Format Caption", "type": "tools.textAggregator", "parameters": {"template": "{{Generate Caption.choices[1].message.content}}\n\n{{cta}}"}},
      {"id": 4, "name": "Create Image Prompt", "type": "openai.chatgpt.createCompletion", "parameters": {"model": "gpt-4o-mini", "prompt": "Write a short DALL-E prompt for an illustration matching this caption: {{Format Caption}}"}},
      {"id": 5, "name": "Generate Image", "type": "openai.dalle.createImage", "parameters": {"prompt": "{{Create Image Prompt.choices[1].message.content}}", "size": "1024x1024"}},
      {"id": 6, "name": "Download Image", "type": "http.getFile", "parameters": {"url": "{{Generate Image.data[1].url}}"}},
      {"id": 7, "name": "Upload Asset", "type": "storage.upload", "parameters": {"filename": "daily-post-{{formatDate(now; 'YYYY-MM-DD')}}.png", "data": "{{Download Image.content}}"}},
      {"id": 8, "name": "Publish Post", "type": "social.publish", "parameters": {"platform": "instagram", "caption": "{{Format Caption}}", "image": "{{Upload Asset.url}}"}},
      {"id": 9, "name": "Log Row", "type": "sheets.addRow", "parameters": {"timestamp": "{{now}}", "platform": "Instagram", "caption": "{{Format Caption}}", "asset_url": "{{Upload Asset.url}}"}}
    ],
    "links": [
      {"from_module": 0, "to_module": 1},
      {"from_module": 1, "to_module": 2},
      {"from_module": 2, "to_module": 3},
      {"from_module": 3, "to_module": 4},
      {"from_module": 4, "to_module": 5},
      {"from_module": 5, "to_module": 6},
      {"from_module": 6, "to_module": 7},
      {"from_module": 7, "to_module": 8},
      {"from_module": 8, "to_module": 9}
    ]
  },
  "metadata": {
    "description": "Creates AI-written social posts daily with auto-generated visuals and logs the publication.",
    "version": "1.0.0",
    "variables": ["topic", "tone", "cta", "hashtags"],
    "notes": "Replace connector module types (social.publish, storage.upload, sheets.addRow) with the providers you use (e.g., Meta/IG, Google Drive, Airtable)."
  }
}
```

## Import instructions
1. Save the JSON above as `social_media_ai_automation.json`.
2. In Make.com, open **Scenarios → Import blueprint** and upload the file.
3. Map your connectors:
   - Replace `social.publish` with Instagram/Facebook/X module of your choice.
   - Replace `storage.upload` with Google Drive, S3, or Dropbox.
   - Replace `sheets.addRow` with Google Sheets or Airtable.
4. Update scheduling, hashtags, and CTA variables to match your brand voice.

## Notes
- The OpenAI modules use `gpt-4o-mini` for cost efficiency; switch to `gpt-4.1` for higher quality if desired.
- Add moderation/validation steps if posting to multiple platforms.
- Use a router module to branch to different platforms or languages per day.
