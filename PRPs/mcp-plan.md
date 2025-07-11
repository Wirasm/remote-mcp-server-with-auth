We want to create a MCP server using this repos template

The goal of the MCP server is to create a simple version of taskmaster mcp that instead of parsing PRD's we parse PRP's

Key features:

- LLM powered PRP information extraction using anthropic
- crud operation on tasks, documentation, tags, etc to and from the DB

Here is an example of a PRP, both template and filled:

read these:
PRPs/ai_docs/prp_example_template.md
PRPs/ai_docs/filled_prp_example.md

**VERY IMPORTANT** Do not use complex regex or complex parsing patterns, we use cluade-4-sonnet for parsing PRPs

We need tools for parsing PRPs this tool should take a filled PRP and use anthropic to extract the tasks into tasks and save them to the db, including surrounding documentation from the prp like the goals what whys, target users, etc.

We want to be able to perform CRUD operations on tasks, documetnation, tags, etc

we need a task fetch tool to get the tasks from the

we want to be able to list all tasks

We want to be able to add information to a task

we want to be able to fetch the additional documentation from the db

We want to be able to modify the additional documentation

The db tables needs to be updated to match our new data models

ITS VERY IMPORTANT THAT WE CREATE ONE TASK PER FILE TO KEEP CONCERNS SEPERATE
