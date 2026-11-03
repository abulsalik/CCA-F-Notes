# CCA-F-Notes
These are my notes and understanding of the material that covers CCA-F Claudes Certified Architect - Foundations exam. This should have definitions and models to help me understand how to make a application with claude efficently so I can hopefully not use too many tokens when building with AI. 

<b>Roles</b>
- 🤖 App = Manager
- 🧠 Claude = Consultant
- 👤 User = Customer
These above should be kept in mind as I will be using them when I go through and make examples of how Claude, the Application, and the User interact when requests are made and what task each side handles.

<b>General Vocab</b>
- 🗄️ session_id = in example terms it is a drawer, it is created by the app to organize the conversation between it, Claude, and the user
- 📁 request = in example it is a folder. it contains the message, the system prompt, and the tools. It is resent every time to the stateless Claude, it is how the app, the user, and Clade communicate. 
- 📝 message = in example it is the history, it separates the app and Claude and allows Claude to recall what it has said.
- 📞 call = is one request sent from the app to Claude and Claude back to the app.
- ‼️ system prompt = the rule card, the developer writes it, the app sends it, and Claude reads it in messages. Example: You are an agent who helps customers find their orders
- 🛠️ tool = it is what the app contains that it utilizes after being told by Claude
- stateless = Claude remembers nothing between calls, needs to be rebuilt and sent through messages every time
- 🔄 agentic loop = the user and assistant going back and forth with tool_use and tool_result requests until end_turn is reached. The loop lives in the app, not Claude
- token = small chunk of text about 3/4 of a word
- enum = enumeration, fixed list of allowed values that constraints Claude to select from instead of creating its own
- context window = all Claude can handle in one call in terms of what you've written and what it can send back
- 🔑 API = a code provided by Anthropic that will let you code in Claude to your application
- 😠 sentiment = the customer/users mood
- 🤓 confidence = Claude's own rating of certainty
- input schema = the rules for what a tool accepts, Example: order_id needs to be numerical, customer_id needs to be text
- 🎟️ confirmation token = a one time token that is generated and is required by the following code or else the call is rejected

<b>Concept Vocab</b>
- fallible = mistakes can happen no matter how good the prompt. The prompt is not the rule
- deterministic = same input always gives the same output
- non-deterministic = same inputs can give different outputs
- traps = solutions that may seem like they will solve the issue but ignore the underlining
  - ❌ fine-tuning = retaining the model | ✔️ This is never right as your levers are prompts, tools, context, and code
  - ❌ add-more = more tools, more agents, and bigger context window | ✔️ more options make choices worse and makes it easy to be non-deterministic, trim and reduce till you have only what is needed
  - ❌ mood/confidence = angry customer or Claude is unsure | ✔️ escalate to the next steps only when the policy triggers not based off of sentiment or confidence
  - ❌ prompt over enforcement = saying "Claude you must ALWAYS stop..." | ✔️ prompts cant guarantee anything because confidence can affects Claude's decisions, non-deterministic
  - ❌ trusting results = result comes back tool failed so you believe it found nothing | ✔️ found nothing is the correct answered but it stated it failed so there is an issue with the code or tools and needs to be handled
- atomic = two steps that happen at once, Example: find a seat and book it
- race condition = problem caused by a gap between the steps, Example: you find a seat but someone else books it and your booking fails
- split tools when actions need different inputs = if one tool is missing fields or has too many than adjust or separate it into multiple tools to make it easier for Claude to choose the correct tool
- show all results before Claude decides = two different inputs may share the same candidate so Claude should show the user and let the user decide which is to be used and not let it decide on its own

<b>Claude Vocab</b>
- ⚠️ stop_reason = it is given on each of Claude's response and tells the app why Claude stopped, examples look like: stop_reason: tool_use OR stop_reason: end_turn
- ➡️ tool_use = it is Claude saying, "Run this tool, call me again when the results"
- ⬅️ tool_result = it is send by the app to messages which is taken by Claude to use further
- 🛑 end_turn = it is Claude saying, " I am finished, show this reply to the user"
- "role:" = roles given inside the app put into session_id and messages that are passed along to Claude to help it know what it has said and what the app has said. Roles alternate as user then assistant
    - role: 👤 user = often the customer and occasionally the application sending anything to Claude
    - role: 🧠 assistant = always Claude
    - role: 🫵🏼 system = always the developer, specifically their rules
- max_token = set by developer for how long Claude's response can be. Limits Claude's single response, will cut off response mid-sentence, returns stop_reason: max_tokens

<b>General Rules</b>
- Claude is stateless, it never sees anything besides what is provided in messages and re-reads conversations as new since it holds no history.
  - Claude reads the request and decides what to do with it, for instance to use a tool which sends to request back to the app.
- The app cannot do anything by itself and is built to only handle requests and send information back and forth.
- The back and forth of the app and Claude is called the agentic loop.
- Claude never reads the folder aka the request it only reads the history aka the message to rebuild what has happened so far.
- The prompt is not the rule the code is so if you want it to happen every time each time make it deterministic, enforce by code.
  - Prompts are for judgement and code is for guarantees.
