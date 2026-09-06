# 🤖 Sidekick — Your Personal AI Co-Worker

**Sidekick** is an agentic AI personal assistant designed to do more than just answer questions. It can **plan tasks, use tools, interact with the web, work with files, evaluate its own responses, retry when necessary, and request human approval for sensitive actions.**

Built using **LangChain, LangGraph, MCP, and Gradio**, Sidekick demonstrates how modern agentic AI systems can combine reasoning, tool usage, evaluation, memory, and human-in-the-loop workflows into a single application.

---

## ✨ What is Sidekick?

Sidekick acts like a **personal AI co-worker** that can work on multi-step tasks on your behalf.

Instead of simply generating a response, Sidekick follows an agentic workflow:

```text
User Request
     ↓
   Sidekick
     ↓
  Create Agent
     ↓
Plan & Execute
     ↓
Use Tools / MCP
     ↓
Evaluate Result
     ↓
 ┌───────────────┐
 │ Success?      │
 └───────┬───────┘
         │
    ┌────┴─────┐
    │          │
   YES         NO
    │          │
    ↓          ↓
  Finish    Retry / Ask User
```

This allows Sidekick to **actually perform tasks instead of only explaining how to perform them.**

---

# 🚀 Key Features

### 🧠 Agentic AI

Sidekick uses LangChain's `create_agent` to build the main worker agent.

The agent can:

* Understand user requests
* Break complex tasks into steps
* Decide which tools to use
* Execute actions
* Recover from tool failures
* Continue working until the task is completed

---

### 🛠️ Tool Calling

Sidekick has access to multiple types of tools.

#### Built-in Tools

* 🔎 Google search using Google Serper
* 📚 Wikipedia lookup
* 📱 Push notifications using Pushover
* 🙋 Human assistance requests

#### MCP Tools

Sidekick also connects to Model Context Protocol (MCP) servers.

Currently included:

* 🌐 **Playwright MCP** — browser interaction
* 📁 **Filesystem MCP** — sandbox file operations

This allows the agent to interact with external environments rather than being limited to text generation.

---

### 🌐 Browser Automation

Sidekick can use **Playwright MCP** to interact with websites.

The browser runs as an MCP server and exposes browser capabilities as tools that the agent can call.

For example, the agent can:

* Navigate websites
* Read web pages
* Search for information
* Interact with web applications
* Perform multi-step browser workflows

The browser is configured in isolated mode for the Sidekick session.

---

### 📂 Sandbox Filesystem

Sidekick has access to a dedicated sandbox directory.

```text
project/
└── sandbox/
```

The filesystem MCP server is restricted to this directory, allowing the agent to work with files without giving it unrestricted access to the entire machine.

---

### 📋 Live Task Planning

Sidekick uses `TodoListMiddleware` to maintain a live task plan.

The Gradio interface continuously displays the agent's current TODO list:

```text
Plan

✓ Search for the required information
✓ Compare the results
◯ Prepare the final response
```

This makes the agent's progress visible while it works.

---

### 🔍 Self-Evaluation Loop

One of the main features of Sidekick is its **evaluator loop**.

After the worker finishes an attempt, a separate evaluator checks whether the response satisfies the user's success criteria.

```text
             ┌───────────────┐
             │ Worker Agent  │
             └───────┬───────┘
                     ↓
               Execute Task
                     ↓
              Generate Answer
                     ↓
             ┌───────────────┐
             │   Evaluator   │
             └───────┬───────┘
                     ↓
             Success Criteria?
                ↙         ↘
             YES           NO
              ↓             ↓
            Finish        Feedback
                            ↓
                         Retry
```

Sidekick allows up to **3 attempts** before returning the best available result.

The evaluator considers:

* The original user request
* The success criteria
* Tools used by the agent
* The agent's latest response

---

### 🔄 Automatic Recovery

Tool calls can occasionally fail.

Instead of crashing the entire agent, Sidekick uses custom middleware to convert tool errors into messages that the model can understand.

```text
Tool Failure
     ↓
TolerateToolErrors
     ↓
Error converted to ToolMessage
     ↓
Agent receives feedback
     ↓
Agent tries another approach
```

This makes the agent more resilient to real-world tool failures.

---

### 🛡️ Safety Middleware

Sidekick includes several middleware components.

#### PII Protection

`PIIMiddleware` is used to detect and handle sensitive information such as:

* Email addresses
* Credit card information

Credit card protection is also applied to tool results.

---

### 💰 Model Call Limits

`ModelCallLimitMiddleware` limits the number of model calls during an agent execution.

Current configuration:

```text
Maximum model calls per run: 30
```

This helps prevent runaway agent loops and unnecessary API usage.

---

### 🙋 Human-in-the-Loop

Some actions should not happen automatically.

Sidekick uses `HumanInTheLoopMiddleware` for sensitive operations such as:

* Sending push notifications
* Requesting human assistance

When approval is required, the agent pauses:

```text
Agent
  ↓
Sensitive Action
  ↓
Human Approval Required
  ↓
⏸ Sidekick Paused
  ↓
User Approves
  ↓
Agent Continues
```

