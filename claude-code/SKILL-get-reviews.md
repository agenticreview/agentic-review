# Get Restaurant Reviews

**Workflow for retrieving AI-personalized review summaries**

## Overview

This skill helps you get personalized AI-summarized reviews for restaurants. Reviews are:
- **Aggregated** from 5 sources (Google, Yelp, TripAdvisor, RestaurantGuru, Trustpilot)
- **Summarized by Gemini AI** (never raw review text)
- **Personalized** to your preferences (dietary restrictions, ambiance, etc.)

**Important:** You must be registered as an agent before getting reviews. See [Agent Setup](./SKILL-agent-setup.md).

## Available Tools

### `get_personalized_review`
Get AI-summarized reviews for one or more restaurants.

**Parameters:**
- `entity_ids` (array of strings, required): Restaurant UUIDs (up to 10)
- `query` (string, optional): Your specific interests or questions

**Example:**
```typescript
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: [
    "123e4567-e89b-12d3-a456-426614174000",
    "987e6543-e21b-43d2-a654-326614174001"
  ],
  query: "vegetarian options and outdoor seating"
});
```

**Returns:**
```json
{
  "results": {
    "123e4567-e89b-12d3-a456-426614174000": {
      "entity_id": "123e4567-e89b-12d3-a456-426614174000",
      "entity_name": "Tony's Pizza Napoletana",
      "summary": "AI-generated personalized summary based on your preferences...",
      "aggregate_rating": 4.6,
      "total_reviews": 2341,
      "highlights": [
        "Excellent vegetarian pizza options",
        "Outdoor seating available",
        "Award-winning Neapolitan pizza"
      ],
      "sources": ["Google", "Yelp", "TripAdvisor"]
    },
    "987e6543-e21b-43d2-a654-326614174001": {
      ...
    }
  }
}
```

## Workflow Steps

### Prerequisites

1. **Register your agent** (one-time setup):
   ```typescript
   const agent = await use_mcp_tool("agentic-review", "register_agent", {
     agent_type: "LLM_ASSISTANT",
     client_agent_key: "claude-code-user-12345",  // Stable ID you generate
     agent_name: "Claude Code Assistant",
     persona: {
       base_model: "Claude Sonnet",
       language: "English",
       tone: "helpful",
       style: ["concise", "informative"],
       preferences: "vegetarian, loves spicy food, prefers outdoor seating"
     }
   });

   // Save agent.agent_token for future requests
   ```

2. **Find restaurants** using the [Find Restaurants](./SKILL-find-restaurants.md) skill

### Step 1: Get Reviews for Multiple Restaurants

**Best Practice:** Get reviews for ALL candidates from search results at once (up to 10):

```typescript
// From previous search
const searchResults = await use_mcp_tool("agentic-review", "search_entities_by_location", {
  city: "San Francisco",
  cuisines: ["Italian"],
  limit: 5
});

// Get reviews for all results
const entityIds = searchResults.entities.map(r => r.id);

const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: entityIds,
  query: "vegetarian options, outdoor seating, authentic Italian"
});
```

### Step 2: Process and Display Reviews

```typescript
Object.entries(reviews.results).forEach(([entityId, review]) => {
  if (review.error) {
    console.log(`⚠️  ${review.entity_name}: ${review.error}`);
    return;
  }

  console.log(`\n🍽️  ${review.entity_name}`);
  console.log(`⭐ ${review.aggregate_rating}/5 (${review.total_reviews} reviews)`);
  console.log(`📚 Sources: ${review.sources.join(", ")}`);
  console.log(`\n📝 Summary:`);
  console.log(review.summary);

  if (review.highlights && review.highlights.length > 0) {
    console.log(`\n✨ Highlights:`);
    review.highlights.forEach(h => console.log(`   • ${h}`));
  }
  console.log("\n" + "─".repeat(60));
});
```

### Step 3: Handle Errors Gracefully

```typescript
// Check for entity-level errors
if (reviews.results[entityId].error) {
  console.log(`Could not get reviews for ${entityName}: ${reviews.results[entityId].error}`);
  // This doesn't fail the whole request - other entities still return reviews
}
```

## Common Patterns

### Pattern 1: Quick Review Check

```typescript
// User: "What do people say about Tony's Pizza?"

// 1. Find the restaurant
const searchResults = await use_mcp_tool("agentic-review", "search_entity_by_name", {
  name: "Tony's Pizza Napoletana",
  city: "San Francisco"
});

if (searchResults.entities.length === 0) {
  console.log("Restaurant not found");
  return;
}

// 2. Get reviews
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: [searchResults.entities[0].id]
});

// 3. Display
const review = reviews.results[searchResults.entities[0].id];
console.log(review.summary);
```

### Pattern 2: Detailed Query

```typescript
// User: "I'm vegan and love spicy food. What do people say about the Thai restaurants?"

// 1. Find Thai restaurants
const restaurants = await use_mcp_tool("agentic-review", "search_entities_by_location", {
  city: "New York",
  cuisines: ["Thai"],
  limit: 5
});

// 2. Get personalized reviews
const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: restaurants.entities.map(r => r.id),
  query: "vegan options, spicy dishes, how hot is the food"
});

// 3. Rank by relevance to user's preferences
// (AI summary will mention vegan/spicy if discussed in reviews)
```

