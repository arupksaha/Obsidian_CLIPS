# Agentic System Design Interview Prep: The Listening Edition

Prepared for Arup, from the Educative course "Agentic System Design," with interview framing, worked answers, and extra material added on top. Prepared November 2026.

## How to use this document

This document is written to be heard first. There are no tables, code blocks, or diagrams, because a text to speech voice turns those into noise. Every pattern is described in sentences, so you can follow along on a walk.

It runs about 85 minutes at normal speed. If you only have ten minutes, listen to Section 1, Section 2, and Section 12. After listening, load the file into NotebookLM as a source. Section 13 has prompts for that stage, and you can skip it while listening.

One rule for what you hear. Sections 3 through 8 come from the course, unless a sentence begins with "my note" or "my takeaway." Those are my opinion. Sections 1, 2, 9, 10, and 11 are my own synthesis and additions, built on the course, and Section 11 tells you where it uses the course's answers. Section 14 lists places where the course contradicts itself, and where you should check a number before quoting it.

A quick word on terms you will hear constantly. LLM means large language model. RAG means retrieval augmented generation, which fetches relevant documents and gives them to the model. PII means personally identifiable information.

## Section 1: What this interview is actually testing

An agentic system design interview looks like a normal system design interview with an LLM in the middle. It is not. In normal system design, the components behave deterministically and the hard part is scale. In agentic design, the most important component is probabilistic, it can take a different path on every run, and it can take actions in the world. So the hard part moves. It becomes control, verification, and blast radius.

Here is my read of what the interviewer is listening for. It is judgment, not research on hiring outcomes, though two published guides point the same way. One, on evaluating AI engineers, names problem decomposition, logical flow, error handling, prompt structure, and being able to justify where you drew each agent boundary. The other, a guide to Google's forward deployed engineer loop, describes an agentic and machine learning design round that looks at decomposition, fluency with model integration and RAG, trade offs among cost, latency, reliability, and scalability, and awareness of real customer deployment. It also warns that round names and order vary. Treat both as useful single sources, not as a standard, and ask your recruiter about the actual format.

I would listen for five signals.

First, restraint. Can you say that part of this should not be an agent? The strongest candidates draw a line between what stays deterministic and what needs reasoning.

Second, pattern selection with reasons. Naming ReAct is worth nothing. Saying "the environment changes under me, so a fixed plan goes stale, so I want a step by step loop, and I accept the extra tokens" is worth a lot.

Third, failure thinking. What happens when the model is wrong, the tool times out, the plan goes stale, the loop never ends, or the input contains an attack?

Fourth, measurement. How would you know it works? Task completion, not eloquence.

Fifth, the central thesis of the entire course: optimize the system, not just the model. MACRS, WebVoyager, and MuLan all gained their performance from architecture, not from a bigger model. If you say one sentence out loud in the interview, make it this one: "The LLM is a component, and my job is to design the controllable, observable, and secure environment it runs in."

## Section 2: The answer framework

Here is a framework for any prompt of the form "design an agent that does X." It is my synthesis of the course, built as eight questions. The memory hook is: Goal, Gate, Blast, Shape, Parts, Guard, Break, Prove.

Question one, Goal. What is the outcome, who is the user, and what number proves success? Ask about volume, latency expectations, data sensitivity, and whether actions are reversible. Do not skip the success metric, because it drives everything later.

Question two, Gate. Should this be an agent at all? Separate the steps. Which are fixed rules and which need judgment over unstructured input? Keep the fixed ones as plain code.

Question three, Blast. What is the worst thing that can happen if the agent is wrong? Sort actions into read only, reversible write, and irreversible. This sets your autonomy level. Autonomy is a design choice, and it can differ by action within the same system.

Question four, Shape. One agent or many, and which orchestration pattern? Start from the simplest thing that works, and add agents only when you can name the reason: distinct skills, parallel work, or isolation boundaries.

Question five, Parts. Walk the components. Model choice per step. Tools with clear schemas and least privilege. Instructions. Memory across short term, long term, and external knowledge. Say what enters the context window and what stays out.

Question six, Guard. Place guardrails at the input, the output, and before each tool call. Then decide where humans sit: in the loop, on the loop, or above the loop.

Question seven, Break. Walk the failure modes. Tool failures, stale plans, infinite loops, hallucinated success, cascading errors, partial completion. Then cover latency and cost: how many LLM calls per request, and what you would do to cut them.

Question eight, Prove. How do you evaluate before launch and monitor after? Simulation, a task benchmark, ablation tests that remove a component to prove its value, human review, then a staged rollout.

For a forty five minute interview, a reasonable time budget is this. Five minutes on Goal and Gate. Five on Blast. Ten on Shape and Parts at a high level. Ten going deep on the one or two hardest components. Ten on Guard and Break. Five on Prove. Let the interviewer interrupt you and steer. If they do, follow them, because that is where the points are.

For a forward deployed engineer interview in particular, I would spend extra time on Goal, Gate, and rollout, because customer deployment is what sets that role apart. Say who has to sign off, what data you cannot touch, and what the first narrow workflow is. Also expect more than design. The Google guide lists coding, a loosely specified build session, and a behavioral round with extra weight on working with customers and non engineers, so do not spend all your preparation here.

## Section 3: Foundations

### What an agent is

From the course: an agent perceives its environment, makes decisions, and acts autonomously to achieve goals. That perceive, reason, act framing goes back to classical AI, popularized by Russell and Norvig. Key characteristics are autonomy, goal oriented behavior, a perception and feedback loop, continuity through memory, and flexibility to revise plans.

The course's appendix adds memory and gives four core characteristics: perception, reasoning, action, and memory. If you are asked for the list, give those four.

### Model versus agent

A model computes an output from an input and stops. It has no goals and no initiative. It is often literally an artifact, a set of learned weights consumed by a program. An agent is a system that uses one or more models inside a larger decision process, keeps context over time, uses tools, and pursues a goal.

The course's analogies: a model is a calculator and an agent is a personal assistant. Or, a language model is an engine and an agent is the vehicle that plans the route and decides when to stop for fuel.

### Types of agents, two ways

The first classification is by environment. Software agents act through digital interfaces. Physical agents use sensors and actuators, and must handle uncertainty, delay, and safety. Hybrid agents span both, such as a warehouse robot that consults online inventory.

The second classification is by decision logic, moving from simple to sophisticated.

A simple reflex agent uses only the current input and fixed rules, like a spam filter.

A model based reflex agent keeps internal state about what it cannot currently see, like a car tracking a pedestrian who has passed out of sensor range.

A goal based agent evaluates actions against an objective, like a GPS.

A utility based agent scores outcomes and optimizes across competing priorities. Goal based accepts any route that arrives. Utility based picks the route that best trades off time, fuel, and risk.

A learning agent improves from experience.

A multi agent system distributes work across specialists in a shared environment.

A hierarchical agent system adds tiers, with strategic planning on top and narrow execution below.

Real systems combine these. An autonomous coding assistant is goal based, utility optimizing, learning capable, and multi agent all at once. The useful interview question is: which kinds of reasoning does this problem actually require?

### How we got here

Rule based systems such as MYCIN and XCON, in the 1970s and 80s, were controllable but could not generalize, and required thousands of hand written rules. Learning based agents moved rule authoring to data driven optimization. Reinforcement learning, with AlphaGo in 2016 as the famous example, learns through trial and reward, but needs massive training and structured environments. LLM powered agents bring what the course calls zero shot generalization. They handle novel tasks without retraining, because world knowledge and reasoning are already in the parameters.

The price is reliability, interpretability, bias, latency, and new security holes. Always name the price.

### When to build an agent

From the course, three signals that an agent may be justified.

One, the task requires contextual decision making, such as a refund decision that depends on customer history, tone, and reason for return.

