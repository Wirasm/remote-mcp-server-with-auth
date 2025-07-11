---
name: "PRP Parser MCP Server"
description: A Model Context Protocol server for parsing Product Requirement Prompts, extracting tasks, managing documentation, and providing CRUD operations with PostgreSQL database integration and Anthropic AI-powered parsing.
---

## Purpose

Build a production-ready MCP server that parses Product Requirement Prompts (PRPs) using Anthropic's Claude API, extracts structured information (tasks, documentation, tags), and provides comprehensive CRUD operations for managing PRPs and their components in a PostgreSQL database.

## Core Principles

1. **Context is King**: Include ALL necessary MCP patterns, PRP parsing logic, and database schemas
2. **Validation Loops**: Provide executable tests from TypeScript compilation to production deployment
3. **Security First**: Build-in authentication, authorization, and SQL injection protection
4. **Production Ready**: Include monitoring, error handling, and deployment automation

---

## Goal

Build a production-ready MCP (Model Context Protocol) server with:

- **PRP Parsing**: AI-powered extraction of tasks, goals, context, and documentation from PRP markdown files
- **Task Management**: CRUD operations for tasks extracted from PRPs
- **Documentation Management**: Store and retrieve PRP documentation (goals, why, what, target users)
- **Tag System**: Organize PRPs and tasks with tags for better searchability
- GitHub OAuth authentication with role-based access control
- PostgreSQL database integration with security validation
- Cloudflare Workers deployment with monitoring
- **One Task Per File Pattern**: Separate concerns by implementing each major function in its own file

## Why

- **Developer Productivity**: Enable AI assistants to understand and work with PRPs systematically
- **Task Organization**: Convert unstructured PRPs into actionable, trackable tasks
- **Knowledge Management**: Preserve context and documentation from PRPs for future reference
- **Enterprise Security**: GitHub OAuth with granular permission system ensures secure access
- **Scalability**: Cloudflare Workers global edge deployment for low-latency access
- **Integration**: Seamlessly integrate PRP management into existing development workflows

## What

### MCP Server Features

**Core MCP Tools:**

- `parsePRP` - Parse a PRP markdown file and extract structured information using Claude
- `createTask` - Create a new task with associated metadata
- `updateTask` - Update an existing task's details
- `deleteTask` - Delete a task (soft delete with audit trail)
- `listTasks` - List all tasks with filtering and pagination
- `getTask` - Retrieve a specific task with full details
- `addTaskInfo` - Add additional information to a task
- `createDocumentation` - Store PRP documentation (goals, why, what)
- `updateDocumentation` - Modify existing documentation
- `getDocumentation` - Retrieve documentation by PRP or category
- `createTag` - Create a new tag for organization
- `tagItem` - Apply tags to tasks or PRPs
- `searchByTag` - Find items by tag

**Authentication & Authorization:**

- GitHub OAuth 2.0 integration with signed cookie approval system
- Role-based access control (read-only vs privileged users)
- User context propagation to all MCP tools
- Secure session management with HMAC-signed cookies

**Database Integration:**

- PostgreSQL connection pooling with automatic cleanup
- SQL injection protection and query validation
- Read/write operation separation based on user permissions
- Error sanitization to prevent information leakage
- Comprehensive audit logging for all operations

**Deployment & Monitoring:**

- Cloudflare Workers with Durable Objects for state management
- Optional Sentry integration for error tracking and performance monitoring
- Environment-based configuration (development vs production)
- Real-time logging and alerting

### Success Criteria

- [ ] MCP server passes validation with MCP Inspector
- [ ] GitHub OAuth flow works end-to-end (authorization → callback → MCP access)
- [ ] PRP parsing correctly extracts tasks, documentation, and metadata
- [ ] All CRUD operations work with proper permission validation
- [ ] Database schema supports full PRP data model
- [ ] TypeScript compilation succeeds with no errors
- [ ] Local development server starts and responds correctly
- [ ] Production deployment to Cloudflare Workers succeeds
- [ ] Authentication prevents unauthorized access to write operations
- [ ] Error handling provides user-friendly messages without leaking system details
- [ ] One task per file pattern is properly implemented for separation of concerns

## All Needed Context

### Documentation & References (MUST READ)

