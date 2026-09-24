# Did You Know You Can Program Language Models?

## Language Model Programming: From Calling an LLM to Building AI Programs

Most people meet large language models through a chat box: write a prompt, receive an answer.

Developers can use them differently.

A language model can become a callable component inside a normal program. Python can still provide `if`, `for`, `while`, functions, classes, databases, APIs and deterministic business rules. But some functions can delegate semantic work to a language model: classify text, extract structured information, generate a plan, critique a result, select a tool, or even write the next piece of executable code.

This family of techniques is often described using terms such as **Language Model Programming**, **LLM orchestration**, **AI functions**, **tool-using agents**, and **code agents**.

The important shift is:

```
Traditional program:
input -> deterministic functions -> output

LLM-enabled program:
input -> code -> model call -> code -> model/tool call -> validation -> output

Code agent:
goal -> model writes actions/code -> execution -> observation -> model revises plan -> output
```

The LLM does not replace the programming language. It becomes another computational primitive inside it.

---

## 1. The Basic Primitive: an AI Function

Conceptually, imagine a function such as:

```python
result = run_model(
    prompt="Extract the important decisions from this document",
    model="chosen-model",
    temperature=0.2,
)
```

Now ordinary programming constructs can control model calls:

```python
for document in documents:
    summary = summarize(document)
    review = review_summary(document, summary)

    if not review.accepted:
        summary = revise(document, summary, review.issues)
```

The control flow is ordinary programming. The semantic operations inside `summarize`, `review_summary`, and `revise` may be performed by language models.

This creates a new design space: programs whose components are partly deterministic and partly probabilistic.

---

## 2. Model Gateway and Unified Invocation

### LiteLLM

**Idea:** provide a common interface for calling models from different providers.

Instead of rewriting an application every time the model provider changes, a gateway layer can normalize model invocation, routing, retries and related infrastructure.

Think of this as the lowest layer:

```
application
    |
AI function
    |
model gateway
    |
OpenAI / Anthropic / Gemini / local or other model
```

**What to learn:** model abstraction, routing, fallbacks, retries, budgets, provider differences and observability.

---

## 3. AI Calls as Normal-Looking Functions

### Mirascope

Mirascope explores a developer experience in which model calls can be expressed as Python functions. A function defines the inputs and prompt behavior, while the framework handles the model interaction.

The conceptual benefit is important: an LLM operation can be treated as a reusable software component rather than a loose prompt copied around an application.

**What to learn:** function-oriented LLM calls, prompt templates, tools, structured responses and reusable components.

---

## 4. Typed AI Programs

### Pydantic AI

Free-form text is difficult to compose into reliable software.

A better interface is often:

```
extract_tasks(text) -> list[Task]
classify_request(text) -> Classification
review_report(report) -> ReviewResult
```

Typed frameworks allow developers to define the expected shape of a model result and validate it before the rest of the application uses it.

This does **not** prove that the model's factual conclusion is correct. It proves that the returned data satisfies the required structure. Semantic verification remains a separate problem.

**What to learn:** schemas, validation, structured outputs, dependency injection, tools and agent interfaces.

---

## 5. Programming Rather Than Hand-Writing Every Prompt

### DSPy

DSPy treats LM behavior as programmable modules with signatures, composition and optimization.

Instead of thinking only in terms of:

> "What exact prompt should I write?"

the developer can think in terms of:

> "What are the inputs, outputs, modules, examples and evaluation metric for this program?"

That distinction is fundamental. Once an AI system has a measurable objective, parts of its implementation can be optimized against examples rather than adjusted only by intuition.

Topics worth studying include:

- signatures and modules;
- composition of LM operations;
- evaluation datasets and metrics;
- optimizers;
- model selection and configuration;
- experimental approaches where models use code and additional model calls while solving complex tasks.

**Core lesson:** AI programming increasingly includes both writing a program and defining how its quality will be measured.

---

## 6. Code Agents: Let the Model Express Actions as Code

### Hugging Face smolagents

A conventional tool-calling agent may return something like a JSON instruction:

```
{"tool": "search", "query": "language model programming"}
```

A **CodeAgent** can instead express its actions as Python code.

That matters because code naturally supports:

- variables;
- loops;
- conditions;
- composition;
- reusable functions;
- intermediate objects;
- multiple tool calls;
- calculations.

