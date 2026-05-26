# TinyLlama Short Prompt with 1000-Character Context

Additional shortened prompt tested for TinyLlama-1.1B-Chat-v1.0 after the standard Prompt v3 did not reliably produce parseable binary labels.

```text
Classify the Turkish student answer.

0 = not high-quality
1 = high-quality

If unsure, choose 0.

Return only 0 or 1.

[input_text_with_context_truncated_to_1000_characters]

Answer:
```

## Notes

This shorter prompt was tested with the context truncated to 1000 characters. It still did not reliably produce parseable 0/1 labels.