Two, the rules are too complex to maintain, such as a vendor security review with evolving compliance requirements.

Three, the workflow relies on unstructured or natural language data, such as insurance claims written in customers' own words.

And the anti signal: if a deterministic solution suffices, use it. The course poses a drill here and leaves the answer to you. My answers: turning off office lights at 7 pm, rule based. Flagging transactions over ten thousand dollars, rule based. Reading, categorizing, and escalating customer emails, agent. My note: raise this early in an interview. It shows restraint for very little effort.

## Section 4: The components of an agent

### The three core components

From the course: model, tools, and instructions. The model thinks, the tools do, and the instructions guide. Instructions are not a separate runtime component. They are directives and context given to the model.

### Model selection

The goal is not the biggest model. It is the model that fits. There are four factors.

Task complexity. Extraction and simple summaries can use small models. Planning, judgment, and code generation benefit from capable models.

Latency. An agent runs in a loop, so model latency multiplies. The appendix's example: ten seconds per call across a five step loop is a fifty second wait.

Cost. Agents burn far more tokens than chatbots, since they think across multiple turns. The answer is a router: send simple steps to cheap models, and reserve expensive models for high level decisions.

Context length. Long documents and conversations need larger windows, or better context management.

An interview tip: talk in tiers, such as "a fast tier for classification and a strong tier for planning," rather than naming specific model versions. Model names age quickly, and tiers show you understand the reasoning. ChainBuddy gives you a concrete example of tiering, which we will reach in Section 8.

### Tools

A tool is any external function, API, or environment the agent can use: APIs, web search, code execution, databases, device controls. Without tools, the agent can only simulate a task in text.

The engineering principle from the appendix: tools should be discrete and composable, each doing one thing and returning a structured result. Define inputs and outputs with schemas. Put an abstraction layer in front of third party APIs, so you can swap services without touching agent logic.

### Instructions

Instructions come as a system message defining role, examples demonstrating behavior, constraints, and task definitions. Compare "be helpful" with "answer billing questions only, escalate refund requests over five hundred dollars to a human, and never discuss competitor pricing." The second is testable. Precision in instructions is one of the highest leverage activities in agent work.

The course also notes that the short prompts quoted from research papers are essences. Production prompts include context, few shot examples, output format requirements, safety rules, and persona.

### Why modularity matters

Keep model, tools, and instructions independently changeable. You can then upgrade models without a rewrite, add tools without redesign, adapt instructions to new domains, isolate bugs, and contain failures. A course quiz tests this, and this is my answer, unconfirmed because I did not submit the quiz. A team hardcodes live inventory data in the prompt to save a tool call. That is the design that violates modularity, because stale data and tangled concerns will hurt as the system scales. A small model for language detection, an API wrapper as a tool, and system instructions for tone are all fine.

### The architecture blueprint

The course zooms out to four elements in a control flow. The input interface receives information. The reasoning core interprets and plans. Tools and actions execute. Memory and context supply continuity. The loop then repeats, and actions influence future inputs.

### Perception

Perception converts raw input into signals the reasoning core can use: speech to text, optical character recognition on images, parsing JSON from an API, intent detection. It enables context awareness, multimodal interaction, and dynamic reactivity, meaning the agent can monitor for changes rather than wait for commands.

A principle from WebVoyager that generalizes well: the agent's perception must match the logic of its environment. If your environment is visual, a text only representation will fail no matter how good your model is.

### Memory

There are three layers in the core lessons. Short term memory holds recent context in a session, often the context window or a small buffer. Long term memory persists across sessions, such as preferences and past decisions, and needs explicit storage. External or knowledge memory looks things up from outside sources, using RAG and vector databases. The appendix adds episodic memory: past actions and outcomes, which lets an agent learn from experience.

The key mechanism: the agent does not load the knowledge base. Information is stored as embeddings, and semantic search retrieves only the snippets relevant to this step.

Memory touches the loop at three points. Before reasoning, retrieve. During reasoning, use it as context. After acting, store new information.

The full loop in the course is five words worth memorizing: Perceive, Recall, Reason, Act, Store.

The course's diagnostic case: a support agent repeats questions, forgets recent tickets, gives generic answers even though FAQs exist, and stays cheerful with an angry user. Every one of those symptoms is a Memory and Context failure, even though the model, tools, and prompt all exist. And when a user says "still having the issue I reported yesterday," the Recall step bridges the sessions, by pulling long term memory of the prior ticket. This is a nice pattern to use live: map symptoms to a component.

## Section 5: Orchestration patterns

Orchestration is how an agent manages its decision flow and how several agents coordinate. The course treats these as implementation strategies and not as a fixed, mutually exclusive taxonomy. Real systems compose them.

### Single agent patterns

The tool calling loop. The agent gets a task, picks a tool, runs it, observes, and continues until done. Use it for dynamic tasks with exploration, like debugging or knowledge synthesis. Weakness: long tasks turn into brittle chains that are hard to debug and recover.

ReAct, short for reasoning and acting. It alternates an explicit thought with an action, then observes. Use it when the environment is unpredictable, or when transparency matters, as in regulated domains. Weakness: verbose, so more tokens and latency, and long reasoning chains can still be hard to interpret. The course's clearest justification comes from web browsing: pop-ups and changing pages break fixed plans, so react step by step.

Plan and execute. Create the full plan first, then run the steps. Use it when the goal is complex but known up front, and sequential accuracy matters, like a multi part report. Weakness: the plan goes stale if the environment changes, and re-planning costs money. The upstream error risk also matters. As MuLan shows, a bad plan poisons everything after it.

### Multi agent patterns

Manager worker, also called orchestrator worker. A manager decomposes and delegates, workers execute with their own tools and instructions, and the manager integrates results, using shared memory for tasks and results. Strong coordination. Weakness: a single point of failure and a possible bottleneck, plus poor delegation sinks everything.

Decentralized handoff. Peers pass control based on task state, with no central manager. Flexible and resilient. Weakness: risk of conflicts, infinite loops, deadlocks, and dropped responsibilities, and much harder debugging. It fits simulations, real time collaboration, and distributed monitoring, or cases where no single agent has the full context.

Hybrid. Common in practice. The course's smart parking exercise lands on a hybrid centered on a manager.

### Two more workflow patterns

The appendix adds reflection and routing to planning and ReAct, making four core workflow patterns.

Reflection: the agent or a separate critic reviews output before it goes further. Use it for high stakes outputs.

Routing: a classifier sends each task to the right specialist or model. This improves accuracy, and it improves safety too, because specialists can have tighter instructions and tools, and routing can enforce access control.

Also keep a clean distinction in your vocabulary. Architecture is the shape of the system: how many agents, and how they relate. Workflow is how the system moves over time. Interviewers appreciate people who do not blur these.

### One agent or many: the four factors

From the course. Task scope and autonomy boundaries: are there independently solvable subgoals? Specialization: are distinct competencies needed? Sequential versus parallel: can work run concurrently? Observability and control: single agents are easier to monitor and debug, while multi agent systems need coordination and shared memory.

My recommended stance: default to one agent with good tools, and make the case for each extra agent. Each one adds LLM calls, handoff formats, and failure surface. MACRS needs four LLM calls per turn, for example. It earns that cost, but you must be able to say how.

### The course's research assistant case, answered

The prompt: gather competitor announcements, analyze customer reviews, analyze internal sales reports, then synthesize a recommendation. Tasks can run in parallel, synthesis must be coordinated, and a human must approve before stakeholders see it.

My answer: manager worker with a human approval gate. Three specialist workers run in parallel, each with scoped tools, so the review analyst has no access to sales data unless needed. The manager integrates. A critic checks each claim against its source. The human approves at the end. Decentralized handoff would be a poor fit, because the synthesis needs a single owner and parallel results must converge.

