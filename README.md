### Endpoint

- Current: `wss://agent.deepgram.com/agent`
- New: `wss://agent.deepgram.com/v1/converse` (transition with 2-week migration period)

### Connection

WebSocket-based, real-time bidirectional communication.

---

### Client Messages

| Type                 | Structure                                                                                                                    | Notes                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Settings             | `{ "type": "Settings", ...Settings }`                                                                                        | Initializes the voice agent and sets up audio transmission formats                       |
| UpdateInstructions   | `{ "type": "UpdateInstructions", "instructions": "" }`                                                                       | Allows giving additional instructions to the Think model in the middle of a conversation |
| UpdateSpeak          | `{ "type": "UpdateSpeak ", "speak": { "provider": { "type": "", "model": "" }, "endpoint": { "url": "", "headers": {} } } }` | Enables changing the Speak model during the conversation                                 |
| InjectAgentMessage   | `{ "type": "InjectAgentMessage", "content": "" }`                                                                            | Triggers an immediate statement from the agent                                           |
| FunctionCallResponse | `{ "type": "FunctionCallResponse", "id": "", "name": "", "content": "" }`                                                    | Sends the result of a function call back to the server                                   |
| Binary Audio         | `[binary data]`                                                                                                              | Audio input per settings                                                                 |

#### Settings Example

