# Use Select AI Actions with Python

## Introduction

This lab uses Oracle Select AI for Python to interact with Autonomous AI Database through natural-language prompts. You will connect from Python, create an AI provider credential and Select AI profile, then use Select AI actions to generate and review SQL, run queries, narrate and explain results, chat with the AI model, summarize text, inspect prompts, and translate text.

Estimated time: 45 minutes.

### Objectives

In this lab, you will:

- Connect to Autonomous AI Database using Select AI for Python.
- Create an AI provider credential and a Select AI profile.
- Load the profile in Python.
- Generate and review SQL from natural-language prompts.
- Execute generated SQL and inspect query results.
- Narrate query results and explain SQL.
- Use chat, summarization, prompt inspection, and translation actions.

### Prerequisites

Before starting, make sure that:

- Your Autonomous AI Database is provisioned and running.
- You created the `ADB_USER` database user and granted the required Select AI privileges.
- The database wallet is downloaded and extracted on the OCI Compute instance.
- Select AI for Python and the OCI SDK are installed in your Python virtual environment.
- The OCI CLI is configured on the Compute instance and its config file is available.
- Network access to the selected AI provider endpoint is configured.
- The workshop sample tables and data are created in the `ADB_USER` schema.

## Before You Begin

Activate the Python virtual environment you created in the previous lab. Set the connection, credential, and provider environment variables in the same shell where you will run the scripts:

```bash
<copy>
export SELECT_AI_USER="ADB_USER"
export SELECT_AI_PASSWORD="<adb_user_password>"
export SELECT_AI_DB_CONNECT_STRING="<adb_service_name>"
export SELECT_AI_WALLET_DIR="/home/opc/wallet"
export SELECT_AI_WALLET_PASSWORD="<wallet_password>"
export SELECT_AI_CREDENTIAL_NAME="my_oci_ai_profile_key"
export SELECT_AI_PROFILE_NAME="oci_ai_profile"
export SELECT_AI_SCHEMA="ADB_USER"
export SELECT_AI_OCI_REGION="<supported-gen-ai-region>"
export SELECT_AI_OCI_MODEL="<oci-genai-model>"
export SELECT_AI_OCI_COMPARTMENT_ID="<compartment_ocid>"
export OCI_CONFIG_PROFILE="DEFAULT"
</copy>
```

Use a service alias from the wallet's `tnsnames.ora` file for `SELECT_AI_DB_CONNECT_STRING`. For example:

```bash
<copy>
export SELECT_AI_DB_CONNECT_STRING="<dbname>_low"
</copy>
```

Verify that the wallet files exist and list the service aliases:

```bash
<copy>
ls -l "$SELECT_AI_WALLET_DIR/tnsnames.ora"
ls -l "$SELECT_AI_WALLET_DIR/ewallet.pem"
grep "^[A-Za-z0-9_]" "$SELECT_AI_WALLET_DIR/tnsnames.ora"
</copy>
```

The examples in this lab use OCI Generative AI. Set the region, model, and compartment values for the service you are using. The OCI config profile must contain the user OCID, tenancy OCID, fingerprint, and private key path.

## Task 1: Connect to Autonomous AI Database from Python

Create a reusable helper file named `select_ai_common.py`:

```python
<copy>
import os
import sys
from pathlib import Path

import select_ai


def required_env(name):
    value = os.getenv(name)
    if not value:
        print(f"Missing required environment variable: {name}")
        sys.exit(1)
    return value


def connect_to_adb():
    user = required_env("SELECT_AI_USER")
    password = required_env("SELECT_AI_PASSWORD")
    dsn = required_env("SELECT_AI_DB_CONNECT_STRING")
    wallet_dir = os.getenv("SELECT_AI_WALLET_DIR", "/home/opc/wallet")
    wallet_password = os.getenv("SELECT_AI_WALLET_PASSWORD")
    wallet_path = Path(wallet_dir)

    if not (wallet_path / "tnsnames.ora").exists():
        print(f"tnsnames.ora not found in {wallet_dir}")
        sys.exit(1)

    connect_args = {
        "user": user,
        "password": password,
        "dsn": dsn,
        "wallet_location": wallet_dir,
        "config_dir": wallet_dir,
    }
    if wallet_password:
        connect_args["wallet_password"] = wallet_password

    select_ai.connect(**connect_args)


def load_profile():
    profile_name = os.getenv("SELECT_AI_PROFILE_NAME", "oci_ai_profile")
    return select_ai.Profile(profile_name=profile_name)
</copy>
```

Create `task1_connect.py`:

```python
<copy>
from select_ai_common import connect_to_adb

connect_to_adb()
print("Connected to the Database")
</copy>
```

Run the script:

```bash
<copy>
python3.12 task1_connect.py
</copy>
```

A successful connection displays:

```text
<copy>
Connected to the Database
</copy>
```

## Task 2: Create an AI Provider Credential

This example creates an OCI Generative AI credential using the user, tenancy, fingerprint, and private key in your OCI config file.

Create `task2_create_credential.py`:

```python
<copy>
import os
import sys

import oci
import select_ai

from select_ai_common import connect_to_adb


connect_to_adb()
credential_name = os.getenv(
    "SELECT_AI_CREDENTIAL_NAME",
    "my_oci_ai_profile_key",
)
oci_config_profile = os.getenv("OCI_CONFIG_PROFILE", "DEFAULT")

try:
    default_config = oci.config.from_file(profile_name=oci_config_profile)
    oci.config.validate_config(default_config)

    with open(default_config["key_file"], "r") as key_file:
        private_key_contents = key_file.read()

    credential = {
        "credential_name": credential_name,
        "user_ocid": default_config["user"],
        "tenancy_ocid": default_config["tenancy"],
        "private_key": private_key_contents,
        "fingerprint": default_config["fingerprint"],
    }

    select_ai.create_credential(
        credential=credential,
        replace=True,
    )
    print(f"Created credential: {credential_name}")
except Exception as error:
    print("Failed to create credential.")
    print(error)
    sys.exit(1)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task2_create_credential.py
</copy>
```

Expected output:

```text
<copy>
Created credential: my_oci_ai_profile_key
</copy>
```

## Task 3: Create a Select AI Profile

The profile defines the AI provider, credential, and database objects Select AI can use when generating SQL. This profile uses the workshop tables: `CUSTOMERS`, `PRODUCTS`, `ORDERS`, `ORDER_ITEMS`, and `RETURNS`.

Create `task3_create_profile.py`:

```python
<copy>
import os
import sys

import select_ai

from select_ai_common import connect_to_adb


connect_to_adb()
profile_name = os.getenv("SELECT_AI_PROFILE_NAME", "oci_ai_profile")
credential_name = os.getenv(
    "SELECT_AI_CREDENTIAL_NAME",
    "my_oci_ai_profile_key",
)
schema_name = os.getenv("SELECT_AI_SCHEMA", "ADB_USER")
region = os.getenv("SELECT_AI_OCI_REGION", "<supported-gen-ai-region>")
model = os.getenv("SELECT_AI_OCI_MODEL")
compartment_id = os.getenv("SELECT_AI_OCI_COMPARTMENT_ID")

try:
    provider_args = {
        "region": region,
        "oci_apiformat": "GENERIC",
    }
    if model:
        provider_args["model"] = model
    if compartment_id:
        provider_args["oci_compartment_id"] = compartment_id

    provider = select_ai.OCIGenAIProvider(**provider_args)
    profile_attributes = select_ai.ProfileAttributes(
        credential_name=credential_name,
        object_list=[
            {"owner": schema_name, "name": "CUSTOMERS"},
            {"owner": schema_name, "name": "PRODUCTS"},
            {"owner": schema_name, "name": "ORDERS"},
            {"owner": schema_name, "name": "ORDER_ITEMS"},
            {"owner": schema_name, "name": "RETURNS"},
        ],
        provider=provider,
        enforce_object_list=True,
        comments=True,
        constraints=True,
        temperature=0.1,
        max_tokens=1024,
    )
    profile = select_ai.Profile(
        profile_name=profile_name,
        attributes=profile_attributes,
        description="Select AI for Python workshop profile",
        replace=True,
    )
    print(f"Created Select AI profile: {profile_name}")
except Exception as error:
    print("Failed to create Select AI profile.")
    print(error)
    sys.exit(1)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task3_create_profile.py
</copy>
```

Expected output:

```text
<copy>
Created Select AI profile: oci_ai_profile
</copy>
```

## Task 4: Enable or Load the Select AI Profile

Load the profile in Python so you can use its actions.

Create `task4_load_profile.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
print(f"Loaded Select AI profile: {profile.profile_name}")
</copy>
```

Run the script:

```bash
<copy>
python3.12 task4_load_profile.py
</copy>
```

Expected output:

```text
<copy>
Loaded Select AI profile: oci_ai_profile
</copy>
```

## Task 5: Use show_sql to Generate SQL from a Natural-Language Prompt

The `show_sql` action generates a SQL statement without executing it.

Create `task5_show_sql.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
prompt = "Show total sales by product category"
sql = profile.show_sql(prompt=prompt)

print("Prompt:")
print(prompt)
print("\nGenerated SQL:")
print(sql)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task5_show_sql.py
</copy>
```

Review the generated SQL and confirm that it uses the `PRODUCTS` and `ORDER_ITEMS` tables. The output should be similar to:

```sql
<copy>
SELECT p."CATEGORY" AS "Product Category",
       SUM(oi."UNIT_PRICE" * oi."QUANTITY") AS "Total Sales"
FROM "ADB_USER"."PRODUCTS" p
INNER JOIN "ADB_USER"."ORDER_ITEMS" oi
    ON p."PRODUCT_ID" = oi."PRODUCT_ID"
GROUP BY p."CATEGORY"
</copy>
```

## Task 6: Use run_sql to Execute a Query

The `run_sql` action generates SQL from a natural-language prompt, executes it, and returns the result as a table.