In smolagents, `CodeAgent` formulates tool calls in code and executes them. The framework also supports different code execution environments.

Conceptually:

```python
results = []

for source in sources:
    analysis = analyze(source)

    if needs_verification(analysis):
        evidence = search_for_evidence(analysis)
        analysis = revise(analysis, evidence)

    results.append(analysis)
```

The model may generate the action program; the execution environment performs it.

### Security warning

Generated code must not automatically receive unrestricted access to the host machine, secrets, files or production systems. Code-agent systems require sandboxing, allow-listed tools, resource limits and explicit permission boundaries.

**What to learn:** CodeAgent, tools, multi-step agents, planning, execution environments, sandboxing and human approval.

---

## 7. A Language Specifically for AI Functions

### BAML

BAML is especially interesting because it demonstrates that AI behavior can be described in a dedicated language rather than embedded entirely in Python.

A BAML function contains:

- typed input parameters;
- a return type;
- an LLM client;
- a prompt.

Conceptually:

```baml
function ExtractTasks(text: string) -> Task[] {
    client SomeModel

    prompt #"
        Extract actionable tasks from:
        {{ text }}
    "#
}
```

The application can then call the generated client from a conventional programming language.

This is an important architectural pattern:

```
AI definition language
        |
generated client
        |
Python / TypeScript / other application code
        |
LLM
```

**What to learn:** typed AI functions, model configuration, prompts as program assets, generated clients and separation between AI definitions and application logic.

---

## 8. Language Model Query Languages

### LMQL

LMQL explores the idea that interacting with a language model can itself be expressed as a programming language.

Rather than treating generation as a single opaque API call, a language can combine prompting with program logic and constraints over generated values.

This makes LMQL important conceptually even when a project ultimately uses ordinary Python: it demonstrates that **model generation can be treated as a programmable execution process**.

**What to learn:** prompt programming, variables, constraints, decoding and the relationship between natural-language instructions and formal program structure.

---

## 9. Workflow and State Machines for AI Systems

### LangGraph

A model call is easy. A long-running AI workflow is harder.

Real applications may need to:

- preserve state;
- branch;
- retry;
- pause for human approval;
- call tools;
- resume after interruption;
- coordinate multiple agents;
- persist intermediate results.

Graph/workflow runtimes such as LangGraph address this orchestration layer.

Think of the difference this way:

```
LLM API          = one intelligence operation
AI function      = reusable semantic operation
agent            = chooses actions
workflow runtime = controls the lifecycle of the whole process
```

**What to learn:** state, nodes, edges, checkpoints, persistence, human-in-the-loop and durable execution.

---

## 10. The Stack

These systems are not necessarily competitors. They often occupy different layers.

```
+--------------------------------------------------+
| Application / Product                            |
+--------------------------------------------------+
| Workflow / State / Orchestration                 |
| LangGraph and related runtimes                   |
+--------------------------------------------------+
| Agents / Code-generated actions                  |
| smolagents and related agent systems             |
+--------------------------------------------------+
| AI program composition / optimization            |
| DSPy                                             |
+--------------------------------------------------+
| Typed AI functions                               |
| Pydantic AI / BAML / Mirascope-style interfaces  |
+--------------------------------------------------+
| Model gateway / provider abstraction             |
| LiteLLM and direct provider SDKs                 |
+--------------------------------------------------+
| Foundation models                                |
+--------------------------------------------------+
```

LMQL and BAML are particularly useful when studying how the **language itself** can change to represent AI operations.

---

## 11. Three Levels of AI Programming

### Level 1 — The program calls the model

The developer writes the algorithm.

```python
for item in items:
    result = ai_function(item)
```

### Level 2 — The model chooses tools

The developer defines the available capabilities. The model chooses which capability to invoke and with what arguments.

```
Goal -> model -> tool -> observation -> model -> tool -> answer
```

### Level 3 — The model writes the action program

The developer defines the environment, permissions and available primitives. The model generates code that composes those primitives.

```
Goal
  -> model writes code
  -> sandbox executes code
  -> observations return to model
  -> model modifies strategy
  -> verified result
```

This third level is one of the most interesting directions for AI-native software.

---

## 12. What Changes in Software Engineering?

Traditional programming assumes that a function's implementation is explicitly specified by the programmer.

AI-native programming introduces functions whose internal behavior may be produced by a model.

That changes what the developer must engineer.

The important artifacts become:

