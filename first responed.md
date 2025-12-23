That’s a **good sign**. It means:

✅ Postman successfully hit your webhook
✅ Your workflow started running
❌ n8n couldn’t send a response back because you **didn’t add the “Respond to Webhook” node yet** (or it’s not connected)

So this is a **configuration gap**, not a failure.

---

# Fix in 60 seconds

## Step 1 — Confirm your Webhook node setting

Open **Webhook node** → set:

**Response:** ✅ `Using Respond to Webhook node`

(You already did this earlier, but re-check it.)

---

## Step 2 — Add the missing node

1. Click **➕ Add node**
2. Search: **Respond to Webhook**
3. Add it at the **end** of your current chain
4. Connect it:

**Webhook → Function → Respond to Webhook**

(If you have only Webhook + Function right now, that’s fine.)

---

## Step 3 — Configure “Respond to Webhook” node

Open it and set:

* **Response Body:** JSON

Paste this:

```json
{
  "ok": true,
  "echo": "{{$json.user_message || $json.message || 'no message found'}}",
  "session_id": "{{$json.session_id || 'default'}}"
}
```

If you don’t see `user_message` yet, that’s okay — this is just a test response.

---

## Step 4 — Test again in Postman

Use the same request:

```json
{
  "message": "Hello from Postman",
  "session_id": "test-session-1"
}
```

Now Postman should return **200** with a JSON response.

---

# Why Postman showed “code 0”

In Postman, **code 0** commonly happens when the client doesn’t get a proper HTTP response (timeout / connection closed / workflow didn’t respond). In your case, n8n explicitly said why:

> **No Respond to Webhook node found**

So once you add it, Postman will behave normally.

---

# After this works (next step)

Once Postman gets a clean JSON response, we insert the Vertex AI call **between** Function and Respond:

**Webhook → Function (memory) → HTTP Request (Vertex AI) → Function (extract reply) → Respond**

---

## Do this now and tell me one thing:

After adding **Respond to Webhook**, what status code do you get in Postman — **200** or something else?
