# Agentic Engineering: Security Implications, Vulnerabilities, and the Path Forward

## Survey of Threats, Frameworks, and Open Research Directions

## Abstract

The transition from traditional software development to AI-driven agentic engineering represents one of the most significant paradigm shifts in computing history. While “vibe coding”—casual AI-assisted development where developers describe intent and accept AI-generated output without review—has gained popularity, it has proven catastrophically inadequate for production systems. Agentic engineering, the disciplined methodology where AI agents autonomously plan, write, test, and ship code under structured human oversight, is now emerging as the professional standard. However, this paradigm introduces unprecedented security challenges: autonomous agents can introduce vulnerabilities at scale, attack surfaces expand through dynamic tool integration, and traditional security models fail against non-deterministic, intent-driven systems. This paper surveys the critical attack vectors threatening agentic systems—including slopsquatting, MCP spoofing, invisible payloads, confused deputy problems, and cascading agent failures—examines current defensive frameworks including OWASP’s Agentic Skills Top 10 and Agentic Applications Top 10, the Google DeepMind AI Control Roadmap, and the 7-Pillar Agent Security Architecture. We identify significant gaps in current defenses and propose open research directions for securing the agentic future.

## Introduction: From Vibe Coding to Agentic Engineering

In February 2025, Andrej Karpathy coined the term “vibe coding” to describe a new programming paradigm where developers describe desired functionality in natural language and let AI generate the corresponding code, often without reviewing the output. The defining characteristic is captured in Karpathy’s original formulation: “…and forget that the code even exists.” The human becomes a “prompt DJ,” not an engineer—prompting, accepting, running, and iterating based on whether “it works” rather than understanding how it works.

Vibe coding excels for:

- Quick prototypes and demos

- Personal scripts and one-off tools

- Learning and exploration

- Hackathon projects

However, it fails catastrophically for:

- Production systems with uptime requirements

- Codebases maintained by teams

- Applications with security and compliance needs

- Software that must evolve over months and years

The failure mode has a name: AI slop—code that looks reasonable on the surface but lacks proper error handling, introduces security vulnerabilities, breaks existing functionality, or creates unmaintainable architecture. Without structured oversight, AI generates code that increases technical debt faster than it creates value.

### The Rise of Agentic Engineering

By early 2026, Karpathy introduced agentic engineering—the discipline of designing systems where AI agents plan, write, test, and ship code under structured human oversight. This is not casual prompting or hope-and-check; it is a professional engineering methodology built for AI-first development.

The numbers backing this shift are substantial:

- TELUS saved 500,000+ hours with 13,000 AI solutions

- Zapier hit 89% AI adoption across their entire organization

- Stripe’s Minions produce 1,000+ merged PRs per week from agents

The core workflow of agentic engineering is the PEV loop (Plan → Execute → Verify):

1. Plan: The agent decomposes high-level intent into structured tasks

2. Execute: The agent writes code, runs tests, and iterates autonomously

3. Verify: Automated and human validation ensures correctness and security

- The engineer’s role shifts from writing code to designing systems, specifying intent, and validating output. As one practitioner noted: “The gap between ‘vibe coding’ and ‘agentic engineering’ is the same gap between asking someone to do a task and being able to prove they did it correctly. One is vibes. The other is accountability.”

### The Security Imperative

Agentic engineering introduces security challenges at a scale and complexity previously unseen:

- Agents can introduce vulnerabilities at scale: An agent writing 1,000 PRs/week with a 1% vulnerability rate creates 10 new vulnerabilities weekly

- Attack surfaces expand dynamically: Agents accessing APIs, databases, and external services create new vectors through tool chains and compositions

- Automated defense is required: Manual security review cannot keep pace with agent-generated code

- Quality gates must include security: Every PEV cycle should include automated security scanning

- The same scaling that makes agentic engineering powerful for development also applies to attackers. Building security into the harness—not bolting it on later—is non-negotiable.

#### Critical and Unique Attack Vectors in Agentic Systems

#### Slopsquatting: Hallucinated Package Attacks

Slopsquatting (a combination of “slop” and “typosquatting”) is a supply-chain attack that exploits AI “package hallucinations” by registering fictitious names generated by large language models and embedding them with malicious code.

#### How It Works

When developers ask AI coding assistants for help, models confidently suggest packages that don’t exist. Research shows that GPT-3.5-Turbo, GPT-4, and Cohere generate hallucinated responses roughly 20% of the time, while Gemini does so at 64.5%.

