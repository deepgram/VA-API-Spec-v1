### Example 1: Multilingual Settings with ElevenLabs

```json
{
  "type": "Settings",
  "experimental": false,
  "audio": {
    "input": {...},
    "output": {
      "encoding": "linear16",
      "sample_rate": 16000,
      "container": "none"
    }
  },
  "agent": {
    "language": "multi",
    "listen": {...},
    "think": {...},
    "speak": {
      "provider": {
        "type": "eleven_labs",
        "model_id": "eleven_flash_v2_5"
      },
      "endpoint": {
        "url": "https://api.elevenlabs.io/v1/text-to-speech/bIHbv24MWmeRgasZH58o",
        "headers": {
          "xi-api-key": "$ELEVENLABS_API_KEY"
        }
      }
    },
    "greeting": ""
  }
}
```

> **Note:** If passing audio to Twilio, change the `sample_rate` to `8000` in the `audio.output` section.

---

### Example 2: Multilingual Settings with Cartesia

```json
{
  "type": "Settings",
  "experimental": false,
  "audio": {
    "input": {...},
    "output": {
      "encoding": "linear16",
      "sample_rate": 16000,
      "container": "none"
    }
  },
  "agent": {
    "language": "multi",
    "listen": {...},
    "think": {...},
    "speak": {
      "provider": {
        "type": "cartesia",
        "model_id": "sonic-english",
        "voice": {
          "mode": "id",
          "id": "a167e0f3-df7e-4d52-a9c3-f949145efdab"
        }
      },
      "endpoint": {
        "url": "https://api.cartesia.ai/tts/bytes",
        "headers": {
          "x-api-key": "$CARTESIA_API_KEY"
        }
      }
    },
    "greeting": ""
  }
}
```