### Frameworks

LangChain and LangGraph. LangChain has chains and the broadest integrations. LangGraph models workflows as stateful graphs with nodes, conditional edges, cycles, and persistence, and suits ReAct loops, reflection cycles, and multi agent coordination. The trade off is a steep learning curve and deep abstraction. My note: if you have built with LangGraph, use it as your default example, and be ready to discuss when it would be overkill.

AutoGen, from Microsoft. Agents are conversational entities that exchange messages. Strong for debate and verification, such as a programmer and executor iterating until tests pass, and it can include humans as agents.

CrewAI. A role based crew with a manager, workers, and shared memory. It has the lowest conceptual onboarding cost, and it suits well understood business processes, at the price of less low level control.

Google ADK. Used in the course's implementation chapters. It has workflow primitives such as a sequential agent and a loop agent, and it is natural on Google Cloud.

Others the course mentions are the OpenAI Agents SDK and Semantic Kernel. Choose based on single versus multi agent, integrations, team experience, target cloud, and how much control you need over the loop. Concepts transfer, and frameworks churn. Do not let an interview turn into a framework trivia contest.

## Section 6: Trust, guardrails, and oversight

When an agent acts, trust becomes a design requirement. A chatbot that gives a wrong answer is embarrassing. An agent that sends money to the wrong account is an incident.

### Where guardrails sit

Guardrails are separate modules that operate alongside the model and intercept data at three points. Input guardrails run before the LLM sees input. Output guardrails run after it generates a response or plan. Tool use guardrails run before a tool executes. In multi agent systems, they must also mediate agent to agent messages and shared memory.

### Types of guardrails

Contextual grounding checks keep answers on task and supported by sources, using RAG, prompt constraints, and post response checks.

Safety and moderation filters block harmful content, using moderation APIs, classifiers, blocked categories, and rate limits on sensitive capabilities.

PII and data protection filters detect, redact, and restrict, using regular expressions or classifiers, masking before the model sees content, permission based access to sources, and audit logs. The course's healthcare example: retrieve only data tied to the current user or session, and mask identifiers unless authorized.

Tool use safeguards are allowlisting, argument validation, preconditions, rate limits, and simulating before executing. A password reset tool, for example, should verify the requester is authenticated, the user exists, and the request has not repeated too often.

Rules based output validation uses regex checks, format checks like JSON schema, semantic checks by a second model, and blocklists and allowlists. For example, verify that a required disclaimer is present.

Advanced safeguards include a critic or evaluator agent, voting and ensembling for critical decisions, self critique, multi agent oversight, quantitative safety monitoring, automated rollback and recovery, and compliance and explainability layers that record why a decision was made.

### Trade offs

Over blocking creates false positives and frustration. Under blocking lets attacks through. Every layer adds latency and complexity. Too many approvals add friction. Best practices: layer defenses, start with the critical risks such as PII, safety, and irreversible actions, monitor false positives and negatives, be transparent with users, and balance safety with utility.

### Controlling memory

Memory is a liability without constraints. Avoid retaining PII, filter before storing, set expirations, and restrict topics. A course quiz gets at this, and again these are my answers, not confirmed against its key. To stop PII exposure during generation and memory storage, use classifier or regex filters on outputs, and mask fields before storing. Blocking all user data kills the product. Argument validation and rate limiting address different risks.

The matching exercise in the same lesson is good practice for naming the right guardrail. My answers: slurs in an invoice note call for a safety filter. A missing disclaimer calls for rules based output validation. Repeatedly rescheduling a payment calls for a rate limit on the tool. Retrieving the wrong client's data calls for data protection and access control. A vague summary calls for a relevance check.

### Human oversight

The course's driving analogy is worth keeping.

Human in the loop is a student driver with an instructor who must approve the turn. The system waits. It works through approval gates and escalation triggers. Use it for irreversible or high stakes actions, low confidence, and ambiguity.

Human on the loop is a driver using cruise control. The system runs, and a person watches and can take over, through dashboards and intervention interfaces.

Human above the loop is reviewing the trip data afterward. It relies on audit trails and logs, and it serves accountability and improvement.

Other models are user triggered flagging, multiple reviewers, and automated monitoring that escalates on thresholds.

The costs are latency, money, scalability limits, and reviewer fatigue. If you escalate everything, reviewers start rubber stamping and miss the one that matters. So place touchpoints deliberately. Set thresholds, like approval for transfers above a certain amount. Give reviewers a good interface with the agent's reasoning, data, and proposed action. Have the agent write a short summary for the reviewer. And feed interventions back so that escalations decline over time.

### Minimal footprint

The appendix calls this the most important safety practice. Each agent gets the minimum tools and permissions for its task. An agent that reads records does not get write access. An agent that drafts email does not get to send it without approval. This is also a debugging feature, because narrow agents fail in narrow ways.

## Section 7: Production challenges and how to answer them

This section condenses the course's challenges lesson. For each, know the problem, the trade off, and two or three fixes.

High inference latency. Loops and multi agent calls multiply delay, and speed trades against cost. Fixes: route to smaller models, cache deterministic results and idempotent tool outputs, run work in parallel, process asynchronously when no one is waiting, and use model optimization such as quantization, pruning, and distillation, or better hardware. My note: also stream partial progress to users.

Output uncertainty and hallucination. Strict validation helps but can over block and slow things. Fixes: ground answers with RAG, validate outputs, ensemble critical decisions, score confidence and escalate below a threshold, and instruct the model to say "I don't know." In multi step work, a hallucinated intermediate result cascades, so verify at critical steps and not only at the end.

Memory consistency. Trade offs among storage cost, retrieval latency, and consistency. Fixes: separate the memory types, define which agents may read and write what, retain selectively, and version and audit critical data.

Scalability. Fixes: modular architecture, multi agent distribution, load balancing and auto scaling across LLM endpoints and tools, batching, and removing redundant calls.

Integration complexity. Real systems have odd formats, authentication, and errors. Fixes: standardized tool schemas, API abstraction layers, retries and fallbacks, and explicit message formats between agents.

Security and privacy. New attack surface includes prompt injection, jailbreaks, and data leakage. Fixes: layered guardrails, authentication and authorization, secure deployment, and differential privacy for the most sensitive cases.

Evaluation. Traditional model metrics fall short. Fixes: task oriented metrics, simulated users, human reviews and A B tests, and rich logging of reasoning steps and tool use.

Human in the loop overhead. Covered in the last section. Place touchpoints only where they matter.

Fault tolerance. Components fail mid run, and failures cascade. Fixes: graceful degradation, such as a support agent disabling a broken product lookup while still handling FAQs. Retries with exponential backoff for transient errors. Checkpoints so that work resumes from the last good state. Handoffs to a human or recovery agent. And error boundaries so one component's failure stays contained.

The course closes this lesson with a production readiness checklist. Performance and cost tested at peak load. Reliability with validated decisions and graceful degradation. Memory consistency and retention policies. Security with filters, enforced tool policy, and robust authorization. Human oversight with clear checkpoints and audit trails. Observability that traces reasoning and tool use. And a modular architecture with clear documentation.

## Section 8: The case studies

Case studies do real work in an interview, because they let you back up a design choice with evidence. For each one, remember the problem, the architecture, the evidence, and the idea you can steal.

### MACRS: multi agent conversational recommendation

The problem. Traditional recommenders were goal directed but rigid, using slot filling and state machines. LLM only recommenders are fluent, but they lose the plot, because they lack explicit planning and state management over long conversations. So the core trade off is fluency versus control.

The architecture has two modules. The first is multi agent act planning. Three responder agents independently generate candidates: an asking agent, a recommending agent, and a chit chat agent. A planner agent then picks one, reasoning over user preferences, dialogue progress, candidate value, and engagement. This is manager worker with an interesting twist: generate options, then select.