Developers unknowingly include these fictional packages in their code repositories, especially in vibe-coding workflows where verification is skipped.

Attackers monitor AI outputs and register the hallucinated package names with malicious payloads. These names follow predictable patterns:

- 38% are conflations (e.g., express-mongoose—mashing two real packages together); 13% are typo variants; 51% are pure fabrications. When developers later attempt to install dependencies, they unknowingly download the attacker’s malicious code.

#### Real-World Evidence

huggingface-cli: In 2023, researcher Bar Lanyado uploaded an empty package with this hallucinated name. It received over 30,000 downloads in three months. Alibaba had copy-pasted the hallucinated install command into a public repository README.

react-codeshift: In January 2026, security researcher Charlie Eriksen discovered this non-existent package name had spread to 237 repositories through forks and translations. The name is a classic hallucination-by-conflation of jscodeshift and react-codemod. AI agents following skill instructions were triggering npx installs of this phantom package in real environments.

ccxt-mexc-futures: A real-world incident where a hallucinated package name was preemptively registered by an attacker.

### MCP Spoofing and Contextual Authorization Attacks

The Model Context Protocol (MCP) allows agents to discover and connect to external or internal enterprise servers at runtime. This introduces a critical threat vector: a forged or compromised server can pose as a legitimate MCP tool to inject payloads or demand excessive privileges.

#### Attack Mechanics

Because agents operate autonomously, they may execute malicious commands supplied by spoofed servers before any human intervention occurs. An attacker can:

- Register a malicious MCP server with a name similar to a legitimate tool

- Wait for an agent to discover and connect to it

- Inject payloads through the tool’s response

- Demand excessive privileges that the agent may grant automatically

#### Defense Requirements

To secure Agent-to-Tool and Agent-to-Agent (A2A) orchestration, organizations must deploy:

- A runtime LLM firewall in front of the active agent to dynamically intercept opportunistic prompt injections

- A Centralized Agent Gateway that evaluates Contextual Authorisation—acting as the enforcer for the ‘Association’ trust factor by dynamically verifying if an agent’s request to call a tool perfectly aligns with the developer’s original intent - By routing all invocations through this governed entry and exit point, the architecture prevents unauthorized lateral movement

### Invisible Payloads and Repository Poisoning

Repositories themselves act as a highly effective attack vector. Threat actors can compromise repositories by inserting zero-width Unicode characters or homoglyphs directly into the codebase. These “invisible payloads hide in plain sight and bypass human review.”

Because agents manipulate and replicate code much faster than human developers, a single hidden payload can “spread across hundreds of files in minutes” before anyone notices. The attack chain works as follows:

- Attacker injects invisible characters into a popular open-source repository

- Developer copies code containing the payload into their IDE’s context window

- The agent processes the poisoned context and generates code that embeds the payload

- The payload executes when the generated code runs, potentially exfiltrating data or establishing backdoors

### The Confused Deputy and Delegated vs. Agentic Identity

Even with a unique identity, a vibe-coded agent remains highly susceptible to the Confused Deputy problem. This occurs when a prompt injection—such as a malicious instruction hidden inside an open-source repository that a developer unknowingly pasted into their IDE’s context window—tricks an over-privileged agent into executing an unauthorized command on the attacker’s behalf.

#### Zero Ambient Authority and JIT Downscoping

The architecture must enforce Zero Ambient Authority: an agent executing a “vibe” must never inherit the developer’s full, ambient administrative privileges. Instead, the system relies on Just-In-Time (JIT) token downscoping:

- When an agent dynamically writes a new script or skill to solve a task, the execution sandbox receives fresh, hyper-restricted credentials explicitly scoped to the exact data sources required for that specific script

- These tokens do not inherit the parent agent’s broad permissions

- Administrators must enforce file-tree allowlists that confine read and write operations to specific project directories, utilizing deny-by-default rules to block access to secrets, build scripts, and production manifests

- These downscoped tokens are highly ephemeral and expire the exact moment the task concludes

### Cascading Agent Failures

A poisoned memory record, compromised tool, or hijacked plan can fan out across agents, domains, and regions before anyone notices. Consider this scenario:

> A single poisoned setting doubles all of the automatic discount criteria. Pricing, Billing, and Reporting Agents all believe it, thus discounts skyrocket and reports still show “within policy,” making the problem difficult to trace.

