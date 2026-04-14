# Python AI Strands Agent Template (`python_ai_strands_template`)

A Copier template for bootstrapping production-ready AI agent projects using [Strands Agents SDK](https://github.com/strands-agents/sdk-python) with AWS Bedrock, Streamlit UI, and a modern Python toolchain: `uv`, `ruff`, tests, docs, Docker, and releases.

Built from patterns extracted from real-world Strands agent projects in the AI Arena ecosystem (649, 887, 999, 1128).

## Why this template
- Start fast with an **orchestrator + sub-agents** architecture out of the box.
- Includes working examples of `@tool` decorated functions, agent factories, and parallel execution.
- Streamlit chat UI with session management ready to go.
- pydantic-settings for configuration with `.env` support.
- Keep quality automated with linting, formatting, and tests.
- Docker support for containerized deployment.

## Technology stack
- [Strands Agents SDK](https://github.com/strands-agents/sdk-python) for building AI agents.
- [AWS Bedrock](https://aws.amazon.com/bedrock/) for LLM inference (Claude).
- [Streamlit](https://streamlit.io/) for the web chat UI.
- [pydantic-settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/) for typed configuration from environment variables.
- [Copier](https://copier.readthedocs.io/) for project scaffolding and updateable generation.
- [uv](https://docs.astral.sh/uv/) for dependency management, virtual environments, lockfiles, and packaging.
- [ruff](https://docs.astral.sh/ruff/) for linting and formatting.
- `pre-commit` to run hooks before commits.
- `pytest` and `pytest-cov` for tests and coverage.
- [MkDocs](https://www.mkdocs.org/) for documentation, themed with Material.
- [Docker](https://www.docker.com/) for containerized builds.
- [Commitizen](https://commitizen-tools.github.io/commitizen/) for Conventional Commits, versioning, and changelog automation.
- `AGENTS.md.jinja` to generate project-specific coding-agent instructions.

## Quick start

### 1. Create the project folder

```bash
mkdir -p <project_name>
cd <project_name>
```

### 2. Install [`uv`](https://github.com/astral-sh/uv)

Installation instructions are [here](https://docs.astral.sh/uv/getting-started/installation/).

### 3. Create the project using [copier](https://github.com/copier-org/copier):

```bash
uvx copier copy https://github.com/<your-username>/python_ai_strands_template.git .
```

> [!IMPORTANT]
> Copier always generates a `.copier-answers.yml` file. Commit the file with the other files and **never** change it manually.

### 4. Setup and first push

```bash
git init --initial-branch=main
cp .env.example .env
# Edit .env with your AWS credentials
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

# CLI mode
uv run python main.py "Hello, agent!"
```

## Generated project structure

```
your-project/
├── app.py                          # Streamlit chat UI entry point
├── main.py                         # CLI entry point
├── pyproject.toml                  # Dependencies and tool config
├── Makefile                        # Build targets
├── .env.example                    # Environment variable template
├── src/<package>/
│   ├── config/
│   │   ├── settings.py             # pydantic-settings (AWS, LLM, session)
│   │   ├── aws_client.py           # Bedrock session & AWS client factory
│   │   └── prompts.py              # System prompts for all agents
│   ├── agents/
│   │   ├── orchestrator.py         # Main agent with parallel sub-agent execution
│   │   └── example_agent.py        # Starter sub-agent factory
│   ├── tools/
│   │   └── example_tools.py        # @tool decorated example functions
│   └── ui/
│       └── components.py           # Reusable Streamlit components
├── tests/
│   ├── conftest.py                 # Shared fixtures (mocked AWS creds)
│   ├── test_example_tools.py       # Tool tests using .__wrapped__()
│   └── test_settings.py            # Settings tests
├── docker/
│   ├── Dockerfile                  # Multi-stage build with uv
│   └── docker-compose.yml          # Service definition
├── docs/                           # MkDocs documentation
├── scripts/                        # Build helper scripts
└── data/                           # Runtime data (gitignored)
```

## Update an existing project

```bash
cd your-project
uvx copier update --defaults
```

Resolve any conflicts and commit the changes.
