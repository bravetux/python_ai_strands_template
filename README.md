# Python AI Strands Agent Template (`python_ai_strands_template`)

A Copier template for bootstrapping production-ready AI agent projects using the [Strands Agents SDK](https://github.com/strands-agents/sdk-python) with AWS Bedrock, a Streamlit chat UI, and a modern Python toolchain: `uv`, `ruff`, tests, docs, Docker, and releases.

Built from patterns extracted from real-world Strands agent projects in the AI Arena ecosystem.

## Why this template
- Start fast with an **orchestrator + specialist sub-agents** architecture out of the box.
- Includes working examples of `@tool` decorated functions, agent factories, and parallel sub-agent execution.
- Streamlit chat UI with session management ready to go.
- Keep quality automated with linting, formatting, type checking, and tests.
- Publish docs and releases with built-in helper scripts and Make targets.

## Technology stack
- [Strands Agents SDK](https://github.com/strands-agents/sdk-python) for building AI agents with tool use, conversation management, and session persistence.
- [AWS Bedrock](https://aws.amazon.com/bedrock/) for LLM inference (Claude).
- [Streamlit](https://streamlit.io/) for the web chat UI.
- [pydantic-settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/) for typed configuration from environment variables.
- [Copier](https://copier.readthedocs.io/) for project scaffolding and updateable generation.
- [uv](https://docs.astral.sh/uv/) for dependency management, virtual environments, lockfiles, and packaging via `uv_build`.
- [ruff](https://docs.astral.sh/ruff/) for linting and formatting.
- `pre-commit` to run hooks before commits.
- `pytest` and `pytest-cov` for tests and coverage.
- [MkDocs](https://www.mkdocs.org/) for documentation, themed with Material and extended via mkdocstrings for API docs, mkdocs-gen-files for generated pages, mkdocs-literate-nav for Markdown-driven navigation, mkdocs-section-index for clickable section indexes, mkdocs-autorefs for cross-page references, pymdown-extensions for richer Markdown, and mike for versioned docs publishing.
- [Docker](https://www.docker.com/) for containerized builds.
- [Commitizen](https://commitizen-tools.github.io/commitizen/) for Conventional Commits, versioning, and changelog automation.
- `AGENTS.md.jinja` to generate a project-specific `AGENTS.md` during scaffolding and keep coding-agent instructions consistent across projects.

## Quick start

### 1. Create the project folder

```bash
mkdir -p <project_name>
cd <project_name>
```

### 2. Install [`uv`](https://github.com/astral-sh/uv)

Installation instructions are [here](https://docs.astral.sh/uv/getting-started/installation/).
It's recommended to install the latest version from [github releases](https://github.com/astral-sh/uv/releases).

If you have already installed `uv`, please ensure you're using the latest version by running `uv self update`.

### 3. Create the project using [copier](https://github.com/copier-org/copier):

Launch the following command and answer carefully to the prompts:

```bash
uvx copier copy https://github.com/<your-username>/python_ai_strands_template.git .
```

You will be prompted for:

| Prompt | Description |
|--------|-------------|
| `package_name` | Name of the Python AI agent package |
| `project_description` | Short description of the project |
| `github_username` | GitHub username or organization name |
| `python_version` | Python version constraint (`>=3.12`, `>=3.11`, or `>=3.10`) |
| `include_docker` | Whether to include Docker support |

> [!IMPORTANT]
> Copier always generates a `.copier-answers.yml` file. Commit the file with the other files and **never** change it manually.

### 4. Setup and first push

```bash
git init --initial-branch=main
cp .env.example .env
# Edit .env with your AWS credentials and Bedrock model ID
make install
git add .
git commit -m "feat: first commit"
git remote add origin <remote_repository_URL>
git push --set-upstream origin main
```

### 5. Run the agent

```bash
# Streamlit UI
make run

# CLI — single query
uv run python main.py "Hello, agent!"

# CLI — interactive REPL
uv run python main.py
```

## Generated project structure

```
your-project/
├── app.py                              # Streamlit chat UI entry point
├── main.py                             # CLI entry point (single query + REPL)
├── pyproject.toml                      # Dependencies and tool config
├── Makefile                            # Build targets
├── .env.example                        # Environment variable template
├── AGENTS.md                           # Coding-agent guidelines
├── src/<package>/
│   ├── config/
│   │   ├── settings.py                 # pydantic-settings (AWS, LLM, session)
│   │   ├── aws_client.py              # Bedrock session & AWS client factory
│   │   └── prompts.py                 # System prompts for all agents
│   ├── agents/
│   │   ├── orchestrator.py            # Main agent: parallel execution, hooks, session
│   │   └── example_agent.py           # Starter sub-agent factory
│   ├── tools/
│   │   └── example_tools.py           # @tool decorated example functions
│   └── ui/
│       └── components.py              # Reusable Streamlit components
├── tests/
│   ├── conftest.py                    # Shared fixtures (mocked AWS creds)
│   ├── test_example_tools.py          # Tool tests using .__wrapped__()
│   └── test_settings.py              # Settings loading tests
├── docker/
│   ├── Dockerfile                     # Multi-stage build with uv, non-root user
│   └── docker-compose.yml            # Service definition
├── docs/                              # MkDocs documentation sources
├── scripts/                           # Build helper scripts
└── data/                              # Runtime data (gitignored)
```

## Architecture

The generated project follows the **orchestrator + specialist sub-agents** pattern used across all AI Arena projects:

```
User Query
    │
    ▼
┌──────────────┐
│ Orchestrator  │  ← SummarizingConversationManager, FileSessionManager, audit hooks
│   Agent       │
└──┬───┬───┬───┘
   │   │   │
   ▼   ▼   ▼        ← ThreadPoolExecutor (parallel)
┌───┐┌───┐┌───┐
│ A ││ B ││ C │     ← SlidingWindowConversationManager, focused tools
└───┘└───┘└───┘
   │   │   │
   ▼   ▼   ▼
┌──────────────┐
│  Aggregated   │
│   Response    │
└──────────────┘
```

### Key patterns

- **Agent factories** — Each agent is created via a `create_*_agent()` function that returns a configured `Agent` instance.
- **Tool decorator** — Functions decorated with `@tool` from `strands` become tools the agent can call. Test them via `my_tool.__wrapped__(args)` to bypass the decorator.
- **Conversation managers** — Orchestrators use `SummarizingConversationManager` (long-running), sub-agents use `SlidingWindowConversationManager` (short-lived).
- **Session persistence** — `FileSessionManager` stores conversation state to disk across restarts.
- **Audit hooks** — `BeforeToolCallEvent` / `AfterToolCallEvent` hooks log every tool invocation.
- **Configuration** — `pydantic-settings` loads from `.env` with typed defaults and aliases.

## Adding a new sub-agent

1. **Create tools** in `src/<package>/tools/my_tools.py`:

   ```python
   from strands import tool

   @tool
   def fetch_data(query: str) -> dict:
       """Fetch data for the given query."""
       return {"result": "..."}
   ```

2. **Create agent** in `src/<package>/agents/my_agent.py`:

   ```python
   from strands import Agent
   from strands.agent.conversation_manager import SlidingWindowConversationManager
   from strands.models import BedrockModel
   from <package>.config.aws_client import get_bedrock_session
   from <package>.config.settings import settings
   from <package>.tools.my_tools import fetch_data

   def create_my_agent() -> Agent:
       model = BedrockModel(
           boto_session=get_bedrock_session(),
           model_id=settings.bedrock_model_id,
           temperature=settings.agent_temperature,
       )
       return Agent(
           model=model,
           tools=[fetch_data],
           conversation_manager=SlidingWindowConversationManager(window_size=10),
           system_prompt="You are a specialist agent for ...",
       )
   ```

3. **Register in orchestrator** — Import in `orchestrator.py`, add to `run_parallel_agents()`:

   ```python
   from <package>.agents.my_agent import create_my_agent

   def run_parallel_agents(query: str) -> dict:
       with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
           futures = {
               "example": executor.submit(create_example_agent(), query),
               "my_agent": executor.submit(create_my_agent(), query),
           }
           # ...
   ```

4. **Write tests** in `tests/test_my_tools.py` using `.__wrapped__()`.

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `AWS_ACCESS_KEY_ID` | | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | | AWS secret key |
| `AWS_SESSION_TOKEN` | | Optional: for SSO / STS temporary credentials |
| `AWS_DEFAULT_REGION` | `us-east-1` | AWS region |
| `BEDROCK_MODEL_ID` | `anthropic.claude-sonnet-4-20250514-v1:0` | Bedrock model to use |
| `AGENT_TEMPERATURE` | `0.1` | LLM temperature |
| `AGENT_TOP_P` | `0.9` | LLM top-p sampling |
| `AGENT_MAX_TOKENS` | `4096` | Max response tokens |
| `SESSION_BACKEND` | `file` | Session storage backend |
| `SESSION_DIR` | `.sessions` | Directory for file-based sessions |

## Makefile commands

| Command | Description |
|---------|-------------|
| `make install` | Installs dependencies and pre-commit hooks |
| `make sync` | Synchronizes dependencies |
| `make run` | Starts the Streamlit UI |
| `make lint` | Runs Ruff linting and format checking |
| `make tests` | Runs pytest with coverage |
| `make type-check` | Runs type annotation checking |
| `make build` | Builds the package |
| `make init-docs` | Initializes the documentation branch |
| `make github-pages` | Publishes documentation to GitHub Pages |
| `make github-tag` | Creates a version tag |
| `make print-version` | Prints the current version |
| `make docker-build` | Builds the Docker image |
| `make docker-up` | Starts the Docker container |

## Documentation

The project documentation is generated automatically by parsing the source code, provided that the code is properly commented with appropriate docstrings.

To initialize the documentation, run:
```bash
make init-docs
```

To publish the generated documentation to GitHub Pages, run:
```bash
make github-pages
```

## Update an existing project

1. Move inside your project and make sure that there are no local changes (in case you have local changes, commit or stash them).

2. Update your project to the latest Git tag of the template with the following command:
   ```bash
   uvx copier update --defaults
   ```
3. Resolve any conflicts and commit the changes.