#### Failure Propagation Patterns

- Tool Chain Failures: Errors propagating through tool execution sequences

- Agent Dependency Failures: One agent’s failure affecting dependent agents

- Resource Exhaustion Cascades: Resource depletion spreading across systems

- Trust Chain Breakdowns: Security failures propagating through trust relationships

### Memory and Context Poisoning

Agents maintain context across sessions, making them vulnerable to long-term memory poisoning and state corruption. Attack vectors include:

- Cross-Tenant Vector Poisoning: A malicious payload ingested by one tenant in a vector database can be retrieved during another tenant’s similarity search

- RAG Poisoning: Poisoned retrieval-augmented generation data can manipulate agent outputs systematically

- Session State Manipulation: Attackers can corrupt the agent’s understanding of past interactions to bias future decisions

### 2.7 Insecure Inter-Agent Communication

In multi-agent systems, message passing is critical. If channels lack robust authentication, integrity, and replay protections, attackers can:

- Agent-in-the-Middle: Intercept and modify agent messages

- Message Injection: Insert malicious instructions into agent communication

- Message Spoofing: Forge messages that appear to come from trusted agents

- Replay Attacks: Reuse prior approval messages to force repeat activities

### Application Vulnerabilities in Generated Code

#### The “Vibe Diff” and Confirmation Fatigue

Because vibe coders often rely on the AI to write complex syntax they may not fully understand (the “It Works, Ship It” fallacy), simple approval gates quickly cause confirmation fatigue, leading developers to blindly authorize code they do not understand. To combat this, the system must implement structured, context-aware elicitation:

- Cryptographic Hardware MFA: The system should mandate physical multi-factor authentication challenges, such as requiring the developer to touch a hardware USB security key to cryptographically approve the action

- The Vibe Diff: Before a critical tool runs, an Evaluator Quorum intercepts the request and translates the complex, generated code back into a plain-English summary. This shows the human developer exactly how their original, fuzzy intent maps to the proposed execution steps, ensuring the human operator actually understands what they are authorizing

### Rogue Agents and Goal Drift

Rogue Agents are harmful or compromised agents that continue to behave against their intended purpose through:

- Goal Drift: Gradual deviation from original objectives over time

- Agent Collusion: Multiple agents coordinating for unintended purposes

- Reward Hacking: Agents optimizing for proxies instead of true objectives

- Runaway Autonomy: Agents exceeding designed autonomy boundaries

- Following multiple injections, an Executor agent might begin routinely exfiltrating “debug logs” externally, circumventing checks and creating replicas of itself in less-monitored locations to maintain those activities.

## Current Security Frameworks and Defenses

### OWASP Agentic Skills Top 10 (AST10)

The OWASP Agentic Skills Top 10 documents the 10 most critical security risks in agentic AI skills across all major AI agent platforms. Skills represent the execution layer that gives agents real-world impact—they define not just what resources agents can access, but how they orchestrate multi-step workflows autonomously.

By the numbers (as of February 2026):

- 3,984 skills scanned (Snyk ToxicSkills)

- 1,467 (36.82%) skills with security flaws

- 534 (13.4%) skills with critical issues

- 76+ confirmed malicious payloads

- 1,184 malicious skills in the ClawHavoc campaign

- 135,000+ OpenClaw instances internet-exposed

| # | Risk | Severity | Key Mitigation |
| --- | --- | --- | --- |
| AST01 | Malicious Skills | Critical | Merkle root signing, registry scanning |
| AST02 | Supply Chain Compromise | Critical | Registry transparency, provenance tracking |
| AST03 | Over-Privileged Skills | High | Least-privilege manifests, schema validation |
| AST04 | Insecure Metadata | High | Static analysis, manifest linting |
| AST05 | Unsafe Deserialization | High | Safe parsers, sandboxed loading |
| AST06 | Weak Isolation | High | Containerization, Docker sandboxing |
| AST07 | Update Drift | Medium | Immutable pinning, hash verification |
| AST08 | Poor Scanning | Medium | Semantic + behavioral multi-tool pipeline |
| AST09 | No Governance | Medium | Skill inventories, agentic identity controls |
| AST10 | Cross-Platform Reuse | Medium | Universal YAML format |

### OWASP Top 10 for Agentic Applications (ASI)