The second module is feedback aware reflection, with two levels. Information level reflection infers what the user wants, based on behavior such as clicks, skips, and engagement, and it updates a user profile shared with the responders. Strategy level reflection asks how to behave better. It detects failures, such as rejections, disengagement, and repetition, using a hybrid of fast rules and a lightweight classifier. It then generates strategy suggestions for the responders and corrective experiences for the planner.

A key mechanism detail: adaptation happens by changing what goes into the agents' prompts on the next turn. No model retraining occurs. It is in session adaptation, not long term learning.

The cost is four LLM calls per turn, which adds latency and spend. The course justifies this with better goal performance, modularity, and interpretability.

The evaluation used a GPT 3.5 based user simulator, since simulators give reproducibility. Metrics were Success Rate, Hit Ratio at K, and average number of turns. As presented in the course, MACRS scored 58.3 percent success, 69.5 percent hit ratio, and 5.9 turns, versus the best baseline at 48.7 percent, 64.3 percent, and 6.8 turns. Removing any component reduced performance. The course also gives an honest caveat: a simulator from the same model family as the system can bias results positively, and real users are still needed.

My takeaway: separate proposing from deciding, and split reflection into "what does the user want" and "how should we behave."

### Eureka: an agent that designs rewards

The problem. In reinforcement learning, a reward function is the rulebook. Designing one is slow, expert heavy, and error prone, and agents exploit loopholes, such as flinging a pen to rack up points. The reward's quality is only visible after training, so you cannot judge it by reading it. That is why a loop is necessary.

Eureka, from NVIDIA, is a meta agent: an agent that builds the learning conditions for other agents.

The architecture has four mechanisms. First, zero shot generation: the LLM reads raw environment code and a task description, and writes reward code with no templates or examples. It often uses only the observation portion of the code to save context. Second, evolutionary search: each iteration samples multiple candidates, typically sixteen, which gives fault tolerance since some will be broken. Each candidate trains a policy, and the highest fitness wins and seeds the next round. Mutations change a coefficient, change a functional form, or add a component. Third, reward reflection, which solves credit assignment. A single fitness score says nothing about why. So reward code must return a dictionary of components, and those values are tracked during training and summarized into text for the LLM. The prompt adds heuristics. If success is always near zero, rewrite the function. If a component barely changes, rescale, rewrite, or discard it. If one component dominates in magnitude, rescale it. Fourth, fast evaluation. Isaac Gym is GPU accelerated and up to three orders of magnitude faster than CPU simulation, which is what makes the search affordable. One training run, using an algorithm called proximal policy optimization, for each candidate during the search, and five for the final report.

Human feedback enters in two ways. Human initialization treats a person's reward as the first iteration. This beats both people alone and Eureka alone. The insight: humans are good at picking relevant variables, and weak at weighting them. Human reward reflection replaces automated feedback with text like "the torso is too low." In a user study, fifteen of twenty preferred the human guided version, which traded speed for stability.

The evidence: twenty nine environments across ten robot types, with an average normalized improvement of fifty two percent over human rewards, and a twenty eight point six percent drop without reflection. It also discovered rewards weakly or even negatively correlated with human ones that still performed better. A clean experimental design detail: all rewards used the same training algorithm and hyperparameters, so differences came only from the reward. The limitation: it does not carry learning from one task to another.

On the unsafe behavior quiz in that lesson, I would choose a runtime behavioral monitor that detects and penalizes unsafe actions during training. A hardcoded penalty is more brittle, randomness does not address the problem, and a smaller LLM is irrelevant. That is my answer, and I did not confirm it against the key.

My takeaway: make your components expose their internals, because rich feedback beats a single score. Sample many candidates when generation is cheap and errors are common. And invest in fast evaluation, since the speed of your evaluator sets the speed of your agent's improvement.

### The Eureka style implementation in Google ADK

The course rebuilds a small version with Brax's HalfCheetah on a Colab GPU. Its structure is a good template for a loop based agent.

A sequential agent runs a setup agent once, then enters a loop. The setup agent initializes shared state, loads config, prepares an artifacts folder, and creates a baseline rollout for comparison. Inside the loop, a reward designer generates candidates. An evaluator trains and scores them. A selector picks a winner. A reflector explains why. An optional human reflection agent adds comments. An exit checker decides whether to stop. An increment agent advances the state.

Seven production lessons are hiding here. First, separate agents from tools: agents decide and tools execute. Second, write generation rules like a validator, not like documentation, because you are designing for execution reliability. The designer's output must use an exact delimiter, and every reward must return its total and its terms. That is a parsing contract between agents. Third, expect imperfect output anyway. The evaluator strips stray formatting, extracts valid functions, and compiles each one through a gate before training. Fourth, let one bad candidate fail without stopping the iteration. Save the code first, capture the error, write an artifact, and still record it on the leaderboard so failure stays visible. Treat nonessential steps, like rendering, as non-fatal. Fifth, use the LLM only where judgment is needed. The selector is deterministic and evidence based, with no LLM. The reflector does use an LLM, and it sees all candidates rather than just the winner, because comparing them is what reveals why one worked. Sixth, be careful with state. Reset iteration scoped evidence so the selector cannot reuse stale scores, and preserve context and progress across iterations. In the course's words, reset evidence, preserve context, preserve progress. Seventh, never hang on a human. The human feedback step is optional, and it continues when no one is at the keyboard.

The lesson also warns that a high score does not guarantee desired behavior. A policy can score well while moving inefficiently or even in the wrong direction. So inspect behavior, not just the metric. Your success metric is itself a design decision. Termination is both bounded and adaptive. A maximum iteration count is the hard safety net, and an exit checker can stop early when the score passes a threshold or plateaus for a set number of rounds. The order of agents in the loop definition is the system, so a behavior change has one obvious place to live. When reading a run, the course's sequence is: the leaderboard for who won, rollouts for what was truly learned, and logs as the flight recorder for where something went wrong.

### ChainBuddy: solving the blank page problem

The problem. People know the question they want answered, but they stall facing an empty canvas. A simple question, like whether a "mathematician" persona improves accuracy, requires personas, prompt templates, models, an evaluator, and a chart. ChainBuddy generates an editable starter pipeline inside ChainForge, a visual LLM evaluation tool. The course notes that as of late 2025 it was still listed as a work in progress and not in the main release.

The architecture has two phases. Phase one is a requirement gathering chat assistant. It keeps a running internal state of goal, requirements, and preferences. The course calls this Chain of Reasoning: Chain of Thought improves a single answer, while this guides an entire project. An early open ended questioner stressed users out in pilots, so the team switched to a "smart intake form" with up to three targeted questions at a time, and users can hit generate whenever they like.

The output is a structured specification, what the course calls a work ticket, as JSON. Hand off a work ticket, not a transcript.

Phase two is a LangGraph based backend using plan and execute. A planner creates a layered blueprint: input, processing, and evaluation layers. It also creates variable traces, which record where each variable originates and what consumes it. Workers then build: a text fields agent, a prompt agent, an evaluator agent that chooses code for objective criteria or an LLM scorer for subjective ones, an edge agent that draws connections from the traces, and a layout agent. Output is an executable JSON file. Users can edit the blueprint before workers run.

The team also used heterogeneous model tiering: a faster, cheaper model for chat and a stronger one for generation. Clarifying intent early also avoids spending expensive compute on the wrong build.

The evaluation was a within subjects study with twelve participants, using cognitive load surveys, usability scales, blind expert ratings, and interviews. Tasks felt easier, and work quality rose. Users were more likely to include evaluators, a direct effect of the planner's structure. The surprise: participants did not rate their success much higher, even though experts did. The course attributes this mismatch to the Dunning Kruger effect, and it also raises overreliance and users copying the assistant's patterns.