```json
{
  "type": "Settings",
  "experimental": false, // default is false
  "audio": {
    "input": {
      "encoding": "", // string
      "sample_rate": // int
    },
    "output": { // optional
      "encoding": "",
      "sample_rate": ,
      "bitrate": ,
      "container": ""
    }
  },
  "agent": {
    "language": "", // optional
    "listen": { // optional, to deepgram's latest model
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
        "temperature": // float, optional
      },
      "endpoint": { // optional
        "url": "",
        "headers": {} // optional
      },
      "functions": [ // optional
        {
          "name": "",
          "description": "",
          "parameters": {},
          "endpoint": { // optional, if not passed, function called client-side
            "url": "",
            "method": "",
            "headers": {}, // optional
          }
        }
      ],
      "instructions": "" // optional
    },
    "speak": { // optional, defaults to latest deepgram TTS model
      "provider": {
        "type": "",
        ... // provider specific fields required
      },
      "endpoint": { // optional if provider.type = 'deepgram', required for non-deepgram TTS providers
        "url": "",
        "headers": {} // optional
      }
    },
    "context": { // optional
      "messages": [
        {
          "type": "ConversationText",
          "role": "",
          "content": ""
        },
        {
          "type": "FunctionCallRequest",
          ...
        },
        {
          "type": "FunctionCallResponse",
          ...
        },
      ],
      "replay": false
    },
    "fallbacks": [ // optional
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

| Type                         | Structure                                                                                                             | Notes                                                                                  |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **`welcome`**                | `{ "type": "Welcome", "message": "Connection established" }`                                                          | Confirms that the WebSocket connection has been successfully opened.                   |
| **`SettingsApplied`**        | `{ "type": "SettingsApplied", "message": "Settings applied successfully" }`                                           | Confirms that the configuration settings have been applied.                            |
| **`ConversationText`**       | `{ "type": "ConversationText", "role": "user" \| "assistant", "content": "" }`                                        | Provides the text of what was spoken by either the user or the agent.                  |
| **`UserStartedSpeaking`**    | `{ "type": "UserStartedSpeaking" }`                                                                                   | Notifies the client that the user has begun speaking.                                  |
| **`AgentThinking`**          | `{ "type": "AgentThinking" }`                                                                                         | Informs the client that the agent is processing information.                           |
| **`FunctionCallRequest`**    | `{ "type": "function_call_request", "functions": [{ "id": "", "name": "", "arguments": "", "client_side": false }] }` | Sent when the agent needs to make a function call; requests client function execution. |
| **`FunctionCallingMessage`** | `{ "type": "FunctionCallingMessage", "id": "", "name": "", "arguments": "", "client_side": false }`                   | Provides information about a function being called.                                    |
| **`AgentStartedSpeaking`**   | `{ "type": "AgentStartedSpeaking" }`                                                                                  | Signals that the server has begun streaming the agent’s audio response.                |
| **`AgentAudioDone`**         | `{ "type": "AgentAudioDone" }`                                                                                        | Indicates that the server has finished sending the final audio segment to the client.  |
| **`Error`**                  | `{ "type": "Error", "message": "" }`                                                                                  | Notifies the client of fatal errors that occurred on the server side.                  |
| **`Warning`**                | `{ "type": "Warning", "message": "" }`                                                                                | Notifies the client of non-fatal errors or warnings.                                   |
| **`Binary Audio`**           | `[binary data]`                                                                                                       | Audio output sent as binary data, per the settings configuration.                      |

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

### Settings

| Parameter                      | Type/Details                                                            | Notes                                          |
| ------------------------------ | ----------------------------------------------------------------------- | ---------------------------------------------- |
| type                           | String, "Settings"                                                      | Identifies config type                         |
| experimental                   | Boolean, default false                                                  | Enables undocumented features                  |
| audio.input.encoding           | String                                                                  | Input audio encoding                           |
| audio.input.sample_rate        | Integer                                                                 | Input audio sample rate                        |
| audio.output.encoding          | String, optional                                                        | Output audio encoding                          |
| audio.output.sample_rate       | Integer, optional                                                       | Output audio sample rate                       |
| audio.output.bitrate           | Integer, optional                                                       | Output audio bitrate                           |
| audio.output.container         | String, optional                                                        | Output file container                          |
| context.messages               | Array of ConversationText \| FunctionCallRequest \|FunctionCallResponse | Conversation history                           |
| context.replay                 | Boolean                                                                 | Replay last message on reconnect               |
| agent.listen.provider.type     | "deepgram"                                                              | STT provider                                   |
| agent.listen.provider.model    | String                                                                  | STT model                                      |
| agent.listen.provider.keyterms | Array of strings, optional                                              | Prompt key-term recognition (nova-3 'en' only) |
| agent.think                    | can be object or list of objects to support fallback                    | LLM settings                                   |
| agent.think.provider.type      | "open_ai", "anthropic", "x_ai", "amazon_bedrock"                        | LLM provider (supports name aliases)           |
| agent.think.provider.model     | String                                                                  | LLM model                                      |
| agent.think.provider.temp      | Number, optional (0-2 OpenAI, 0-1 Anthropic)                            | Response randomness                            |
| agent.think.endpoint.url       | String                                                                  | Custom LLM endpoint                            |
| agent.think.functions          | Array of Function objects                                               | Callable functions                             |
| agent.think.instructions       | String, optional                                                        | LLM system prompt                              |
| agent.think.endpoint.headers   | Object, optional                                                        | Custom headers for LLM                         |
| agent.speak                    | can be object or list of objects to support fallback                    | TTS configuration                              |
| agent.speak.provider.type      | "deepgram", "eleven_labs", "cartesia", "open_ai"                        | TTS provider                                   |
| agent.speak.provider.model     | String, varies by provider                                              | TTS model                                      |
| agent.speak.endpoint.url       | String                                                                  | Custom TTS endpoint                            |
| agent.speak.endpoint.headers   | Object, optional                                                        | Custom headers for TTS                         |
| agent.language                 | String, optional, default "en"                                          | Agent language                                 |

#### Provider-Specific Speak Parameters

- **deepgram**: `model`
- **eleven_labs**: `model_id`, `voice_id`, `language_code` (optional)
- **cartesia**: `model_id`, `voice`, `mode: "id"`, `id`
- **open_ai**: `model`, `voice`

---

### Notes

- Audio: Binary messages will match `audio.input`/`audio.output` settings.
- Rollout: 2-week dev/test, then 2-week migration period announced via email/Slack.
- Internal Features: Hidden unless `experimental: true`.
