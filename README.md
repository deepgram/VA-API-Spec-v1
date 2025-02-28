### Endpoint

- Current: `ws://agent.deepgram.com/agent`
- New: `ws://agent.deepgram.com/v1/converse` (transition with 2-week migration period)

### Connection

WebSocket-based, real-time bidirectional communication.

---

### Client Messages

| Type                  | Structure                                                                                                                    | Notes                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| SettingsConfiguration | `{ "type": "SettingsConfiguration", ...SettingsConfiguration }`                                                              | Initializes the voice agent and sets up audio transmission formats                       |
| UpdateInstructions    | `{ "type": "UpdateInstructions", "instructions": "" }`                                                                       | Allows giving additional instructions to the Think model in the middle of a conversation |
| UpdateSpeak           | `{ "type": "UpdateSpeak ", "speak": { "provider": { "type": "", "model": "" }, "endpoint": { "url": "", "headers": {} } } }` | Enables changing the Speak model during the conversation                                 |
| InjectAgentMessage    | `{ "type": "InjectAgentMessage", "content": "" }`                                                                            | Triggers an immediate statement from the agent                                           |
| FunctionCallResponse  | `{ "type": "FunctionCallResponse", "id": "", "name": "", "content": "" }`                                                    | Sends the result of a function call back to the server                                   |
| Binary Audio          | `[binary data]`                                                                                                              | Audio input per settings                                                                 |

#### Settings Example

```json
{
  "type": "SettingsConfiguration",
  "type": "",
  "experimental_features": false,
  "audio": {
    "input": {
      "encoding": "",
      "sample_rate": 0
    },
    "output": {
      "encoding": "",
      "sample_rate": 0,
      "bitrate": 0,
      "container": ""
    }
  },
  "agent": {
    "language": "",
    "listen": {
      "provider": {
        "type": "",
        "model": "",
        "keyterms": [""]
      }
    },
    "think": {
      "provider": {
        "type": "",
        "model": "",
        "temperature": 0
      },
      "endpoint": {
        "url": "",
        "headers": {}
      },
      "functions": [
        {
          "name": "",
          "description": "",
          "parameters": {}
        }
      ],
      "instructions": ""
    },
    "speak": {
      "provider": {
        "type": "",
        "model": ""
      },
      "endpoint": {
        "url": "",
        "headers": {}
      }
    },
    "context": {
      "messages": [
        {
          "role": "",
          "content": ""
        }
      ],
      "replay": false
    },
    "fallbacks": [
      {
        "think": {},
        "speak": {}
      }
    ]
  }
}
```

#### Function Call Response Example

```json
{
  "type": "FunctionCallResponse",
  "id": "",
  "name": "",
  "content": ""
}
```

---

### Server Messages

| Type                           | Structure                                                                                                             | Notes                                                                                  |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **`welcome`**                  | `{ "type": "welcome", "message": "Connection established" }`                                                          | Confirms that the WebSocket connection has been successfully opened.                   |
| **`settings_applied`**         | `{ "type": "settings_applied", "message": "Settings applied successfully" }`                                          | Confirms that the configuration settings have been applied.                            |
| **`conversation_text`**        | `{ "type": "conversation_text", "role": "user" \| "assistant", "content": "" }`                                       | Provides the text of what was spoken by either the user or the agent.                  |
| **`user_started_speaking`**    | `{ "type": "user_started_speaking" }`                                                                                 | Notifies the client that the user has begun speaking.                                  |
| **`agent_thinking`**           | `{ "type": "agent_thinking" }`                                                                                        | Informs the client that the agent is processing information.                           |
| **`function_call_request`**    | `{ "type": "function_call_request", "functions": [{ "id": "", "name": "", "arguments": "", "client_side": false }] }` | Sent when the agent needs to make a function call; requests client function execution. |
| **`function_calling_message`** | `{ "type": "function_calling_message", "id": "", "name": "", "arguments": "", "client_side": false }`                 | Provides information about a function being called.                                    |
| **`agent_started_speaking`**   | `{ "type": "agent_started_speaking" }`                                                                                | Signals that the server has begun streaming the agent’s audio response.                |
| **`agent_audio_done`**         | `{ "type": "agent_audio_done" }`                                                                                      | Indicates that the server has finished sending the final audio segment to the client.  |
| **`errors`**                   | `{ "type": "error", "message": "" }`                                                                                  | Notifies the client of fatal errors that occurred on the server side.                  |
| **`warnings`**                 | `{ "type": "warning", "message": "" }`                                                                                | Notifies the client of non-fatal errors or warnings.                                   |
| **`Binary Audio`**             | `[binary data]`                                                                                                       | Audio output sent as binary data, per the settings configuration.                      |