My takeaway: decouple conversation from heavy work. Make the output editable, keeping the human as final arbiter. And use dual evaluation: never rely on user perception alone.

### WebVoyager: the multimodal web agent

The problem. An agent must complete tasks on live, messy websites. HTML only agents drown in verbose markup, and they miss visual logic. Calendars are the classic trap: the grid is obvious to a person and a maze in the code. Vision creates a new difficulty, the grounding problem. The model can correctly decide "click search," but connecting that to one exact element is the bottleneck. In the course's analogy, a head chef says "sauté the vegetables," and the line cook has to pick the right pan and the right bowl for table seven. Action generation is the what. Action grounding is the which.

The architecture is a multimodal ReAct loop, chosen because a fixed plan fails on pages that change. Observation combines a screenshot and text. Set of Mark prompting runs a script that draws numbered boxes over interactive elements, so the model says "click seventeen" instead of describing a blue button. A text list adds each element's label and type. The course generalizes these into two patterns: grounded action referencing and dual modality observation.

Context clipping keeps the entire history of thoughts and actions, but only the three most recent screenshots, which balances focus and recall. The action space has seven commands: click, input, scroll, wait, back, jump to search engine, and answer. Jump to search engine is a recovery action, and answer is the termination action. A small, explicit action space is itself a design decision. The environment uses a fixed viewport so coordinates stay consistent.

The evaluation used 643 tasks across fifteen live sites. The dataset used a self instruct method: humans write seeds, an LLM expands them, and humans verify. Human judges reviewed full trajectories and judged the outcome, avoiding a brittle golden path requirement, since many paths can be valid. GPT 4V served as a scalable proxy judge, agreeing with humans 85.3 percent of the time.

Results: 59.1 percent success, against 40.1 percent for text only and 30.8 percent for a general purpose agent. Failure analysis is the best part. Navigation and getting stuck, 44.4 percent of failures. Visual grounding, 24.8 percent. Hallucinated completion, 21.8 percent. Prompt misalignment, 9 percent. So the largest problem was not vision at all. It was recovery. An agent needs to notice it is stuck, and a final verification step would address falsely claimed success.

### The web agent implementation

The course's ADK build offers three modes. Strict mode is a heavily constrained single agent limited to seven numeric action commands, with a long prompt holding twelve guidelines, a decision policy, and few shot examples. Single agent mode is a conversational WebVoyager style agent, and the course calls it the simplest for demonstrations because it avoids transferring context between agents. Multi agent mode has a coordinator with no web tools of its own, a planner with no tools at all, a browser agent as the hands, and a vision agent as the eyes. The stated reason for the split is that one context window holding fifteen tools can get confused. Each mode is wrapped in a loop agent with a default cap of fifteen iterations, to prevent infinite loops and runaway cost. If building the loop fails, it falls back to the plain agent.

The core problem it solves is that LLM calls are stateless while a browser is highly stateful. A single shared runtime object holds the browser session, the elements from the last observation, action history, visited pages, and counters that detect when the agent is stuck, such as a no progress streak. That is how a tool call for element five knows what element five means.

Grounding has layered defenses. If the model invents an element number, the tool returns an error the model can correct from. A click first tries a stable selector, then falls back to the pixel center of the element's box. Instructions say never to invent IDs, to re-observe when an action changes nothing, to try something different rather than repeat, and to open a date widget and observe again before choosing a day.

Completion is verified. The agent must call a verification tool with evidence and get a verified status before declaring success, which directly targets the hallucinated completion failure in the WebVoyager study. It must not take irreversible actions unless asked. Each run writes screenshots, labeled screenshots, and a JSON lines process log, treated as evidence you can replay. If a site blocks access, the right behavior is an honest statement of the limitation, not an invented answer.

The course also names limits of labeling elements from the page's HTML structure. It is blind to canvas and some cross origin frames, boxes clutter dense pages, and pages can shift between capturing coordinates and clicking. It says the industry is moving toward pure vision parsing, models that output coordinates directly, and the accessibility tree, which gives cleaner semantics than raw HTML. My note: mention the accessibility tree in an interview, since it is a pragmatic win.

### SkillWeaver: the self improving web agent

This thought exercise, based on the SkillWeaver paper, starts from a cost. A basic agent re-learns the same flight search every time, wasting latency and tokens, and adding fresh chances for error. The course estimates five to seven LLM calls for the search, which a reusable skill reduces to one.

Its solution has three stages. Skill practice: complete the task once with the normal ReAct loop, producing a successful trajectory. API synthesis: an LLM turns that trajectory into a general function, replacing specific airports with parameters, and adding a signature and docstring. Skill honing: generate test cases, run them, feed errors back to the LLM, and add the skill to the library only after it holds up.

The library stores each function with its signature, a detailed docstring, and a usage log of past successes and failures. For selection, an LLM router sees the task plus all signatures and docstrings, and it calls a matching skill instead of starting the loop.

My takeaway: convert repeated reasoning into tested, deterministic code, so the agent gets cheaper and more reliable with use. My note: a router that sees every skill will strain as the library grows, so retrieve candidates first, and retire skills that start failing when sites change.

### MuLan: agentic image generation

The problem. One shot text to image generation gets spatial relationships and attribute bindings wrong, and if it fails, your only move is to start over.

The architecture works like a painter, using three parts. An LLM global planner lists objects in painting order, left to right and bottom to top, converting a difficult spatial reasoning task into a simple sorting task. Progressive generation uses a stock diffusion model to add one object at a time. A local planner sets a rough mask, and backward cross attention guidance pushes the object into that region. When objects overlap, it generates candidates at ten, thirty, and fifty percent overlap and uses a CLIP score to pick, CLIP being a model that rates how well an image matches text. VLM feedback uses a vision language model as a critic after each stage, checking presence, attributes, and spatial relations, and it triggers a redo with adjusted parameters.

It is training free. Intelligence comes from orchestrating existing models. It also creates natural human checkpoints, where a user can say "make the vase smaller" mid process, turning them from prompter into director.

The evaluation used 200 hard prompts, curated partly from a benchmark and partly generated, selected because leading models fail them. The scoring is a structured questionnaire on object completeness, attribute binding, and spatial relationships, answered by both GPT 4V and humans. One lovely detail: the critic during generation, LLaVA, differs from the final judge, GPT 4V, so the system does not grade its own homework. Removing the feedback loop dropped overall score from 77.55 percent to 60 percent. Limitations: slower, and planner errors propagate.

My takeaway: use different critics for different jobs, a cheap one for narrow checks and a strong one for overall judgment. And separate your runtime critic from your evaluation judge.

### AI Hospital: high stakes simulation

A thought exercise on a simulated clinic. It is educational and not medical advice. Static multiple choice exams test recall, not the interactive process of diagnosis. So design a loop with four roles: an intern doctor, a patient, an examiner who supplies test results, and a medical director who evaluates. The examiner gives results only through the doctor, as in real practice. For collaboration, each intern drafts independently first, so that reasoning comes before group influence. The director finds the disagreements and makes them the agenda, and the interns debate those points toward consensus. For long term improvement without retraining the LLM, there are two stores. A medical case base holds successful cases, retrieved with RAG. An experience base holds corrective rules, derived by comparing a wrong diagnosis with the ground truth, stored once validated. The final idea, called SEAL, Simulacrum based Evolutionary Agent Learning, is a closed loop that yields accelerated experience, learning without manually labeled data, and transfer to real benchmarks such as medical exams.

My takeaway: evaluate in interactive environments, and use memory rather than fine tuning to accumulate experience.

### OpenClaw: the agent as infrastructure

This case flips the view: the LLM is one component, and the assistant behaves more like an operating system. It has five constraints: continuous presence, cross platform persistence, physical agency, long term memory, and proactive autonomy.

