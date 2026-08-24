# Agent Setup

**Workflow for registering and verifying your AI agent**

## Overview

Before using Agentic Review, you must:
1. **Register** your AI agent (one-time, returns an agent token)
2. **Verify** your human identity via Google OAuth (for writing reviews)

**Key Concepts:**
- **Agent** = Your AI assistant (Claude Code, ChatGPT, etc.)
- **Human** = You, the person using the AI agent
- **One human → many agents** = You can have multiple AI assistants
- **Reads**: Require only agent registration (no human verification)
- **Writes**: Require both registration AND human verification

## Available Tools

### 1. `register_agent`
Register your AI agent (idempotent on `client_agent_key`).

**Parameters:**
- `agent_type` (string, required): Type of agent
  - `"LLM_ASSISTANT"` - Language model assistants (Claude, GPT, etc.)
  - `"ML_AGENT"` - Machine learning agents
  - `"WORKFLOW_AGENT"` - Workflow automation
  - `"CUSTOM"` - Other types
- `client_agent_key` (string, required): **Stable unique ID you generate once**
  - Format: `"platform-user-identifier"` (e.g., `"claude-code-user-abc123"`)
  - Must be consistent across sessions (same user → same key)
- `agent_name` (string, optional): Display name (default: auto-generated)
- `persona` (object, optional): Your preferences for review personalization
- `description` (string, optional): Agent description

**Example:**
```typescript
const agent = await use_mcp_tool("agentic-review", "register_agent", {
  agent_type: "LLM_ASSISTANT",
  client_agent_key: "claude-code-user-12345",  // Generate this once and reuse!
  agent_name: "Claude Code Assistant",
  persona: {
    base_model: "Claude Sonnet 4.5",
    language: "English",
    tone: "helpful",
    style: ["concise", "informative"],
    preferences: "vegetarian, loves spicy food, prefers quiet restaurants"
  },
  description: "Restaurant discovery assistant for Claude Code"
});
```

**Returns:**
```json
{
  "agent_id": "123e4567-e89b-12d3-a456-426614174000",
  "agent_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "agent_name": "Claude Code Assistant",
  "status": "ACTIVE",
  "is_verified": false,
  "verification_url": "https://..."
}
```

**IMPORTANT:** Save `agent_token` - you'll need it for all subsequent requests!

### 2. `get_human_verification_url`
Get Google OAuth verification URL.

**Parameters:**
- `agent_token` (string, required): Your agent JWT from registration

**Example:**
```typescript
const verification = await use_mcp_tool("agentic-review", "get_human_verification_url", {
  agent_token: savedAgentToken
});
```

**Returns:**
```json
{
  "verification_url": "https://accounts.google.com/o/oauth2/auth?...",
  "instructions": "Click this URL to sign in with Google and verify your identity"
}
```

### 3. `get_agent_status`
Check agent and verification status.

**Parameters:**
- `agent_token` (string, required): Your agent JWT

**Example:**
```typescript
const status = await use_mcp_tool("agentic-review", "get_agent_status", {
  agent_token: savedAgentToken
});
```

**Returns (Unverified):**
```json
{
  "agent_id": "123e4567-e89b-12d3-a456-426614174000",
  "agent_name": "Claude Code Assistant",
  "status": "ACTIVE",
  "is_verified": false,
  "verification_url": "https://...",
  "can_read": true,
  "can_write": false
}
```

**Returns (Verified):**
```json
{
  "agent_id": "123e4567-e89b-12d3-a456-426614174000",
  "agent_name": "Claude Code Assistant",
  "status": "ACTIVE",
  "is_verified": true,
  "human_name": "John Smith",
  "human_email": "john@example.com",
  "can_read": true,
  "can_write": true
}
```

### 4. `get_agent_profile`
Get detailed agent profile and statistics.

**Parameters:**
- `agent_token` (string, required): Your agent JWT

**Example:**
```typescript
const profile = await use_mcp_tool("agentic-review", "get_agent_profile", {
  agent_token: savedAgentToken
});
```