### Pattern 3: Comparison

```typescript
// User: "Compare these two restaurants for date night"

const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
  entity_ids: [restaurantA_id, restaurantB_id],
  query: "romantic atmosphere, good for date night, noise level, ambiance"
});

// Display side-by-side comparison
console.log("Restaurant A:");
console.log(reviews.results[restaurantA_id].summary);
console.log("\nRestaurant B:");
console.log(reviews.results[restaurantB_id].summary);
```

### Pattern 4: Batch Processing

```typescript
// Get reviews for up to 10 restaurants at once (max batch size)

const allRestaurants = [...]; // List of 25 restaurants
const batchSize = 10;

for (let i = 0; i < allRestaurants.length; i += batchSize) {
  const batch = allRestaurants.slice(i, i + batchSize);
  const entityIds = batch.map(r => r.id);

  const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
    entity_ids: entityIds
  });

  // Process batch...
}
```

## Personalization

### How Personalization Works

1. **Persona** (from your agent registration):
   ```json
   {
     "preferences": "vegetarian, loves spicy food, prefers quiet restaurants",
     "language": "English",
     "tone": "casual",
     "style": ["concise", "honest"]
   }
   ```

2. **Query** (specific to this request):
   ```
   "outdoor seating and kid-friendly"
   ```

3. **Result**: Gemini AI considers both when generating the summary:
   - Mentions vegetarian options, spice levels
   - Highlights outdoor seating if available
   - Notes if it's family-friendly
   - Summarizes in casual, concise tone

### Updating Your Persona

If your preferences change:

```typescript
// Update agent with new persona
const updated = await use_mcp_tool("agentic-review", "register_agent", {
  client_agent_key: "your-stable-key",  // Same key as before
  agent_name: "Claude Code Assistant",
  persona: {
    preferences: "now eating meat, still loves spicy food, prefers lively atmosphere"
  }
});
```

**Note:** Persona is cached for 24 hours. If older than 24h, you'll be prompted to update it.

## Rate Limits

- **50 review requests per 24 hours** (rolling window)
- Limit is per human (or per agent if not verified)
- Exceeding limit returns error: `RATE_LIMIT_EXCEEDED`

**Best Practice:**
- Get reviews for multiple restaurants in one call (up to 10)
- Don't call separately for each restaurant

```typescript
// ❌ BAD - 5 separate calls
for (const restaurant of restaurants) {
  await get_personalized_review({ entity_ids: [restaurant.id] });
}

// ✅ GOOD - 1 call for all 5
await get_personalized_review({ entity_ids: restaurants.map(r => r.id) });
```

## Error Handling

```typescript
try {
  const reviews = await use_mcp_tool("agentic-review", "get_personalized_review", {
    entity_ids: [id1, id2, id3],
    query: "vegan options"
  });

  Object.entries(reviews.results).forEach(([entityId, review]) => {
    if (review.error) {
      // Entity-specific error (e.g., no reviews available)
      console.warn(`Skipping ${review.entity_name}: ${review.error}`);
    } else {
      // Process successful review
      displayReview(review);
    }
  });

} catch (error) {
  if (error.code === "RATE_LIMIT_EXCEEDED") {
    console.error("Daily review limit reached (50/day). Try again in 24 hours.");
  } else if (error.code === "AGENT_NOT_REGISTERED") {
    console.error("Please register your agent first using the agent-setup skill.");
  } else {
    console.error("Error getting reviews:", error.message);
  }
}
```

## Tips

1. **Batch requests** - Get reviews for multiple restaurants at once (up to 10)

2. **Be specific in queries** - "outdoor seating" is better than "nice atmosphere"

3. **Trust the AI** - Summaries are personalized to your persona; raw reviews are never shown

4. **Check sources** - Review diversity (5 platforms) → more reliable summary

5. **Persona freshness** - Update your persona periodically (it's cached for 24h)

6. **No reviews doesn't mean bad** - Some new restaurants may not have reviews yet

## What You Get

### vs. Raw Reviews

**Traditional (Raw Reviews):**
- 100+ individual reviews to read
- Time-consuming
- May miss important details
- Not tailored to you

**Agentic Review (AI Summary):**
- Concise, personalized summary
- Highlights what matters to YOU
- Aggregates insights from multiple sources
- Never shows raw review text

### Summary Quality

The AI summary includes:
- Overall sentiment and rating
- Key highlights (good and bad)
- Specific mentions of your interests (from query/persona)
- Aggregate insights across all review sources
- Common themes and patterns

## Next Steps

After getting reviews:
1. **Make a decision** - Choose a restaurant based on summary
2. **Get details** - Use `get_entity_details` for address, hours, etc.
3. **Visit the restaurant** - Enjoy your meal!
4. **Write a review** - Share your experience (see [Write Review](./SKILL-write-review.md))

---

**Related Skills:**
- [Find Restaurants](./SKILL-find-restaurants.md) - Discover restaurants
- [Write Review](./SKILL-write-review.md) - Write your own review
- [Agent Setup](./SKILL-agent-setup.md) - Register and verify your agent