```yaml
# CRITICAL MCP PATTERNS - Read these first
- docfile: PRPs/ai_docs/mcp_patterns.md
  why: Core MCP development patterns, security practices, and error handling

# PRP STRUCTURE AND EXAMPLES - Essential for parsing logic
- file: PRPs/ai_docs/prp_example_template.md
  why: Template structure that PRPs follow - defines what to extract

- file: PRPs/ai_docs/filled_prp_example.md
  why: Example of a filled PRP - shows actual content to parse

# EXISTING CODEBASE PATTERNS - Study these implementations
- file: src/index.ts
  why: Complete MCP server with authentication, database, and tools - MIRROR this pattern

- file: src/github-handler.ts
  why: OAuth flow implementation - USE this exact pattern for authentication

- file: src/database.ts
  why: Database security, connection pooling, SQL validation - FOLLOW these patterns

- file: src/simple-math.ts
  why: Basic MCP server without auth - good starting point for tool registration

- file: wrangler.jsonc
  why: Cloudflare Workers configuration - COPY this pattern for deployment

# OFFICIAL MCP DOCUMENTATION
- url: https://modelcontextprotocol.io/docs/concepts/tools
  why: MCP tool registration and schema definition patterns

- url: https://modelcontextprotocol.io/docs/concepts/resources
  why: MCP resource implementation if needed

# ANTHROPIC DOCUMENTATION
- url: https://docs.anthropic.com/claude/docs/intro-to-claude
  why: Claude API usage for PRP parsing

- url: https://github.com/anthropics/anthropic-sdk-typescript
  why: TypeScript SDK for Claude integration
```

### Current Codebase Tree (Run `tree -I node_modules` in project root)

```bash
/
├── src/
│   ├── index.ts                 # Main authenticated MCP server ← STUDY THIS
│   ├── index_sentry.ts         # Sentry monitoring version
│   ├── simple-math.ts          # Basic MCP example ← GOOD STARTING POINT
│   ├── github-handler.ts       # OAuth implementation ← USE THIS PATTERN
│   ├── database.ts             # Database utilities ← SECURITY PATTERNS
│   ├── utils.ts                # OAuth helpers
│   └── workers-oauth-utils.ts  # Cookie security system
├── PRPs/
│   ├── templates/prp_mcp_base.md  # This template
│   └── ai_docs/                   # Implementation guides ← READ ALL
├── wrangler.jsonc              # Cloudflare config ← COPY PATTERNS
├── package.json                # Dependencies
└── tsconfig.json               # TypeScript config
```

### Desired Codebase Tree (Files to add/modify)

```bash
/
├── src/
│   ├── prp-parser/             # NEW: PRP Parser MCP implementation
│   │   ├── index.ts            # Main PRP parser MCP server
│   │   ├── tools/              # One file per tool (separation of concerns)
│   │   │   ├── parse-prp.ts    # PRP parsing with Claude
│   │   │   ├── task-crud.ts    # Task CRUD operations
│   │   │   ├── documentation-crud.ts # Documentation management
│   │   │   ├── tag-management.ts # Tag system operations
│   │   │   └── search-tools.ts # Search and query tools
│   │   ├── database/           # Database layer
│   │   │   ├── schema.ts       # Database schema definitions
│   │   │   ├── migrations.sql  # SQL migrations for PRP tables
│   │   │   └── queries.ts      # Prepared database queries
│   │   ├── parsers/            # PRP parsing logic
│   │   │   ├── prp-parser.ts   # Main PRP parsing logic
│   │   │   ├── task-extractor.ts # Extract tasks from PRPs
│   │   │   └── validation.ts   # PRP structure validation
│   │   └── types.ts            # TypeScript interfaces and types
├── wrangler-prp-parser.jsonc   # NEW: Cloudflare config for PRP parser
└── .dev.vars                   # Updated with ANTHROPIC_API_KEY
```

### Known Gotchas & Critical Patterns