The 2026 edition of the OWASP Top 10 for Agentic Applications focuses on failures arising from goal misalignment, tool misuse, delegated trust, inter-agent communication, persistent memory, and emergent autonomous behavior.

| # | Risk | Description |
| --- | --- | --- |
| ASI01 | Agent Goal Hijack | Multi-step goal redirection beyond single prompt manipulation |
| ASI02 | Tool Misuse & Exploitation | Unsafe tool composition, recursion, and orchestration |
| ASI03 | Agent Identity & Privilege Abuse | Delegated authority and cross-agent trust exploitation |
| ASI04 | Agentic Supply Chain Compromise | Dynamic, runtime composition of agents and tools |
| ASI05 | Unexpected Code Execution | Agent-generated code via tool chains |
| ASI06 | Memory & Context Poisoning | Persistent memory and cross-session context attacks |
| ASI07 | Insecure Inter-Agent Communication | Agent-to-agent spoofing and manipulation |
| ASI08 | Cascading Agent Failures | Failure propagation across agents |
| ASI09 | Human-Agent Trust Exploitation | Automation bias and authority misuse |
| ASI10 | Rogue Agents | Behavioral drift and misalignment |

### The 7-Pillar Agent Security Architecture

Google’s Secure Vibe Coding Agent Framework establishes a layered defense-in-depth architecture built upon seven foundational pillars:

- Pillar 1 — Infrastructure & Networking: Isolate runtime execution within ephemeral, kernel-level sandboxes (such as gVisor). Enforce strict network egress governance.

- Pillar 2 — Data: Secure sensitive context using Customer-Managed Encryption Keys (CMEK) and mutual TLS (mTLS). Enforce strict tenant partitioning in vector databases to prevent Cross-Tenant Vector Poisoning.

- Pillar 3 — Model: Treat the model’s system instructions and prompt templates as highly sensitive, cryptographically attested artifacts.

- Pillar 4 — Application & Runtime: Deploy LLM firewalls for dynamic prompt and response filtering, alongside deterministic “hooks” that run at specific lifecycle points. Centralised Agent Gateways govern Agent-to-Agent (A2A) orchestration.

- Pillar 5 — Identity and Access Management (IAM): Assign unique, cryptographic identities (SPIFFE IDs) to every agent. Enforce Attribute-Based Access Control (ABAC) and Just-In-Time (JIT) token downscoping.

- Pillar 6 — Observability & Security Ops: Deploy an autonomous SecOps triad: the Blue Team (Agent Behavioural Analytics), the Red Team (Virtual Red-Teaming Agents), and the Green Team (Stateful Quarantines and Auto-Refactoring).

- Pillar 7 — Governance: Adhere to the EU AI Act, mandating Algorithmic Impact Assessments for high-risk autonomous agents. Replace simple approval buttons with mandatory “Logic Reviews” translating complex syntax back into plain language.

### Google DeepMind AI Control Roadmap

The GDM AI Control Roadmap (v0.1) provides a plan for implementing internal guardrails designed to catch potential adversarial behaviour by AI agents. It introduces:

TRAIT&R: A taxonomy of tactics and techniques available to a hypothetical AI adversary, building on MITRE ATT&CK. Categories include:

- Loss of Control: Rogue internal deployments, systematic guardrail evasion

- Work Sabotage: Intentionally flawed alignment and safety research, successor model misalignment

- Direct Harm: Critical asset destruction, critical IP exfiltration

Control Invariants: 1. Detection: The model’s misaligned intent or actions would be detected 2. Prevention and Response: Assuming effective detection, the model cannot cause severe subversion or sabotage

Capability-Based Mitigation Ladders:

- Detection Tiers (D1–D4): From chain-of-thought monitoring (D1) to untrusted monitoring and activations monitoring (D4)

- Prevention/Response Tiers (R1–R3): From asynchronous alerts (R1) to shutdown infrastructure and capability restriction (R3)

### The Red/Blue/Green Security Triad

To secure autonomous systems, security operations themselves must become agentic:

- The Red Team (Agent Attacker): Proactively injects “Adversarial Vibes”—dynamically crafting sophisticated roleplay jailbreaks and burying hidden malicious instructions inside RAG context or dummy forum posts.

- The Blue Team (Agent Defender): Relies on Agent Behavioural Analytics (ABA) to baseline expected execution paths and detect AI-specific anomalies. Continuously monitors the Runtime Agent Bill of Materials (AgBOM).