Create `task6_run_sql.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
prompt = "Show total sales by product category"
dataframe = profile.run_sql(prompt=prompt)

print("Prompt:")
print(prompt)
print("\nQuery Result:")
print(dataframe.to_string(index=False))
</copy>
```

Run the script:

```bash
<copy>
python3.12 task6_run_sql.py
</copy>
```

The output should show sales grouped by product category. For the sample data, the totals are:

| Category | Total sales |
| --- | ---: |
| Electronics | 4295.00 |
| Accessories | 59.98 |
| Furniture | 748.00 |

## Task 7: Use narrate to Generate a Natural-Language Response

The `narrate` action generates SQL, runs it, and returns the result as a natural-language response.

Create `task7_narrate.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
prompt = "Narrate the sales performance by product category"
response = profile.narrate(prompt=prompt)

print("Prompt:")
print(prompt)
print("\nNarrative Response:")
print(response)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task7_narrate.py
</copy>
```

The response should summarize category sales. For example:

- Electronics leads with total sales of 4295.
- Accessories shows total sales of 59.98.
- Furniture records total sales of 748.

## Task 8: Use explain_sql to Explain a Query

The `explain_sql` action generates SQL for a prompt and returns a natural-language explanation of the query logic.

Create `task8_explain_sql.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
prompt = "Explain the SQL for total sales by product category"
explanation = profile.explain_sql(prompt=prompt)

print("Prompt:")
print(prompt)
print("\nSQL Explanation:")
print(explanation)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task8_explain_sql.py
</copy>
```

The explanation should describe how the query joins `PRODUCTS` with `ORDER_ITEMS`, multiplies unit price by quantity, and groups sales totals by category. The sample query does not filter by order status or subtract returns.

## Task 9: Use chat to Interact with the AI Model

The `chat` action sends a prompt directly to the AI model. It is useful for general questions, explanations, and content generation.

Create `task9_chat.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
prompt = (
    "Explain how natural language to SQL helps business users "
    "analyze database data."
)
response = profile.chat(prompt=prompt)

print("Prompt:")
print(prompt)
print("\nChat Response:")
print(response)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task9_chat.py
</copy>
```

The output should be a natural-language response to the prompt. The exact wording depends on the configured AI model.

## Task 10: Use summarize to Summarize Text

The `summarize` action creates a shorter version of supplied text.

Create `task10_summarize.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
content = """
Select AI for Python enables developers to use natural language prompts
to interact with Oracle Autonomous AI Database. Developers can generate SQL,
execute SQL, explain SQL statements, narrate query results, provide feedback,
summarize text, and build AI-powered applications using Python.
"""
summary = profile.summarize(content=content)

print("Original Text:")
print(content)
print("\nSummary:")
print(summary)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task10_summarize.py
</copy>
```

The output should contain a shorter summary of the supplied text.

## Task 11: Use show_prompt to Inspect the Prompt Sent to the LLM

The `show_prompt` action displays the prompt that Select AI sends to the language model. Use it to inspect how profile attributes, object lists, comments, constraints, and provider settings shape the prompt.

Create `task11_show_prompt.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
prompt = "Show total sales by product category"
llm_prompt = profile.show_prompt(prompt=prompt)

print("Original Prompt:")
print(prompt)
print("\nPrompt Sent to the LLM:")
print(llm_prompt)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task11_show_prompt.py
</copy>
```

The script displays the full prompt, including the schema context for the profile and the natural-language question. The exact prompt content depends on the profile and library version.

## Task 12: Use translate to Translate Text

The `translate` action translates text from one language to another.

Create `task12_translate.py`:

```python
<copy>
from select_ai_common import connect_to_adb, load_profile

connect_to_adb()
profile = load_profile()
text = "Thank you for completing the Select AI for Python workshop."
response = profile.translate(
    text=text,
    source_language="en",
    target_language="de",
)

print("Original Text:")
print(text)
print("\nTranslated Text:")
print(response)
</copy>
```

Run the script:

```bash
<copy>
python3.12 task12_translate.py
</copy>
```

The response should display the text translated into German. For example:

```text
<copy>
Vielen Dank, dass Sie den Select AI for Python Workshop abgeschlossen haben.
</copy>
```

## Summary

In this lab, you connected to Autonomous AI Database from Python, created an AI provider credential and Select AI profile, loaded the profile, and used Select AI actions with database data and text.

You used `show_sql`, `run_sql`, `narrate`, `explain_sql`, `chat`, `summarize`, `show_prompt`, and `translate`. You can now use these actions to generate SQL, execute queries, explain query logic, inspect prompts, summarize content, translate text, and build AI-powered Python applications.

You may now proceed to the next lab.

## Learn More

* [Select AI for Python](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-py.html)
* [Use AI Keyword to Enter Prompts](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-keyword-prompts.html)
* [Examples of Using Select AI](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-examples.html)

Copyright (c) 2026 Oracle Corporation.

Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.3
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
A copy of the license is included in the section entitled [GNU Free Documentation License](https://oracle-livelabs.github.io/adb/shared/adb-15-minutes/introduction/files/gnu-free-documentation-license.txt)
