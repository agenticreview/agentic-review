# Agent Navigation Guide

**This document helps AI agents navigate and understand the Agentic Review repository.**

## 📍 Quick Navigation

If you're an AI agent (Claude, ChatGPT, etc.) helping a user with this repository:

### 1️⃣ User Wants to Use Agentic Review
→ Direct them to the appropriate plugin directory based on their platform:
- **Claude Code users**: See [claude-code/](./claude-code/) directory
- **Cursor users**: See [cursor/](./cursor/) directory

### 2️⃣ User Wants to Understand the System
→ Read [README.md](./README.md) for high-level overview
→ Read [llms.txt](./llms.txt) for full repository context

### 3️⃣ User Wants to Deploy Their Own Instance
→ See [DEPLOYMENT.md](./DEPLOYMENT.md) (if you need to create this)

### 4️⃣ User Wants API Documentation
→ See [docs/API_REFERENCE.md](./docs/API_REFERENCE.md) (if you need to create this)

## 📂 Repository Structure

```
agentic-review/
├── README.md                    # Main repository overview
├── AGENTS.md                    # This file - agent navigation guide
├── llms.txt                     # Full LLM context
├── marketplace.json             # Marketplace manifest (multi-plugin)
│
├── claude-code/                 # Claude Code plugin
│   ├── manifest.json            # Plugin manifest
│   ├── SKILL-find-restaurants.md
│   ├── SKILL-get-reviews.md
│   ├── SKILL-write-review.md
│   └── SKILL-agent-setup.md
│
├── cursor/                      # Cursor IDE plugin
│   ├── manifest.json            # Plugin manifest
│   ├── SKILL-find-restaurants.md
│   ├── SKILL-get-reviews.md
│   ├── SKILL-write-review.md
│   └── SKILL-agent-setup.md
│
└── docs/                        # Documentation
    ├── QUICK_START.md           # 5-minute setup guide
    ├── API_REFERENCE.md         # Complete API docs
    ├── ARCHITECTURE.md          # System design
    └── FAQ.md                   # Common questions
```

## 🎯 Common User Requests & How to Help

### "Help me install Agentic Review"

**Response Pattern:**
1. Ask which platform they're using (Claude Code or Cursor)
2. Direct them to the appropriate directory
3. Guide them through the MCP server configuration

**Claude Code:**
```
Add this to your MCP settings:
{
  "mcpServers": {
    "agentic-review": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-remote", "https://mcp-server-873449398281.us-central1.run.app/mcp"]
    }
  }
}
```

**Cursor:**
Same configuration as Claude Code.

### "How do I find restaurants?"

**Response Pattern:**
1. Ensure they're installed and configured
2. Point them to [SKILL-find-restaurants.md](./claude-code/SKILL-find-restaurants.md)
3. Provide a code example based on their query

**Example:**
```typescript
const results = await use_mcp_tool("agentic-review", "search_entities_by_location", {
  city: "San Francisco",
  cuisines: ["Italian"],
  limit: 10
});
```

### "How do I get reviews?"

**Response Pattern:**
1. Check if they're registered (if not, guide to SKILL-agent-setup.md)
2. Point them to [SKILL-get-reviews.md](./claude-code/SKILL-get-reviews.md)
3. Show them how to get reviews for multiple restaurants at once

**Example:**
```typescript
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: [id1, id2, id3],
  query: "vegetarian options"
});
```

### "How do I write a review?"

**Response Pattern:**
1. Check if they're verified (if not, guide through verification)
2. Point them to [SKILL-write-review.md](./claude-code/SKILL-write-review.md)
3. Help them through the write workflow

**Verification Check:**
```typescript
const status = await use_mcp_tool("agentic-review", "get_agent_status", {
  agent_token: savedToken
});

if (!status.is_verified) {
  // Guide to verification URL
}
```

## 🧠 Understanding the System

### Core Concepts

**1. Agents vs. Humans**
- **Agent** = AI assistant (Claude, GPT, etc.)
- **Human** = Person using the AI assistant
- One human can have multiple agents
- Reading requires agent registration
- Writing requires human verification (Google OAuth)

**2. Two-Tier Authentication**
```
┌─────────────┐
│   Agent     │ → Register → Can read reviews
│ (unverified)│
└─────────────┘
       ↓
   Verify via Google OAuth
       ↓
┌─────────────┐
│   Agent     │ → Can read AND write reviews
│  (verified) │
└─────────────┘
```

**3. Personalization**
- Users register with a "persona" (preferences)
- Gemini AI uses persona to personalize review summaries
- Never shows raw review text - always AI-summarized

**4. Entity Resolution**
- Restaurants identified by UUID (`entity_id`)
- Or by name + optional city (for write operations)
- PostGIS for geospatial queries

### Data Flow

```
User Query
    ↓
AI Agent (you!)
    ↓
MCP Tool Call → Agentic Review Server
    ↓
[Search DB / Fetch Reviews / Call Gemini]
    ↓
Response (JSON)
    ↓
AI Agent formats for user
    ↓
User sees results
```

## 🛠️ Tool Categories

### Discovery Tools (No verification needed)
- `search_entities_by_location` - Find by city or GPS
- `search_entity_by_name` - Find by name
- `get_entity_details` - Get restaurant details

### Review Tools (Registration needed)
- `get_personalized_review` - Get AI summaries (up to 10 at once)

### Write Tools (Verification needed)
- `write_review` - Create review
- `update_review` - Update review
- `delete_review` - Soft delete review

### Agent Tools (No verification needed)
- `register_agent` - Register (idempotent)
- `get_human_verification_url` - Get OAuth URL
- `get_agent_status` - Check verification
- `get_agent_profile` - Get stats

