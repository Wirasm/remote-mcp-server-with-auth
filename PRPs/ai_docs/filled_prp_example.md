# Web Search Tool Implementation PRP

## Overview

Implement a web search tool for the Task Executor agent using OpenAI's hosted web search capability. This will enable the agent to search the internet for real-time information about real estate markets, neighborhoods, schools, and local amenities.

## Critical Context

### Documentation References

- **OpenAI Agents SDK - Hosted Tools**: https://sdk.openai.com/agents/tools#hosted-tools | PRPs/ai_docs/agents_sdk_tools.md
  - Web search is a hosted tool that runs on OpenAI servers
  - Requires `webSearchTool()` import from `@openai/agents`
  - No custom execution logic needed - handled by OpenAI

- **SDK Version**: `@openai/agents@^0.0.2` (current in package.json)
  ```typescript
  import { webSearchTool } from "@openai/agents";
  const searchTool = webSearchTool({
    name: "web_search_preview", // optional custom name
    searchContextSize: "medium", // 'low' | 'medium' | 'high'
  });
  ```

### Existing Patterns to Follow

#### Tool Structure (from `src/lib/agents/tools/lead-tool.ts`)

```typescript
import { tool } from "@openai/agents";
import { z } from "zod";
import { logger } from "@/lib/logger";

export const toolName = tool({
  name: "tool_name",
  description: "Clear description",
  parameters: z.object({
    /* schema */
  }),
  execute: async (params, runContext) => {
    const toolLogger = logger.child("tool_name");
    // Implementation
  },
});
```

#### Tool Integration (from `src/lib/agents/task-executor-agent.ts`)

```typescript
const agent = new Agent({
  name: "TaskExecutor",
  instructions: TASK_EXECUTOR_INSTRUCTIONS,
  model: AGENT_CONFIG.TASK_EXECUTOR.MODEL,
  tools: [saveLeadTool], // Add new tools here
  modelSettings: {
    /* ... */
  },
});
```

### Key Differences: Hosted vs Function Tools

- **Hosted tools** (like web search) run on OpenAI servers
- **Function tools** (like save_lead) run locally with custom logic
- Hosted tools don't have an `execute` function - they're handled by the model

## Implementation Blueprint

### 1. Create Web Search Tool Wrapper

Since we want to add context and logging around the hosted tool, create a wrapper:

```typescript
// src/lib/agents/tools/web-search-tool.ts

import { webSearchTool } from "@openai/agents";
import { tool } from "@openai/agents";
import { z } from "zod";
import { logger } from "@/lib/logger";
import { SEARCH_CONFIG } from "@/lib/constants";
import type { ToolContext } from "./lead-tool";

/**
 * Web search tool for real estate market information
 * Uses OpenAI's hosted web search capability
 * @module lib/agents/tools/web-search-tool
 */

// Direct export of hosted tool for simple usage
export const hostedWebSearch = webSearchTool({
  name: "web_search",
  searchContextSize: SEARCH_CONFIG.CONTEXT_SIZE,
});

// Wrapper function tool for enhanced control and logging
export const searchMarketDataTool = tool({
  name: "search_market_data",
  description: "Search the web for real estate market information, neighborhood data, school ratings, and local amenities",
  parameters: z.object({
    query: z.string().describe("Search query for real estate information"),
    location: z.string().optional().describe("Specific location to focus the search"),
    searchType: z
      .enum(["market_trends", "neighborhood", "schools", "amenities", "general"])
      .optional()
      .describe("Type of real estate information to search for"),
  }),
  execute: async ({ query, location, searchType }, runContext) => {
    const toolLogger = logger.child("search_market_data");
    const conversationId = runContext?.context?.conversationId;

    try {
      // Enhance query with location context
      const enhancedQuery = location ? `${query} in ${location}` : query;

      toolLogger.info("Performing web search", {
        conversationId,
        searchType: searchType || "general",
        hasLocation: !!location,
      });

      // Note: In a real implementation, you might:
      // 1. Call an external search API
      // 2. Cache results
      // 3. Filter/process results
      // For now, return instruction to use hosted tool

      return `To search for "${enhancedQuery}", please use the web_search tool directly. This will provide current market information.`;
    } catch (error) {
      toolLogger.error("Web search failed", error);
      throw new Error("Failed to search for market information. Please try again.");
    }
  },
});
```

