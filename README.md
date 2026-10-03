# Event Scheduler Agent

A conversational AI agent that helps you plan a meeting from start to finish. You chat with it in a Jupyter notebook; it works out the details with you, looks up the venue, checks travel time and weather, and then sends you everything you need to block off your calendar.

## What it does

You tell the agent about a meeting you want to schedule. It asks follow-up questions until it knows:

- where the event will happen
- the date and time
- the event title
- the address you'll be travelling from

Once it has those details, it:

1. **Searches Google Maps** to find the venue or suggest places that fit what you describe
2. **Calculates travel distance and time** between your starting point and the venue (walking, cycling, or driving)
3. **Checks the weather** at the destination for the time of the event (up to 10 days out)
4. **Creates Google Calendar links** for two pre-filled events: one for the travel time and one for the meeting itself
5. **Emails you a summary** with all the details and the calendar links
6. **Sends a one-line text message and Slack message** confirming the meeting is booked

## How it works

The notebook runs a simple agent loop. Each turn, the conversation is sent to the LLM along with a set of tools it can call. If the model asks for a tool, the notebook runs it, feeds the result back, and lets the model continue. When the model has nothing left to call, it's your turn to type.

### Tools

Each tool is defined as a Pydantic model, which gives the LLM a typed schema to fill in.

| Tool | Purpose |
| --- | --- |
| `EventDate` | Captures the event's year, month, day, hour, and minute |
| `GetEventLink` | Returns a link that creates a pre-populated Google Calendar event |
| `SendText` | Sends an SMS (US numbers, 160 characters or fewer, no URLs) |
| `SendSlackMessage` | Sends a Slack message |
| `SendEmail` | Sends an email with a subject and body |
| `GoogleMapsSearch` | Searches Google Maps, optionally restricted to a radius around a location; returns place details including a `place_id` |
| `GoogleMapsTravelDistance` | Finds the travel distance and time between two `place_id`s by `WALK`, `BICYCLE`, or `DRIVE` |
| `GoogleMapsWeather` | Gets current or hourly forecast weather for a latitude and longitude |

The tools are executed through the `xlkitlearn_api` gateway (`APIGateway`), which handles the calls to Google Maps, email, SMS, and Slack.

## Requirements

- Python 3.12+
- Jupyter Notebook or JupyterLab
- An [OpenRouter](https://openrouter.ai/) API key
- The `xlkitlearn_api` package available in your environment
- Python packages:

```bash
pip install openai pydantic
```

## Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git
   cd <your-repo>
   ```

2. **Save your API key** in a plain text file somewhere outside the repository, containing only the key.

3. **Update the key path** in the notebook so it points to your file:

   ```python
   with open(r"path\to\your\API Router Key.txt", 'r') as f:
       key_API = f.read().strip()
   ```

4. **Check the client and model settings.** The notebook uses the `openai` library pointed at OpenRouter:

   ```python
   client = openai.OpenAI(
       base_url="https://openrouter.ai/api/v1",
       api_key=key_API,
   )
   ```

   The model is set in the agent loop (`model = 'openai/gpt-5.5'`). You can swap in any tool-calling model listed at [openrouter.ai/models](https://openrouter.ai/models).

## Usage

1. Open `Event Scheduler.ipynb` and run the cells from top to bottom.
2. When the `User message` prompt appears, type your reply and press Enter.
3. Keep chatting until the agent has what it needs. It will call its tools and print each call and response as it goes.
4. To end the session, submit an empty message.

Example opening message:

> I'm meeting a client for coffee next Tuesday at 2pm somewhere near Union Square. I'll be coming from 500 Howard St.

After each model response, the notebook prints the prompt tokens, completion tokens, cost of that call, and the running total, so you can see what a session costs.

## Notes

- **Keep your API key out of GitHub.** Store it outside the repository or add the key file to `.gitignore`.
- The agent is given your current date, time, and timezone at startup, so relative dates like "next Tuesday" resolve correctly.
- Text messages are limited to US phone numbers and 160 characters.
- Weather forecasts are only available up to 10 days ahead.
- If a tool call fails, the agent is told there was an error and carries on rather than crashing the session.