The layers are as follows. A channel layer normalizes messages from WhatsApp, Slack, and others, and handles delivery quirks. A gateway control plane, a long running WebSocket server, handles routing, lifecycle, and authentication. An agent runtime assembles context, runs the LLM loop, intercepts tool calls, and writes each turn to append only logs. A memory layer uses JSON lines logs and SQLite for hybrid search, both vector and keyword. And a tool substrate covers shell, files, and browser, often sandboxed in Docker with human approval for risky commands.

The major decision is persistent process over serverless. Serverless fails here because of cold starts, the difficulty of keeping WebSockets alive, and rebuilding context on every turn. The price is that you own uptime, crash recovery, and memory growth.

Autonomy comes from the same agent running under different triggers: cron jobs fire, and if nothing matters, the agent finishes silently. You do not need a separate monitoring subsystem.

Security is defense in depth. Bind to localhost by default. Authenticate at the gateway. Scope permissions by session, so that a group chat gets a sandbox while the main session may get host access. Use non root, read only, capability dropped containers. Require human approval for high risk commands. On prompt injection, the course argues that most vulnerabilities are failures of the system to correctly label information, so it separates system text, user text, and tool output with structured formatting and metadata. That is useful but not sufficient, and Section 10 explains why.

Context survival treats tokens as a budget: compact old turns with an LLM summary, and if compaction fails, reset the session instead of crashing. It values survival over fidelity.

My takeaway is the distinction between load bearing and swappable decisions. Persistent gateway, session scoping, and a delegated agent loop are structural. SQLite, the sandbox technology, and the way tokens are streamed are swappable. Use this language with interviewers.

## Section 9: The patterns across all case studies

Pull back and you will see the same few moves repeating.

Decompose and specialize. MACRS splits conversation roles. ChainBuddy splits building roles. MuLan splits a scene into objects. Decomposition lets weaker components solve harder problems.

Generate, then select. MACRS has three candidates and a planner. Eureka has sixteen candidates and a fitness score. MuLan has overlap candidates and CLIP. When outputs are uncertain, sample and judge.

Close the loop with rich feedback. Eureka's component values, MACRS's two reflections, and MuLan's critic all do this. A single score tells you that it failed, while rich feedback tells you why.

Hand off structure, not chat. ChainBuddy's JSON ticket and variable traces, and Eureka's component dictionary. Structured contracts keep agents from misunderstanding each other.

Make the environment legible to the model. WebVoyager's numbered boxes, Eureka's environment code, and MuLan's sorting rule. Change the problem's representation before asking the model to be smarter.

Budget the context. WebVoyager's clipping, Eureka's observation only code, and OpenClaw's compaction. Context is a scarce resource to be managed on purpose.

Create checkpoints. Progressive designs produce natural points for critics and humans.

Evaluate the system with ablations. Remove a part and measure the drop. Eureka lost 28.6 percent without reflection, and MuLan fell from 77.55 to 60 without its critic. Proposing an ablation in your interview is an easy way to sound rigorous.

Fail informatively. Failure analysis in WebVoyager, leaderboard error entries in the ADK build, and logged planner decisions in MACRS.

## Section 10: Beyond the course

Everything in this section is my addition. I checked the protocol governance and the lethal trifecta against sources listed in Section 14. The rest is general engineering knowledge. These are topics I would prepare because the course leaves them thin, though I cannot promise any particular interviewer will ask.

### Tool and agent protocols

The course discusses standardized tool definitions but does not cover protocols. Know the Model Context Protocol, or MCP, which standardizes how an application exposes tools and data to a model. Know the Agent to Agent protocol, or A2A, aimed at communication between agents across systems. A simple way to put it: MCP connects agents to tools, and A2A connects agents to agents. Both are now reported to sit under the Linux Foundation's Agentic AI Foundation, with MCP a founding project in December 2025 and A2A joining in August 2026. I found that A2A date in one secondary source, so check it before quoting. Also point out that a protocol solves connectivity, not trust. You still need authorization and argument checks at the tool boundary.

### Prompt injection, taken seriously

The course's labeling approach helps, but models do not reliably respect labels. So assume injection will sometimes succeed, and limit what a hijacked agent can do. Simon Willison's "lethal trifecta" is the concept to know: access to private data, exposure to untrusted content, and the ability to communicate externally. When an agent has all three, an attacker can steal data. Break at least one leg. For example, split the agent that reads untrusted content from the agent that holds privileges, passing only validated structured data between them. Enforce permissions in the tool layer, using the end user's identity, not the agent's broad service account. Never put the security decision in the prompt. Willison also says he is deeply suspicious of guardrail products claiming to catch most attacks, since 95 percent would be a failing grade in web security. That is why the architecture has to carry the weight. My note: this is one of the most useful points to raise unprompted.

### Idempotency and durable execution

If an agent retries a payment after a timeout, did it pay twice? Tool calls need idempotency keys. Long workflows need durable execution, where each step is persisted so a crash resumes without repeating side effects. Multi step writes need compensating actions, the saga pattern from distributed systems, since you often cannot roll back an email. The course mentions checkpointing and rollback. These terms go one level deeper.

### Loop control and budgets

Set hard limits per run: maximum steps, tokens, wall clock time, and dollars. Detect loops by watching for repeated identical calls or no change in state. Define what happens on hitting a limit, normally a partial result plus an honest status or a handoff. Unbounded loops are the classic agent cost incident.

### Context engineering

The prompt is now assembled at runtime from many sources, and that assembly is the design problem. Keep tool results short, or store large results externally and pass a reference. Summarize old steps. Keep tool lists small, or retrieve relevant tools on demand, because too many similar tools degrade selection. Keep stable content at the start of the prompt so that prompt caching can save money, when your provider offers it.

### Evaluation in depth

Build a golden dataset from real traces. Evaluate at three levels: step level, such as whether the right tool was selected with the right arguments; trajectory level, such as whether the path was efficient and safe; and outcome level, such as whether the task was truly done. Mix code based checks with LLM judges, and calibrate the judges against humans, as WebVoyager did. Since agents are non deterministic, run each case several times and report distributions. Run evals in continuous integration, so a prompt change is treated like a code change. Then track production metrics: task success, escalation rate, cost per successful task, tool error rate, and time to completion. Cost per successful task is better than cost per request, since a cheap failure is not a bargain.

### Quick capacity arithmetic

If you are asked to estimate, compute LLM calls per task times tokens per call times price, then multiply by volume. Latency is the sum along the sequential path, plus the slowest branch of each parallel step. Provider rate limits often bite before your own servers do, so mention queues, backpressure, and fallback across models. Tasks that run for minutes should be asynchronous, using a job queue and status updates rather than a held connection.

### Trace based observability

Treat each run as a distributed trace, with spans for each LLM call and tool call, carrying inputs, outputs, tokens, and cost. The course mentions LangSmith. OpenTelemetry style tracing is the general concept.

### Rollout

Run in shadow mode first: the agent proposes, and people act, so you can compare. Then move to assistive, with human approval. Then allow autonomy for low risk segments, expanding by measured performance. Keep a kill switch. Version prompts, tools, and models together, so you can roll back. For a forward deployed role, this staged path is often the most persuasive thing you can describe.

## Section 11: Worked designs, spoken as you would say them

Practice these out loud. Each follows Goal, Gate, Blast, Shape, Parts, Guard, Break, Prove.

### Design one: customer support agent that can issue refunds

Goal. Resolve billing and account issues end to end. I would measure resolution rate without recontact, and confirm with customer satisfaction and cost per resolved case. Clarifying question: what refund amounts and what volume?

Gate. Authentication and refund policy limits stay as deterministic code. Understanding messy messages and deciding what is going on needs an LLM.

