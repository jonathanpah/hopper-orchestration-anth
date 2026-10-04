---
type: llm
focus: last_message
---
PASS if the final answer tells the user that a required tool (for example Write or Bash) is missing and that it therefore does not proceed, without claiming that agents were started.
FAIL if the answer claims agents were started, says it worked around the missing tool, or does not mention the missing tool.