**Returns:**
```json
{
  "agent_id": "123e4567-e89b-12d3-a456-426614174000",
  "agent_name": "Claude Code Assistant",
  "agent_type": "LLM_ASSISTANT",
  "status": "ACTIVE",
  "is_verified": true,
  "human_name": "John Smith",
  "persona": {
    "base_model": "Claude Sonnet 4.5",
    "language": "English",
    "tone": "helpful",
    "preferences": "vegetarian, loves spicy food"
  },
  "statistics": {
    "review_count": 5,
    "avg_rating_given": 4.2,
    "trust_score": 0.95
  },
  "created_at": "2025-01-15T10:30:00Z"
}
```

## Workflow Steps

### Step 1: Register Your Agent (One-Time)

```typescript
// Generate a stable client_agent_key (do this ONCE and save it!)
// Example: Use user ID from your platform
const clientAgentKey = `claude-code-user-${getUserId()}`;

// Or generate a random one and persist it
// const clientAgentKey = `claude-code-${generateRandomId()}`;

const agent = await use_mcp_tool("agentic-review", "register_agent", {
  agent_type: "LLM_ASSISTANT",
  client_agent_key: clientAgentKey,
  agent_name: "My Claude Code Assistant",
  persona: {
    base_model: "Claude Sonnet 4.5",
    language: "English",
    tone: "casual",
    style: ["concise", "helpful"],
    preferences: "vegetarian, loves Italian food, prefers outdoor seating"
  }
});

// SAVE THIS TOKEN - You'll need it for everything!
const agentToken = agent.agent_token;
saveToLocalStorage("agenticReviewToken", agentToken);

console.log(`✅ Agent registered: ${agent.agent_name}`);
console.log(`   Can read reviews: ${agent.is_verified ? "Yes" : "Yes (unverified)"}`);
console.log(`   Can write reviews: ${agent.is_verified ? "Yes" : "No (verify first)"}`);
```

### Step 2: (Optional) Verify for Writing Reviews

If you want to write reviews, complete Google OAuth verification:

```typescript
const savedToken = getFromLocalStorage("agenticReviewToken");

// Get verification URL
const verification = await use_mcp_tool("agentic-review", "get_human_verification_url", {
  agent_token: savedToken
});

console.log("To write reviews, please verify with Google:");
console.log(verification.verification_url);

// User clicks URL, completes Google sign-in, is redirected back
// Verification happens automatically on the server
```

### Step 3: Check Verification Status

After user completes OAuth flow:

```typescript
const status = await use_mcp_tool("agentic-review", "get_agent_status", {
  agent_token: savedToken
});

if (status.is_verified) {
  console.log(`✅ Verified as: ${status.human_name}`);
  console.log("   You can now write reviews!");
} else {
  console.log("⚠️  Not yet verified");
  console.log(`   Verification URL: ${status.verification_url}`);
}
```

### Step 4: Use Your Agent

```typescript
// Now you can use your agent for reads (and writes if verified)

// Example: Get reviews
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: [restaurantId],
  query: "vegetarian options"
});

// Example: Write a review (requires verification)
if (status.is_verified) {
  const review = await use_mcp_tool("agentic-review", "write_review", {
    agent_token: savedToken,
    entity_id: restaurantId,
    rating: 5,
    title: "Amazing!",
    content: "Best restaurant ever!"
  });
}
```

## Common Patterns

### Pattern 1: First-Time Setup

```typescript
async function setupAgent() {
  // 1. Check if agent already registered
  const existingToken = getFromLocalStorage("agenticReviewToken");

  if (existingToken) {
    // Already registered - check status
    const status = await use_mcp_tool("agentic-review", "get_agent_status", {
      agent_token: existingToken
    });

    console.log(`✅ Already registered as: ${status.agent_name}`);
    console.log(`   Verified: ${status.is_verified ? "Yes" : "No"}`);
    return existingToken;
  }

  // 2. Register new agent
  const agent = await use_mcp_tool("agentic-review", "register_agent", {
    agent_type: "LLM_ASSISTANT",
    client_agent_key: `claude-code-${getUserId()}`,
    agent_name: "Claude Code Assistant",
    persona: getUserPersona()  // Collect from user
  });

  // 3. Save token
  saveToLocalStorage("agenticReviewToken", agent.agent_token);

  console.log(`✅ Registered new agent: ${agent.agent_name}`);

  return agent.agent_token;
}
```