---

#### Function Call Request Example

```json
{
  "type": "function_call_request",
  "functions": [
    {
      "id": "",
      "name": "",
      "arguments": "",
      "client_side": false
    }
  ]
}
```

### Settings Configuration

| Parameter                      | Type/Details                                     | Notes                               |
| ------------------------------ | ------------------------------------------------ | ----------------------------------- |
| type                           | String, "SettingsConfiguration"                  | Identifies config type              |
| experimental_features          | Boolean, default false                           | Enables undocumented features       |
| audio.input.encoding           | String                                           | Input audio encoding                |
| audio.input.sample_rate        | Integer                                          | Input audio sample rate             |
| audio.output.encoding          | String                                           | Output audio encoding               |
| audio.output.sample_rate       | Integer                                          | Output audio sample rate            |
| audio.output.bitrate           | Integer, optional                                | Output audio bitrate                |
| audio.output.container         | String, optional                                 | Output file container               |
| context.messages               | Array of {role, content}                         | Conversation history                |
| context.replay                 | Boolean                                          | Replay last message on reconnect    |
| agent.listen.provider.type     | "deepgram"                                       | STT provider                        |
| agent.listen.provider.model    | String                                           | STT model                           |
| agent.listen.provider.keyterms | Array of strings, optional                       | Fine-tune recognition (nova-3 only) |
| agent.think.provider.type      | "open_ai", "anthropic", "x_ai", "amazon_bedrock" | LLM provider                        |
| agent.think.provider.model     | String                                           | LLM model                           |
| agent.think.provider.temp      | Number, optional (0-2 OpenAI, 0-1 Anthropic)     | Response randomness                 |
| agent.think.endpoint.url       | String, optional                                 | Custom LLM endpoint                 |
| agent.think.functions          | Array of Function objects                        | Callable functions                  |
| agent.think.instructions       | String, optional                                 | LLM system prompt                   |
| agent.think.endpoint.headers   | Object, optional                                 | Custom headers for LLM              |
| agent.speak.provider.type      | "deepgram", "eleven_labs", "cartesia", "open_ai" | TTS provider                        |
| agent.speak.provider.model     | String, varies by provider                       | TTS model                           |
| agent.speak.endpoint.url       | String, optional                                 | Custom TTS endpoint                 |
| agent.speak.endpoint.headers   | Object, optional                                 | Custom headers for TTS              |
| agent.language                 | String, optional, default "en"                   | Agent language                      |
| agent.fallbacks                | Array of think/speak objects, optional           | Provider fallback                   |

#### Provider-Specific Speak Parameters

- **deepgram**: `model`
- **eleven_labs**: `model_id`, `voice_id`, `language_code` (optional)
- **cartesia**: `model_id`, `voice`, `mode: "id"`, `id`
- **open_ai**: `model`, `voice`

---

### Notes

- Audio: Binary messages match `audio.input`/`audio.output` settings.
- Rollout: 2-week dev/test, then 2-week migration period announced via email/Slack.
- Internal Features: Hidden unless `experimental_features: true`.