Blast. Reading an account is low risk. Refunds cost money, and account changes can enable fraud. So refunds under a set limit are automatic if policy checks pass, and those above it need human approval.

Shape. Start with a router and a single support agent. Add specialists only if billing and technical issues need clearly different tools and instructions. Inside the agent, use a ReAct style loop, since the path depends on what the lookups return.

Parts. A fast tier model for routing and a strong tier model for resolution. Tools: account lookup, order status, knowledge base search, refund, and ticket creation. Short term memory for this conversation. Long term memory for prior tickets and tone history. RAG for policy docs.

Guard. Input filtering for injection. PII masking. The refund tool checks identity, amount, and a once per order limit in code, independent of the model. Output validation for policy claims. Escalation on repeated frustration, low confidence, or a user request, with a summary handed to the human.

Break. Idempotency keys on refunds. If the account API is down, degrade to FAQs and open a ticket. Step cap per conversation. A refund in progress is checkpointed.

Prove. Replay historic tickets in a simulator, measure resolution and policy compliance, and run an ablation without memory. Shadow mode, then small amounts, then expand.

### Design two: insurance claims intake, from the course's knowledge check

The course gives an answer here, so I will give its version first. Step one, receiving the written description, is planning: work out what is present, what is missing, and what steps follow. Step two, searching the policy database, is ReAct, because the number of searches depends on what comes back. Step three, asking follow up questions, is reflection plus ReAct. Reflection detects the gap, and ReAct closes it. Step four is reflection on the draft assessment, plus routing by claim value to a human queue.

The course prefers multi agent: an orchestrator, a policy retrieval agent, a conversation agent tuned for empathy, and an assessment agent. Its reasoning is that these tasks differ in tools, tone, and failure modes, and can be tested separately. It names the cost as schema design and added latency. Its best line is that "which single pattern applies" is sometimes the wrong question. Ask what the agent must detect, and what it must do about it.

My note: I would start simpler, with one agent plus a separate critic, and split when the evaluation shows a reason. Both positions are defensible if you name the trade off. I would also make the value threshold plain code rather than a model decision.

### Design three: freight quotation, from the course's landscape lesson

The current process reads a request by email, checks three carrier portals, applies margin rules, gets senior review over a threshold, and emails the quote.

The course's answer. Reading the email is perception, including spotting what is missing early, since that is cheaper than discovering it later. Checking portals is ReAct with tool use, one cycle per portal, each able to fail on its own, with retry, a cached rate, or a flag. Margin rules are a rules engine called as a tool, followed by a light sanity check, because a wildly out of range quote signals an upstream error. Senior review is human in the loop through routing, and the agent must pause and wait. Notifying afterward is oversight in name only. Sending is a tool call, and the highest harm step, since it commits the company.

The course recommends CrewAI, with intake, market research, pricing, and dispatch agents, partly because stakeholders can understand those roles. It says LangGraph is reasonable if routing grows complex, and AutoGen is a poor fit since there is nothing to debate. Its risks are portal fragility, pricing rule drift, needing version control and sign off, an overly broad email tool, and missing observability. It says the question is really asking whether you think beyond the happy path.

My note: I lean toward LangGraph, because a durable pause for human approval is the crux of this workflow, but the course's answer is fine if you defend it. I would also ask whether carrier APIs can replace the portals.

### Design four: smart parking, from the course's thought exercise

The system already has sensor, booking, user, and notification microservices that are reactive. The need for agents comes from proactive rerouting, dynamic pricing, personalization, and reasoning through sensor failures.

The course's answer is a hybrid. A central manager runs the main workflow, which is easier to manage and debug, while zone agents hand off directly to neighboring zones when one is full, for resilience.

Driver agent, evolved from the user service. A natural language request triggers it. It extracts entities such as location and price, handles clarification, and sends a structured request to the manager. It remembers the conversation, plus the profile and past choices. Its pattern is ReAct, because it talks to a person and benefits from visible step by step progress.

Zone agent, evolved from the sensor service. A manager query or sensor data triggers it. It prices dynamically, predicts which spots will free up, and infers a failed sensor's status from neighbors. Its tools cover sensors, local events, and traffic. It keeps an occupancy cache and history. The pattern is plan and execute, since the steps are predictable and easy to audit.

Reservation agent, evolved from the booking service. A reserve request triggers it. It verifies the user, locks the spot, processes payment, finalizes, and notifies, keeping a transaction log. The pattern is plan and execute, with a rollback that unlocks the spot if payment fails.

The course's guardrails are, first, a PII redaction filter between agents and in logs, enforced by the driver and manager, masking something like a license plate. Second, a tool safeguard on payments, enforced by the reservation agent, with argument validation such as a maximum duration, and a rate limit per user to prevent double billing.

My note: I would push back in two places, and an interviewer may enjoy it. A booking has fixed steps, so it needs a coded workflow, not an LLM generating a plan. And a rate limit is a weak defense against double charging, where an idempotency key is the right tool. I would also enforce price caps in code.

### A quick extra: the DevOps agent, from the glossary

An agent monitors build logs, restarts services, and recalls past failures. The course says it is goal based, model based, and learning, all at once. A simple reflex agent fails for three matching reasons. It cannot handle partial observability, since a timeout in service A may come from a leak in service C. It has no memory, so it repeats a restart that already caused an outage. And it cannot weigh competing options. The lesson: agent types map to the specific problems that require them.

### Design five: a deep research agent

Clarify depth, sources, and citation standards. Use manager worker. The manager makes a research plan, and asks the user to confirm it for long runs. Workers search in parallel, each with a narrow question and its own clean context, and return structured findings with citations. A critic verifies that each claim has a supporting source. Treat web pages as untrusted, and give those workers no private data or sending ability, which breaks the trifecta. Budget by searches and tokens, with a stop rule for diminishing new information. Evaluate on citation accuracy and coverage against an expert rubric.

### Design six: coding agent

Tools: read, search, edit, run tests, in a sandbox with no production credentials and restricted network. Use a ReAct loop: observe test output, reason, revise. Keep context under control by searching rather than loading the repository. Humans review through pull requests, so the merge is the gate. Add verification by requiring tests to pass, and watch for the agent modifying tests to get a green result. Evaluate on a task benchmark, and on merge rate and rework rate in production.

### Design seven: your home turf, a behavioral case management assistant

This one is built around your own work on BCM and EAP clinical workflows, so you can speak from experience. Expect a healthcare interviewer to push hard here. Use your own spec's details wherever they differ from this sketch.

Goal. Reduce clinician administrative load and breaches of service level agreements in intake triage, case routing, note summarization, risk flagging, outreach, and session authorization, without any drop in safety. Metrics: time to first contact, service level breach rate, clinician edit rate on summaries, and sensitivity of risk flag detection.

Gate. Service level clocks, scoring of validated instruments such as the PHQ 9 and GAD 7 questionnaires, and authorization rule checks are deterministic code. LLMs read narrative text, draft summaries, and suggest routing.

Blast. Authority boundaries vary by workflow. Summaries are drafts a clinician signs. Routing is a suggestion. Risk is the asymmetric case: the agent may raise a flag but must never lower or clear one. Missing a risk is much worse than an extra review, so tune for recall, and route to a human immediately. The agent gives no clinical determination.

Shape. A LangGraph stateful workflow with conditional routing and interrupt points for human approval, using specialized workers per workflow. A persistent checkpoint fits long lived cases well. I would avoid decentralized handoffs here, because auditors want one clear control path.

