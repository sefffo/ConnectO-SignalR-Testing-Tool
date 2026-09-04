# 💡 ConnectO — SignalR Real-Time Testing Tool

A lightweight, single-page browser tool for manually testing **ConnectO**'s real-time messaging and notification features powered by **ASP.NET Core SignalR**.  
No install. No setup. Just open, paste your JWT, and start firing events.

🌐 **Live Tool:** [connect-o-signal-r-testing-tool.vercel.app](https://connect-o-signal-r-testing-tool.vercel.app/)

---

## 🧭 Overview

The tool connects to two SignalR hubs simultaneously:

| Hub | Endpoint | What it tests |
|---|---|---|
| **ChatHub** | `/hubs/chat` | Messages, typing indicators, read receipts, join/leave groups |
| **NotificationHub** | `/hubs/notifications` | DM creation, group events, role changes, group deletion |

Each hub has its own live log panel with color-coded entries so you can tell at a glance what's happening — events, sent actions, errors, and server confirmations.

---

## 🚀 Quick Start

### 1. Open the Tool in Two Tabs

> The tool is designed for **two-user simulation**. Open it in two separate browser tabs (or two different browsers) — each tab represents a different logged-in user.

```
https://connect-o-signal-r-testing-tool.vercel.app/
```

### 2. Get a JWT Token

Log in to the ConnectO API (e.g. via **API Dog** or **Swagger**) as a test user and copy the JWT access token from the response.

### 3. Configure & Connect

In each tab:

1. **Paste** your JWT into the **"JWT Token"** field at the top.
2. **Set the Server URL** — defaults to `http://localhost:5000`. Change it to your deployed API base URL if testing against a remote environment.
3. Click **🔗 Connect to Both Hubs**.

The two status indicators next to **ChatHub** and **NotificationHub** will turn **green** once both connections are established.

---

## 📋 Feature Walkthrough

### 💬 ChatHub Panel (Left)

#### Join a Conversation

Before you can send or receive messages in a conversation, both users must join the SignalR group for that conversation.

1. Paste the **Conversation ID** (GUID) into the "Conversation ID" field.  
   *(Or let it auto-fill — see the Notification panel below.)*
2. Click **↩ Join Conversation**.
3. Do the same in the second tab for the other user.

> Click **↪ Leave Conversation** to remove a user from the group and test that they stop receiving messages.

#### Send a Message

1. Make sure the **Conversation ID** field is filled.
2. Type your message in the **"Message"** input.
3. Click **▶ Send Message**.

The tool calls `POST /api/message` with `contentType: 0` (Text). On success:
- The **Chat Log** shows a green ✅ entry with the new message ID.
- The **message ID is auto-filled** in the "Message ID" field below.
- The other tab's Chat Log shows a `📨 MessageReceived` event with the message content and sender ID.

#### Typing Indicator

The typing indicator fires **automatically** as you type in the message input — no need to click anything. It sends `typing = true` while you type, then `typing = false` after a 2-second pause.

You can also trigger it manually:
- **✏ Typing: ON** — broadcasts that the user is typing.
- **✏ Typing: OFF** — broadcasts that the user stopped.

In the other tab, you'll see a live `✏️ [userId]... is typing` label appear above the chat log, which disappears after 3 seconds.

#### Mark as Read

After a message is received, its ID is auto-filled in the **"Message ID"** field.

1. Click **✔ Mark as Read**.
2. The tool calls `POST /api/message/read` with `upToMessageId`.
3. The other tab's Chat Log receives a `✅ ReadReceipt` event showing the reader's ID and how many messages were marked.

---

### 🔔 NotificationHub Panel (Right)

This panel listens passively to all 6 notification events. You don't trigger most of them directly — they fire as a result of API actions you perform (e.g. creating a conversation, updating a group).

#### Create a DM Conversation

1. Paste the **target user's GUID** into the "Target User ID" field.
2. Click **+ Create DM Conversation**.

This calls `POST /api/conversation/dm`. Both users' Notification Logs will show a `👤 NewConversation` event, and the **Conversation ID field is auto-filled** in the Chat panel — ready to join and start chatting immediately.

#### Observed Notification Events

| Event | Color | Triggered when |
|---|---|---|
| `👤 NewConversation` | Purple | A DM is created with you |
| `👥 NewGroupConversation` | Purple | You are added to a new group |
| `✏️ GroupUpdated` | Purple | Group name/info is changed |
| `⛔ RemovedFromGroup` | Red | You are removed from a group |
| `🔑 RoleChanged` | Purple | Your role in a group is changed |
| `🗑️ GroupDeleted` | Red | A group you're in is deleted |

---

## 🎨 Log Color Guide

| Color | Meaning |
|---|---|
| 🟣 **Purple** | Incoming SignalR event received from the server |
| 🟢 **Green** | Successful action or connection confirmed |
| 🟠 **Orange** | Action sent (outbound invoke or REST call) |
| 🔴 **Red** | Error or disconnection |
| ⚫ **Gray** | Informational (e.g. reconnecting, typing indicator) |

---

## 🧪 End-to-End Test Scenario

Here's a full flow to validate that messaging and real-time features are all working together:

1. **Tab A** — Paste User A's JWT, set server URL, click **Connect**.
2. **Tab B** — Paste User B's JWT, same server URL, click **Connect**.
3. **Tab B** — Paste User A's GUID into "Target User ID", click **+ Create DM**.
4. **Both tabs** — Notification log shows `NewConversation` and the Conversation ID auto-fills.
5. **Both tabs** — Click **↩ Join Conversation**.
6. **Tab A** — Type a message → observe typing indicator in Tab B's chat log.
7. **Tab A** — Click **▶ Send Message**.
8. **Tab B** — `MessageReceived` event appears; Message ID auto-fills.
9. **Tab B** — Click **✔ Mark as Read**.
10. **Tab A** — `ReadReceipt` event confirms the message was read.

---

## 🔧 Local Development

This is a static single-file tool (`index.html`). No build step needed.

```bash
git clone https://github.com/sefffo/ConnectO-SignalR-Testing-Tool.git
cd ConnectO-SignalR-Testing-Tool

# Just open it in your browser:
open index.html
# or serve it:
npx serve .
```

Point the **Server URL** to your local ConnectO API (`http://localhost:5000` by default).

> Make sure your ASP.NET Core backend has CORS configured to allow the tool's origin, and that SignalR hubs allow JWT authentication.

---

## 🛠 Tech Stack

- **Vanilla HTML/CSS/JS** — zero dependencies, no framework
- **Microsoft SignalR JS Client** v8.0.0 (via CDN)
- **Deployed on Vercel** as a static site

---

## 📁 Repository

| File | Description |
|---|---|
| `index.html` | The entire tool — UI, SignalR client, REST calls |
| `README.md` | This file |

---

> Built by [@sefffo](https://github.com/sefffo) as part of the **ConnectO** social media & chat platform.
