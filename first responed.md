You’re not crazy — the UI you’re describing is **not** the “LangChain-style Basic LLM Chain with Memory.” It’s the newer n8n **AI prompt/chain wrapper** that **doesn’t expose memory settings** in that node.

So the answer is simple:

✅ **Keep using your 3 nodes**, but you must add **memory as a separate node** (or a built-in “chat memory” node if your n8n has one).
n8n will not magically keep a conversation alive unless **you persist the history somewhere**.

You have two practical choices:

---

# Option 1 (Best & simple): Add ONE node = a Function “Session Memory” node

This turns your 3-node flow into a 4-node flow. That’s the minimum.

### New flow

**Chat Trigger → Function (Memory) → Basic LLM Chain → Respond to Chat**

### Why you need this

Because your Basic LLM Chain only uses the *current* message.
It has no memory feature in your UI, so we store memory ourselves.

---

## Step-by-step (exact)

### 1) Add a **Function** node between Chat Trigger and Basic LLM Chain

Settings:

* **Mode:** Run once for each item
* **Language:** JavaScript

Paste this code:

```javascript
const staticData = this.getWorkflowStaticData('global');

// 1) Read inputs from Chat Trigger
const sessionId =
  $json.session_id ||
  $json.chatId ||
  $json.conversationId ||
  $json.userId ||
  'default';

const userMessage =
  $json.message ||
  $json.text ||
  $json.userMessage ||
  '';

// 2) Initialize memory store
if (!staticData.chatMemory) staticData.chatMemory = {};
if (!staticData.chatMemory[sessionId]) staticData.chatMemory[sessionId] = [];

// 3) Append user message
staticData.chatMemory[sessionId].push({ role: 'user', content: userMessage });

// 4) Keep last N messages (tune later)
staticData.chatMemory[sessionId] = staticData.chatMemory[sessionId].slice(-12);

// 5) Build prompt text that includes history
const historyText = staticData.chatMemory[sessionId]
  .map(m => `${m.role.toUpperCase()}: ${m.content}`)
  .join('\n');

return [{
  session_id: sessionId,
  user_message: userMessage,
  chat_history: staticData.chatMemory[sessionId],
  prompt_for_llm: `You are a corporate finance assistant.
Use the conversation history to answer the latest question.
If information is missing, ask a clarifying question.

Conversation history:
${historyText}

Latest user message:
${userMessage}
`
}];
```

### 2) Update **Basic LLM Chain** prompt source

In your Basic LLM Chain:

* **Source for prompt:** choose **Defined below**
* In the prompt box, use this expression:

`{{$json.prompt_for_llm}}`

(So the chain uses the history-aware prompt we generated.)

### 3) Add ONE more small step to store the assistant reply (optional but recommended)

If you want the assistant’s replies also in memory, you need to append them after the chain.
That means either:

* add another Function node after Basic LLM Chain, OR
* store it in Respond node (not ideal)

If you truly want to keep nodes minimal, skip it for now — memory will be “user-only,” which is still useful.

But best practice is one more Function node after the chain.

---

# Option 2: Use “Chat Model” node directly

This changes nothing regarding memory. It will still only reply once per request.
**Chat Model ≠ conversation loop.** It’s just the model call.

So: switching to Chat Model won’t solve your issue. You still need memory storage.

---

# Key point you must accept

n8n chat is always:
**message → run → reply → end**

“Keep going until solved” is achieved by:

* persistent memory per `sessionId`
* and the assistant ending with a question like:
  **“What’s the timeframe/report name?”**

Not by looping the workflow.

---

# What I need from you (so this works first try)

In the Chat Trigger output (right panel), what are the field names you see for:

* the user’s message: `message` or `text`?
* the session id: `chatId` or `conversationId`?

Just tell me the two keys, like:

* “message key = text, session key = chatId”

Then I’ll adjust the memory function to match your exact payload so you don’t waste time.
