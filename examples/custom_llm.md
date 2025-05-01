### Example: Settings for Custom OpenAI Account

```json
{
  "type": "Settings",
  "audio": {...},
  "agent": {
    "listen": {...},
    "think": {
      "provider": {"type": "open_ai", "model": "gpt-4o"},
      "prompt": "You are a helpful assistant.",
      "endpoint": {
        "url": "https://api.openai.com/v1/chat/completions",
        "headers": {"authorization": "bearer $OPENAI_API_KEY"}
      }
    },
    "speak": {...}
  }
}
```
