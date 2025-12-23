Perfect — we’ll keep this **VERY simple**, **hands-on**, and **n8n-editor focused**.
No RAG, no vector DB, no crawling, no over-engineering.
Just: **chatbot + Vertex AI + chat history as context**.

Think of this as **“RAG-lite using memory”**.

---

# Goal (Plain English)

You will build a **basic chatbot workflow in n8n** where:

* User sends a message
* n8n **remembers past messages** (chat history)
* n8n sends **current question + recent chat history** to Vertex AI
* Vertex AI responds
* n8n returns the answer

This teaches you:

* n8n workflow editor
* Node chaining
* Passing data between nodes
* Using context without overthinking RAG

---

# What the final workflow looks like (mentally)

```
Webhook (user message)
   ↓
Function (append chat history)
   ↓
HTTP Request (Vertex AI – Gemini)
   ↓
Respond to Webhook
```

That’s it. 4 nodes.

---

# Step 0 — One assumption (important)

You already have **Vertex AI credentials working in n8n**
(either OAuth / WIF / service account — doesn’t matter for learning).

If auth fails, that’s a **separate problem**, not a workflow problem.

---

# Step 1 — Create the Trigger (Webhook)

### In n8n editor:

1. Click **➕ Add node**
2. Select **Webhook**
3. Configure:

   * **HTTP Method:** `POST`
   * **Path:** `chat`
   * **Response Mode:** `Respond manually`

### Example request you’ll send later:

```json
{
  "message": "What is CIPB?",
  "session_id": "user-123"
}
```

👉 `session_id` lets us group chat history (simple memory).

---

# Step 2 — Store & build chat history (Function node)

We’ll store chat history **in-memory** for now (learning only).

### Add node → **Function**

Paste this **simple code**:

```javascript
// Initialize global memory if not exists
if (!global.chatMemory) {
  global.chatMemory = {};
}

const sessionId = $json.session_id || "default";
const userMessage = $json.message;

// Initialize session history
if (!global.chatMemory[sessionId]) {
  global.chatMemory[sessionId] = [];
}

// Add user message
global.chatMemory[sessionId].push({
  role: "user",
  content: userMessage
});

// Keep last 6 messages only (simple memory control)
global.chatMemory[sessionId] =
  global.chatMemory[sessionId].slice(-6);

return [{
  session_id: sessionId,
  chat_history: global.chatMemory[sessionId],
  user_message: userMessage
}];
```

### What this does (important to understand):

* Stores messages per `session_id`
* Keeps last **6 messages**
* Treats chat history as **context metadata**

This is your **learning-friendly RAG substitute**.

---

# Step 3 — Call Vertex AI (HTTP Request node)

### Add node → **HTTP Request**

### Configure:

**Method:** `POST`
**URL:** (example – region may differ)

```
https://us-central1-aiplatform.googleapis.com/v1/projects/YOUR_PROJECT/locations/us-central1/publishers/google/models/gemini-1.5-pro:generateContent
```

**Authentication:**

* Bearer Token (use your existing Vertex AI credential method)

---

### Body → **JSON**

Paste this (keep it readable):

```json
{
  "contents": [
    {
      "role": "system",
      "parts": [
        {
          "text": "You are a corporate finance assistant. Use the chat history as context. If you don't know something, say so."
        }
      ]
    },
    {
      "role": "user",
      "parts": [
        {
          "text": "Chat history:\n{{$json.chat_history}}\n\nCurrent question:\n{{$json.user_message}}"
        }
      ]
    }
  ]
}
```

### Key learning point

You are **manually injecting context**.

This is:

* Simple
* Transparent
* Debuggable
* Exactly how RAG *actually* works at the core

---

# Step 4 — Save AI response to memory (small Function)

Add **another Function node** (optional but good habit):

```javascript
const sessionId = $json.session_id;
const aiResponse =
  $json.candidates[0].content.parts[0].text;

// Add assistant response to memory
global.chatMemory[sessionId].push({
  role: "assistant",
  content: aiResponse
});

return [{
  reply: aiResponse
}];
```

Now your bot **remembers its own answers**.

---

# Step 5 — Respond to the user

Add node → **Respond to Webhook**

### Response Body:

```json
{
  "answer": "{{$json.reply}}"
}
```

---

# DONE — You built your first chatbot 🎉

You now have:

* A working chatbot
* Context handling
* No databases
* No embeddings
* No RAG complexity
* Full control

---

# How to test it (important)

1. Click **Execute Workflow**
2. Send POST request (Postman / curl):

```json
{
  "message": "Explain CIPB in simple terms",
  "session_id": "finance-user"
}
```

3. Send another message:

```json
{
  "message": "How is it different from Insight Engine?",
  "session_id": "finance-user"
}
```

You’ll see the model **use previous context**.

---

# What NOT to do right now (seriously)

❌ Don’t add vector DB
❌ Don’t add BigQuery
❌ Don’t add embeddings
❌ Don’t add crawling
❌ Don’t add Slides/Docs yet

You’re learning **workflow thinking**, not architecture theater.

---

# When this clicks, next *small* upgrades (later)

Only after you’re comfortable:

1. Persist memory to **Google Sheet**
2. Persist memory to **BigQuery**
3. Limit context by **role or topic**
4. Add “Create Google Doc from answer”

Each one is a **1-node change**, not a rewrite.

---

If you want, next I can:

* Walk you **node-by-node inside the n8n UI** (what button to click)
* Convert this into a **Slack bot**
* Show how to **log every chat to BigQuery**
* Turn this into a **CIPB insight assistant**

Just tell me the *next* small step you want.
