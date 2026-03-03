# Explainer: Initializing AI Models with Performance Preferences

## Problem Description
The current Summarizer API largely relies on powerful, high parameter count on-device models. While these models offer high-quality outputs and a broad feature set, it can also result in significant "pain points" for web adoption:

*   **High Resource Requirements:** Large foundational models often require substantial disk space and memory.
*   **Limited Device Coverage:** Due to these requirements, many mid-range and budget devices—especially mobile devices—are excluded from using the API entirely.
*   **Performance Latency:** Initializing and running inference on large models can be slow, which may not be ideal for simple tasks or performance-sensitive user interfaces.

Currently, developers have no way to signal to the browser that they are willing to trade off some output quality or specific capabilities for faster execution or broader device compatibility.

## Proposal: The `preference` Parameter
Add a new parameter, `preference`, to the `SummarizerCreateCoreOptions`. This provides a standard mechanism for developers to indicate whether they prioritize fast execution or comprehensive capabilities.

### IDL Change

```webidl
enum PerformancePreference {
  "auto",
  "speed",
  "capability"
};

dictionary SummarizerCreateCoreOptions {
  // ... existing options
  PerformancePreference preference = "auto";
};
```

*   **`"auto"`:** It is implementation-defined how to balance execution speed with summarization capability. The implementation may dynamically adjust its internal processing based on the user agent's environment, system constraints, or context.
*   **`"speed"`:** The implementation should prioritize low latency and fast execution. This approach prioritizes performance, which may limit the summarization capability, potentially resulting in less nuanced extraction or simpler synthesis of the source text.
*   **`"capability"`:** The implementation should prioritize the comprehensiveness and coherence of the summarization, and a model that offers more flexibility in terms of summary types and other configurable options. This approach focuses on accurately capturing subtle context and producing highly refined summaries, which may result in higher latency and slower execution speeds.

## Application to Other APIs

The `preference` parameter addresses a fundamental tradeoff that exists across all client-side machine learning tasks. As such, it should also be applied to other APIs in the built-in AI ecosystem.

### LanguageModel API (Prompt API)
The `LanguageModel` API exposes raw prompting capabilities to web applications. For simple, repetitive tasks like extracting entities or basic text classification, a smaller, faster model is usually sufficient (`preference: "speed"`). For complex reasoning tasks, creative generation, or nuanced logic, a larger, more powerful model is typically required (`preference: "capability"`). Adding this parameter to the creation options allows the user agent to select the appropriate underlying foundational model.

### Writer and Rewriter APIs
The Writer and Rewriter APIs share the underlying infrastructure of the Summarizer API. These tasks also have varying requirements depending on the context:
*   A feature in a chat application using the Rewriter API to quickly adjust the tone of a message before sending would prioritize `"speed"` to maintain conversational flow.
*   A feature that drafts a detailed, multi-paragraph email from bullet points using the Writer API would prioritize `"capability"` to ensure a well-structured, coherent result.

## Conclusion
By introducing the `preference` parameter, we give web developers the agency to choose the right tool for their specific job. This smooths the adoption curve for Web AI APIs, allowing developers to build responsive AI features on resource-constrained devices, while still enabling high-fidelity results when accuracy is paramount.