1. **Function contract** — what goes in and what must come out.
2. **Model policy** — which model is allowed to perform the operation.
3. **Context** — what information the model can see.
4. **Tools** — what external actions it can perform.
5. **Constraints** — what it must never do.
6. **Budget** — calls, tokens, time and money.
7. **Evaluation** — how quality is measured.
8. **Verification** — how incorrect output is detected.
9. **Permissions** — what actions require human approval.
10. **Observability** — what happened during execution.
11. **Versioning** — which prompt/program/model produced a result.
12. **Sandboxing** — where generated code is allowed to execute.

In other words, the programmer increasingly designs an **execution environment for intelligence**.

---

## 13. Important Distinction: AI Functions Are Probabilistic

A normal function might guarantee:

```
2 + 2 -> 4
```

An AI function is different:

```
review(document) -> probable judgment
```

The same semantic task may produce different outputs across models, model versions, prompts or runs.

Therefore serious AI programs need explicit evaluation and verification.

Useful techniques include:

- typed outputs;
- deterministic validation rules;
- source-grounded checks;
- test datasets;
- model-independent assertions;
- confidence thresholds where appropriate;
- independent review stages;
- human approval for high-impact actions;
- logging and replay.

---

## 14. A Practical Learning Experiment

Build the same task three ways:

**Task:** Given a collection of source documents, produce a verifiable brief.

### Version A — Explicit Python workflow

The developer writes the complete control flow and calls AI functions at predetermined points.

### Version B — Code agent

Provide tools such as:

```
read_source()
analyze_source()
search_evidence()
review_claim()
write_brief()
```

Let the code agent decide how to combine them.

### Version C — AI program / optimization framework

Represent the task as composable LM modules, define examples and an evaluation metric, and test whether the program can be improved systematically.

Measure:

- factual consistency with sources;
- omissions;
- unsupported claims;
- task completion rate;
- number of model calls;
- token usage;
- latency;
- monetary cost;
- reproducibility;
- human corrections required.

Do not judge the systems only by which answer "looks smarter."

---

## 15. Suggested Study Order

1. Call one model from Python.
2. Turn the call into a reusable AI function.
3. Require structured typed output.
4. Compose several AI functions with normal Python control flow.
5. Add tools.
6. Build a tool-using agent.
7. Build a code agent in a sandbox.
8. Add state and durable workflow execution.
9. Create an evaluation dataset.
10. Optimize the AI program against measurable criteria.
11. Study BAML and LMQL as examples of AI-oriented language design.
12. Design your own abstraction only after understanding what existing systems already solve.

---

## 16. Systems to Explore

| System | Main idea | Study it for |
|---|---|---|
| LiteLLM | Unified model interface and gateway | Multi-model invocation and routing |
| Mirascope | Pythonic LLM functions | AI calls as reusable functions |
| Pydantic AI | Typed agent/application framework | Structured outputs and validation |
| DSPy | Programming and optimizing LM systems | Composable AI programs and evaluation |
| smolagents | Agents that can express actions as code | Code agents and tool composition |
| BAML | Dedicated language for typed AI functions | AI-specific language design |
| LMQL | Language/model query programming | Constraints and programmable generation |
| LangGraph | Stateful agent/workflow runtime | Long-running orchestration and persistence |

---

## 17. The Big Idea

The interesting question is no longer only:

> "How do I write a better prompt?"

A more powerful question is:

> **"How do I build a program in which language models are first-class computational components?"**

And then:

> **"Can a language model itself construct part of that program while operating inside a controlled environment?"**

The answer to both is already **yes**.

The emerging engineering discipline combines conventional programming, language-model calls, typed contracts, tools, agents, generated code, workflow runtimes, evaluation and security boundaries.

That is **Language Model Programming** in the broad practical sense.

---

## Primary documentation

- DSPy — https://dspy.ai/
- Hugging Face smolagents — https://huggingface.co/docs/smolagents/
- BAML — https://docs.boundaryml.com/
- LMQL — https://lmql.ai/
- Pydantic AI — https://ai.pydantic.dev/
- Mirascope — https://mirascope.com/docs/
- LiteLLM — https://docs.litellm.ai/
- LangGraph — https://docs.langchain.com/oss/python/langgraph/

_Last reviewed: 2026-09-24. APIs and experimental agent frameworks evolve quickly; verify current documentation before implementation._
