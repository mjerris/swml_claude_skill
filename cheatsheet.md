# SWML Cheatsheet

## Play URL Schemes

| Scheme | Example | Description |
|--------|---------|-------------|
| `https://` | `https://cdn.example.com/audio.mp3` | Play audio file |
| `say:` | `say:Hello caller` | Text-to-speech |
| `silence:` | `silence:2.0` | Silence in seconds |
| `ring:` | `ring:us` or `ring:20.0:us` | Ring tone (optional duration) |

## Most-Used Method Parameters

### play
```yaml
- play:
    urls: ["say:Hello", "https://example.com/audio.mp3"]
    volume: 10          # -40.0 to 40.0
    say_voice: Polly.Matthew
    say_language: en-US
    loop: 0             # 0 = loop forever
```

### prompt
```yaml
- prompt:
    play: ["say:Press 1 for yes, 2 for no"]
    max_digits: 1
    terminators: "#"
    digit_timeout: 5.0
    initial_timeout: 5.0
    speech_timeout: 10.0
    speech_language: en-US
    speech_hints: ["yes", "no", "help"]
```
Output vars: `prompt_result`, `prompt_value`, `prompt_digit_terminator`, `prompt_speech_confidence`

### connect
```yaml
- connect:
    to: "+15559876543"        # single destination
    from: "+15551230000"
    timeout: 30
    # OR serial: [{to: "..."}, {to: "..."}]
    # OR parallel: [{to: "..."}, {to: "..."}]
    # OR serial_parallel: [[{to: "..."}], [{to: "..."}]]
    result:
      case:
        connected: [hangup]
      default:
        - play: "say:No answer"
```
Output vars: `connect_result` (connected|failed), `connect_failed_reason`

### record
```yaml
- record:
    beep: true
    format: wav          # wav or mp3
    stereo: false
    direction: both      # speak, hear, both
    initial_timeout: 5.0
    end_silence_timeout: 3.0
    terminators: "#"
```
Output vars: `record_url`, `record_result`, `record_duration`

### record_call
```yaml
- record_call:
    format: wav
    stereo: true
    direction: both
    control_id: my_recording
```

### request
```yaml
- request:
    url: "https://api.example.com/data"
    method: POST           # GET, POST, PUT, DELETE
    headers:
      Authorization: "Bearer token123"
      Content-Type: application/json
    body:
      caller: "%{call.from}"
    timeout: 10
    save_variables: true   # save response body keys as vars
```

### ai (minimal)
```yaml
- ai:
    prompt:
      text: "You are a helpful agent."
    post_prompt:
      text: "Summarize the conversation."
    post_prompt_url: "https://example.com/report"
```

### ai (full config)
```yaml
- ai:
    prompt:
      text: "You are a customer service agent for Acme Corp."
      temperature: 0.7
      top_p: 0.9
      confidence: 0.6
    post_prompt:
      text: "Summarize: topic, resolution, sentiment."
    post_prompt_url: "https://example.com/report"
    hints: ["account", "billing", "support", "cancel"]
    params:
      attention_timeout: 30000
      inactivity_timeout: 120000
      end_of_speech_timeout: 1500
      ai_volume: 10
    languages:
      - name: English
        code: en-US
        voice: elevenlabs.rachel
    SWAIG:
      defaults:
        web_hook_url: "https://example.com/swaig"
      functions:
        - function: check_order
          purpose: "Look up a customer's order status by order number"
          argument:
            type: object
            properties:
              order_id:
                type: string
                description: "The order number"
      native_functions:
        - stop_and_transfer
```

## Variable Expansion

```yaml
# Access variables set by 'set' method
- play: "say:Your code is %{vars.code}"

# Access call properties (read-only)
- play: "say:Calling from %{call.from}"

# Access subroutine parameters
- play: "say:Hello %{params.name}"

# JavaScript expressions
- set:
    greeting: "%{vars.name ? 'Hi ' + vars.name : 'Hello'}"
```

## Branching Patterns

### switch (on variable value)
```yaml
- switch:
    variable: prompt_value
    case:
      "1": [{ transfer: "https://example.com/a.swml" }]
      "2": [{ transfer: "https://example.com/b.swml" }]
    default:
      - play: "say:Invalid option"
```

### cond (conditional array)
```yaml
- cond:
    - - "vars.balance > 100"
      - - play: "say:Premium customer"
    - - "vars.balance > 0"
      - - play: "say:Standard customer"
    - - play: "say:No balance found"
```

### if (condition/then/else)
```yaml
- if:
    condition: "vars.authenticated === true"
    then:
      - play: "say:Welcome back"
    else:
      - play: "say:Please verify your identity"
```

### goto (with condition and loop limit)
```yaml
- goto:
    when: "vars.prompt_value !== '1'"
    label: menu
    max: 3
```

## Subroutine Pattern

```yaml
version: 1.0.0
sections:
  main:
    - answer
    - execute:
        dest: play_greeting
        params:
          name: "caller"
        result:
          case:
            ok: [{ play: "say:Continuing..." }]
          default: [hangup]
    - hangup

  play_greeting:
    - play: "say:Welcome %{params.name}"
    - return: "ok"
```

## YAML vs JSON Side-by-Side

**YAML:**
```yaml
version: 1.0.0
sections:
  main:
    - answer
    - play:
        urls:
          - "say:Hello"
          - "silence:1"
        volume: 10
    - hangup
```

**JSON:**
```json
{
  "version": "1.0.0",
  "sections": {
    "main": [
      "answer",
      {
        "play": {
          "urls": ["say:Hello", "silence:1"],
          "volume": 10
        }
      },
      "hangup"
    ]
  }
}
```