### 2. Update Task Executor Agent

```typescript
// src/lib/agents/task-executor-agent.ts (updated)

import { Agent } from "@openai/agents";
import { saveLeadTool } from "./tools/lead-tool";
import { hostedWebSearch } from "./tools/web-search-tool";
import { logger } from "@/lib/logger";
import { AGENT_CONFIG } from "@/lib/constants";

const TASK_EXECUTOR_INSTRUCTIONS = `You are a task execution agent that handles specific actions delegated by the main conversational agent.

ROLE:
- Execute specific tasks efficiently
- Return structured results
- Handle errors gracefully
- Never engage in conversation

AVAILABLE ACTIONS:
- save_lead: Save or update lead information in the database
- web_search: Search the internet for real estate market information, neighborhoods, schools, and amenities

GUIDELINES:
- Focus only on the task at hand
- Return concise status updates
- If a task fails, return a clear error message
- Do not ask questions or engage in dialogue
- For web searches, provide relevant and current information
- Summarize search results concisely`;

export function createTaskExecutorAgent(): any {
  const agentLogger = logger.child("task-executor-agent");

  try {
    const agent = new Agent({
      name: "TaskExecutor",
      instructions: TASK_EXECUTOR_INSTRUCTIONS,
      model: AGENT_CONFIG.TASK_EXECUTOR.MODEL,
      tools: [
        saveLeadTool,
        hostedWebSearch, // Add hosted web search tool
      ],
      modelSettings: {
        temperature: AGENT_CONFIG.TASK_EXECUTOR.TEMPERATURE,
        maxTokens: AGENT_CONFIG.TASK_EXECUTOR.MAX_TOKENS,
      },
    });

    agentLogger.info("Task executor agent created successfully with web search");
    return agent;
  } catch (error) {
    agentLogger.error("Failed to create task executor agent", error);
    throw new Error("Failed to initialize task executor agent");
  }
}
```

### 3. Add Configuration Constants

```typescript
// src/lib/constants.ts (additions)

/**
 * Search configuration
 * @constant
 */
export const SEARCH_CONFIG = {
  /** Search context size for web search tool */
  CONTEXT_SIZE: "medium" as const,
  /** Maximum search results to consider */
  MAX_RESULTS: 5,
  /** Cache TTL in seconds */
  CACHE_TTL: 300,
  /** Search types */
  SEARCH_TYPES: {
    MARKET_TRENDS: "market_trends",
    NEIGHBORHOOD: "neighborhood",
    SCHOOLS: "schools",
    AMENITIES: "amenities",
    GENERAL: "general",
  } as const,
} as const;
```

## Database Persistence

### Overview

Following the established pattern from `lead-tool.ts`, we'll persist web search results to enable:

- Search history for leads
- Cached results to reduce API calls
- UI display of market insights
- Analytics on search patterns

### Database Schema

```sql
-- Add to migration: YYYYMMDDHHMMSS_add_property_search_tables.sql

-- Property searches table for web and Zillow searches
CREATE TABLE property_searches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id TEXT NOT NULL,
  search_type TEXT NOT NULL CHECK (search_type IN ('web_search', 'zillow')),
  query TEXT NOT NULL,
  search_params JSONB DEFAULT '{}', -- location, searchType, etc.
  results JSONB NOT NULL,
  result_summary TEXT, -- AI-generated summary for UI display
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,

  -- Foreign key to leads table
  CONSTRAINT fk_property_searches_conversation
    FOREIGN KEY (conversation_id)
    REFERENCES leads(conversation_id)
    ON DELETE CASCADE
);

-- Indexes for performance
CREATE INDEX idx_property_searches_conversation ON property_searches(conversation_id);
CREATE INDEX idx_property_searches_type ON property_searches(search_type);
CREATE INDEX idx_property_searches_created ON property_searches(created_at DESC);

-- Updated timestamp trigger
CREATE TRIGGER update_property_searches_updated_at
  BEFORE UPDATE ON property_searches
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- Row Level Security
ALTER TABLE property_searches ENABLE ROW LEVEL SECURITY;

-- Anonymous users can only insert (via agents)
CREATE POLICY "property_searches_anon_insert" ON property_searches
  FOR INSERT TO anon
  WITH CHECK (true);

-- Authenticated users have full access to their searches
CREATE POLICY "property_searches_auth_all" ON property_searches
  FOR ALL TO authenticated
  USING (
    conversation_id IN (
      SELECT conversation_id FROM leads
      WHERE user_id = (SELECT auth.uid())
    )
  );
```