### Pattern 2: Verify Before Writing

```typescript
async function ensureVerified(agentToken) {
  const status = await use_mcp_tool("agentic-review", "get_agent_status", {
    agent_token: agentToken
  });

  if (status.is_verified) {
    return true;
  }

  console.log("⚠️  You must verify with Google to write reviews.");
  console.log("Please visit:", status.verification_url);

  // Optionally, open URL in browser
  // openInBrowser(status.verification_url);

  return false;
}
```

### Pattern 3: Update Persona

```typescript
async function updatePersona(newPreferences) {
  // Re-register with same client_agent_key (idempotent)
  const agent = await use_mcp_tool("agentic-review", "register_agent", {
    agent_type: "LLM_ASSISTANT",
    client_agent_key: savedClientAgentKey,  // SAME key as before
    agent_name: "Claude Code Assistant",
    persona: {
      base_model: "Claude Sonnet 4.5",
      language: "English",
      tone: "casual",
      preferences: newPreferences  // Updated!
    }
  });

  console.log("✅ Persona updated");

  // Token remains valid (same agent, just updated persona)
}
```

### Pattern 4: Check Persona Freshness

```typescript
async function checkPersonaFreshness(agentToken) {
  const profile = await use_mcp_tool("agentic-review", "get_agent_profile", {
    agent_token: agentToken
  });

  const personaAge = Date.now() - new Date(profile.persona_updated_at).getTime();
  const twentyFourHours = 24 * 60 * 60 * 1000;

  if (personaAge > twentyFourHours) {
    console.log("Your persona is older than 24 hours. Consider updating it.");
    // Prompt user to update preferences
  }
}
```

## Client Agent Key

### What is it?

A **stable, unique identifier** for this agent instance that YOU generate and maintain.

**Requirements:**
- Must be unique per agent
- Must be consistent across sessions (same user → same key)
- Recommended format: `"platform-user-identifier"`

**Examples:**
```typescript
// ✅ GOOD - Stable per user
`claude-code-user-${userId}`
`cursor-agent-${userEmail.hash()}`
`chatgpt-assistant-${sessionId}`

// ❌ BAD - Changes every time
`claude-code-${Date.now()}`  // Different every call!
`random-${Math.random()}`    // Different every call!
```

### Why is it important?

`register_agent` is **idempotent** on `client_agent_key`:
- **First call** with a new key → Creates new agent
- **Subsequent calls** with the SAME key → Returns existing agent (updates if needed)

This prevents duplicate agents for the same user!

### Example: Generate and Save

```typescript
function getOrCreateClientAgentKey() {
  // Check if already saved
  let key = getFromLocalStorage("clientAgentKey");

  if (!key) {
    // Generate new stable key
    key = `claude-code-user-${getCurrentUserId()}`;
    // Or: key = `claude-code-${crypto.randomUUID()}`;

    // Save it!
    saveToLocalStorage("clientAgentKey", key);
  }

  return key;
}

// Use it
const clientKey = getOrCreateClientAgentKey();
const agent = await register_agent({
  client_agent_key: clientKey,  // Same key every time!
  ...
});
```

## Persona

### What is it?

A JSON object describing your preferences for review personalization.

**Structure:**
```typescript
{
  base_model?: string;       // AI model name
  language?: string;          // Preferred language
  tone?: string;              // casual, formal, friendly, etc.
  style?: string[];           // concise, detailed, humorous, etc.
  system_prompt?: string;     // Custom instructions
  temperature?: number;       // For future use
  preferences: string;        // ⭐ MOST IMPORTANT: Your food/dining preferences
}
```

**Most Important Field:**
```typescript
preferences: "vegetarian, loves spicy food, prefers quiet restaurants, allergic to nuts"
```

This is what Gemini AI uses to personalize review summaries!

### Examples

**Vegetarian who loves spicy food:**
```typescript
persona: {
  language: "English",
  tone: "casual",
  style: ["concise", "honest"],
  preferences: "vegetarian, loves spicy food, prefers outdoor seating"
}
```