- a switch on a tool such as true or false is not a safety mechanism, Claude can call a tool with false.
- The confirmation Token system shows a impact and generates a one time confirmation token, the impact is reviewed and then the next call is sent, if that call doesn't have the token then the code rejects the call.
___
___
# Example of agentic loop
## <b>The agentic loop: customer, manager, consultant</b>
A walkthrough of one support chat, showing who does what at each step.

| Analogy | Real term | Job |
|---|---|---|
| Customer (Dana) | **user** | Sends messages |
| Manager | **App** | Runs tools, keeps history, reads `stop_reason` |
| Consultant | **Claude** | Reads the folder, writes replies |
| Rule card | **System prompt** | Developer's instructions for Claude |
| Folder | **The request** (`system` + `messages` + `tools`) | Rebuilt in full on every call |
| Drawer 7781 | **session_id** + stored history | Stays at the app, never sent to Claude |

## The whole loop at a glance

```mermaid
sequenceDiagram
    participant D as Customer (Dana)
    participant A as App (manager)
    participant C as Claude (consultant)

    D->>A: "Order 4417 hasn't arrived."
    Note over A: Opens drawer 7781 (session_id)<br/>Builds folder: system + tools + messages
    A->>C: Request 1
    C-->>A: "Let me check that" + tool_use: lookup_order 4417<br/>stop_reason: tool_use
    Note over A: Runs lookup_order itself<br/>Adds tool_result to messages
    A->>C: Request 2 (full history again)
    C-->>A: "Held at a hub, new date Oct 25"<br/>stop_reason: end_turn
    A->>D: Relays the answer
    Note over A: Saves everything in drawer 7781
```

## Step by step

Click each step to open it.

<details>
<summary><b>Step 1: Before anyone talks</b> (App)</summary>

The manager keeps a drawer for every conversation, labeled with a session_id (7781). He also keeps a rule card: "You are a support agent for Acme. Be brief." That rule card is the system prompt.

> ⚠️ **Watch:** the system prompt is the rule card the manager hands to the consultant: words for Claude. It is not the app"

</details>

<details>
<summary><b>Step 2: Dana walks in</b> (Customer)</summary>

"Hi, I'm Dana. Order 4417 hasn't arrived."

</details>

<details>
<summary><b>Step 3: Manager builds the folder</b> (App)</summary>

