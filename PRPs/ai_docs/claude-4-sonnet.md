# Migrating to Claude 4

This page provides guidance on migrating from Claude 3.7 models to Claude 4 models (Opus 4 and Sonnet 4).

In most cases, you can switch to Claude 4 models with minimal changes:

1. Update your model name:
   - From: `claude-3-7-sonnet-20250219`
   - To: `claude-sonnet-4-20250514` or `claude-opus-4-20250514`

2. Existing API calls should continue to work without modification, although API behavior has changed slightly in Claude 4 models (see [API release notes](/en/release-notes/api) for details).

## What's new in Claude 4

### New refusal stop reason

Claude 4 models introduce a new `refusal` stop reason for content that the model declines to generate for safety reasons, due to the increased intelligence of Claude 4 models:

```json
{
  "id": "msg_014XEDjypDjFzgKVWdFUXxZP",
  "type": "message",
  "role": "assistant",
  "model": "claude-sonnet-4-20250514",
  "content": [{ "type": "text", "text": "I would be happy to assist you. You can " }],
  "stop_reason": "refusal",
  "stop_sequence": null,
  "usage": { "input_tokens": 564, "cache_creation_input_tokens": 0, "cache_read_input_tokens": 0, "output_tokens": 22 }
}
```

When migrating to Claude 4, you should update your application to [handle `refusal` stop reasons](/en/docs/test-and-evaluate/strengthen-guardrails/handle-streaming-refusals).

### Summarized thinking

With extended thinking enabled, the Messages API for Claude 4 models returns a summary of Claude's full thinking process. Summarized thinking provides the full intelligence benefits of extended thinking, while preventing misuse.

While the API is consistent across Claude 3.7 and 4 models, streaming responses for extended thinking might return in a "chunky" delivery pattern, with possible delays between streaming events.

<Note>
  Summarization is processed by a different model than the one you target in your requests. The thinking model does not see the summarized output.
</Note>

For more information, see the [Extended thinking documentation](/en/docs/build-with-claude/extended-thinking#summarized-thinking).

### Interleaved thinking

Claude 4 models support interleaving tool use with extended thinking, allowing for more natural conversations where tool uses and responses can be mixed with regular messages.

<Note>
  Interleaved thinking is in beta. To enable interleaved thinking, add [the beta header](/en/api/beta-headers) `interleaved-thinking-2025-05-14` to your API request.
</Note>

For more information, see the [Extended thinking documentation](/en/docs/build-with-claude/extended-thinking#interleaved-thinking).

### Updated text editor tool

The text editor tool has been updated for Claude 4 models with the following changes:

- **Tool type**: `text_editor_20250429`
- **Tool name**: `str_replace_based_edit_tool`
- The `undo_edit` command is no longer supported in Claude 4 models.

<Note>
  The `str_replace_editor` text editor tool remains the same for Claude Sonnet 3.7.
</Note>

If you're migrating from Claude Sonnet 3.7 and using the text editor tool:

```python
# Claude Sonnet 3.7
tools=[
    {
        "type": "text_editor_20250124",
        "name": "str_replace_editor"
    }
]

# Claude 4
tools=[
    {
        "type": "text_editor_20250429",
        "name": "str_replace_based_edit_tool"
    }
]
```

For more information, see the [Text editor tool documentation](/en/docs/agents-and-tools/tool-use/text-editor-tool).

### Token-efficient tool use no longer supported

[Token-efficient tool use](/en/docs/agents-and-tools/tool-use/token-efficient-tool-use) is only available in Claude Sonnet 3.7.

If you're migrating from Claude Sonnet 3.7 and using token-efficient tool use, we recommend removing the `token-efficient-tools-2025-02-19` [beta header](/en/api/beta-headers) from your requests.

The `token-efficient-tools-2025-02-19` beta header can still be included in Claude 4 requests, but it will have no effect.

### Extended output no longer supported

The `output-128k-2025-02-19` [beta header](/en/api/beta-headers) for extended output is only available in Claude Sonnet 3.7.

If you're migrating from Claude Sonnet 3.7, we recommend removing `output-128k-2025-02-19` from your requests.

The `output-128k-2025-02-19` beta header can still be included in Claude 4 requests, but it will have no effect.

## Performance considerations

### Claude Sonnet 4

- Improved reasoning and intelligence capabilities compared to Claude Sonnet 3.7
- Enhanced tool use accuracy

### Claude Opus 4

- Most capable model with superior reasoning and intelligence
- Slower than Sonnet models
- Best for complex tasks requiring deep analysis

## Migration checklist

- [ ] Update model id in your API calls
- [ ] Test existing requests (should work without changes)
- [ ] Remove `token-efficient-tools-2025-02-19` beta header if applicable
- [ ] Remove `output-128k-2025-02-19` beta header if applicable
- [ ] Handle new `refusal` stop reason
- [ ] Update text editor tool type and name if using it
- [ ] Remove any code that uses the `undo_edit` command
- [ ] Explore new tool interleaving capabilities with extended thinking
- [ ] Review [Claude 4 prompt engineering best practices](/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices) for optimal results
- [ ] Test in development before production deployment

## Need help?

- Check our [API documentation](/en/api/overview) for detailed specifications.
- Review [model capabilities](/en/docs/about-claude/models/overview) for performance comparisons.
- Review [API release notes](/en/release-notes/api) for API updates.
- Contact support if you encounter any issues during migration.