### Updated Tool Implementation with Persistence

```typescript
// src/lib/agents/tools/web-search-tool.ts (updated execute function)

export const searchMarketDataTool = tool({
  name: "search_market_data",
  description: "Search the web for real estate market information, neighborhood data, school ratings, and local amenities",
  parameters: z.object({
    query: z.string().describe("Search query for real estate information"),
    location: z.string().optional().describe("Specific location to focus the search"),
    searchType: z
      .enum(["market_trends", "neighborhood", "schools", "amenities", "general"])
      .optional()
      .describe("Type of real estate information to search for"),
  }),
  execute: async ({ query, location, searchType }, runContext) => {
    const toolLogger = logger.child("search_market_data");
    const conversationId = runContext?.context?.conversationId;

    try {
      if (!conversationId) {
        throw new Error("No conversation ID in context");
      }

      // Enhance query with location context
      const enhancedQuery = location ? `${query} in ${location}` : query;

      toolLogger.info("Performing web search", {
        conversationId,
        searchType: searchType || "general",
        hasLocation: !!location,
      });

      // Get Supabase client
      const supabase = await createClient();

      // Check for recent cached results (last 5 minutes)
      const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000).toISOString();
      const { data: cachedSearch } = await supabase
        .from("property_searches")
        .select("results, result_summary")
        .eq("conversation_id", conversationId)
        .eq("query", enhancedQuery)
        .eq("search_type", "web_search")
        .gte("created_at", fiveMinutesAgo)
        .order("created_at", { ascending: false })
        .limit(1)
        .single();

      if (cachedSearch) {
        toolLogger.info("Returning cached search results", { conversationId });
        return cachedSearch.result_summary || "Cached search results found";
      }

      // Note: Actual web search would happen here via hosted tool
      // For now, return instruction to use hosted tool
      const searchInstruction = `To search for "${enhancedQuery}", please use the web_search tool directly. This will provide current market information.`;

      // In production, after getting actual results:
      // 1. Parse and summarize results
      // 2. Store in database
      const searchParams = {
        location,
        searchType: searchType || "general",
      };

      // Example of how to store results (when integrated with actual search)
      /*
      const { error: dbError } = await supabase
        .from("property_searches")
        .insert({
          conversation_id: conversationId,
          search_type: "web_search",
          query: enhancedQuery,
          search_params: searchParams,
          results: actualSearchResults, // From web search API
          result_summary: summarizedResults, // AI-generated summary
          updated_at: new Date().toISOString()
        })

      if (dbError) {
        toolLogger.error("Failed to save search results", dbError)
        // Don't throw - still return results even if save fails
      }
      */

      return searchInstruction;
    } catch (error) {
      toolLogger.error("Web search failed", error);

      // Re-throw user-friendly errors
      if (error instanceof Error && error.message === "No conversation ID in context") {
        throw new Error("Unable to perform search without conversation context");
      }

      throw new Error("Failed to search for market information. Please try again.");
    }
  },
});
```

### Database Type Generation

After creating the migration, regenerate types:

```bash
supabase gen types typescript --local > src/lib/database.types.ts
```

### Persistence Patterns Following Lead Tool

1. **Upsert vs Insert**: Use INSERT for searches (each search is unique)
2. **Empty Field Handling**: Filter out null/empty params before saving
3. **Timestamps**: Always include updated_at
4. **Error Handling**: Log but don't fail if save fails (return results anyway)
5. **Validation**: Validate search params before executing
6. **Logging**: Use child logger with operation context

### Testing Database Operations