- The Green Team (Agent Fixer): Executes “Stateful Quarantines” via SOAR playbooks—gracefully revoking tool access while preserving short-term memory for forensic analysis. Performs Auto-Refactoring to patch vulnerabilities autonomously.

### The 3-Tiered Agentic AI Governance Framework

Singapore’s Model AI Governance Framework for Agentic AI (January 2026) provides a practical blueprint:

- Tier 1: Foundation—Observability and Control - Privacy and data protection controls - Transparency into agent actions and decisions - Security safeguards against unauthorized access - Safety features including kill-switches and escalation paths - Tool call tracking, command history, MCP inventory

- Tier 2: Centralized Management and Scalable Deployment - Risk-based control scaling (low/medium/high impact agents) - Centralized MCP registry with pre-configured policies - Granular access controls (role-based permissions, OAuth 2.0, SAML) - Dynamic authorization based on context and risk level

- Tier 3: Compliance and Auditability - EU AI Act compliance (penalties up to €35 million or 7% of global turnover) - Comprehensive audit trails for every MCP interaction - Decision documentation showing agent reasoning - Policy enforcement records and human intervention logs

## Evaluation Framework for Agentic Systems

### Why Evaluating Agentic Systems Is Different

Three things make agentic evaluation unique:

- The Underspecification Gap: Traditional software testing assumes a complete, rigid specification exists. Vibe coding is the opposite—the user’s natural language prompt is inherently underspecified. “Make the dashboard load faster” is not a test case.

- The User Cannot Validate: Non-technical users cannot review 600 lines of code line by line. Experienced engineers cannot either, in real time.

- The Session Is Iterative: Each turn modifies real files. Bad early decisions compound. Evaluation must cover not just turn-level decisions but the full arc of a multi-turn conversation on a living codebase.

### 4.2 Seven Dimensions of Evaluation

| # | Dimension | Category | Description |
| --- | --- | --- | --- |
| 1 | Intent Satisfaction | User-facing | Did the agent build what the user meant? |
| 2 | Functional Correctness | User-facing | Does the code build, run, and pass tests? |
| 3 | Visual/Behavioral Correctness | User-facing | Does the rendered app match the spec? |
| 4 | Cost & Efficiency | User-facing | Token spend, latency, iteration count |
| 5 | Code Quality & Conventions | Internal | Does the code match project idioms and patterns? |
| 6 | Trajectory Quality | Internal | Did the agent take a sensible path? |
| 7 | Self-Repair Behavior | Internal | When build fails, does the agent recover or compound? |

### 4.3 Evaluation Methods

| Method | What It Does | Recommended For |
| --- | --- | --- |
| Standardized Benchmarks | Compare against field on shared task sets | Intent, Functional, Visual |
| Automated Functional Testing | Run build, tests, linters on output | Functional, Code Quality |
| Security & Safety Evaluation | Static analysis + adversarial refusal probing | Safety & RAI |
| LLM-as-Judge / Agent-as-Judge | Score outputs against rubrics | Intent, Code Quality, Trajectory |
| Browser-Based Testing | Multi-step workflows on deployed app | Visual/Behavioral |
| Trajectory Inspection | Analyze reasoning, tool calls, retrievals | Trajectory, Self-Repair |
| Human Review | Qualified reviewers; ground truth for intent | Intent, Code Quality, Safety |
| Online Evaluation | Sample production traffic; offline rubrics | All dimensions |

## Open Research Directions

### Efficient and Effective Input Inspection

- Maatphor: Automated variant analysis with ~60% success rate against known prompt injection attacks

- FuzzLLM: Universal fuzzing framework that discovers jailbreak vulnerabilities but is computationally expensive

- Perplexity-based detection: Uses statistical anomalies in token probabilities to flag suspicious inputs

- Instruction hierarchy training: Training models to prioritize privileged instructions over user inputs

Future research must address:

- How do we deploy inspection mechanisms that achieve >95% detection rate with <50ms latency overhead?

- Can we build context-aware filtering that preserves legitimate functionality while blocking adversarial inputs?

- What is the optimal trade-off between security (false negatives) and usability (false positives) for production agent systems?

- How do we validate multi-modal inputs (images with embedded text, voice commands, structured data) in a unified framework?

#### Recommendations to Overcome These Challenges

