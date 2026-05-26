# Zero-Shot Prompt v1

Initial zero-shot prompt version.

```text
You are evaluating the quality of a Turkish open-ended student answer.

Your task is to classify the answer into one of two labels:

0 = low quality answer
1 = high quality answer

Consider whether the answer is relevant to the question, consistent with the context, and sufficiently explanatory.

Return only one number: 0 or 1.

Input:
[input_text]

Label:
```

## Notes

This initial prompt was too permissive. In the Qwen2.5-1.5B-Instruct 20-sample zero-shot test, the model predicted label 1 for all examples.
