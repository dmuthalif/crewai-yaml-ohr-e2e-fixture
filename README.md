# crewai-yaml-ohr-e2e-fixture

A standard CrewAI **YAML-config** Crew for the DigitalOcean Managed Agents
(OHR) `crewai` template. It is the unmodified output of
`crewai create crew e2e_yaml --classic` (crewai 1.15.11): a `@CrewBase` class
in `src/e2e_yaml/crew.py` with agents and tasks in `src/e2e_yaml/config/*.yaml`.

## Agent spec

```yaml
name: crewai-yaml-demo
agent: crewai
size: mars-2vcpu-4gb
template: crewai
secrets:
  OPENAI_API_KEY: "${DO_TOKEN}"            # DO API token; DO inference is OpenAI-compatible
env:
  OPENAI_BASE_URL: "https://inference.do-ai.run/v1"
  MODEL: "openai-gpt-oss-120b"
  CREWAI_DISABLE_TELEMETRY: "true"
  FRAMEWORK_REPO: "https://github.com/dmuthalif/crewai-yaml-ohr-e2e-fixture.git"
  FRAMEWORK_REPO_SHA: "<full 40-char commit SHA of this repo>"
  FRAMEWORK_OBJECT: "e2e_yaml.crew:E2EYaml"
  FRAMEWORK_INPUT_PROMPT_KEY: "topic"
  FRAMEWORK_INPUT_DEFAULTS: '{"current_year": "2026"}'
```

```bash
doctl agents start --spec agent.yaml
doctl agents prompt <session> "AI agents"
```

The team needs the `open_harness_crewai` flipper enabled.

## Why the last three env vars are required

A stock scaffold does not run without them:

| Setting | Without it |
|---|---|
| `FRAMEWORK_OBJECT` | The scaffold's `pyproject.toml` has `[tool.crewai] type = "crew"` but no `crew = "module:attr"`, so boot preflight fails: `FRAMEWORK_OBJECT is required (module:attr) ...` |
| `FRAMEWORK_INPUT_PROMPT_KEY` | The prompt text is passed as `{input}`, but the YAML tasks use `{topic}`, so the kickoff fails: `Template variable 'topic' not found in inputs dictionary` |
| `FRAMEWORK_INPUT_DEFAULTS` | `tasks.yaml` also uses `{current_year}`, which the prompt does not supply |

The mediator instantiates the `@CrewBase` class and calls its `crew()` method.
It installs dependencies from `pyproject.toml` at boot (`pip install -e .`).

Verified on production on 2026-09-28: the kickoff for topic "AI agents"
completed in about 3 minutes with `run.completed`.