```typescript
// src/lib/agents/tools/__tests__/web-search-tool.test.ts (additions)

describe("searchMarketDataTool persistence", () => {
  const mockSupabaseClient = {
    from: jest.fn().mockReturnValue({
      select: jest.fn().mockReturnValue({
        eq: jest.fn().mockReturnValue({
          eq: jest.fn().mockReturnValue({
            eq: jest.fn().mockReturnValue({
              gte: jest.fn().mockReturnValue({
                order: jest.fn().mockReturnValue({
                  limit: jest.fn().mockReturnValue({
                    single: jest.fn().mockResolvedValue({ data: null })
                  })
                })
              })
            })
          })
        })
      }),
      insert: jest.fn().mockResolvedValue({ error: null })
    })
  }

  beforeEach(() => {
    (createClient as jest.Mock).mockResolvedValue(mockSupabaseClient)
  })

  it("should check cache before searching", async () => {
    const cachedResult = {
      results: { /* cached data */ },
      result_summary: "Cached market data for Austin"
    }

    mockSupabaseClient.from().select()...single
      .mockResolvedValueOnce({ data: cachedResult })

    const result = await searchMarketDataTool.execute(
      { query: "home prices", location: "Austin, TX" },
      { context: { conversationId: "test-123" } }
    )

    expect(result).toBe("Cached market data for Austin")
    expect(mockSupabaseClient.from).toHaveBeenCalledWith("property_searches")
  })

  it("should handle missing conversation ID", async () => {
    await expect(
      searchMarketDataTool.execute(
        { query: "test" },
        { context: {} }
      )
    ).rejects.toThrow("Unable to perform search without conversation context")
  })
})
```

### 4. Create Tests

```typescript
// src/lib/agents/tools/__tests__/web-search-tool.test.ts

import { hostedWebSearch, searchMarketDataTool } from "../web-search-tool";
import { webSearchTool } from "@openai/agents";
import { SEARCH_CONFIG } from "@/lib/constants";
import { logger } from "@/lib/logger";

jest.mock("@openai/agents", () => ({
  webSearchTool: jest.fn().mockReturnValue({ name: "web_search", type: "hosted" }),
  tool: jest.fn((config) => config),
}));

jest.mock("@/lib/logger", () => ({
  logger: {
    child: jest.fn().mockReturnValue({
      info: jest.fn(),
      error: jest.fn(),
    }),
  },
}));

describe("Web Search Tool", () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe("hostedWebSearch", () => {
    it("should create hosted web search tool with correct configuration", () => {
      expect(webSearchTool).toHaveBeenCalledWith({
        name: "web_search",
        searchContextSize: SEARCH_CONFIG.CONTEXT_SIZE,
      });
      expect(hostedWebSearch).toEqual({ name: "web_search", type: "hosted" });
    });
  });

  describe("searchMarketDataTool", () => {
    it("should have correct configuration", () => {
      expect(searchMarketDataTool).toMatchObject({
        name: "search_market_data",
        description: expect.stringContaining("real estate market information"),
      });
    });

    it("should validate parameters with Zod schema", () => {
      const params = searchMarketDataTool.parameters;

      // Valid input
      expect(() =>
        params.parse({
          query: "average home prices",
          location: "Austin, TX",
          searchType: "market_trends",
        }),
      ).not.toThrow();

      // Invalid search type
      expect(() =>
        params.parse({
          query: "test",
          searchType: "invalid",
        }),
      ).toThrow();
    });

    it("should enhance query with location when provided", async () => {
      const mockContext = { context: { conversationId: "test-123" } };
      const mockLogger = logger.child("search_market_data");

      const result = await searchMarketDataTool.execute({ query: "home prices", location: "Austin, TX" }, mockContext);

      expect(mockLogger.info).toHaveBeenCalledWith(
        "Performing web search",
        expect.objectContaining({
          conversationId: "test-123",
          hasLocation: true,
        }),
      );
      expect(result).toContain("home prices in Austin, TX");
    });

    it("should handle errors gracefully", async () => {
      const mockLogger = logger.child("search_market_data");
      const mockError = new Error("Search API error");

      // Mock the execute function to throw
      const execute = searchMarketDataTool.execute;
      searchMarketDataTool.execute = jest.fn().mockRejectedValue(mockError);

      await expect(searchMarketDataTool.execute({ query: "test" }, null)).rejects.toThrow("Failed to search");

      searchMarketDataTool.execute = execute; // Restore
    });
  });
});
```

```typescript
// src/lib/agents/__tests__/task-executor-agent.test.ts (update)

// Add test for web search tool integration
it("should include web search tool", () => {
  createTaskExecutorAgent();

  const callArgs = (Agent as jest.Mock).mock.calls[0][0];
  expect(callArgs.tools).toHaveLength(2);
  expect(callArgs.tools[1]).toEqual(hostedWebSearch);
});

it("should have instructions mentioning web search", () => {
  createTaskExecutorAgent();

  const callArgs = (Agent as jest.Mock).mock.calls[0][0];
  expect(callArgs.instructions).toContain("web_search: Search the internet");
  expect(callArgs.instructions).toContain("real estate market information");
});
```

