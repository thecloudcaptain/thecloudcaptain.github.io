---
layout: post
title: "How to Build a Custom GitHub Copilot Agent with Prompts and Skills"
description: "A practical introduction to GitHub Copilot custom agents, reusable skills, prompt files and repository instructions using a simple Azure Networking Agent example."
date: 2026-08-19
author: The Cloud Captain
categories:
  - GitHub
  - AI
tags:
  - GitHub Copilot
  - Custom Agents
  - Agent Skills
  - Prompt Files
  - Azure Networking
  - AI Agents
permalink: /articles/building-a-custom-github-copilot-agent/
---

GitHub Copilot can do much more than respond to one-off prompts.

With **custom agents**, **agent skills**, **prompt files**, and **repository instructions**, you can give Copilot a defined role, reusable specialist knowledge, repeatable tasks, and persistent rules for how it should work inside a repository.

That sounds more complicated than it is.

A useful way to think about the four building blocks is:

```text
Agent        = Who is doing the work?
Skill        = What specialist knowledge can it use?
Prompt       = What task do I want it to perform?
Instructions = What rules apply across the repository?
```

In this article, we will build a simple **Azure Networking Agent**.

Its responsibility will be to review Azure network designs and infrastructure code, identify risks, and provide recommendations around areas such as:

- Virtual Networks and subnets
- Azure Firewall
- routing
- Private Endpoints
- Private DNS
- ExpressRoute
- network security

The objective is not to create the most sophisticated agent possible. It is to understand the structure first.

---

## 1. Start with the Repository Structure

For a repository-scoped implementation, we can organise the files like this:

```text
my-cloud-project/
│
├── .github/
│   │
│   ├── agents/
│   │   └── azure-networking.agent.md
│   │
│   ├── skills/
│   │   └── azure-networking/
│   │       └── SKILL.md
│   │
│   ├── prompts/
│   │   └── review-network.prompt.md
│   │
│   └── copilot-instructions.md
│
├── docs/
│   └── network-architecture.md
│
└── infra/
    └── network.tf
```

Each part has a different purpose.

```text
.github/agents/
    Defines specialised agents.

.github/skills/
    Contains reusable specialist capabilities.

.github/prompts/
    Contains reusable task prompts.

.github/copilot-instructions.md
    Contains repository-wide guidance.
```

Keeping these responsibilities separate becomes increasingly useful as the number of agents and tasks grows.

---

## 2. Create the Azure Networking Agent

Repository-level custom agents are defined as Markdown files inside:

```text
.github/agents/
```

Create:

```text
.github/agents/azure-networking.agent.md
```

A simple agent profile could look like this:

```markdown
---
name: azure-networking
description: Reviews Azure network architecture and infrastructure for connectivity, routing, DNS, private access and network security considerations.
tools: ["read", "search"]
---

You are an Azure Networking specialist.

Your responsibility is to review Azure network architecture and infrastructure configurations and provide practical recommendations.

Focus on:

- Virtual Networks and subnet design
- IP address planning
- routing and User Defined Routes
- Azure Firewall
- Network Security Groups
- Private Endpoints
- Private DNS
- ExpressRoute and hybrid connectivity
- inbound and outbound connectivity
- network resiliency

When reviewing a solution:

1. Understand the existing design before recommending changes.
2. Identify architectural or operational risks.
3. Explain why each issue matters.
4. Recommend practical remediation.
5. Clearly identify assumptions when information is missing.
6. Do not modify infrastructure unless explicitly asked.
```

The YAML section at the top is the agent configuration.

The Markdown below it defines the agent's behaviour.

In this example we have also restricted the agent to the `read` and `search` tools because its initial purpose is assessment and review rather than implementation.

The important point is that the agent definition describes a **role**.

Instead of repeatedly prompting Copilot with:

> Act as an Azure networking specialist. Review routing, Private DNS, Private Endpoints, Azure Firewall...

we define that responsibility once and select the agent when we need it.

---

## 3. Give the Agent Specialist Knowledge with a Skill

The agent knows its role, but we may also want reusable organisational guidance for how Azure networking should be assessed.

This is where an **Agent Skill** becomes useful.

Create:

```text
.github/skills/azure-networking/SKILL.md
```

For example:

```markdown
---
name: azure-networking
description: Azure networking design and review guidance. Use when assessing Azure VNets, routing, firewalls, private connectivity, hybrid connectivity or DNS.
---

# Azure Networking Review Guidance

When analysing Azure networking, evaluate the following areas.

## Addressing

- Check for overlapping address spaces.
- Confirm that subnet sizing allows for expected growth.
- Identify address ranges that could create hybrid connectivity conflicts.

## Routing

- Review User Defined Routes and route propagation.
- Identify asymmetric routing risks.
- Confirm the intended path for inbound and outbound traffic.

## Private Connectivity

- Evaluate whether Azure PaaS services require Private Endpoints.
- When Private Endpoints are used, review the related DNS requirements.
- Confirm that public network access aligns with the intended security model.

## DNS

- Identify the authoritative DNS path.
- Review Private DNS zone placement and virtual network links.
- Consider hybrid name resolution requirements.

## Security

- Review NSG scope and overly broad rules.
- Review Azure Firewall placement and policy intent.
- Prefer least-privilege network access.

## Output

For each finding provide:

1. Finding
2. Risk
3. Recommendation
4. Assumptions or dependencies
```