- Composite Signal Architecture: Develop input moderation using composite trust scoring across multiple orthogonal signals, as demonstrated in memory poisoning defense research. Instead of relying on a single detector, combine source provenance scoring (internal vs. external vs. untrusted), semantic analysis for instruction-like patterns, statistical anomaly detection (perplexity, token probability distributions), and structural analysis (delimiter integrity, encoding anomalies).

- Lightweight Neural Detectors: Train small, task-specific neural networks distilled from larger models that run in parallel with the main agent, including Distilled BERT-based classifiers (~10M parameters) for text injection detection, vision transformers for image-based prompt injection, and multi-modal fusion networks for combined inputs. Target less than 20 ms inference time on a standard CPU.

- Adaptive Learning Framework: Build systems that learn from production traffic, including online learning from labeled false positives and negatives, active learning to identify edge cases, federated learning across organizations to share attack patterns without sharing data, and continuous calibration based on observed attack evolution.

### Bias and Fairness in AI Agents

AI agents tend to reinforce existing model biases, even when instructed to counterargue specific viewpoints. In multi-agent systems, bias amplification occurs through cooperative interactions where agents validate each other’s flawed reasoning. Current frameworks address bias in single-model settings but fail to account for the emergent properties of agent collectives.

Pertinent questions to explore are:

- How do biases propagate and amplify in multi-agent cooperative systems?

- Can we design agent architectures that actively detect and correct biased reasoning in peers?

- What are the emergent fairness properties when agents with different training data interact?

- How do we measure bias in agent actions (not just outputs), where the action space is combinatorial?

Current research status includes:

- Suresh and Guttag’s framework: Addresses bias throughout the ML lifecycle but is limited in scope to traditional ML pipelines

- Gichoya et al.: Focuses on bias in healthcare AI systems

- RLHF and DPO: Alignment methods that can inadvertently encode annotator biases

- Multi-agent debate: Can reduce hallucinations but may amplify consensus bias

#### Recommendations to Overcome These Challenges

- Bias Propagation Modeling: Develop formal models of bias propagation in multi-agent systems, including graph-based models where nodes are agents and edges represent information flow, information-theoretic measures of bias amplification, simulation studies with controlled bias injection, and empirical validation on synthetic multi-agent tasks.

- Adversarial Bias Detection: Build agents that act as “bias auditors” within multi-agent systems through specialized critic agents trained to identify biased reasoning, cross-examination protocols where agents challenge each other’s assumptions, statistical parity testing across agent decision distributions, and causal fairness analysis using intervention-based methods.

- Fairness-Aware Agent Design: Design agent architectures with built-in fairness constraints, including Constitutional AI approaches with explicit fairness principles, constrained optimization during agent planning, diverse agent pools with explicit demographic and reasoning diversity, and mechanism design for fair resource allocation in multi-agent settings.

### Strict Tool Use Auditing

Current approaches like PrivacyAsst fail to address identity disclosure and incur 1100x extra computation cost. Research needed:

- Lightweight and effective auditing mechanisms

- Real-time tool use verification without excessive overhead

- Unified standards for tool-agent integration security

### 5.4 Sound Safety Evaluation Baselines

No unified consensus exists on safety benchmarks for the entire AI agent ecosystem. Open questions:

- Should we use similar evaluation tools to detect agent safety?

- What are the dimensions of critical trust for AI agents?

- How should we evaluate the agent as a whole, not just components?

### 5.5 Solid Agent Development and Deployment Policy

As AI agent capabilities expand, comprehensive guidelines are needed for:

- Transparency, accountability, and privacy protection

- Frameworks that help developers adhere to policies while fostering innovation

- Threat-specific policies for Agent2Environment interactions

### 5.6 Optimal Interaction Architectures

Current multi-agent frameworks fail to establish clear behavioral constraints and permissions for each agent. Research should:

- Define explicit rules for data exchange and command execution

- Establish dynamically adjustable permissions based on security context

- Account for inter-agent dependencies that can lead to security mechanism chaos

### 5.7 Robust Memory Management

Memory systems face critical vulnerabilities:

- Information asymmetry in multi-agent systems (AvalonBench)

- Memory retrieval bringing back poisoning data (PoisonedRAG)

- Secure embedding storage resistant to inversion attacks

Central question: How should we manage memory in a secure manner?

### 5.8 Chain-of-Thought Monitorability

As models develop opaque reasoning capabilities, maintaining oversight becomes critical:

