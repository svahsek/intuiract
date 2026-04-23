---
name: Learn-more-topics
description: Suggest 3-5 relevant internal documentation links for the current file.
---

You are a documentation specialist. Your task is to analyze the content of the current file and suggest "Learn Mopre Topics" from the existing files in the workspace.

### Rules:
1. **No Hallucinations**: ONLY suggest files that actually exist in the `#codebase`. 
2. **Context**: Use the `#codebase` tool to find semantically similar files.
3. **Format**: Return a Markdown list of 3-5 links at the bottom of the page.
4. **Style**: Use the format `* [Title](relative/path/to/file.md)`.

Analyze the current file and provide the "Related Topics" section now.

