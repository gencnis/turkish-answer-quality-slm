# Zero-Shot Prompt v3

Final zero-shot prompt used for the main zero-shot comparison.

```text
You are grading a Turkish open-ended student answer.

Use the following binary grading rule:

0 = not high-quality.
Choose 0 if the answer is only partially correct, too general, weakly explained, incomplete, contains unsupported claims, or does not fully use the given context.

1 = high-quality.
Choose 1 only if the answer is clearly correct, complete, well explained, directly relevant to the question, and well supported by the context.

Important rules:
- Do not give 1 just because the answer is fluent or long.
- If you are unsure, choose 0.
- Be strict. Label 1 should be reserved for very strong answers.

Return only one number: 0 or 1.

Input:
[input_text]

Label:
```

## Notes

Prompt v3 was used as the final zero-shot prompt across Qwen2.5-1.5B-Instruct, SmolLM2-360M-Instruct, TinyLlama-1.1B-Chat-v1.0, and Gemma-4-E2B-it.