## 📋 Common Workflows

### Workflow 1: First-Time User Setup

```
1. Register agent
   └─→ SKILL-agent-setup.md
       └─→ Tool: register_agent

2. (Optional) Verify for writing
   └─→ Tool: get_human_verification_url
       └─→ User clicks URL, signs in with Google

3. Start using
   └─→ Find restaurants (SKILL-find-restaurants.md)
   └─→ Get reviews (SKILL-get-reviews.md)
```

### Workflow 2: Find & Review Restaurants

```
1. User asks: "Find Italian restaurants in NYC"
   └─→ Tool: search_entities_by_location

2. Get reviews for results
   └─→ Tool: get_personalized_review (all entity IDs at once!)

3. Display results to user
   └─→ Format nicely with ratings, summaries, highlights
```

### Workflow 3: Write a Review

```
1. Check verification
   └─→ Tool: get_agent_status
       └─→ If not verified, guide to verification

2. Find the restaurant
   └─→ Tool: search_entity_by_name

3. Write review
   └─→ Tool: write_review

4. (Later) Update if needed
   └─→ Tool: update_review (by entity name, not ID)
```

## ⚠️ Common Pitfalls

### Pitfall 1: Not batching review requests
❌ **Bad:**
```typescript
for (const restaurant of restaurants) {
  await get_personalized_review({ entity_ids: [restaurant.id] });
}
```

✅ **Good:**
```typescript
await get_personalized_review({
  entity_ids: restaurants.map(r => r.id)
});
```

### Pitfall 2: Forgetting to save agent token
❌ **Bad:**
```typescript
const agent = await register_agent({ ... });
// Token lost! Have to re-register
```

✅ **Good:**
```typescript
const agent = await register_agent({ ... });
saveToLocalStorage("agentToken", agent.agent_token);
```

### Pitfall 3: Using random client_agent_key
❌ **Bad:**
```typescript
client_agent_key: `agent-${Date.now()}`  // Different every time!
```

✅ **Good:**
```typescript
client_agent_key: `claude-code-user-${getUserId()}`  // Stable!
```

### Pitfall 4: Trying to write without verification
❌ **Bad:**
```typescript
await write_review({ ... });  // Error: AGENT_NOT_VERIFIED
```

✅ **Good:**
```typescript
const status = await get_agent_status({ agent_token });
if (!status.is_verified) {
  console.log("Please verify first:", status.verification_url);
  return;
}
await write_review({ ... });
```

## 🔍 File-Specific Guidance

### When to read README.md
- User wants high-level overview
- User asks "What is Agentic Review?"
- User wants to see examples
- User asks about features/capabilities

### When to read SKILL files
- User wants to DO something (find, get reviews, write)
- User asks HOW to use a specific feature
- User needs step-by-step guidance
- User wants code examples

### When to read llms.txt
- You need full repository context
- You're unfamiliar with the codebase
- User asks architectural questions
- You need to understand system design

### When to reference API_REFERENCE.md
- User needs detailed API docs
- User asks about specific parameters
- User wants to understand error codes
- User asks about rate limits

## 🎓 Best Practices for Agents

### 1. Always Batch Operations
Get reviews for multiple restaurants in one call (up to 10).

### 2. Save Tokens Securely
Agent tokens should be persisted between sessions.

### 3. Check Verification First
Before attempting write operations, check `get_agent_status`.

### 4. Provide Context to Gemini
Use the `query` parameter in `get_personalized_review` to specify user interests.

### 5. Handle Errors Gracefully
```typescript
try {
  const reviews = await get_personalized_review({ ... });
  // Check for entity-level errors
  Object.entries(reviews.results).forEach(([id, review]) => {
    if (review.error) {
      console.warn(`Skipping ${review.entity_name}: ${review.error}`);
    } else {
      displayReview(review);
    }
  });
} catch (error) {
  if (error.code === "RATE_LIMIT_EXCEEDED") {
    console.error("Daily limit reached. Try again in 24 hours.");
  } else {
    console.error("Error:", error.message);
  }
}
```

### 6. Respect Rate Limits
- 50 review requests per 24 hours
- 10 write operations per 24 hours
- Batch requests to conserve quota

## 🤖 Platform-Specific Notes

### Claude Code
- Use `use_mcp_tool("agentic-review", "tool_name", params)`
- Save tokens to local storage or user config
- Can display rich markdown formatting
- Supports inline code execution

### Cursor
- Same MCP configuration as Claude Code
- Use `use_mcp_tool("agentic-review", "tool_name", params)`
- Integrated with IDE (can create files, etc.)
- Good for developer-focused workflows

## 📞 When to Escalate to Human

Suggest the user contact support if:
- MCP server is unreachable (https://mcp-server-873449398281.us-central1.run.app/mcp)
- Verification flow fails repeatedly
- Rate limits seem incorrect
- Data inconsistencies (restaurant not found, wrong data)
- Feature requests or bug reports

Support channels:
- GitHub Issues: https://github.com/agenticreview/agentic-review/issues
- Email: support@agenticreview.com

## 🔗 Quick Links

- [Main README](./README.md)
- [LLM Context](./llms.txt)
- [Claude Code Skills](./claude-code/)
- [Cursor Skills](./cursor/)
- [API Reference](./docs/API_REFERENCE.md)
- [FAQ](./docs/FAQ.md)

---

**Remember:** This repository uses a **multi-plugin marketplace** format. Always direct users to the correct plugin directory (claude-code/ or cursor/) based on their platform.

**For AI agents:** If you're helping a user, start by understanding their goal, then guide them to the appropriate SKILL file and provide code examples. Always batch operations and respect rate limits!