The Gradio UI exposes an:

**"Approve and continue"**

button whenever the agent is paused.

---

# 🧩 Architecture

The project follows a layered agentic architecture.

```text
                    ┌─────────────────────┐
                    │      Gradio UI      │
                    │                     │
                    │ Chat + Plan +       │
                    │ Controls            │
                    └──────────┬──────────┘
                               │
                               ↓
                    ┌─────────────────────┐
                    │      Sidekick       │
                    │                     │
                    │ Task Management     │
                    │ Evaluator Loop      │
                    │ Human Approval      │
                    └──────────┬──────────┘
                               │
                               ↓
                    ┌─────────────────────┐
                    │ LangChain Agent     │
                    │ create_agent        │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              ↓                ↓                ↓
        Built-in Tools     MCP Tools       Middleware
              │                │                │
              │                │                ├─ PII
              │                │                ├─ HITL
              │                │                ├─ TODO
              │                │                └─ Call Limits
              ↓                ↓
       Google Serper       Playwright MCP
       Wikipedia           Filesystem MCP
       Pushover
```

---

# 🛠️ Technology Stack

| Technology         | Purpose                                  |
| ------------------ | ---------------------------------------- |
| **Python**         | Core programming language                |
| **LangChain**      | Agent and tool orchestration             |
| **LangGraph**      | Agent execution, state and checkpointing |
| **OpenAI Models**  | Agent reasoning and evaluation           |
| **MCP**            | Connecting agents with external tools    |
| **Playwright MCP** | Browser automation                       |
| **Filesystem MCP** | Sandbox file operations                  |
| **Gradio**         | Web-based user interface                 |
| **Google Serper**  | Web search                               |
| **Wikipedia API**  | Knowledge lookup                         |
| **Pushover**       | Push notifications                       |
| **Pydantic**       | Structured evaluator output              |
| **python-dotenv**  | Environment variable management          |

---

# 📁 Project Structure

```text
Sidekick/
│
├── app.py                 # Gradio application and UI
│
├── sidekick.py            # Core Sidekick agent and evaluator loop
│
├── sidekick_tools.py      # Built-in tools and MCP connections
│
├── styles.py              # UI theme and custom styling
│
├── sandbox/               # Agent-accessible filesystem
│
├── community_contributions/
│                          # Additional community contributions
│
├── requirements.txt       # Python dependencies (if used)
├── pyproject.toml         # Project configuration (if used)
├── .env                   # Environment variables
└── README.md              # Project documentation
```

---

# ⚙️ How It Works

## 1. Application Startup

When the application starts, the Gradio interface initializes the Sidekick.

```python
sidekick = Sidekick()
await sidekick.setup()
```

The setup process:

1. Creates the sandbox directory
2. Starts MCP sessions
3. Loads MCP tools
4. Creates the LangChain agent
5. Configures middleware
6. Initializes the evaluator

---

## 2. User Sends a Task

The user provides:

```text
Your request to the Sidekick
```

and optionally:

```text
What are your success criteria?
```

The request is passed to the Sidekick.

---

## 3. Agent Executes the Task

The worker is created using:

```python
create_agent(
    model="openai:gpt-5.4-mini",
    tools=self.tools,
    system_prompt=WORKER_PROMPT,
    middleware=[...],
    checkpointer=self.memory,
)
```

The agent can then select the appropriate tools to complete the task.

---

## 4. Evaluator Checks the Result

After the worker finishes, the evaluator receives:

* User request
* Success criteria
* Tools used
* Latest assistant response

The evaluator returns structured output:

```python
class EvaluatorOutput(BaseModel):
    feedback: str
    success_criteria_met: bool
    user_input_needed: bool
```

---

## 5. Retry if Necessary

If the criteria aren't met, Sidekick sends the evaluator's feedback back to the worker.

The worker then gets another opportunity to complete the task.

Maximum attempts:

```python
MAX_ATTEMPTS = 3
```

---

# 🧠 LangChain + LangGraph

Sidekick demonstrates how **LangChain and LangGraph can work together to build an agentic application**.

### LangChain

LangChain is primarily used for:

* Agent creation
* Model integration
* Tool integration
* Middleware
* Structured output

The main worker is created with:

```python
create_agent(...)
```

### LangGraph

LangGraph provides the underlying execution and state-management capabilities used by the agent.

Sidekick also uses:

```python
InMemorySaver()
```

as the checkpointer.

A unique thread ID is created for every Sidekick instance:

```python
self.sidekick_id = str(uuid.uuid4())
```

This allows the agent's execution state to persist during the session.

---

# 🔌 Model Context Protocol (MCP)

MCP allows Sidekick to connect an AI agent to external capabilities through standardized tool interfaces.

Sidekick currently uses:

### Playwright MCP

Used for browser automation.

```text
AI Agent
   ↓
MCP Client
   ↓
Playwright MCP Server
   ↓
Browser
```

### Filesystem MCP

Used for interacting with the Sidekick sandbox.

