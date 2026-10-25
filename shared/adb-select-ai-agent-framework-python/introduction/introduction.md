# Introduction

## About this Workshop

In this workshop, you will learn how to use the Select AI Agent Framework with Python to build and run intelligent, database-connected agent workflows. The Select AI Agent Framework enables developers to create AI agents that can interpret natural-language requests, plan actions, use tools, evaluate results, and return responses directly from Python applications.

You will begin by connecting to your Autonomous AI Database, creating an AI provider credential, and creating a Select AI profile. You will then use the select_ai.agent Python module to create tools, tasks, agents, and teams.

This workshop introduces the main components of the Select AI Agent Framework. You will learn how to create database and custom tools, define tasks, configure an agent’s role and behaviour, assemble an agent team, and run the team with natural-language prompts.

You will also learn how agents can use database tools such as natural-language-to-SQL, PL/SQL functions, retrieval-augmented generation, web search, notifications, and custom functions.

The workshop introduces two multi-agent workflow patterns:

- Sequential workflow pattern.

- Supervisor agent pattern.

You will learn how to use a sequential team for predictable, fixed-order processing and a supervisor-led team for dynamic runtime routing between specialized worker agents.

By the end of this workshop, you will understand how to use Select AI Agent Framework for Python to build agentic workflows that interact with Autonomous AI Database data and services.

## Concepts of Select AI Agent Framework for Python

Select AI Agent Framework is Oracle AI Database’s capability for creating and managing interactive and autonomous AI agents. Agents can reason about user requests, plan a sequence of actions, call approved tools, reflect on tool results, and maintain conversational context.

Before you begin, review these key concepts:

Select AI Agent Framework for Python: A Python interface that enables applications to create and run Select AI agents using the select_ai.agent module.

Connection: Establishes access from Python to Oracle Database using a standalone connection or connection pool.

Provider: Defines the AI service, model, endpoint, region, and provider-specific settings used by Select AI.

Credential: Stores the authentication details required to access the selected AI provider.

Profile: Combines the provider, credential, database object scope, and generation settings into a reusable Select AI configuration.

Tool: Defines an action that an agent can perform. Tools can use built-in capabilities such as natural-language-to-SQL, RAG, web search, and notifications, or can call PL/SQL procedures and custom functions.

Task: Defines the instructions, objectives, and tools that an agent uses to complete a specific piece of work.

Agent: Represents an AI worker with a defined role, instructions, AI profile, and access to selected tools.

Team: Combines agents and tasks into an executable workflow. A team can coordinate one or more agents using sequential or supervisor-based patterns.

Agent execution: The process in which an agent interprets a request, plans actions, calls tools, evaluates results, and generates a response.

Reflection: Enables the agent to evaluate tool results and revise its plan when additional actions or clarification are required.

Conversation memory: Maintains context from previous prompts, tool calls, and responses so that follow-up questions can refer to earlier interactions.

Sequential workflow pattern: A fixed workflow in which agent-task pairs run in a configured order and pass results between dependent tasks.

Supervisor agent pattern: A dynamic workflow in which a supervisor agent selects and coordinates specialized worker agents at run time.

Worker agent: A specialized agent that performs a delegated task in a supervisor-led team.

Task dependency: A relationship that allows a downstream task in a sequential workflow to receive the result of an upstream task.

Self-contained subtask: A complete request sent by a supervisor to a worker agent. It includes the context and instructions the worker needs because the worker does not automatically inherit the supervisor’s full conversation.

Synchronous and asynchronous APIs: The framework supports standard Python execution and asynchronous workflows using async and await.

Command Line Interface: Provides an optional terminal-based workflow for testing Select AI profiles, agent objects, prompts, and team interactions.

## Estimated Time

1 hour 30 minutes

## Objectives

In this workshop, you will:

- Set up an Autonomous AI Database for Select AI Agent Framework use cases.

- Create and configure an OCI Compute instance for Python development.

- Install Python 3.12 and Select AI for Python.

- Connect from Python to Autonomous AI Database using database credentials and wallet configuration.

- Verify the Select AI for Python installation with a connection smoke test.

- Create an AI provider credential for OCI Generative AI.

- Create and load a Select AI profile for agent interactions.

- Understand the roles of tools, tasks, agents, and teams.

- Create a Select AI agent using the select_ai.agent Python module.

- Create tools for natural-language-to-SQL, PL/SQL, RAG, web search, notifications, or custom functions.

- Define tasks that specify agent objectives, instructions, and available tools.

- Configure an agent role and associate it with a Select AI profile.

- Assemble agents and tasks into an agent team.

- Create a multi-agent team using the sequential workflow pattern.

- Configure agent-task pairs to run in a fixed order.

- Pass results between dependent tasks.

- Create a supervisor agent and specialized worker agents.

- Configure a supervisor-led team for runtime task routing.

- Send self-contained subtasks from a supervisor to worker agents.

- Run an agent team with natural-language prompts.

- Use multi-turn conversations with an agent team.

- Understand how agents plan, use tools, and evaluate tool results.

- Review agent execution history and conversation details for troubleshooting.

- Use synchronous and asynchronous Select AI Agent Framework APIs.

- Select the appropriate workflow pattern for a business use case.

- Build a foundation for developing AI-powered database applications with Python.

You may now proceed to the next lab.

## Learn More

* [Select AI Agent Framework](https://docs.oracle.com/en-us/iaas/autonomous-database-serverless/doc/select-ai-agent1.html)
* [Select AI Agent Framework Interfaces](https://docs.oracle.com/en/database/oracle/oracle-database/26/selai/select-ai-agent-interfaces.html)
* [Examples of Using the Select AI Agent Framework](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/examples-using-select-ai-agent.html)
* [Multi-Agent Pattern for AI Agents](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/multi-agent-pattern-ai-agents.html)

## Acknowledgements

- **Created By:** Chandrakanth Putha, Oracle AI Database Product Manager
- **Contributors:** Mark Hornick, Product Management for Select AI and Machine Learning
- **Last Updated By:** Dhanish Kumar, Senior Member of Technical Staff
