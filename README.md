# AI Gratitude Journal Bot

Welcome to the **AI Gratitude Journal Bot**, a no-code automation workflow built on n8n to help users track daily gratitudes for mental wellness. This project uses a Telegram bot to capture natural language messages, an AI agent to process and categorize them, and Google Sheets to store and visualize data. Perfect for anyone looking to combine AI, no-code tools, and positive psychology!

## Table of Contents
- [Overview](#overview)
- [Workflow Structure](#workflow-structure)
- [Node Details](#node-details)
  - [Telegram Trigger Node](#telegram-trigger-node)
  - [AI Agent Node](#ai-agent-node)
  - [Google Sheets Node](#google-sheets-node)
- [Setup Instructions](#setup-instructions)
- [System Prompt](#system-prompt)
- [Usage Example](#usage-example)
- [Contributing](#contributing)
- [License](#license)

## Overview
This workflow enables users to send gratitude messages via Telegram (e.g., "Grateful for coffee and a good chat today"). An AI agent processes the input using natural language processing (NLP), extracts gratitude items, assigns categories (e.g., relationships, achievements), scores sentiment, and logs data into a structured Google Sheet for tracking and trend analysis. Built for accessibility, it promotes mental health and demonstrates AI automation for non-technical users.

## Workflow Structure
The workflow consists of three main nodes in n8n:
1. **Telegram Trigger**: Captures user messages sent to the bot.
2. **AI Agent**: Processes messages using a language model with memory and a custom system prompt.
3. **Google Sheets**: Appends processed data to a spreadsheet for storage and visualization.

**Flow**: Telegram Trigger → AI Agent → Google Sheets

<figure>
  <img src="https://github.com/SuzanneMondal/AI-Gratitude-Journal-Bot/blob/main/images/workflow_n8n_canvas.png" alt="Gratitude Bot Workflow" width="400">
  <figcaption>Fig. 1 - AI Gratitude Bot Workflow using n8n and Telegram.</figcaption>
</figure>

## Node Details

### Telegram Trigger Node
- **Function**: Listens for incoming messages sent to the configured Telegram bot.
- **Input**: Telegram message (text) from a user.
  - Example: `{"message": {"text": "Grateful for morning coffee and a team win", "chat": {"id": 123456}, ...}}`
- **Output**: JSON object containing the message text and metadata (e.g., chat ID, timestamp).
  - Example: `{"message": {"text": "Grateful for morning coffee and a team win", "chat_id": 123456, "date": "2025-10-14T15:21:00+05:30"}}`
- **Configuration**:
  - Bot Token: Generated via BotFather on Telegram.
  - Updates: Set to "message" to capture text inputs.
  - Webhook: Enabled for real-time updates (ensure n8n server is accessible).

### AI Agent Node
- **Function**: Processes the Telegram message using an AI language model (e.g., Grok, OpenAI GPT) to extract gratitudes, categorize them, score sentiment, and generate insights.
- **Input**: JSON from Telegram node, specifically `{{ $node["Telegram"].json["message"]["text"] }}`.
- **Output**: Structured JSON with parsed data for Google Sheets.
  - Example Output:
    ```json
      "date": "2025-10-14 15:21",
      "raw_entry": "Grateful for morning coffee and a team win",
      "gratitude_items": ["morning coffee", "team win"],
      "categories": ["daily routines", "achievements"],
      "sentiment_score": 0.85,
      "mood_insight": "Positive vibe from productivity and routines.",
      "weekly_summary": "N/A",
      "reflection_prompt": "What small win are you excited for tomorrow?",
      "bot_response": "Love your positivity! Coffee and wins are the best. 🌟 Logged to your journal!"    
    ```

- **Configuration**: Structured JSON with parsed data for Google Sheets.
  - Model: Choose a model like Grok or GPT-4o (set API key in n8n credentials).
  - Prompt: Uses a custom system prompt (see below) to guide NLP tasks.
  - Memory: Context window (e.g., 5 previous messages) for continuity in responses.
  - Tools: Configured to output structured JSON for downstream nodes.
 
### Google Sheets Node
- **Function**: Appends AI-processed data to a Google Sheet for storage and trend analysis.
- **Input**: JSON from AI Agent node (e.g., `{{ $node["AI Agent"].json }}`).
- **Output**: Data written to a Google Sheet row; no direct output to other nodes.
- **Configuration**:
  - Google Sheets API: Enabled with OAuth2 credentials.
  - Spreadsheet ID: Unique ID of the target Google Sheet.
  - Range: Set to append rows (e.g., `Sheet1!A:I`).
  - Columns Mapped: Date, Raw Entry, Gratitude Items, Categories, Sentiment Score, Mood Insight, Weekly Summary, Reflection Prompt, Notes.

## Setup Instructions

### Prerequisites
  - n8n installed (self-hosted or cloud)
  - Telegram account and BotFather access
  - Google account with Sheets API enabled
  - AI model API access (e.g., xAI Grok, OpenAI, or local LLM)

### Step-by-Step Setup
1. **Clone the Repository**:
```bash
   git clone https://github.com/yourusername/ai-gratitude-journal-bot.git
```
2. **Set Up n8n**:
   - Install n8n locally (`npm install n8n`) or use n8n Cloud.
   - Import the workflow JSON file (included in `/workflow` folder).
  
3. **Configure Telegram**:
   - Create a bot via BotFather on Telegram to get a bot token.
   - Set the token in the Telegram Trigger node.

4. **Set Up AI Agent**:
   - Add credentials for your AI model (e.g., xAI for Grok, OpenAI for GPT).
   - Copy the system prompt (below) into the AI Agent node.
  
5. **Configure Google Sheets**:
   - Create a Google Sheet with headers: Date, Raw Entry, Gratitude Items, Categories, Sentiment Score, Mood Insight, Weekly Summary, Reflection Prompt, Notes.
   - Enable Google Sheets API in Google Cloud Console; add OAuth2 credentials to n8n.
   - Set Spreadsheet in the Google Sheets node.

## System Prompt
The AI Agent uses this prompt to process messages:

```plaintext
You are Gratitude Journal AI, a supportive assistant for mental wellness tracking. Your role is to process natural language messages from a Telegram bot, extract gratitude entries, categorize them, perform basic sentiment analysis, update a Google Sheet structure, and respond encouragingly to promote daily journaling habits.

Core Instructions:
- Input: A single natural language message (e.g., "Grateful for a walk in the park and team support today."). Assume current date/time is provided (e.g., [CURRENT_DATE_TIME: 2025-10-14 15:21]).
- Output Format: JSON with:
  {
    "date": "YYYY-MM-DD HH:MM",
    "raw_entry": "full message",
    "gratitude_items": ["item1", "item2", ...],
    "categories": ["category1", "category2", ...] (from: health, work/achievements, relationships, daily routines, nature, self-growth, family, hobbies, other),
    "sentiment_score": number (0.0-1.0; positive language = higher),
    "mood_insight": "1-sentence tone summary",
    "weekly_summary": "trend or N/A",
    "reflection_prompt": "open-ended question for next entry",
    "bot_response": "encouraging reply (<100 words, positive)"
  }
- Processing:
  1. Extract 3-5 gratitude items; infer if vague.
  2. Assign 1-3 categories; prioritize fit.
  3. Score sentiment (0=neutral, 1=highly positive).
  4. Generate concise insights and prompts.
  5. Reply warmly, ending with an emoji (e.g., 🌟).
- Edge Cases:
  - Off-topic: Redirect to gratitudes.
  - Empty: Prompt for details.
  - Privacy: No data storage beyond session.
- Tone: Empathetic, like a wellness coach.
```

## Usage Example

- **User Message**: "Grateful for morning yoga, a call with mom, and finishing a project."
- **Bot Response**: "Amazing shares! Yoga and family time are so uplifting. 🌿 Logged to your journal—check the insights!"
- **Sheet Entry**:
  - Date: 2025-10-14 15:21
  - Raw Entry: Grateful for morning yoga, a call with mom, and finishing a project.
  - Gratitude Items: morning yoga, call with mom, finishing project
  - Categories: health, family, work/achievements
  - Sentiment Score: 0.90
  - Mood Insight: Uplifted by wellness and productivity.
  - Weekly Summary: N/A
  - Reflection Prompt: What's a family moment you cherish this week?

 <figure>
  <img src="https://github.com/SuzanneMondal/AI-Gratitude-Journal-Bot/blob/main/images/google_sheet_journal.png" alt="My Journal Sheet" width="600">
  <figcaption>Fig. 2 - A capture of my gratitude journal sheet.</figcaption>
</figure>
 
## Contributing
Contributions are welcome! Fork the repo, tweak the workflow (e.g., add visualization nodes), and submit a pull request. Issues or feature ideas? Open a ticket!

## License
MIT License - Free to use, modify, and distribute.

##
Built with ❤️ by Suzanne for mental wellness and AI enthusiasts!