```text
AI Agent
   ↓
MCP Client
   ↓
Filesystem MCP Server
   ↓
sandbox/
```

This architecture makes it possible to add additional MCP servers and capabilities later.

---

# 🔐 Environment Variables

Create a `.env` file in the project root.

Example:

```env
OPENAI_API_KEY=your_openai_api_key

SERPER_API_KEY=your_serper_api_key

PUSHOVER_TOKEN=your_pushover_token

PUSHOVER_USER=your_pushover_user
```

**Never commit your `.env` file or API keys to GitHub.**

Add this to `.gitignore`:

```gitignore
.env
__pycache__/
*.pyc
sandbox/
```

---

# 📦 Installation

## Prerequisites

Make sure you have:

* Python 3.11+
* Node.js
* npm
* Git
* An OpenAI API key
* Google Serper API key
* Pushover credentials if push notifications are required

Playwright MCP and the filesystem MCP server are installed through `npx`.

---

## Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git

cd YOUR_REPOSITORY
```

---

## Install Python Dependencies

If using `uv`:

```bash
uv sync
```

Then run:

```bash
uv run app.py
```

Alternatively, if using pip:

```bash
pip install -r requirements.txt
```

Then:

```bash
python app.py
```

---

# ▶️ Running Sidekick

Start the application:

```bash
uv run app.py
```

Gradio will start the web interface.

The application contains:

* 💬 Chat interface
* 📋 Live task plan
* 📝 Request input
* 🎯 Success criteria
* ▶️ Go button
* 🔄 Reset button
* 🙋 Approval workflow

---

# 🖥️ User Interface

The Sidekick interface is designed around a simple workflow:

```text
┌─────────────────────────────────────────────┐
│                  Sidekick                   │
│          Your personal co-worker            │
├────────────────────────────┬────────────────┤
│                            │                │
│        Chat                 │      Plan      │
│                            │                │
│                            │   ✓ Task 1     │
│                            │   ✓ Task 2     │
│                            │   ◯ Task 3     │
│                            │                │
├────────────────────────────┴────────────────┤
│ Your request to the Sidekick                │
├─────────────────────────────────────────────┤
│ What are your success criteria?             │
├─────────────────────────────────────────────┤
│ Reset       Approve and continue      Go!   │
└─────────────────────────────────────────────┘
```

---

# 🎯 Example Use Cases

Sidekick can be extended for many agentic workflows.

### 🌐 Web Research

```text
Find the latest information about a topic,
compare multiple sources, and summarize the findings.
```

### ✈️ Travel Research

```text
Find flights between two destinations,
compare options, and summarize the best choices.
```

### 📁 File Operations

```text
Analyze the files in the sandbox and prepare
a summary of the important information.
```

### 🔎 Information Gathering

```text
Research a topic using web search and Wikipedia,
then provide a concise report.
```

### 📱 Notifications

```text
Complete the task and send me a push notification
when it is finished.
```

---

# 🧪 Design Philosophy

Sidekick is built around several principles:

### 1. Don't just answer — act

The agent should use tools and perform actions whenever necessary.

### 2. Make progress visible

The live TODO panel allows users to see what the agent is working on.

### 3. Evaluate instead of blindly trusting

A separate evaluator checks whether the agent actually satisfied the requested criteria.

### 4. Retry intelligently

When the result isn't good enough, feedback is sent back to the worker.

### 5. Keep humans in control

Sensitive actions can pause execution and require explicit approval.

### 6. Limit runaway execution

Model-call limits and maximum retry attempts prevent uncontrolled loops.

---

# 🚧 Future Improvements

Potential extensions include:

* 🧠 Persistent long-term memory
* 🗄️ Database-backed checkpoints
* 👥 Multiple specialized agents
* 🔌 Additional MCP servers
* 📊 Agent execution analytics
* 🔐 More advanced security policies
* 🧪 Automated agent evaluation
* 🌍 Multi-user support
* ☁️ Cloud deployment
* 📱 Improved mobile experience
* 🔄 Background task execution

---

# 🤝 Contributing

Contributions are welcome!

If you'd like to improve Sidekick:

1. Fork the repository
2. Create a new branch

```bash
git checkout -b feature/my-feature
```

3. Make your changes
4. Commit your changes

```bash
git commit -m "Add new feature"
```

5. Push your branch

```bash
git push origin feature/my-feature
```

6. Open a Pull Request

---

# 📜 License

This project is intended for educational and experimental purposes.

Add an appropriate license file to the repository if you plan to distribute the project publicly.

---

# ⭐ Acknowledgements

Built using the modern agentic AI ecosystem around:

* LangChain
* LangGraph
* Model Context Protocol
* Gradio
* OpenAI
* Playwright

---

## 💡 The Big Idea

> **Sidekick isn't just a chatbot. It's an AI worker that can plan, act, evaluate, retry, and ask for human approval when needed.**

The project demonstrates a practical approach to building **tool-using, self-evaluating, human-in-the-loop AI agents** using LangChain, LangGraph, and MCP.