```typescript
// CRITICAL: Cloudflare Workers require specific patterns
// 1. ALWAYS implement cleanup for Durable Objects
export class PRPParserMCP extends McpAgent<Env, Record<string, never>, Props> {
  async cleanup(): Promise<void> {
    await closeDb(); // CRITICAL: Close database connections
  }

  async alarm(): Promise<void> {
    await this.cleanup(); // CRITICAL: Handle Durable Object alarms
  }
}

// 2. ALWAYS validate SQL to prevent injection (use existing patterns)
const validation = validateSqlQuery(sql); // from src/database.ts
if (!validation.isValid) {
  return createErrorResponse(validation.error);
}

// 3. ALWAYS check permissions before write operations
const ALLOWED_USERNAMES = new Set(["coleam00", "admin2"]);
if (!ALLOWED_USERNAMES.has(this.props.login)) {
  return createErrorResponse("Insufficient permissions");
}

// 4. ALWAYS use withDatabase wrapper for connection management
return await withDatabase(this.env.DATABASE_URL, async (db) => {
  // Database operations here
});

// 5. ALWAYS use Zod for input validation
import { z } from "zod";
const PRPParseSchema = z.object({
  prpContent: z.string().min(1).max(50000),
  extractTasks: z.boolean().default(true),
  extractDocs: z.boolean().default(true),
});

// 6. TypeScript compilation requires exact interface matching
interface Env {
  DATABASE_URL: string;
  GITHUB_CLIENT_ID: string;
  GITHUB_CLIENT_SECRET: string;
  OAUTH_KV: KVNamespace;
  ANTHROPIC_API_KEY: string; // NEW: For Claude API
}

// 7. IMPORTANT: One task per file pattern
// Each tool should be in its own file under src/prp-parser/tools/
// This keeps concerns separated and makes the codebase maintainable
```

## Implementation Blueprint

### Data Models & Types

Define TypeScript interfaces and Zod schemas for type safety and validation.

```typescript
// src/prp-parser/types.ts

// User authentication props (inherited from OAuth)
type Props = {
  login: string; // GitHub username
  name: string; // Display name
  email: string; // Email address
  accessToken: string; // GitHub access token
};

// PRP data model
interface PRP {
  id: string;
  name: string;
  description: string;
  goal: string;
  why: string[];
  what: string;
  successCriteria: string[];
  context: PRPContext;
  tasks: Task[];
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

interface PRPContext {
  documentation: DocumentationRef[];
  codebaseTree: string;
  desiredTree: string;
  knownGotchas: string;
}

interface DocumentationRef {
  type: 'url' | 'file' | 'docfile';
  path: string;
  why: string;
  section?: string;
  critical?: string;
}

interface Task {
  id: string;
  prpId: string;
  order: number;
  description: string;
  fileToModify?: string;
  pattern?: string;
  pseudocode?: string;
  status: 'pending' | 'in_progress' | 'completed';
  additionalInfo: Record<string, any>;
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
}

interface Tag {
  id: string;
  name: string;
  description?: string;
  color?: string;
  createdBy: string;
  createdAt: Date;
}

// Zod schemas for validation
const ParsePRPSchema = z.object({
  prpContent: z.string().min(1).max(50000).describe("Markdown content of the PRP"),
  extractTasks: z.boolean().default(true).describe("Whether to extract tasks"),
  extractDocs: z.boolean().default(true).describe("Whether to extract documentation"),
});

const CreateTaskSchema = z.object({
  prpId: z.string().uuid().describe("ID of the associated PRP"),
  description: z.string().min(1).max(1000).describe("Task description"),
  order: z.number().int().positive().describe("Task order"),
  fileToModify: z.string().optional().describe("File to modify for this task"),
  pattern: z.string().optional().describe("Code pattern to follow"),
  pseudocode: z.string().optional().describe("Pseudocode for implementation"),
  tags: z.array(z.string()).default([]).describe("Tags for the task"),
});

const UpdateTaskSchema = z.object({
  id: z.string().uuid().describe("Task ID"),
  description: z.string().optional(),
  status: z.enum(['pending', 'in_progress', 'completed']).optional(),
  additionalInfo: z.record(z.any()).optional(),
  tags: z.array(z.string()).optional(),
});

// Environment interface
interface Env {
  DATABASE_URL: string;
  GITHUB_CLIENT_ID: string;
  GITHUB_CLIENT_SECRET: string;
  OAUTH_KV: KVNamespace;
  ANTHROPIC_API_KEY: string;
}
```

