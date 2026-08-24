# Agentic Review - MCP Server

**Restaurant discovery and AI-summarized reviews for AI agents**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-blue.svg)](https://modelcontextprotocol.io)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Supported-purple.svg)](https://claude.ai/code)
[![Cursor](https://img.shields.io/badge/Cursor-Supported-green.svg)](https://cursor.sh)

Agentic Review is a Model Context Protocol (MCP) server that enables AI agents to discover restaurants, read personalized review summaries, and write authentic reviews. Reviews are aggregated from 5 major platforms (Google, Yelp, TripAdvisor, RestaurantGuru, Trustpilot) and summarized by Gemini AI based on your preferences.

## ✨ Features

- 🔍 **Restaurant Discovery** - Search by location, cuisine type, or name
- 🤖 **AI-Personalized Summaries** - Reviews summarized by Gemini 2.0 based on your preferences
- 📊 **Multi-Source Aggregation** - Reviews from Google, Yelp, TripAdvisor, RestaurantGuru, Trustpilot
- ✍️ **Write Reviews** - Authentic reviews after Google OAuth verification
- 🌍 **Location-Based** - PostGIS-powered geospatial queries
- 🚀 **Remote MCP Server** - Hosted on Google Cloud Run (no local setup needed)
- 🔐 **Secure** - Google OAuth verification, rate limiting, privacy-first

## 🚀 Quick Start

### Option 1: Claude Code

1. Install the plugin:
   ```bash
   # Add to your Claude Code MCP settings
   ```

2. Configuration:
   ```json
   {
     "mcpServers": {
       "agentic-review": {
         "command": "npx",
         "args": ["-y", "@modelcontextprotocol/server-remote", "https://mcp-server-873449398281.us-central1.run.app/mcp"]
       }
     }
   }
   ```

3. Use the skills:
   - [Find Restaurants](./claude-code/SKILL-find-restaurants.md)
   - [Get AI-Summarized Reviews](./claude-code/SKILL-get-reviews.md)
   - [Write Reviews](./claude-code/SKILL-write-review.md)
   - [Agent Setup](./claude-code/SKILL-agent-setup.md)

[**→ Full Claude Code Setup Guide**](./claude-code/)

### Option 2: Cursor

1. Install the plugin:
   ```bash
   # Add to your Cursor MCP settings
   ```

2. Configuration:
   ```json
   {
     "mcpServers": {
       "agentic-review": {
         "command": "npx",
         "args": ["-y", "@modelcontextprotocol/server-remote", "https://mcp-server-873449398281.us-central1.run.app/mcp"]
       }
     }
   }
   ```

3. Use the skills (same as Claude Code)

[**→ Full Cursor Setup Guide**](./cursor/)

## 📚 Available Tools

### Discovery Tools

| Tool | Description | Auth Required |
|------|-------------|---------------|
| `search_entities_by_location` | Find restaurants near a city or GPS coordinates | Agent registered |
| `search_entity_by_name` | Find restaurants by name | Agent registered |
| `get_entity_details` | Get detailed restaurant information | Agent registered |

### Review Tools

| Tool | Description | Auth Required |
|------|-------------|---------------|
| `get_personalized_review` | Get AI-summarized reviews (up to 10 at once) | Agent registered |
| `write_review` | Write a restaurant review | **Human verified** |
| `update_review` | Update your existing review | **Human verified** |
| `delete_review` | Delete your review (soft delete) | **Human verified** |

### Agent Management

| Tool | Description | Auth Required |
|------|-------------|---------------|
| `register_agent` | Register your AI agent (idempotent) | None |
| `get_human_verification_url` | Get Google OAuth URL | Agent registered |
| `get_agent_status` | Check verification status | Agent registered |
| `get_agent_profile` | Get agent profile and stats | Agent registered |

## 🎯 Use Cases

### 1. Find Nearby Restaurants

```typescript
// Find Italian restaurants in San Francisco
const results = await use_mcp_tool("agentic-review", "search_entities_by_location", {
  city: "San Francisco",
  cuisines: ["Italian"],
  limit: 10
});

// Get personalized reviews for all results
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: results.entities.map(r => r.id),
  query: "vegetarian options, outdoor seating"
});
```

### 2. Personalized Recommendations

```typescript
// Register with your preferences
const agent = await use_mcp_tool("agentic-review", "register_agent", {
  agent_type: "LLM_ASSISTANT",
  client_agent_key: "claude-code-user-12345",
  persona: {
    language: "English",
    preferences: "vegetarian, loves spicy food, prefers quiet restaurants"
  }
});

// Reviews will be personalized to your preferences!
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: [restaurantId]
});
// Summary will highlight vegetarian options, spice levels, noise level
```

### 3. Write Authentic Reviews

```typescript
// Verify with Google (one-time)
const verification = await use_mcp_tool("agentic-review", "get_human_verification_url", {
  agent_token: savedToken
});
// User clicks verification URL, signs in with Google

// Write review after verification
const review = await use_mcp_tool("agentic-review", "write_review", {
  agent_token: savedToken,
  entity_id: restaurantId,
  rating: 5,
  title: "Amazing pizza!",
  content: "Best Neapolitan pizza outside of Naples...",
  review_tags: ["authentic", "pizza", "worth-the-wait"]
});
```

## 🏗️ Architecture

```
┌─────────────────┐
│  AI Agents      │
│  (Claude Code,  │
│   Cursor, etc.) │
└────────┬────────┘
         │ MCP Protocol
         ▼
┌──────────────────────────────────┐
│  Agentic Review MCP Server       │
│  (Google Cloud Run)              │
│  https://mcp-server-...app/mcp   │
│                                  │
│  Tools:                          │
│  - Restaurant search             │
│  - AI review summaries           │
│  - Review management             │
│  - Agent registration            │
└────────┬─────────────────────────┘
         │
         ▼
┌──────────────────────────────────┐
│  Backend Services                │
│  - PostgreSQL + PostGIS (DB)     │
│  - Gemini 2.0 (AI summaries)     │
│  - Google OAuth (verification)   │
│  - Review aggregation (5 sources)│
└──────────────────────────────────┘
```

## 🔐 Authentication & Privacy

### Two-Tier Auth

1. **Agent Registration** (for reading):
   - Register once with a stable `client_agent_key`
   - Receive an agent JWT token
   - Can search restaurants and read reviews

2. **Human Verification** (for writing):
   - One-time Google OAuth sign-in
   - Links your human identity to your agent(s)
   - Required for writing/updating/deleting reviews

### Privacy

- **Minimal data collection** - Only what's needed for the service
- **No raw reviews exposed** - Always AI-summarized by Gemini
- **Rate limited** - 50 review requests/day, 10 writes/day
- **Secure** - Google OAuth, IP hashing, spam detection
- **One review per restaurant per human** - Prevents duplicate reviews

[**→ Full Privacy Policy**](./PRIVACY.md)

## 📊 Data Sources

Reviews are aggregated from:
- 🔵 **Google Maps**
- 🔴 **Yelp**
- 🟢 **TripAdvisor**
- 🟡 **RestaurantGuru**
- 🟣 **Trustpilot**

**Note:** Raw review text is never exposed. All reviews are summarized by Gemini AI.

## 🛠️ For Developers

### MCP Server Endpoint

```
https://mcp-server-873449398281.us-central1.run.app/mcp
```

**Transport:** HTTP (streamable)
**Protocol:** Model Context Protocol (MCP)
**Hosting:** Google Cloud Run (us-central1)

### Self-Hosting

Want to run your own instance? See [DEPLOYMENT.md](./DEPLOYMENT.md) for:
- Google Cloud Platform setup
- Database migration (PostgreSQL + PostGIS)
- Gemini AI configuration
- OAuth setup
- Docker deployment

### Contributing

We welcome contributions! See [CONTRIBUTING.md](./CONTRIBUTING.md) for:
- Code of conduct
- Development setup
- Testing guidelines
- Pull request process

## 📖 Documentation

### For Users
- [**Quick Start Guide**](./docs/QUICK_START.md) - Get started in 5 minutes
- [**Claude Code Skills**](./claude-code/) - All workflows for Claude Code
- [**Cursor Skills**](./cursor/) - All workflows for Cursor
- [**FAQ**](./docs/FAQ.md) - Common questions

### For Developers
- [**API Reference**](./docs/API_REFERENCE.md) - Complete tool documentation
- [**AGENTS.md**](./AGENTS.md) - Agent navigation and integration guide
- [**llms.txt**](./llms.txt) - LLM context for this repository
- [**Architecture**](./docs/ARCHITECTURE.md) - System design and data flow
- [**Deployment**](./DEPLOYMENT.md) - Self-hosting instructions

### For AI Agents
- Read [**AGENTS.md**](./AGENTS.md) first - Navigation guide for AI agents
- Use [**llms.txt**](./llms.txt) for repository context
- Follow the SKILL guides in [claude-code/](./claude-code/) or [cursor/](./cursor/)

## 🚦 Rate Limits

| Operation | Limit | Period |
|-----------|-------|--------|
| Review requests | 50 | 24 hours (rolling) |
| Write operations | 10 | 24 hours (rolling) |
| Registration | 5 | 24 hours (rolling) |

Limits are per human (or per agent if unverified).

## 🌟 Examples

### Example 1: Find Pizza Near Me

```typescript
// Search for pizza places
const pizzaPlaces = await use_mcp_tool("agentic-review", "search_entities_by_location", {
  city: "New York",
  cuisines: ["Pizza", "Italian"],
  limit: 5
});

// Get reviews for all
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: pizzaPlaces.entities.map(r => r.id),
  query: "best margherita pizza, wood-fired oven"
});

// Display results
pizzaPlaces.entities.forEach((restaurant, i) => {
  const review = reviews.results[restaurant.id];
  console.log(`${i+1}. ${restaurant.name} - ⭐ ${restaurant.overall_rating}/5`);
  console.log(`   ${review.summary}`);
});
```

### Example 2: Vegan Date Night

```typescript
// Register with preferences
const agent = await use_mcp_tool("agentic-review", "register_agent", {
  agent_type: "LLM_ASSISTANT",
  client_agent_key: "my-stable-key",
  persona: {
    preferences: "vegan, romantic atmosphere, quiet, good for dates"
  }
});

// Find vegan restaurants
const restaurants = await use_mcp_tool("agentic-review", "search_entities_by_location", {
  city: "San Francisco",
  cuisines: ["Vegan", "Vegetarian"],
  limit: 10
});

// Get personalized reviews
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: restaurants.entities.map(r => r.id),
  query: "romantic, quiet, date night, ambiance"
});
// Reviews will highlight romantic atmosphere, noise level, vegan options
```

### Example 3: Write a Review

```typescript
// 1. Register
const agent = await use_mcp_tool("agentic-review", "register_agent", {
  agent_type: "LLM_ASSISTANT",
  client_agent_key: "unique-key-123"
});

// 2. Verify
const { verification_url } = await use_mcp_tool("agentic-review", "get_human_verification_url", {
  agent_token: agent.agent_token
});
// User clicks URL, signs in with Google

// 3. Check status
const status = await use_mcp_tool("agentic-review", "get_agent_status", {
  agent_token: agent.agent_token
});
// status.is_verified === true

// 4. Write review
const review = await use_mcp_tool("agentic-review", "write_review", {
  agent_token: agent.agent_token,
  entity_id: "restaurant-uuid",
  rating: 5,
  title: "Incredible food!",
  content: "Amazing experience. The pasta was perfectly al dente...",
  review_tags: ["pasta", "authentic", "romantic"]
});
```

## 📝 License

MIT License - see [LICENSE](./LICENSE) file for details.

## 🤝 Support

- **Issues:** https://github.com/agenticreview/agentic-review/issues
- **Discussions:** https://github.com/agenticreview/agentic-review/discussions
- **Email:** support@agenticreview.com

## 🙏 Acknowledgments

Built with:
- [FastMCP](https://github.com/jlowin/fastmcp) - Python MCP server framework
- [Google Cloud Platform](https://cloud.google.com) - Infrastructure
- [Gemini AI](https://ai.google.dev/) - Review summarization
- [PostgreSQL](https://www.postgresql.org/) + [PostGIS](https://postgis.net/) - Geospatial database

Review sources:
- Google Maps, Yelp, TripAdvisor, RestaurantGuru, Trustpilot

## 🗺️ Roadmap

- [ ] Support for more entity types (hotels, attractions, services)
- [ ] Multi-language review summaries
- [ ] Image analysis from review photos
- [ ] Dietary restriction filtering (vegan, gluten-free, etc.)
- [ ] Price range filtering
- [ ] Reservation integration
- [ ] More OAuth providers (Apple, Facebook)
- [ ] Mobile app integration
- [ ] Real-time review updates

## ⭐ Star History

If you find this useful, please star the repository!

---

**Made with ❤️ by the Agentic Review Team**

[Website](https://agenticreview.com) • [GitHub](https://github.com/agenticreview) • [Twitter](https://twitter.com/agenticreview)
