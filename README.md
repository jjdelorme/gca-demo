# Gemini Code Assist
- Where is it best? 
    - Simple questions... in context (open tab, cusor position, etc...)
    - Simpler, single file changes
- Agent Mode
    - Goal oriented
    - Using multiple files
    - Uses tools
    - "ReAct" (reason) and take actions (acting) to solve a problem. It iterates through a loop of thinking, taking an action.

    * Test Agent Mode:
        ```
        Setup a proxy in angular so that I can avoid CORS errors when calling the server on port 8083 in development?
        ```

# Overview of agentic development
A few demonstrations of the Gemini CLI.

## The Agent Loop
* State the goal in clear terms, try to avoid ambiguity, provide a means to verify (i.e. tests)
* Manage context! (`100% context left`)
* Use tools! `/tools`
* Adding context with @filename.ext, urls or asking to use the websearch tool.
    - *NOTE: for security reasons, files can only be added relative to the workspace directory*

## Save a chat
```
/chat save my-chat
/chat resume my-chat
/chat share my-chat.md
```

## More tools!
Examples of using MCP Servers
* Context7
    - `Use context7 to find the latest documentation on how to create a signal in Angular and show me an example`
* Playwright, Chrome Dev Tools, Puppeteer
    - Automate the browser to test / debug / verify a change
* Git
    - Minimize registration of MCP Servers... i.e. using the git CLI vs. github MCP server
    - Allow / deny MCP Servers
* Figma
    - Remote vs. Local MCP Servers; demonstrates OAuth
    - Read designs; ask it to implement or read a color for example
    

### Configuring individual MCP Servers
```json
  "mcpServers": {
    "github": {
      "httpUrl": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer $GITHUB_MCP_PAT"
      },
      "excludeTools": [
        "create_repository",
        "delete_file",
        "fork_repository",
        "get_commit"
      ]
    },
    ...
```

## Context is everything!
* **Context Poisoning**: When a hallucination or other error makes it into the context, where it is repeatedly referenced.
* **Context Distraction**: When a context grows so long that the model over-focuses on the context, neglecting what it learned during training.
* **Context Confusion**: When superfluous information in the context is used by the model to generate a low-quality response.
* **Context Clash**: When you accrue new information and tools in your context that conflicts with other information in the prompt.

Use the `/compress` & `/clear` commands

### Memory & GEMINI.md
* `/memory` and `GEMINI.md`
* `/init` command will take a stab at generating, but add your specific rules to this.
* Hierarchical, concatenated scopes
    - system, user, project, ...
* What should you put in GEMINI.md
    - Manage size...
    - Clarify anything that might be ambiguous

### Custom Slash Commands
```bash
mkdir -p .gemini/commands/
```

#### ice-breaker.toml:
```yaml
description = "Create an ice breaker"
prompt = "Based on the user's input {{args}} create an ice breaker statement"
```

#### Install with Gemini Extensions 
```bash
gemini extensions install https://github.com/jjdelorme/plan-commands
```

## Vibe Coding
* What
    - Vibe Engineering? 
    - Imagine a brand new PhD graduate with 0 practical experience; your new intern
* Why
    - Cost of experimentation is almost 0
    - Great at grunt work
* When
    - You still apply *YOUR* critical thinking to the defition of `how`
    - Best for rapid prototyping, exploration, and experimentation
* How
    - Context / Goals / Verifiable Success Crtieria; "you will be done when..."
    - Humans always in the loop
* *BEWARE*
    - AI Slop
    - Code Review Fatigue
    - "When COBOL was released people said **we no longer need programmers**"; *said those coming from assembly*

### Planning (Spec-Driven Development)
```markdown
/plan:new 
Create a new AiQuoteService that will generate a Quote object using a "theme" parameter with the full AI prompt; Generate one single short, witty random quote based on the theme in {theme}".  

AiQuoteService will use the gemini-2.5-flash-lite model in Vertex AI with the max tokens output set to 64.  The model will be invoked using spring-ai-starter-model-vertex-ai-gemini.  Use context7 to lookup the documentation on how to implement this with spring-ai.  

Add a new endpoint /generate-quote to the QuoteController which takes the parameter "prompt" and returns the Quote object by calling the AiQuoteService.

Do not implement any tests in this plan.
```

- `/plan:implement`
- In quotes-web: `Change the /random-quote end point call to /generate-quote`

### Yolo Mode
CTRL-Y / `-y`


## Advanced

### Override System Prompt

#### Write out the system prompt

```bash
GEMINI_WRITE_SYSTEM_MD=1 gemini -p "Why is the sky blue?"    
```

#### Override the system prompt
```bash
GEMINI_SYSTEM_MD=1
# or
GEMINI_SYSTEM_MD=path_to_your_md
```

# Installation & Configuration
Installation instruction and documentation for Gemini CLI: https://github.com/google-gemini/gemini-cli 

## * Including / Excluding ToolsConfiguration
* [settings.json Files](https://github.com/google-gemini/gemini-cli/blob/main/docs/get-started/configuration.md#settings-files)
* `/etc/gemini-cli/settings.json` (Linux), `C:\ProgramData\gemini-cli\settings.json` (Windows) taking precedence over all other settings files

### Authentication
* `/auth` Options explained
* Login with Google (*With Gemini Code Assist license*), limited to 1,500/2,000 requests per day (standard/enterprise)

### Telemetry
```json
  "telemetry": {
    "enabled": true,
},
...
```

## Tips
* LLMs perform better when given rigid constraints rather than polite suggestions. They don't get offended by all-caps.
    - Use terms like **"CRITICAL PROTOCOL"**, **"MUST ADHERE RIGIDLY"**, and **"NEVER EXECUTE"**
* Provide examples (few-shot)
- Provide explicit ✅ and ❌ examples. *Note: the models recognize icons as tokens*
* **BEWARE**: *IT'S NOT ALWAYS GOING TO LISTEN TO YOU*