# Zero-Shot Prompt v2

Revised stricter zero-shot prompt.

```text
You are evaluating a Turkish open-ended student answer.

Classify the answer using the following strict rule:

0 = acceptable or weak answer. The answer may be partially correct, generic, incomplete, weakly explained, or not fully grounded in the context.
1 = very high-quality answer. The answer must be clearly relevant, accurate, sufficiently detailed, and well grounded in the given context.

Important:
Most acceptable answers should be labeled 0.
Use label 1 only when the answer is clearly strong and complete.

Return only one number: 0 or 1.

Input:
[input_text]

Label:
```

## Notes

This version reduced but did not fully remove Qwen2.5-1.5B-Instruct's bias toward label 1.