### List of Tasks (Complete in order)

```yaml
Task 1 - Project Setup:
  COPY wrangler.jsonc to wrangler-prp-parser.jsonc:
    - MODIFY name field to "prp-parser-mcp"
    - ADD ANTHROPIC_API_KEY to vars section
    - KEEP existing OAuth and database configuration
    - UPDATE main field to "src/prp-parser/index.ts"

  UPDATE .dev.vars file:
    - ADD ANTHROPIC_API_KEY=your_api_key
    - KEEP existing OAuth and database variables

Task 2 - Database Schema:
  CREATE src/prp-parser/database/migrations.sql:
    - CREATE TABLE prps (id, name, description, goal, why, what, success_criteria, context, created_by, created_at, updated_at)
    - CREATE TABLE tasks (id, prp_id, order, description, file_to_modify, pattern, pseudocode, status, additional_info, created_at, updated_at)
    - CREATE TABLE tags (id, name, description, color, created_by, created_at)
    - CREATE TABLE task_tags (task_id, tag_id, created_at)
    - CREATE TABLE prp_tags (prp_id, tag_id, created_at)
    - ADD indexes for performance
    - ADD foreign key constraints
    - ENABLE Row Level Security

  CREATE src/prp-parser/database/schema.ts:
    - DEFINE TypeScript types matching database schema
    - EXPORT database type definitions

Task 3 - PRP Parser Implementation:
  CREATE src/prp-parser/parsers/prp-parser.ts:
    - IMPLEMENT Anthropic Claude integration
    - USE claude-3-sonnet for parsing PRPs
    - EXTRACT tasks, goals, documentation, success criteria
    - VALIDATE extracted data structure
    - HANDLE parsing errors gracefully

  CREATE src/prp-parser/parsers/task-extractor.ts:
    - PARSE task list from PRP content
    - EXTRACT task order, descriptions, file modifications
    - IDENTIFY pseudocode and patterns
    - MAINTAIN task relationships

  CREATE src/prp-parser/parsers/validation.ts:
    - VALIDATE PRP structure completeness
    - CHECK required sections exist
    - ENSURE task format is correct

Task 4 - Tool Implementations (One File Per Tool):
  CREATE src/prp-parser/tools/parse-prp.ts:
    - IMPLEMENT parsePRP tool using Anthropic API
    - PARSE markdown content into structured data
    - SAVE parsed data to database
    - RETURN parsing results

  CREATE src/prp-parser/tools/task-crud.ts:
    - IMPLEMENT createTask, updateTask, deleteTask, getTask, listTasks
    - VALIDATE permissions for write operations
    - USE database transactions for consistency
    - IMPLEMENT soft delete pattern

  CREATE src/prp-parser/tools/documentation-crud.ts:
    - IMPLEMENT createDocumentation, updateDocumentation, getDocumentation
    - STORE PRP documentation separately
    - ENABLE version tracking
    - SUPPORT markdown formatting

  CREATE src/prp-parser/tools/tag-management.ts:
    - IMPLEMENT createTag, tagItem, removeTag, searchByTag
    - VALIDATE tag uniqueness
    - SUPPORT tag hierarchies
    - ENABLE tag-based filtering

  CREATE src/prp-parser/tools/search-tools.ts:
    - IMPLEMENT searchPRPs, searchTasks
    - SUPPORT full-text search
    - ENABLE filtering by status, tags, dates
    - IMPLEMENT pagination

Task 5 - Main MCP Server:
  CREATE src/prp-parser/index.ts:
    - EXTEND McpAgent class following existing patterns
    - REGISTER all tools from tool files
    - IMPLEMENT permission-based tool availability
    - ADD comprehensive error handling
    - IMPLEMENT cleanup and alarm methods

Task 6 - Environment Configuration:
  SETUP Cloudflare KV namespace (if new):
    - RUN: wrangler kv namespace create "OAUTH_KV" --config wrangler-prp-parser.jsonc
    - UPDATE wrangler-prp-parser.jsonc with namespace ID

  SET production secrets:
    - RUN: wrangler secret put ANTHROPIC_API_KEY --config wrangler-prp-parser.jsonc
    - REUSE existing GitHub OAuth and database secrets

Task 7 - Testing Implementation:
  CREATE test PRPs in PRPs/test/:
    - ADD sample valid PRP
    - ADD sample invalid PRP
    - ADD complex PRP with multiple tasks

  TEST parsing functionality:
    - VERIFY task extraction accuracy
    - CHECK documentation parsing
    - VALIDATE error handling

Task 8 - Local Testing:
  TEST basic functionality:
    - RUN: wrangler dev --config wrangler-prp-parser.jsonc
    - VERIFY server starts without errors
    - TEST OAuth flow: http://localhost:8788/authorize
    - TEST MCP endpoint: http://localhost:8788/mcp

  TEST with MCP Inspector:
    - RUN: npx @modelcontextprotocol/inspector
    - CONNECT to local server
    - TEST all CRUD operations
    - VERIFY PRP parsing works

Task 9 - Production Deployment:
  DEPLOY to Cloudflare Workers:
    - RUN: wrangler deploy --config wrangler-prp-parser.jsonc
    - VERIFY deployment success
    - TEST production OAuth flow
    - VERIFY all tools accessible
```

