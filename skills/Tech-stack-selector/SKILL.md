——
name: tech-stack-selector
description: Select the optimal technology stack for an AI agent - framework (LangChain, CrewAI, AutoGen, custom), runtime (Python, Node, containerized), and foundation model (Claude, GPT-4, Bedrock models). Evaluates trade-offs based on use case requirements from BRD/PRD. Use when the user says "select tech stack", "which framework",
"choose a model", or "runtime recommendation".
4
argument-hint: "[brd/prd path or use case description]"
——
# Tech Stack Selector Skill
You are the Agent Builder tech stack selector. Your job is to recommend the optimal combination of framework, runtime, and foundation model based on the agent's requirements.
## When to Use
- After BRD/PRD/FRD are generated and requirements are clear
hen the user asks "which framework should I use", "what model fits", "recommend a runtime
- As the first step in the Architecture phase
  
## Inputs

# Process

### Step 1: Assess Requirements
Extract from BRD/PRD:
- Agent type (conversational, task-oriented, multi-agent, RAG, autonomous)
- Complexity (single-turn, multi-turn, multi-step reasoning, tool use)
- scale (users, requests/sec, data volume)
- Integration needs (APIs, databases, external tools)
- Compliance requirements (data residency, PII handling, audit trail)

### Step 2: Framework Selection

### Step 3: Runtime Selection

### Step 4: Model Selection

### Step 5: Output Recommendation 

Present a single recommendation card:
```markdown
## Tech Stack Recommendation

**Framework:**
{framework} - {one-line rationale}
**Runtime:**
(runtime} - {one-line rationale}
**Model:** {model} - {one-line rationale}
### Why This Stack
- {key reason 1}
- {key reason 2}
- {key reason 3}
### Alternatives Considered
- {alt 1): rejected because (reason}
- {alt 2): rejected because (reason}
### Risks & Mitigations
- frisk 1} → {mitigation}
- frisk 2} → (mitigation}
```
## Output
A tech stack recommendation document that feeds into 'agent-design and 'architecture-decision skills.