He opens drawer 7781 (empty, since it's a new chat), files Dana's message, and hands the consultant a folder:

```
system:   rule card
tools:    get_customer, lookup_order, process_refund
messages:
  user:   Hi, I'm Dana. Order 4417 hasn't arrived.
```

> ⚠️ **Watch:** the session_id stays in the manager's office. The consultant never sees the drawer, only the folder.

</details>

<details>
<summary><b>Step 4: Consultant replies</b> (Claude)</summary>

He says "Let me check that, Dana" and writes a note: look up order 4417. Then he sticks a label on his reply.

`stop_reason: tool_use`

> ⚠️ **Watch:** the consultant asks. He never runs the lookup himself, and he doesn't loop.

</details>

<details>
<summary><b>Step 5: Manager reads the label</b> (App)</summary>

The label says `tool_use`, so the consultant isn't finished. The manager does NOT relay "Let me check that" and walk out. He runs `lookup_order` himself: shipped Oct 20, held at a hub, new delivery Oct 25.

> ⚠️ **Watch:** `stop_reason` is a label on every reply, read by the app. It isn't a stop command. `tool_use` means "keep going."

</details>

<details>
<summary><b>Step 6: Manager hands back the full folder</b> (App)</summary>

The consultant remembers nothing from a minute ago, so the folder must hold everything again, now with the result added:

```
system:   rule card
messages:
  user:      Hi, I'm Dana. Order 4417 hasn't arrived.
  assistant: Let me check that, Dana.
             [tool_use: lookup_order 4417]
  user:      [tool_result: shipped Oct 20, held at hub,
              new delivery Oct 25]
```

> ⚠️ **Stateless:** every hand-off starts from zero. If the result isn't in the folder, it doesn't exist for the consultant. Note the `tool_result` sits under `user`, because it's the app speaking.

</details>

<details>
<summary><b>Step 7: Consultant answers</b> (Claude)</summary>

"Your order shipped Oct 20 but was held at a hub. New delivery date: Oct 25."

`stop_reason: end_turn`

> ⚠️ **Watch:** `end_turn` is the label on the reply, not the reply itself. The answer is the text above.

</details>

<details>
<summary><b>Step 8: Manager closes the turn</b> (App)</summary>

He reads `end_turn`, relays the answer to Dana, and saves the whole exchange in drawer 7781. This turn's loop is over.

</details>

<details>
<summary><b>Step 9: Dana's next message</b> (Customer → App)</summary>

"Can I just get a refund?" The manager opens drawer 7781 and sends the entire history, plus this new line, in a fresh folder.

> ⚠️ **Watch:** if the manager sent only the new line, the consultant would ask "What's your order number?" The fix is the plumbing, not the rule card.

</details>

<details>
<summary><b>Step 10: Same loop, new tool</b> (Claude → App)</summary>

The consultant writes a note, `process_refund 4417`, labeled `tool_use`. The manager decides whether to run it (he holds the keys), runs it, and returns the result. The consultant confirms the refund, labeled `end_turn`. The manager relays it to Dana.

`tool_use → end_turn`

</details>

## stop_reason cheat sheet

| stop_reason | Meaning | What the app does |
|---|---|---|
| `tool_use` | Claude wants a tool run | Runs it, adds `tool_result`, calls Claude again |
| `end_turn` | Claude finished | Shows the reply to the user |
| `max_tokens` | Claude ran out of room | Treats the reply as incomplete |

______________________________________________________________________________________________________________________________________________________________

## <b>Confirmation Token Example</b>
| Tool | Input | What it returns |
|---|---|---|
| `preview_delete_workspace` | `workspace_id` | The impact, plus a one-time `confirmation_token` |
| `execute_delete_workspace` | `workspace_id`, `confirmation_token` | Deletes, but <b>only</b> if the token is valid |

## Step by step

Click each step to open it.

<details>
<summary><b>Step 1: The user asks</b> "Delete the marketing workspace."</summary>

The manager keeps a drawer for every conversation, labeled with a session_id (7781). He also keeps a rule card: "You are a support agent for Acme. Be brief." That rule card is the system prompt.

> ⚠️ **Watch:** the system prompt is the rule card the manager hands to the consultant: words for Claude. It is not the app or "the system."

</details>

<details>
<summary><b>Step 2: Dana walks in</b> (Customer)</summary>

"Hi, I'm Dana. Order 4417 hasn't arrived."

</details>

<details>
<summary><b>Step 3: Manager builds the folder</b> (App)</summary>

He opens drawer 7781 (empty, since it's a new chat), files Dana's message, and hands the consultant a folder:

```
system:   rule card
tools:    get_customer, lookup_order, process_refund
messages:
  user:   Hi, I'm Dana. Order 4417 hasn't arrived.
```

> ⚠️ **Watch:** the session_id stays in the manager's office. The consultant never sees the drawer, only the folder.

</details>

<details>
<summary><b>Step 4: Consultant replies</b> (Claude)</summary>

He says "Let me check that, Dana" and writes a note: look up order 4417. Then he sticks a label on his reply.

`stop_reason: tool_use`

> ⚠️ **Watch:** the consultant asks. He never runs the lookup himself, and he doesn't loop.

</details>

<details>
<summary><b>Step 5: Manager reads the label</b> (App)</summary>

The label says `tool_use`, so the consultant isn't finished. The manager does NOT relay "Let me check that" and walk out. He runs `lookup_order` himself: shipped Oct 20, held at a hub, new delivery Oct 25.

> ⚠️ **Watch:** `stop_reason` is a label on every reply, read by the app. It isn't a stop command. `tool_use` means "keep going."

</details>

<details>
<summary><b>Step 6: Manager hands back the full folder</b> (App)</summary>

The consultant remembers nothing from a minute ago, so the folder must hold everything again, now with the result added:

```
system:   rule card
messages:
  user:      Hi, I'm Dana. Order 4417 hasn't arrived.
  assistant: Let me check that, Dana.
             [tool_use: lookup_order 4417]
  user:      [tool_result: shipped Oct 20, held at hub,
              new delivery Oct 25]
```

> ⚠️ **Stateless:** every hand-off starts from zero. If the result isn't in the folder, it doesn't exist for the consultant. Note the `tool_result` sits under `user`, because it's the app speaking.

</details>

<details>
<summary><b>Step 7: Consultant answers</b> (Claude)</summary>

"Your order shipped Oct 20 but was held at a hub. New delivery date: Oct 25."

`stop_reason: end_turn`

> ⚠️ **Watch:** `end_turn` is the label on the reply, not the reply itself. The answer is the text above.

</details>

<details>
<summary><b>Step 8: Manager closes the turn</b> (App)</summary>

He reads `end_turn`, relays the answer to Dana, and saves the whole exchange in drawer 7781. This turn's loop is over.

</details>

<details>
<summary><b>Step 9: Dana's next message</b> (Customer → App)</summary>

"Can I just get a refund?" The manager opens drawer 7781 and sends the entire history, plus this new line, in a fresh folder.

> ⚠️ **Watch:** if the manager sent only the new line, the consultant would ask "What's your order number?" The fix is the plumbing, not the rule card.

</details>

<details>
<summary><b>Step 10: Same loop, new tool</b> (Claude → App)</summary>

The consultant writes a note, `process_refund 4417`, labeled `tool_use`. The manager decides whether to run it (he holds the keys), runs it, and returns the result. The consultant confirms the refund, labeled `end_turn`. The manager relays it to Dana.

`tool_use → end_turn`

</details>