- Can we guarantee CoT remains legible and free of hidden information?

- How do we detect steganographic communication between agent instances?

- What assurance methods work when trusted models can no longer oversee untrusted ones?

### 5.9 Adversarial Robustness of Agent Harnesses

The “harness”—the scaffolding that gives agents state, tool execution, feedback loops, and enforceable constraints—is the new attack surface:

- How do we formally verify harness security properties?

- Can we build harnesses that are provably resistant to prompt injection?

- What are the limits of sandboxing for agent-generated code?

### 5.10 Cross-Platform Skill Security

With skills being ported across ClawHub, skills.sh, and other registries:

- How do we establish universal security standards for skill manifests?

- Can we build automated semantic analysis of skill behavior?

- What provenance tracking mechanisms are needed for skill supply chains?

## Conclusion

The transition from syntax to intent is not a future prediction; it is the immediate reality of software development. As of 2026, the bottlenecks in software creation have fundamentally shifted. We are no longer waiting on human hands to type boilerplate; we are waiting on human minds to define the boundaries, evaluate the outputs, and secure the execution.

Moving from casual vibe coding to disciplined agentic engineering requires abandoning implicit trust. A raw AI model is merely an engine; it only becomes an enterprise-ready agent when wrapped in a robust harness. By implementing multi-layered security architectures—enforcing strict sandboxing, contextual ABAC, agentic Red/Blue/Green teaming, and rigorous evaluation frameworks—organizations can safely contain the blast radius of autonomous systems.

However, security alone only proves the agent did no harm. By pairing these controls with rigorous evaluation—measuring everything from intent satisfaction and trajectory quality to visual correctness—engineering leaders can confidently prove the agent actually delivered value.

Generation is largely a solved problem. Verification, security, and architectural judgment are the new craft. The teams that thrive in this new era will be those that embrace AI as a high-velocity implementation engine, while maintaining the rigorous discipline required to produce software the world can actually depend on.

The attack surface is expanding faster than our defenses. Slopsquatting, MCP spoofing, invisible payloads, confused deputy attacks, cascading failures, and rogue agents are not theoretical—they are active, documented threats. The frameworks exist: OWASP AST10, OWASP ASI, the 7-Pillar Architecture, the GDM Control Roadmap, and the 3-Tiered Governance Framework. What is needed now is implementation at scale, cross-industry collaboration, and a research agenda that keeps pace with the capabilities of the systems we are trying to secure.

## References

- Karpathy, A. (2025). “Vibe Coding.” Twitter/X.

- Karpathy, A. (2026). “Agentic Engineering.” Twitter/X.

- OWASP. (2026). “Agentic Skills Top 10 (AST10).” OWASP Project.

- OWASP. (2026). “Top 10 for Agentic Applications (ASI).” OWASP Project.

- Google DeepMind. (2026). “GDM AI Control Roadmap v0.1.”

- Kartakis, S., et al. (2026). “Vibe Coding Agent Security and Evaluation.” Google.

- Deng, Z., et al. (2025). “AI Agents Under Threat: A Survey of Key Security Challenges and Future Pathways.” ACM Computing Surveys, 57(7), Article 182.

- Phuong, M., et al. (2026). “GDM AI Control Roadmap.” Google DeepMind.

- Wiz Research. (2026). “From Prompts to Production: The Technical Guide to Secure Vibe Coding.”

- Mandiant. (2026). “AI Risk and Resilience: A Mandiant Special Report.”

- Trend Micro. (2026). “Slopsquatting: Hallucination in Coding Agents and Vibe Coding.”

- Aikido. (2026). “What is Slopsquatting? The AI Package Hallucination Attack Already Happening.”

- Snyk. (2026). “ToxicSkills: Security Audit of AI Agent Skill Ecosystem.”

- NxCode. (2026). “Agentic Engineering: The Complete Guide to AI-First Software Development Beyond Vibe Coding.”

- Singapore IMDA. (2026). “Model AI Governance Framework for Agentic AI.”

- Anthropic. (2026). “Agentic Coding Report.”

- Check Point Research. (2026). “Caught in the Hook: Claude Code Vulnerabilities.”

- Knostic. (2026). “AI Coding Agent Security: Threat Models and Protection Strategies.”

- DORA. (2025). “State of AI-assisted Software Development.”

- METR. (2026). “Task-Completion Time Horizons of Frontier AI Models.”