Parts. Retrieval over policies and care guidelines, ideally with GraphRAG when questions span relationships between members, providers, and authorizations. Memory is scoped per case, with strict access. Substance use records can fall under 42 CFR Part 2. Be current on this, because the 2024 final rule moved it closer to HIPAA, with compliance required by February 16, 2026. A single consent can now cover future treatment, payment, and operations, and covered recipients can generally redisclose under HIPAA. Stricter protections remain. Records cannot be used in proceedings against the patient, and substance use counseling notes kept separately generally need their own consent. So tag records and consent scope at the data layer, and enforce it there, not in a prompt. Have counsel confirm how this applies.

Guard. De-identify or minimize data before model calls where possible. Enterprise deployment with appropriate agreements. Audit every retrieval and action. Grounding checks require summaries to cite source notes. Duty to warn situations such as Tarasoff go immediately to a clinician and are never decided by the agent.

Break. If the model is unavailable, fall back to the manual queue with service level alerts still running, since those are code. Keep a safety net on risk flags by running rules based screening in parallel with the LLM.

Prove. Retrospective evaluation on de-identified historic cases, with clinicians labeling. Report recall on risk with confidence intervals, and check for performance differences across populations. Shadow mode, then clinician assisted drafts. Monitor for drift and automation bias, the same overreliance that the ChainBuddy study warns about.

## Section 12: Rapid fire review

Here are short questions with short answers. Pause after each question and try answering before I do.

What is the difference between a model and an agent? A model maps input to output and stops. An agent uses models in a loop with tools, memory, and goals.

Name the five steps of the agent loop. Perceive, Recall, Reason, Act, Store.

Name the three memory layers. Short term, long term, and external knowledge. Episodic is a fourth from the appendix.

Plan and execute or ReAct? Known, stable task: plan and execute. Changing environment: ReAct. Often both, with a plan at the top and ReAct inside each step.

Manager worker's main weakness? Single point of failure and bottleneck.

Decentralized handoff's main weakness? Loops, dropped responsibility, and hard debugging.

What is credit assignment? Knowing which part of a design caused the result. Eureka solves it by exposing reward components.

What is the grounding problem? Mapping a correct high level intention to the exact element or object in the environment.

What is Set of Mark? Overlaying numbered labels on a screenshot, so the model can reference targets precisely.

What is context clipping? Keeping all thoughts and actions, but only recent observations. WebVoyager keeps three screenshots.

What is an ablation study? Removing one component and measuring the drop, to prove its value.

Human in the loop, on the loop, above the loop? Approves before action, supervises during action, reviews after action.

Three guardrail placement points? Input, output, and before tool use.

Why hand off a work ticket? Structured specs reduce ambiguity and are cheaper than a transcript.

What is heterogeneous model tiering? Using cheaper, faster models for simple steps and stronger models for hard ones.

What is minimal footprint? Least privilege for agents: only the tools and permissions the task requires.

Why not one big prompt that simulates all roles? The course's answer for MACRS has five parts: the prompt gets brittle and hard to debug, state and memory are less explicit, decisions are less interpretable, changes are harder to localize, and without arbitration the dialogue meanders.

Why does a high eval score not end the discussion? The metric can be gamed or may miss intent, as with reward exploits and hallucinated completion. Check behavior and outcomes.

What is the biggest failure category for WebVoyager? Navigation and recovery, at 44.4 percent of failures.

How do you stop runaway loops? Step, cost, and time caps, plus stuck detection, with a defined exit.

Where do you enforce authorization? In the tool layer with code, using the end user's identity. Not in the prompt.

Numbers worth remembering. MACRS: four LLM calls per turn. Eureka: sixteen candidates, fifty two percent improvement, twenty eight point six percent drop without reflection. WebVoyager: 643 tasks, 59.1 percent success, seven actions, three screenshots. MuLan: 77.55 falling to 60 without the critic. ChainBuddy: twelve participants.

Mistakes I would avoid. Jumping to a multi agent diagram before asking about the goal. Using an LLM for arithmetic or policy rules. Treating the system prompt as a security boundary. No exit conditions. No evaluation plan beyond "we would test it." Listing frameworks in place of reasoning. Never naming a trade off.

Phrases worth having ready. "I would keep this step deterministic." "What is the blast radius if this is wrong?" "That is a load bearing decision, and this one is swappable." "I would prove that with an ablation." "I would measure cost per successful task." "The model proposes, and code enforces."

That last one is the course in a sentence, and it is a good place to stop listening. Good luck, Arup.

## Section 13: NotebookLM prompts (skip while listening)

After uploading this file, try these prompts.

1. Act as a staff level interviewer at a company hiring a forward deployed engineer. Give me an agentic system design prompt, ask one question at a time, and critique my answers against the Goal, Gate, Blast, Shape, Parts, Guard, Break, Prove framework.

2. Quiz me on which orchestration pattern fits each of ten new scenarios, and make me justify each choice with a trade off.

3. Using the case studies, give me five "what evidence supports that design choice?" questions.

4. Challenge my behavioral case management design as a skeptical compliance officer, then as a skeptical chief medical officer.

5. Generate an Audio Overview focused only on Section 8, with the hosts comparing how each case study handles feedback.

6. Create flashcards from Section 12.

## Section 14: Source notes and caveats (skip while listening)

Coverage. After a self review, I went back and read the seven implementation lessons and the full loop control lesson, which had come through partially. I also opened the hidden answers on eight pages and reconciled them with this document. The remaining coverage limits are these. I removed code and raw logs from the implementation lessons, and one lesson's page text cut off after 50,000 characters, inside a long log sample, so anything after that is unread. I did not submit quizzes, so answers marked as mine are unconfirmed. I did not run the hands on project, or complete the interactive mock interviews, "Design Your First Agent," "Thinking Through Eureka," and the final "Agentic System Design." Do those yourself, as they are the closest practice the course offers.

Where the course disagrees with itself. The cheatsheet describes MACRS reflection as reviewing recommendations before output, and the glossary lists its agents as preference modeling, act planning, and reflection. The dedicated MACRS lessons instead describe asking, recommending, and chit chat responders plus a planner, with reflection reacting to user feedback in later turns. This document follows the dedicated lessons. The cheatsheet also labels Eureka as a planning example and ChainBuddy as routing, while the Eureka lessons emphasize ReAct style loops and evolutionary search, and ChainBuddy's backend is plan and execute. Trust the dedicated lessons here too.

Numbers. All research figures here are as the course presents them. I did not check them against the original papers. Before quoting one in an interview, especially the MACRS comparison table, look at the paper. Saying "roughly" is safer than false precision.

Interpretation. The course attributes the ChainBuddy perception gap to the Dunning Kruger effect. Twelve participants is a small sample, so treat that as a plausible reading, not an established finding. Also, Eureka's and MuLan's results come from research benchmarks, not production deployments, so present them as evidence for a design idea, not as a guarantee.

Model names. The course names various model versions, and some of its examples are dated. Use tiers in your answers rather than version names.

Audio. The runtime is an estimate at 150 words a minute, and I have not heard this through a real voice. Names such as LLaVA and SQLite may come out oddly, so try five minutes first.

Regulatory. The behavioral health material is a practice sketch, not legal or compliance guidance. The 42 CFR Part 2 summary comes from one law firm's client alert, so defer to your own specification and to counsel.

Sources checked for this version:

- Crowell and Moring, on the Part 2 final rule: https://www.crowell.com/en/insights/client-alerts/hhs-finalizes-significant-modifications-aligning-part-2-regulations-with-hipaa
- Simon Willison, The Lethal Trifecta: https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
- FlowHunt, on A2A joining MCP at the Linux Foundation: https://www.flowhunt.io/blog/agentic-ai-foundation-a2a-mcp-standards/
- PromptLayer, on the agentic system design interview: https://blog.promptlayer.com/the-agentic-system-design-interview-how-to-evaluate-ai-engineers/
- Aced, Google forward deployed engineer interview guide: https://www.aced.io/guides/google-forward-deployed-engineer-interview