## Validation Gates

```bash
# 1. Type checking
npx tsc --noEmit

# 2. Linting
npm run lint

# 3. Unit tests
npm test -- web-search-tool
npm test -- task-executor-agent

# 4. All tests
npm test

# 5. Check database implementation
Use tools to check database migrations
you have mcp tools directly connected tot he db

# 6. Integration tests
npm test -- src/lib/agents/__tests__/

# 7. Build validation
npm run build
```

## Error Patterns & Solutions

### Common Errors

1. **Missing Import Error**

   ```
   Cannot find module '@openai/agents' or its corresponding type declarations
   ```

   **Solution**: Ensure `@openai/agents` is installed and version ^0.0.2

2. **Hosted Tool Not Available**

   ```
   webSearchTool is not a function
   ```

   **Solution**: Check SDK version, hosted tools require v0.0.2+

3. **Search Context Size Invalid**

   ```
   Invalid value for searchContextSize
   ```

   **Solution**: Use only 'low', 'medium', or 'high' values

4. **Rate Limiting**
   ```
   Too many search requests
   ```
   **Solution**: Implement caching and rate limiting in wrapper function

## Progressive Enhancement Path

1. **1**: Basic hosted web search integration
2. **2**: Add result caching to reduce API calls
3. **3**: Implement search result filtering for real estate relevance
4. **4**: Add specialized search functions (MLS integration, Zillow API)
5. **5**: Create search result summarization for better UX

## Usage Examples

### From Sarah (Main Agent)

```
User: "What's the current real estate market like in Austin?"
Sarah: "I'll search for current market information in Austin for you."
[delegates to Task Executor with web_search]
Task Executor: [searches web, returns market data]
Sarah: "Based on current data, the Austin real estate market shows..."
```

### Search Types

- **Market Trends**: "average home prices Austin TX 2024"
- **Neighborhood**: "best neighborhoods families Austin TX"
- **Schools**: "top rated schools Cedar Park TX"
- **Amenities**: "shopping restaurants downtown Austin"

## Security Considerations

1. **No Sensitive Data in Searches**: Ensure PII is not included in search queries
2. **Result Validation**: Consider validating/sanitizing search results
3. **Rate Limiting**: Implement to prevent abuse
4. **Logging**: Log searches for audit trail but exclude sensitive data

## UI Integration Considerations

### API Endpoints Needed

```typescript
// app/api/leads/[id]/searches/route.ts
GET /api/leads/[id]/searches - Get search history for a lead

// app/api/properties/insights/route.ts
GET /api/properties/insights?location={location} - Get market insights for location
```

### UI Components to Build

1. **Search History Component**:
   - Display recent searches with timestamps
   - Show search type badges (market trends, schools, etc.)
   - Click to view full results

2. **Market Insights Card**:
   - Summary of market data from searches
   - Key metrics highlighted
   - Link to detailed view

3. **Lead Activity Timeline**:
   - Chronological view of searches
   - Integrated with property views from Zillow
   - Visual indicators for search types

### Data Aggregation Queries

```typescript
// Get market insights summary for a lead's area of interest
const { data: marketInsights } = await supabase
  .from("property_searches")
  .select("result_summary, search_params")
  .eq("conversation_id", conversationId)
  .eq("search_type", "web_search")
  .in("search_params->searchType", ["market_trends", "neighborhood"])
  .order("created_at", { ascending: false })
  .limit(5);

// Get lead's search patterns
const { data: searchPatterns } = await supabase.rpc("analyze_search_patterns", {
  p_conversation_id: conversationId,
});
```

## Success Criteria

- [ ] Web search tool is available in task executor
- [ ] Hosted tool properly configured with context size
- [ ] Agent can perform web searches when requested
- [ ] Search results are persisted to database
- [ ] Cache reduces duplicate API calls
- [ ] Search history is available for UI display
- [ ] All tests pass including persistence tests
- [ ] Type checking passes
- [ ] Build succeeds
- [ ] Search results are relevant to real estate queries
