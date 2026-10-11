# Introduction

## About this Workshop

In this workshop, you will learn how to use Select AI for Python to apply Select AI actions and generate synthetic data with Oracle Autonomous AI Database. Select AI for Python enables developers to invoke Oracle Database Select AI capabilities directly from Python applications and workflows.

You will begin by connecting to your Autonomous AI Database, creating an AI provider credential, and creating a Select AI profile. You will then use Select AI actions to work with natural language prompts, including generating SQL, reviewing SQL before execution, running generated SQL, explaining SQL statements, narrating query results, translating text, summarizing content, and inspecting the prompt sent to the LLM. The supported actions include `show_sql`, `run_sql`, `narrate`, `explain_sql`, `chat`, `summarize`, `show_prompt`, and `translate`.

This workshop also introduces synthetic data generation using Select AI for Python. You will learn how to populate database tables with AI-generated sample rows for demos, development, testing, and prototyping when representative data is needed without copying production data. Synthetic data generation uses a Select AI profile and can generate rows for a single table or multiple tables.
By the end of this workshop, you will understand how to use Select AI for Python actions to interact with database data through natural language and how to generate synthetic data for building and testing AI-powered database applications


### Concepts of Select AI for Python

Select AI is Oracle AI Database's capability for using natural language and large language models (LLMs) directly with Oracle AI Database data. Instead of writing every SQL statement manually, developers can ask questions in plain English and let Select AI help generate, explain, run, or describe database interactions.
Before you begin, review these key concepts:

**Select AI for Python**: A Python interface that lets applications invoke Select AI capabilities programmatically. It connects Python workflows to an Oracle AI Database.

**Connection**: Establishes access from Python to Oracle Database using a standalone connection or connection pool.

**Provider**: Defines the AI service, model, endpoint, region, and provider-specific settings used by Select AI.

**Credential**: Stores the authentication details required to access the selected AI provider.

**Profile**: Combines the provider, credential, database object scope, and generation settings into a reusable Select AI configuration.

**Actions**: Define what Select AI should do with a prompt, such as `show SQL`, `run SQL`, `explain SQL`, `narrate results`, `chat`, `summarize`, or `translate`.

**Conversation**: Maintains prompt history so follow-up questions can use previous context.

**Synthetic data generation**: Select AI can generate representative, artificial rows for one or more tables. This is useful for demos, prototypes, development, and testing when production data is unavailable or unsuitable to copy.

**Command Line Interface**: Provides an optional terminal-based workflow for testing Select AI profiles, prompts, SQL generation, and chat interactions.

**Natural language to SQL(NL2SQL)**: Converts natural-language questions into SQL statements that can be reviewed, executed, explained or used to narrate query results.

**Estimated Time**: 1 hour 30 minutes

**Objectives**

In this workshop, you will:

- Set up an **Autonomous AI Database** for Select AI for Python use cases.
- Create and configure an **OCI Compute instance** for Python development.
- Install **Python 3.12** and **Select AI for Python**.
- Connect from Python to Autonomous AI Database using database credentials and wallet configuration.
- Verify the Select AI for Python installation with a connection smoke test.
- Create an AI provider credential for OCI Generative AI.
- Create and load a Select AI profile for natural language interactions.
- Use Select AI actions such as `show_sql`, `run_sql`, `narrate`, `explain_sql`, `chat`, `summarize`, `show_prompt`, and `translate`.
- Generate SQL from natural language prompts.
- Execute generated SQL and retrieve results in Python.
- Generate natural language explanations of database query results.
- Generate synthetic data for database tables to support testing, demonstrations, and application development.

You may now proceed to the next lab.

## Learn more

* Links
<!-- * [Oracle Autonomous AI Database Documentation](https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/index.html)
* [Oracle Autonomous AI Database MCP Server Documentation](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/mcp-server.html#GUID-22D738E1-BC06-47F0-9684-CD698DD8C492)
* [Oracle Select AI Agent Documentation](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-agent1.html)
* [`DBMS_CLOUD_AI_AGENT` Package Documentation](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/dbms-cloud-ai-agent-package.html)
* [Additional Autonomous AI Database Workshops](https://apexapps.oracle.com/pls/apex/dbpm/r/livelabs/livelabs-workshop-cards?p100_workshop_series=222) -->

## Acknowledgements
* **Authors:**: 
* **Contributors:** 
* **Last Updated By/Date:** 


Copyright (c) 2026 Oracle Corporation.

Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.3
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
A copy of the license is included in the section entitled [GNU Free Documentation License](files/gnu-free-documentation-license.txt)
