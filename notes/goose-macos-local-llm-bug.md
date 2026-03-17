# [Bug Report] Goose macOS + Local LLM Tool Collision

| Attribute | Details |
| :--- | :--- |
| **Date** | 2026-03-17 |
| **Status** | 🟠 Reported Upstream |
| **Target** | block/goose v1.27.2 |
| **Upstream Issue** | [Issue #7968](https://github.com/block/goose/issues/7968) |

## 📝 Summary
On macOS Sequoia, when using local providers (**Ollama** or **Local Inference**), Goose fails to route `Developer.write` calls to its internal MCP extension. 

The system attempts to execute `write` as a raw shell command, which triggers a name collision with the legacy macOS utility `/usr/bin/write`. This results in no files being created.

## 🔍 Detailed Observations
- **Ollama Behavior (Silent Failure):** Goose claims the tool call was successful and reports a valid path, but no file is created. The UI shows no error.
- **Local Inference Behavior (Explicit Error):** Goose displays a shell error: `write: <filename> is not logged in on <content>`. This is the "smoking gun" proof of the collision with `/usr/bin/write`.
- **The `tree` loop:** The model often tries to run `tree` to explore the directory. Since `tree` is not native to macOS, it returns `command not found`, often causing Goose to enter an execution loop.

## 🛠 Reproduction Steps
1. **Environment:** macOS 15.3 (M1 Max), Goose v1.27.2.
2. **Action:** Prompt: `"Create a file FINAL_TEST.txt with content 'Victoire'."`
3. **Observation:** Goose UI shows a tool call for `write`, but resolves to the shell binary.
4. **Result:** File is missing from disk.

## 📂 Debugging & Logs
For future reference, internal logs for Goose Desktop on macOS are located at:
`~/Library/Application Support/Goose/logs/main.log`

## 💡 Temporary Workarounds
- **Force Shell Extension:** Explicitly ask Goose to use the shell: 
  > *"Use your shell extension to run: `echo 'content' > file.txt`"*
- **Manual Verification:** Confirming permissions by running `touch` via the **Code Execution (Shell)** extension (this bypasses the broken `Developer.write` routing).

## 💻 Environment Details
- **OS:** macOS 15.3 ARM64 (Build 25D2128)
- **Hardware:** Apple M1 Max / 64 GB RAM
- **Permissions:** Full Disk Access was granted (did not resolve the issue).

## 📌 Technical Hypotheses
1. **Tool-Routing Failure:** Local providers fail to correctly map intents to MCP extensions, causing a fallback to "best-effort" shell execution based on the tool's name.
2. **Binary Collision:** Goose's internal tool name `write` is not namespaced enough to avoid collision with the Unix `/usr/bin/write` utility when executed via a standard shell.