### Per Task Implementation Details

```typescript
// Task 3 - PRP Parser Implementation Pattern
// src/prp-parser/parsers/prp-parser.ts

import Anthropic from '@anthropic-ai/sdk';
import { z } from 'zod';
import { PRP, Task, DocumentationRef } from '../types';

export class PRPParser {
  private anthropic: Anthropic;

  constructor(apiKey: string) {
    this.anthropic = new Anthropic({ apiKey });
  }

  async parsePRP(content: string): Promise<PRP> {
    const prompt = `
You are a PRP (Product Requirement Prompt) parser. Extract structured information from the following PRP markdown content.

Extract:
1. Name and description from the YAML frontmatter
2. Goal section
3. Why section (as array of bullet points)
4. What section
5. Success criteria (as array)
6. All documentation references (urls, files, docfiles)
7. Task list with order, descriptions, and any implementation details

Return a JSON object with this structure:
{
  "name": "PRP name",
  "description": "PRP description",
  "goal": "Goal text",
  "why": ["reason1", "reason2"],
  "what": "What section content",
  "successCriteria": ["criteria1", "criteria2"],
  "context": {
    "documentation": [
      {"type": "url", "path": "...", "why": "..."},
      {"type": "file", "path": "...", "why": "..."}
    ],
    "codebaseTree": "tree content",
    "desiredTree": "desired tree content",
    "knownGotchas": "gotchas content"
  },
  "tasks": [
    {
      "order": 1,
      "description": "Task description",
      "fileToModify": "path/to/file.ts",
      "pattern": "pattern to follow",
      "pseudocode": "pseudocode if any"
    }
  ]
}

