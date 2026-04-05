---
name: ai-handoff-readme
description: Automatically inject an AI Assistant setup/deployment prompt into project READMEs. This allows end-users without technical knowledge to easily deploy complex repositories by simply pasting a pre-written prompt into their own AI.
---

# AI Handoff Prompt Integration

When designing software meant to be shared, the largest friction point for non-technical users is environment setup (cloning repos, installing Docker, Node.js, setting paths, etc.). 

By adopting the **AI Handoff Prompt** pattern, you bypass writing endless technical installation guides. Instead, you provide a single prompt that the user copies into their own AI (like Claude, ChatGPT, or Cursor) which then acts as a personalized deployment agent for them.

## The Strategy

Always inject a highly visible section near the top of the `README.md` (usually right before or after the 'Quick Start' section) containing a highly-specific deployment mandate.

### The Standard Template

```markdown
## 🤖 AI Assistant Installation (For ChatGPT/Claude/Cursor)

If you have an AI coding assistant and want it to automatically set this project up for you, simply copy and paste the prompt below into your AI of choice:

> **Copy & Paste this to your AI:**
> *"I want to run a locally-hosted web application called '[Project Name]' on my machine. Please give me the exact terminal commands to clone the repository (from [GitHub Username]/[Project Name]), navigate into the folder, and spin it up using Docker Compose. If I don't have Docker installed, briefly tell me how to get it for my OS first. Once the container is running, tell me what localhost port to open, and explain how I can save it as a PWA on my phone or computer."*
```

### Customizing the Handoff

When adding this to a new repository, modify the prompt variables to direct the AI accurately:
1. **Repository Target:** Clearly specify the format `GhostUsername/RepoName` so the AI knows exactly what to `git clone`.
2. **Framework Specifics:** If the app uses a `.env` file that needs to be copied from `.env.example`, instruct the AI to do this.
3. **Execution Environment:** Tell the AI exactly what runtime it should use (e.g., `Docker Compose`, `npm run dev`, `python main.py`).
4. **Conclusion Hook:** Always instruct the AI to explain what port the app runs on once finished so the user isn't stuck staring at a terminal.

## Why This is a Best Practice

- **Zero-Support Burden:** The AI deployed by the user will read the repo, troubleshoot local machine failures, and explain terminal states without the developer needing to provide tech support.
- **Dynamic Context:** If the user is on Windows Native vs MacOS M3 vs Linux, their AI will automatically write the correct paths and package manager configurations for them based on the prompt.