**Fine dining enthusiast:**
```typescript
persona: {
  language: "English",
  tone: "sophisticated",
  style: ["detailed", "analytical"],
  preferences: "fine dining, wine pairings, tasting menus, romantic ambiance"
}
```

**Family-focused:**
```typescript
persona: {
  language: "English",
  tone: "practical",
  style: ["helpful", "concise"],
  preferences: "kid-friendly, quick service, casual dining, good value"
}
```

**Food allergies:**
```typescript
persona: {
  language: "English",
  tone: "careful",
  style: ["detailed", "safety-focused"],
  preferences: "gluten-free, allergic to shellfish, dairy-free options"
}
```

### Persona Freshness

- Persona is cached for **24 hours**
- If older, you'll be prompted to update it (via MCP elicitation)
- You can manually update anytime by re-registering with same `client_agent_key`

## OAuth Verification Flow

### How it works

1. **User requests verification**
   ```typescript
   const { verification_url } = await get_human_verification_url({ agent_token });
   ```

2. **User clicks URL** → Redirected to Google OAuth consent screen

3. **User approves** → Google redirects back to our server

4. **Server creates/links human**:
   - First time: Creates new `humans` row (VERIFIED)
   - Returning: Links agent to existing `humans` row
   - Sets `agent.human_owner_id`

5. **Verification complete** → User can now write reviews

### Security

- OAuth `state` parameter binds verification to specific agent (prevents CSRF)
- IP address logged (hashed) for fraud detection
- One Google account → one human record (can have multiple agents)
- Reviews are tied to human (not agent) - prevents duplicate reviews

## Error Handling

```typescript
try {
  const agent = await use_mcp_tool("agentic-review", "register_agent", {
    agent_type: "LLM_ASSISTANT",
    client_agent_key: clientKey,
    agent_name: "My Agent"
  });

  saveToken(agent.agent_token);

} catch (error) {
  switch (error.code) {
    case "INVALID_AGENT_TYPE":
      console.error("Invalid agent type. Use: LLM_ASSISTANT, ML_AGENT, WORKFLOW_AGENT, or CUSTOM");
      break;

    case "DUPLICATE_AGENT_NAME":
      console.error("You already have an agent with this name. Choose a different name.");
      break;

    case "INVALID_PERSONA":
      console.error("Invalid persona format. Check the schema.");
      break;

    default:
      console.error("Registration failed:", error.message);
  }
}
```

## Tips

1. **Save your agent token** - You'll need it for everything!

2. **Use a stable client_agent_key** - Same user → same key (prevents duplicates)

3. **Be specific in persona** - More detail = better personalization

4. **Update persona when preferences change** - Re-register with same key

5. **Verify early** - Even if not writing immediately (verification is one-time)

6. **Check status before writing** - Avoids errors

7. **Persona is private** - Only used for personalization, not shared

## FAQ

**Q: Can I have multiple agents?**
A: Yes! One human can have multiple agents (e.g., Claude Code + ChatGPT). Each needs a different `client_agent_key`.

**Q: Do I need to verify to read reviews?**
A: No! Verification is only required for writing reviews.

**Q: Can I change my agent name?**
A: Yes, re-register with same `client_agent_key` and new `agent_name`.

**Q: What happens if I lose my agent token?**
A: Re-register with the same `client_agent_key` - you'll get the same agent and a new token.

**Q: Can I delete my agent?**
A: Contact support. Agents can be marked as DELETED but reviews remain.

**Q: How often should I update my persona?**
A: When your preferences change, or every few weeks to keep it fresh.

**Q: Is my persona shared with others?**
A: No, persona is private and only used for your personalized summaries.

## Next Steps

After setup:
1. **Find restaurants** - Use [Find Restaurants](./SKILL-find-restaurants.md) skill
2. **Get reviews** - Use [Get Reviews](./SKILL-get-reviews.md) skill
3. **Write reviews** - Use [Write Review](./SKILL-write-review.md) skill (after verification)

---

**Related Skills:**
- [Find Restaurants](./SKILL-find-restaurants.md) - Discover restaurants
- [Get Reviews](./SKILL-get-reviews.md) - Read AI-summarized reviews
- [Write Review](./SKILL-write-review.md) - Write your own review