PRP Content:
${content}
`;

    const response = await this.anthropic.messages.create({
      model: 'claude-3-sonnet-20240229',
      max_tokens: 4000,
      messages: [{ role: 'user', content: prompt }],
    });

    // Parse and validate the response
    const parsed = JSON.parse(response.content[0].text);
    return this.validatePRPStructure(parsed);
  }

  private validatePRPStructure(data: any): PRP {
    // Validate with Zod schema
    // Transform to PRP type
    // Add IDs and timestamps
    return data as PRP;
  }
}

// Task 4 - Tool Implementation Pattern
// src/prp-parser/tools/parse-prp.ts

import { z } from "zod";
import { withDatabase } from "../../database";
import { PRPParser } from "../parsers/prp-parser";
import { ParsePRPSchema } from "../types";

export function registerParsePRPTool(server: any, env: Env, props: Props) {
  server.tool(
    "parsePRP",
    "Parse a PRP markdown file and extract structured information using Claude AI",
    ParsePRPSchema,
    async ({ prpContent, extractTasks, extractDocs }) => {
      try {
        // Initialize parser
        const parser = new PRPParser(env.ANTHROPIC_API_KEY);
        
        // Parse PRP content
        console.log(`Parsing PRP for user: ${props.login}`);
        const parsed = await parser.parsePRP(prpContent);
        
        // Save to database
        return await withDatabase(env.DATABASE_URL, async (db) => {
          // Start transaction
          const prpResult = await db`
            INSERT INTO prps (
              name, description, goal, why, what, 
              success_criteria, context, created_by
            ) VALUES (
              ${parsed.name}, ${parsed.description}, ${parsed.goal},
              ${JSON.stringify(parsed.why)}, ${parsed.what},
              ${JSON.stringify(parsed.successCriteria)},
              ${JSON.stringify(parsed.context)}, ${props.login}
            )
            RETURNING id, created_at
          `;
          
          const prpId = prpResult[0].id;
          
          // Insert tasks if requested
          if (extractTasks && parsed.tasks.length > 0) {
            await db`
              INSERT INTO tasks ${db(
                parsed.tasks.map(task => ({
                  prp_id: prpId,
                  order: task.order,
                  description: task.description,
                  file_to_modify: task.fileToModify,
                  pattern: task.pattern,
                  pseudocode: task.pseudocode,
                  status: 'pending'
                }))
              )}
            `;
          }
          
          return {
            content: [{
              type: "text",
              text: `**PRP Parsed Successfully**\n\n` +
                    `**Name:** ${parsed.name}\n` +
                    `**ID:** ${prpId}\n` +
                    `**Tasks Extracted:** ${parsed.tasks.length}\n` +
                    `**Documentation References:** ${parsed.context.documentation.length}\n\n` +
                    `The PRP has been successfully parsed and saved to the database.`
            }]
          };
        });
      } catch (error) {
        console.error('PRP parsing error:', error);
        return {
          content: [{
            type: "text",
            text: `Failed to parse PRP: ${error.message}`,
            isError: true
          }]
        };
      }
    }
  );
}

// Task 5 - Main MCP Server Pattern
// src/prp-parser/index.ts

export class PRPParserMCP extends McpAgent<Env, Record<string, never>, Props> {
  server = new McpServer({
    name: "PRP Parser MCP Server",
    version: "1.0.0",
  });

  async cleanup(): Promise<void> {
    try {
      await closeDb();
      console.log("Database connections closed successfully");
    } catch (error) {
      console.error("Error during database cleanup:", error);
    }
  }

  async alarm(): Promise<void> {
    await this.cleanup();
  }

  async init() {
    console.log(`PRP Parser MCP initialized for user: ${this.props.login}`);
    
    // Register basic tools available to all authenticated users
    registerParsePRPTool(this.server, this.env, this.props);
    registerTaskCRUDTools(this.server, this.env, this.props);
    registerDocumentationTools(this.server, this.env, this.props);
    registerTagTools(this.server, this.env, this.props);
    registerSearchTools(this.server, this.env, this.props);
    
    // Register privileged tools based on user permissions
    const PRIVILEGED_USERS = new Set(['coleam00']);
    if (PRIVILEGED_USERS.has(this.props.login)) {
      // Add any admin-only tools here
      console.log(`Privileged tools enabled for: ${this.props.login}`);
    }
  }
}

// Export OAuth provider with MCP endpoints
export default new OAuthProvider({
  apiHandlers: {
    "/sse": PRPParserMCP.serveSSE("/sse") as any,
    "/mcp": PRPParserMCP.serve("/mcp") as any,
  },
  authorizeEndpoint: "/authorize",
  clientRegistrationEndpoint: "/register",
  defaultHandler: GitHubHandler as any,
  tokenEndpoint: "/token",
});
```

### Integration Points

```yaml
CLOUDFLARE_WORKERS:
  - wrangler-prp-parser.jsonc: New configuration for PRP parser
  - Environment variables: Add ANTHROPIC_API_KEY
  - Durable Objects: Configure MCP agent binding for state persistence

GITHUB_OAUTH:
  - Reuse existing GitHub OAuth app or create new one
  - Callback URL: https://prp-parser.workers.dev/callback
  - Permissions: Read user profile for authentication

DATABASE:
  - PostgreSQL: Add new tables for PRPs, tasks, tags
  - Migrations: Run database migrations before deployment
  - Indexes: Ensure proper indexing for performance

ANTHROPIC_API:
  - API Key: Required for Claude integration
  - Model: claude-3-sonnet-20240229 for PRP parsing
  - Rate Limits: Consider API rate limits in implementation

ENVIRONMENT_VARIABLES:
  - Development: .dev.vars with ANTHROPIC_API_KEY
  - Production: Cloudflare Workers secret for API key
  - Required: All existing vars plus ANTHROPIC_API_KEY
```

