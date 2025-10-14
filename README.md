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
- [Google Sheet Structure](#google-sheet-structure)
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
    {
      "date": "2025-10-14 15:21",
      "raw_entry": "Grateful for morning coffee and a team win",
      "gratitude_items": ["morning coffee", "team win"],
      "categories": ["daily routines", "achievements"],
      "sentiment_score": 0.85,
      "mood_insight": "Positive vibe from productivity and routines.",
      "weekly_summary": "N/A",
      "reflection_prompt": "What small win are you excited for tomorrow?",
      "bot_response": "Love your positivity! Coffee and wins are the best. 🌟 Logged to your journal!"
    }

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
