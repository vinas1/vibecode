# Local Vibe Coding with Cline and LM Studio

Use Cline with LM Studio to run an AI coding agent locally in Visual Studio Code. This setup keeps source code on approved hardware, avoids cloud-model token usage, and provides an agent that can read files, propose changes, and run approved terminal commands.

## Architecture

```text
Visual Studio Code
└── Cline
    └── LM Studio
        └── LM Link
            └── Gemma 4 12B QAT
```

*see more in the* [architecture.md](https://github.com/vinas1/vibecode/blob/main/architecture.md)

| Component | Standard |
|---|---|
| IDE | Visual Studio Code |
| Coding agent | Cline |
| Inference runtime | LM Studio |
| Primary model | Gemma 4 12B QAT |
| Model ID | `google/gemma-4-12b-qat` |
| Remote model routing | LM Link |
| Fallback model | Qwen2.5 Coder 14B |

## Prerequisites

Before starting, confirm the following are installed and available:

- Visual Studio Code
- LM Studio
- Cline for Visual Studio Code
- Gemma 4 12B QAT downloaded locally or available through LM Link
- A project folder opened in Visual Studio Code

## 1. Start the LM Studio Server

1. Open LM Studio.
2. Open the **Developer** tab.
3. Load **Gemma 4 12B QAT** locally or from an LM Link device.
4. Start the LM Studio server.
5. Confirm the server is available at:

   ```text
   http://localhost:1234
   ```

Verify the endpoint from a terminal:

```bash
curl http://localhost:1234/v1/models
```

The response should include:

```text
google/gemma-4-12b-qat
```

Example:

```json
{
  "data": [
    {
      "id": "google/gemma-4-12b-qat",
      "object": "model",
      "owned_by": "organization_owner"
    }
  ],
  "object": "list"
}
```

> If Gemma is loaded on another machine through LM Link, continue using the local endpoint. LM Studio handles routing the request to the linked device.

## 2. Install Cline

### Install from Visual Studio Code

1. Open Visual Studio Code.
2. Open **Extensions**:
   - Windows or Linux: `Ctrl+Shift+X`
   - macOS: `Cmd+Shift+X`
3. Search for **Cline**.
4. Select the official Cline extension.
5. Click **Install**.
6. Open Cline from the Visual Studio Code activity bar.

### Install from the Command Line (install vscode if it's not already availabel)

```bash
brew install --cask visual-studio-code
code --install-extension saoudrizwan.claude-dev --force
```

## 3. Skip the Cline Account Setup

A Cline account is not required when using LM Studio locally.

If Cline displays the **Select a free model** screen:

1. Do not select one of the hosted free models.
2. Do not click **Create my Account**.
3. Click **Back**.
4. Open Cline settings using the gear icon in the upper-right corner.

The hosted account and billing options are not needed for this local setup.

## 4. Connect Cline to LM Studio

Open Cline settings and configure the following:

```text
API Provider: LM Studio
Base URL: http://localhost:1234
Model ID: google/gemma-4-12b-qat
```

Leave the API key blank.

Do not add `/v1` unless the installed Cline version explicitly requests an OpenAI-compatible API URL.

Avoid accidentally entering:

```text
http://localhost:1234/v1/v1
```

## 5. Enable Compact Prompts

In Cline settings, enable:

```text
Use Compact Prompt
```

Compact prompts reduce agent instruction overhead and leave more model context available for source code, repository content, and tool results.

Start a new Cline task when the current task accumulates too much context.

## 6. Configure Privacy Settings

For internal or privacy-sensitive development:

1. Open Cline settings.
2. Find **Cline Telemetry**.
3. Turn telemetry off.

Continue following organizational requirements for approved models, source-code handling, and local AI usage.

## 7. Configure Agent Permissions

Keep human approval enabled while evaluating the agent.

Require approval for:

- File modifications
- File creation
- File deletion
- Terminal commands
- External access
- MCP tool execution

Read-only operations may be auto-approved if desired.

A recommended starting configuration is:

```text
Auto-approve:
- Read files
- Safe commands

Require approval:
- Edit files
- Create files
- Delete files
- Run non-read-only commands
- Access external resources
```

Review every proposed diff before selecting **Save**.

## 8. Use Plan and Act Modes

Cline provides separate modes for analysis and execution.

### Plan Mode

Use **Plan** when the agent should:

- Explore the repository
- Read files
- Identify dependencies
- Explain code
- Propose an implementation
- Avoid making changes

Example:

```text
Review this repository and explain where the API base URL is configured. Do not modify any files.
```

### Act Mode

Use **Act** when the agent should:

- Edit files
- Create files
- Run commands
- Refactor code
- Execute a multi-step development task

Example:

```text
Find where the API base URL is configured, update it to use the environment variable API_BASE_URL, and show me the proposed changes before saving.
```

## 9. Validate the Agent

Run the following tests before using the setup for normal development.

### Test 1: Read a File

```text
Read index.html and report the first three elements inside the head element. Do not modify anything.
```

Expected behavior:

- Cline reads `index.html`.
- Gemma uses the actual file contents.
- No raw JSON or tool-call syntax is displayed.
- No file changes are proposed.

### Test 2: Make a Targeted Edit

```text
Read index.html. Add <!-- Site metadata and icons --> immediately above the first meta element. Make no other changes.
```

Expected behavior:

1. Cline reads `index.html`.
2. Gemma locates the first `<meta>` element.
3. Cline displays an editable side-by-side diff.
4. Cline waits for approval.
5. The change is applied only after **Save** is selected.
6. No unrelated content is modified.

Expected result:

```html
<head>
    <!-- Site metadata and icons -->
    <meta charset="UTF-8">
```

### Test 3: Verify the Change

```text
Run git diff and confirm that only the requested HTML comment was added.
```

Expected behavior:

- Cline requests permission to run the command.
- `git diff` executes in the project terminal.
- Gemma summarizes the actual command output.
- No additional changes are made.

### Test 4: Run a Read-Only Command

```text
Run git status and summarize the result. Do not modify anything.
```

Expected behavior:

- Cline requests command approval.
- The command executes successfully.
- Gemma summarizes the repository state.

### Test 5: Traverse Multiple Files

```text
Find the stylesheet referenced by index.html. Read the stylesheet and identify the body rule. Do not modify anything.
```

Expected behavior:

1. Cline reads `index.html`.
2. Gemma identifies the referenced stylesheet.
3. Cline reads the stylesheet.
4. Gemma identifies the requested CSS rule.
5. No files are modified.

### Test 6: Complete a Multi-File Agent Task

```text
Find the stylesheet referenced by index.html and add a comment immediately above the body rule saying Base page styling. Do not change any CSS properties.
```

Expected behavior:

1. Cline reads `index.html`.
2. Gemma identifies the stylesheet.
3. Cline reads the stylesheet.
4. Gemma locates the `body` rule.
5. Cline proposes one targeted edit.
6. Cline waits for approval.
7. No CSS properties are modified.

Expected result:

```css
/* Base page styling */
body {
```

## 10. Review and Save Changes

When Cline proposes an edit:

1. Review the original and proposed versions in the side-by-side comparison.
2. Confirm that only the requested lines changed.
3. Select **Save** to apply the change.
4. Select **Reject** if the change is incorrect or too broad.
5. Use `git diff` after the edit to verify the final result.

Never approve a full-file rewrite when the request only requires a localized change.

## Recommended Prompt Pattern

Use focused instructions that clearly define the target and boundaries.

```text
Read [file].

Make this change:
[exact requested change]

Constraints:
- Change only what was requested.
- Preserve unrelated content and formatting.
- Do not rewrite the entire file.
- Show the proposed diff before saving.
```

Example:

```text
Read index.html.

Add <!-- Site metadata and icons --> immediately above the first meta element.

Constraints:
- Make no other changes.
- Preserve the current indentation.
- Do not rewrite the entire file.
- Show the proposed diff before saving.
```

## Troubleshooting

### Cline Requests an Account

Do not create an account for local LM Studio usage.

1. Click **Back**.
2. Open the Cline settings gear.
3. Select **LM Studio** as the API provider.
4. Configure the local endpoint and model ID.

### The Model Does Not Appear

Verify the LM Studio endpoint:

```bash
curl http://localhost:1234/v1/models
```

Confirm the response includes:

```text
google/gemma-4-12b-qat
```

If the model is not listed:

1. Confirm the model is loaded.
2. Confirm the LM Studio server is running.
3. Confirm the LM Link device is connected if the model is remote.
4. Restart the LM Studio server if necessary.

### Cline Cannot Connect to LM Studio

Confirm the configured base URL is:

```text
http://localhost:1234
```

Check that the LM Studio Developer server is running.

Avoid duplicate API paths such as:

```text
http://localhost:1234/v1/v1
```

### The Agent Produces a Large or Unrelated Edit

Reject the proposed change and submit a more targeted instruction:

```text
Make one localized change only. Preserve every unrelated line. Do not rewrite the complete file.
```

### The Agent Uses Too Much Context

1. Enable **Use Compact Prompt**.
2. Start a new Cline task.
3. Keep the request focused.
4. Avoid attaching unrelated files.
5. Use Plan mode to locate the target before switching to Act mode.

### Gemma Is Running on an LM Link Device

Continue configuring Cline with:

```text
http://localhost:1234
```

Use the exact model ID:

```text
google/gemma-4-12b-qat
```

LM Studio handles the remote model route through LM Link.

## Security and Usage Guidelines

- Use approved local models.
- Keep source code on approved hardware.
- Review all proposed changes before saving.
- Require approval for terminal commands and file writes.
- Use Plan mode for investigation and Act mode for implementation.
- Run `git diff` after agent-generated changes.
- Do not send credentials, tokens, passwords, or secrets in prompts.
- Do not allow the agent to modify files outside the current workspace.
- Keep changes small, reviewable, and traceable through Git.
- Maintain human review for all production changes.

## A Proposed Vibe Code Standard

```text
IDE: VS Code
Agent: Cline
Inference Runtime: LM Studio
Primary Model: Gemma 4 12B QAT
Model ID: google/gemma-4-12b-qat
Remote Model Routing: LM Link
Fallback Model: Qwen2.5 Coder 14B
```

Cline provides the agent workflow, LM Studio provides local model inference, and Gemma 4 provides the approved local model.

#### Previous state
Pre mid 2026, I encouraged the use of "Continue" plugin with this workflow however, model support is lacking for USA based models along with frequent agent errors, so I no longer recommend the use of Continue until the stability issues are resolved. As a result, Continue is not required for this workflow and should be uninstalled to reduce congnitive load and friction. *Happy vibing!*