## Validation Loop

### Level 1: TypeScript & Configuration

```bash
# CRITICAL: Run these FIRST - fix any errors before proceeding
npm run type-check                 # TypeScript compilation
wrangler types --config wrangler-prp-parser.jsonc  # Generate Cloudflare Workers types

# Expected: No TypeScript errors
# If errors: Fix type issues, missing interfaces, import problems
```

### Level 2: Database Setup

```bash
# Run database migrations
psql $DATABASE_URL < src/prp-parser/database/migrations.sql

# Verify tables created
psql $DATABASE_URL -c "\dt prps, tasks, tags, task_tags, prp_tags"

# Expected: All tables exist with proper structure
# If errors: Check migration SQL, fix syntax errors
```

### Level 3: Local Development Testing

```bash
# Start local development server
wrangler dev --config wrangler-prp-parser.jsonc

# Test OAuth flow (should redirect to GitHub)
curl -v http://localhost:8788/authorize

# Test MCP endpoint (should return server info)
curl -v http://localhost:8788/mcp

# Expected: Server starts, OAuth redirects to GitHub, MCP responds with server info
# If errors: Check console output, verify environment variables, fix configuration
```

### Level 4: MCP Tool Testing

```bash
# Test with MCP Inspector
npx @modelcontextprotocol/inspector

# Connect to: http://localhost:8788/mcp
# Authenticate with GitHub
# Test each tool:
# 1. parsePRP with sample PRP content
# 2. listTasks to verify parsing worked
# 3. createTask to add new task
# 4. updateTask to modify task
# 5. searchByTag to test search functionality

# Expected: All tools work correctly with proper responses
# If errors: Check tool implementations, database queries, permissions
```

### Level 5: Integration Testing

```bash
# Test complete workflow
# 1. Parse a real PRP from PRPs/ai_docs/filled_prp_example.md
# 2. Verify all tasks extracted correctly
# 3. Update task statuses
# 4. Add tags and search by them
# 5. Retrieve documentation

# Test error cases
# 1. Invalid PRP format
# 2. Unauthorized write attempts
# 3. Database connection failures
# 4. Anthropic API errors

# Expected: Graceful error handling, clear error messages
# If errors: Improve error handling, add missing validations
```

## Final Validation Checklist

### Core Functionality

- [ ] TypeScript compilation: `npm run type-check` passes
- [ ] Cloudflare types generated successfully
- [ ] Database migrations applied successfully
- [ ] Local server starts: `wrangler dev` runs without errors
- [ ] MCP endpoint responds: `curl http://localhost:8788/mcp` returns server info
- [ ] OAuth flow works: Authentication redirects and completes successfully
- [ ] PRP parsing extracts all required information correctly
- [ ] All CRUD operations work with proper validation
- [ ] Permission system prevents unauthorized writes
- [ ] One task per file pattern properly implemented

### Production Readiness

- [ ] All tests pass including error cases
- [ ] Anthropic API integration works reliably
- [ ] Database queries are optimized with proper indexes
- [ ] Error messages are user-friendly and secure
- [ ] Logging provides adequate debugging information
- [ ] Production deployment succeeds
- [ ] Monitoring and alerting configured (if using Sentry)

---

## Anti-Patterns to Avoid

### MCP-Specific

- ❌ Don't skip input validation with Zod - always validate tool parameters
- ❌ Don't forget to implement cleanup() method for Durable Objects
- ❌ Don't hardcode user permissions - use configurable permission systems
- ❌ Don't mix concerns - keep one task per file for tools

### PRP Parsing

- ❌ Don't use complex regex for parsing - use Claude AI as specified
- ❌ Don't assume PRP structure - validate before parsing
- ❌ Don't lose context during parsing - preserve all documentation references
- ❌ Don't skip error handling for Anthropic API failures

### Development Process

- ❌ Don't skip the validation loops - each level catches different issues
- ❌ Don't deploy without testing all CRUD operations
- ❌ Don't ignore TypeScript errors - fix all type issues before deployment
- ❌ Don't forget to test with actual PRP content from examples