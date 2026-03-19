# aws-telephony-bot

A Pipecat AI voice agent built with a cascade pipeline (STT → LLM → TTS).

## How It Works

1. Your application triggers an outbound call
2. The bot creates a Daily room with dial-out capabilities
3. Daily initiates the outbound call to the specified phone number
4. When the call is answered, the bot joins the Daily room
5. The bot and the callee are connected, and the bot handles the conversation

## Configuration

- **Bot Type**: Telephony
- **Transport(s)**: Daily PSTN (Dial-out)
- **Pipeline**: Cascade
  - **STT**: Deepgram
  - **LLM**: AWS Bedrock
  - **TTS**: Deepgram
- **Features**:
  - Transcription
  - Krisp Noise Cancellation

## Setup

1. Create a virtual environment and install dependencies

   ```bash
   uv sync
   ```

2. Set up environment variables

   Copy the example file and fill in your API keys:

    ```bash
    cp .env.example .env
    # Edit .env with your API keys
    ```

3. Buy a phone number

   Instructions on how to do that can be found at this [docs link:](https://docs.daily.co/reference/rest-api/phone-numbers/buy-phone-number)

4. Request dial-out enablement

   For compliance reasons, to enable dial-out for your Daily account, you must request enablement via the form. You can find out more about dial-out, and the form at the [link here](https://docs.daily.co/guides/products/dial-in-dial-out#main)

## Environment Configuration

The bot supports two deployment modes controlled by the `ENV` variable:

### Local Development (`ENV=local`)

- Uses your local server for handling dial-out requests and starting the bot
- Default configuration for development and testing

### Production (`ENV=production`)

- Bot is deployed to Pipecat Cloud; requires `PIPECAT_API_KEY` and `PIPECAT_AGENT_NAME`
- Set these when deploying to production environments
- Your FastAPI server runs either locally or deployed to your infrastructure


## Run the Bot Locally

You'll need two terminal windows open:

1. **Terminal 1**: Start the webhook server:

   ```bash
   uv run server.py
   ```

   This runs on port 8080 and handles dial-out requests.

2. **Terminal 2**: Start the bot server:

   ```bash
   uv run bot.py -t daily
   ```

   This runs on port 7860 and handles the bot logic.

3. **Test the dial-out functionality**

   With both servers running, send a dial-out request:

   ```bash
   curl -X POST "http://localhost:8080/dialout" \
     -H "Content-Type: application/json" \
     -d '{
       "dialout_settings": {
         "phone_number": "+1234567890"
       }
     }'
   ```

   The server will create a room, start the bot, and the bot will call the specified number. Answer the call to speak with the bot.

## Project Structure

aws-telephony-bot/
├── server/              # Python bot server
│   ├── bot.py           # Main bot implementation
│   ├── server.py        # FastAPI webhook server for Daily PSTN dial-out
│   ├── server_utils.py  # Utility functions starting the bot
│   ├── pyproject.toml   # Python dependencies
│   ├── env.example      # Environment variables template
│   ├── .env             # Your API keys (git-ignored)
│   ├── Dockerfile       # Container image for Pipecat Cloud
│   └── pcc-deploy.toml  # Pipecat Cloud deployment config
├── .gitignore           # Git ignore patterns
└── README.md            # This file

This example is organized to be production-ready and easy to customize:
- **`server.py`** - FastAPI server that handles dial-out requests

  - Receives dial-out requests via `/dialout` endpoint
  - Creates Daily rooms with dial-out capabilities
  - Routes to local or production bot deployment
  - Uses shared HTTP session for optimal performance


- **`server_utils.py`** - Utility functions for Daily API interactions

  - Data models for call data and agent requests
  - Room creation logic
  - Bot starting logic (production and local modes)
  - Easy to extend with custom business logic

- **`bot.py`** - The voice bot implementation
  - `DialoutManager` class for retry logic
  - Handles the conversation with the person being called
  - Deployed to Pipecat Cloud in production or run locally for development

## Deploying to Pipecat Cloud

This project is configured for deployment to Pipecat Cloud. You can learn how to deploy to Pipecat Cloud in the [Pipecat Quickstart Guide](https://docs.pipecat.ai/getting-started/quickstart#step-2-deploy-to-production).

Refer to the [Pipecat Cloud Documentation](https://docs.pipecat.ai/deployment/pipecat-cloud/introduction) to learn more about configuring, deploying, and managing your agents in Pipecat Cloud.

## Learn More

- [Pipecat Documentation](https://docs.pipecat.ai/)
- [Pipecat GitHub](https://github.com/pipecat-ai/pipecat)
- [Pipecat Examples](https://github.com/pipecat-ai/pipecat-examples)
- [Discord Community](https://discord.gg/pipecat)