The skill is not another agent.

It is a reusable package of specialist guidance that Copilot can use when the task is relevant.

A skill directory can also contain supporting resources such as additional Markdown guidance, examples, or scripts. That allows a relatively small agent profile to make use of deeper specialist knowledge without placing everything inside the `.agent.md` file.

---

## 4. How Does the Agent Know Which Skill to Use?

This is an important distinction.

You do **not** normally need to put something like this inside the agent:

```text
Load .github/skills/azure-networking/SKILL.md
```

GitHub Copilot can discover available skills and determine when they are relevant based on:

- the user's prompt
- the skill name
- the skill description

For example, our skill contains:

```yaml
description: Azure networking design and review guidance. Use when assessing Azure VNets, routing, firewalls, private connectivity, hybrid connectivity or DNS.
```

If the task is:

```text
Review this Terraform configuration for Azure Private Endpoint and DNS issues.
```

the Azure networking skill is clearly relevant.

When Copilot chooses to use the skill, the `SKILL.md` instructions are added to the agent's context.

This is one reason the skill description matters. It is not merely documentation for humans; it helps describe **when the capability should be used**.

---

## 5. Create a Reusable Prompt

The agent represents the role.

The skill represents reusable specialist knowledge.

We still need to tell the agent what we want it to do **for a particular task**.

We could simply type a prompt manually every time, but common tasks can also be stored as reusable prompt files.

Create:

```text
.github/prompts/review-network.prompt.md
```

For example:

```markdown
---
agent: 'azure-networking'
description: 'Review the current Azure network architecture and identify risks and recommended improvements'
---

Review the Azure network architecture and infrastructure configuration in this repository.

Focus on:

- network segmentation
- IP addressing
- routing
- Azure Firewall
- Network Security Groups
- Private Endpoints
- Private DNS
- inbound connectivity
- outbound connectivity
- hybrid connectivity
- resiliency

Provide the result using the following structure:

## Current Architecture
Summarise the network design that can be determined from the repository.

## Findings
List the important findings and risks.

## Recommendations
Provide practical recommended changes.

## Assumptions
Identify missing information or assumptions that should be validated.

Do not make infrastructure changes as part of this review.
```

In supported IDEs such as VS Code, prompt files can then be invoked as reusable commands rather than recreating the same detailed task prompt each time.

Here, the prompt file explicitly targets the `azure-networking` custom agent through its `agent` frontmatter, so the reusable task runs with that specialist role.

At the time of writing, GitHub Copilot prompt files are a **public preview** feature and are supported in VS Code, Visual Studio, and JetBrains IDEs, so it is worth checking current GitHub documentation before designing critical workflows around them.

---

## 6. Add Repository-Wide Instructions

There is one more layer to consider.

Some rules are not specific to the networking agent.

For example, imagine this repository contains Azure infrastructure for a Canadian environment and the organisation has decided that:

- Canada Central is the default Azure region
- Terraform is the required Infrastructure as Code language
- managed identities should be preferred over stored credentials
- secrets must not be committed to source control
- destructive infrastructure changes must be clearly identified

These are repository-level rules.

Create:

```text
.github/copilot-instructions.md
```

For example:

```markdown
# Repository Instructions

This repository contains Azure infrastructure and application configuration.

When working in this repository:

- Use Canada Central as the default Azure region unless another region is explicitly required.
- Use Terraform for Infrastructure as Code.
- Prefer managed identities over stored credentials or access keys where supported.
- Do not place secrets, credentials, tokens or connection strings in source control.
- Follow least-privilege RBAC principles.
- Clearly identify destructive infrastructure changes before making them.
- Review existing modules and patterns before introducing new resource structures.
- Keep implementation consistent with the existing repository naming and tagging conventions.
```

These instructions are persistent repository guidance rather than a one-time task.

The difference is therefore:

```text
Azure Networking Agent
    "This is the role I perform."

Azure Networking Skill
    "This is specialist knowledge I can use."

Network Review Prompt
    "This is the task I need to perform now."

Repository Instructions
    "These rules apply while working in this repository."
```

---

## 7. Putting Everything Together

We now have four separate layers:

```text
                         USER TASK
                            │
                            ▼
                Azure Networking Agent
                            │
                     Role + Behaviour
                            │
                            ▼
                    Relevant Skills
                            │
                 Azure Networking Skill
                            │
                            ▼
                  Repository Context
                            │
             ┌──────────────┴──────────────┐
             ▼                             ▼
      network-architecture.md          network.tf
             │                             │
             └──────────────┬──────────────┘
                            ▼
                    Network Review
```

The repository instructions remain applicable across the workflow.

A typical interaction could be:

1. Open the repository in VS Code.
2. Open GitHub Copilot Chat.
3. Select the `azure-networking` custom agent.
4. Invoke the reusable network review prompt, or provide a task directly.
5. Allow the agent to inspect the architecture and infrastructure files.
6. Review the findings before making changes.

The prompt could also be as simple as:

```text
Review the current Azure network design and identify security, routing, DNS and private connectivity issues.
```

Because the role and specialist guidance already exist, the user does not need to restate the entire networking methodology every time.

---

## 8. Agents, Skills and Prompts Should Have Clear Boundaries

It is tempting to put everything into one very large agent file.

For a small experiment that may work, but it becomes harder to maintain as the scope grows.

Consider these responsibilities separately.

### Put role-specific behaviour in the agent

For example:

```text
You are responsible for reviewing Azure networking.
Do not modify infrastructure unless explicitly requested.
```

### Put reusable specialist guidance in a skill

For example:

```text
How to evaluate Private Endpoints, Private DNS,
routing and firewall architecture.
```

### Put repeatable tasks in prompt files

For example:

```text
Review the current network architecture and produce
findings, risks, recommendations and assumptions.
```

### Put broad repository rules in custom instructions

For example:

```text
Use Canada Central by default.
Use Terraform.
Prefer managed identities.
Do not commit secrets.
```

This separation makes the customisation easier to understand and reuse.

---

## 9. Skills Can Become Organisational Knowledge

The networking example is intentionally simple, but the same pattern can be expanded.

A cloud engineering repository could eventually contain skills such as:

```text
.github/skills/
│
├── azure-networking/
├── azure-landing-zone/
├── azure-firewall/
├── private-endpoints/
├── private-dns/
├── terraform-standards/
├── azure-rbac/
├── well-architected-review/
├── java-modernization/
└── migration-assessment/
```

Different agents can then use the knowledge relevant to their task.

For example:

```text
Azure Network Architect
    ↓
azure-networking
private-dns
azure-firewall

Terraform Engineer
    ↓
terraform-standards
private-endpoints
azure-rbac

Application Modernization Agent
    ↓
java-modernization
```

This is where skills become particularly interesting.

Instead of putting every standard, pattern and implementation guideline into every agent, specialist knowledge can be maintained as reusable capabilities.

---

## 10. Start Small Before Building a Multi-Agent System

There is a lot of discussion around multi-agent systems, orchestration, agent handoffs and autonomous development workflows.

Those concepts are useful, but it is worth understanding the basic building blocks first.

Start with one clear role.

```text
Azure Networking Agent
```

Give it one useful skill.

```text
azure-networking
```

Create one repeatable task.

```text
review-network.prompt.md
```

Add a small set of repository rules.

```text
copilot-instructions.md
```

Then test whether the agent consistently behaves the way you expect.

Once that model works, it becomes much easier to introduce additional agents for architecture, Terraform, security, migration, application modernization, testing or operations.

---

## Final Structure

The completed example looks like this:

```text
my-cloud-project/
│
├── .github/
│   │
│   ├── agents/
│   │   └── azure-networking.agent.md
│   │
│   ├── skills/
│   │   └── azure-networking/
│   │       └── SKILL.md
│   │
│   ├── prompts/
│   │   └── review-network.prompt.md
│   │
│   └── copilot-instructions.md
│
├── docs/
│   └── network-architecture.md
│
└── infra/
    └── network.tf
```

And the mental model remains straightforward:

```text
Agent        = Role
Skill        = Specialist knowledge
Prompt       = Task
Instructions = Repository rules
```

---

## Conclusion

A useful GitHub Copilot agent does not need to start as a large autonomous system.

A well-defined specialist agent can be created with a small number of files that clearly separate:

- the role Copilot should perform
- the specialist knowledge available to it
- the task being requested
- the rules that apply to the repository

For cloud engineering, this provides a practical way to begin capturing repeatable architecture and engineering practices directly alongside the code and infrastructure that teams work with.

The next step is where this model becomes more interesting: instead of defining **one** Azure specialist, multiple agents can represent different roles across an entire cloud engagement — from opportunity identification and RFP analysis through architecture, engineering, security review, migration and operations.

### Written by Usman Mahmood

Founder of The Cloud Captain, sharing practical guidance on cloud architecture, platform engineering, automation and modern infrastructure.

> **P.S. Further reading:** GitHub maintains current documentation for [custom agents](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/create-custom-agents), [agent skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills), [prompt files](https://docs.github.com/en/copilot/tutorials/customization-library/prompt-files), and [repository custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions).
