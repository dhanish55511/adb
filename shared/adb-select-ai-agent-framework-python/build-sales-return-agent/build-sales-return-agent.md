# Build a Sales Return Agent Using the Sequential Pattern with the Select AI Agent Framework for Python

## Introduction

In this lab, you will build a Sales Return Agent using the Sequential pattern with the Select AI Agent Framework for Python. The agent automates a common customer-service workflow: returning a purchased product.

The Sales Return Agent interacts with the customer to identify the order, collect the reason for the return, and process the request by updating the relevant order-status record in Oracle Autonomous AI Database. It uses the database and a large language model (LLM) to provide a natural conversational experience while completing the required backend processing.

You will build the solution in sequence by defining a function and tool, creating a task, configuring an agent with an AI profile, and assembling an agent team. The AI profile identifies the LLM used by the agent.

The lab has two phases. First, you will create a basic Sales Return Agent that converses with the customer and updates the database. Next, you will enhance the agent to generate a customer-facing email confirmation for the completed return request.

Estimated Time: 25 minutes

## Objectives

In this lab, you will:

- Connect to Autonomous AI Database

- Build and run a customer return agent

- Create a Customer Order Update Status function and a tool that uses it to update return status in the database.

- Create a task that coordinates the return action.

- Define the customer return agent’s persona and role.

- Create an agent team and run it with Select AI Agent framework for Python, answering the agent’s questions.

- Add return-confirmation email capability

- Write and test a PL/SQL function that composes a return-confirmation email.

- Create a customemail tool that uses the function and a task with clear instructions for using the tool

- Update the agent to call the email task and produce a ready-to-send preview.

- Add the task to the agent team

- Complete an end-to-end interaction and capture the email preview.

## Before you begin

Set the environment variables:

```bash
<copy>
export SELECT_AI_USER=ADB_USER
export SELECT_AI_PASSWORD='<adb_user_password>'
export SELECT_AI_DB_CONNECT_STRING='<adb_service_name>'
export SELECT_AI_WALLET_DIR=/home/opc/wallet
export SELECT_AI_WALLET_PASSWORD='<wallet_password>'

export SELECT_AI_CREDENTIAL_NAME=my_oci_ai_profile_key
export SELECT_AI_PROFILE_NAME=oci_ai_agent_profile
export SELECT_AI_SCHEMA=ADB_USER

export SELECT_AI_OCI_REGION=<supported-gen-ai-region>
export SELECT_AI_OCI_MODEL=<oci-genai-model>
export SELECT_AI_OCI_COMPARTMENT_ID='<compartment_ocid>'

export OCI_CONFIG_PROFILE=DEFAULT
</copy>
```

For SELECT_AI_DB_CONNECT_STRING, use a service alias from the wallet tnsnames.ora file, such as:

```bash
<copy>
export SELECT_AI_DB_CONNECT_STRING=<dbname>_low
</copy>
```

Verify that the wallet files exist:

```bash
<copy>
ls -l $SELECT_AI_WALLET_DIR/tnsnames.ora
ls -l $SELECT_AI_WALLET_DIR/ewallet.pem
</copy>
```

Verify the available database service names:

```bash
<copy>
grep "^[A-Za-z0-9_]" $SELECT_AI_WALLET_DIR/tnsnames.ora
</copy>
```

## Task 1: Connect to Autonomous AI Database from Python

In this task, you will create a reusable helper file and verify that Python can connect to Autonomous AI Database.

In this task, you will create a reusable helper file and verify that Python can connect to Autonomous AI Database.

Create a file named select_ai_agent_common.py

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

Create a file named task1_agent_connect.py

from select_ai_common import connect_to_adb

connect_to_adb()

print("Connected to the Database")

Run the script:

```bash
<copy>
python3.12 task1_connect.py
</copy>
```

If the connection is successful, the script displays:

Connected to the Database

## Task 2: Create a Tool for Database Updates

Create Update_Order_Status_Tool

Create a new file namedtask2_agent_create_update_order_status_tool.py

```python
<copy>
import select_ai
from select_ai.agent import Tool
from select_ai_agent_common import connect_to_adb
TOOL_NAME = "Update_Order_Status_Tool"
def main():
    connect_to_adb()

    Tool.delete_tool(
        tool_name=TOOL_NAME,
        force=True,
    )
 tool = Tool.create_pl_sql_tool(
        tool_name=TOOL_NAME,
        function="update_customer_order_status",
        description="Tool for updating customer order status in the database table.",
        instruction=(
            "This tool updates the database to reflect a return status change. "
            "Always confirm the customer name and order number with the user "
            "before updating the status."
        ),
        replace=True,
    )
print(f"Created tool: {tool.tool_name}")
    select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task2_agent_create_update_order_status_tool.py
</copy>
```

Expected output:

Created tool: Update_Order_Status_Tool

Verify the Update_Order_Status_Tool

use Tool.list() to verify that Update_Order_Status_Tool was created.

Create a new file name task2_agent_verify_update_order_status_tool.py

```python
<copy>
import select_ai
from select_ai.agent import Tool
from select_ai_agent_common import connect_to_adb
TOOL_NAME = "Update_Order_Status_Tool"
def main():
    connect_to_adb()

    found = False

    for tool in Tool.list(tool_name_pattern=TOOL_NAME):
        found = True
        print(f"Tool Name: {tool.tool_name}")
        print(f"Description: {tool.description}")
        print(f"Attributes: {tool.attributes}")

    if not found:
        print(f"Tool not found: {TOOL_NAME}")

    select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task2_agent_verify_update_order_status_tool.py
</copy>
```

Expected output:

Tool Name: UPDATE_ORDER_STATUS_TOOL

Description: Tool for updating customer order status in the database table.

Attributes: ToolAttributes(instruction='This tool updates the database to reflect a return status change. Always confirm the customer name and order number with the user before updating the status.', function='update_customer_order_status', tool_params=ToolParams(_REQUIRED_FIELDS=None, credential_name=None, endpoint=None, notification_type=None, profile_name=None, recipient=None, sender=None, channel=None, smtp_host=None, subject=None), tool_inputs=None, tool_type=None)

## Task 3: Create a Customer Order Update Status Function

You will create a SQL function to update a customer’s order status and return a message, run an PL/SQL block to set a customers order to pending_return and print the result, then verify the table.

Log in to Oracle Autonomous AI Database as ADB_USER, then run the following SQL statements.

Create a customer order update status function.

```python
<copy>
CREATE OR REPLACE FUNCTION update_customer_order_status (
    p_customer_name IN VARCHAR2,
    p_order_number  IN VARCHAR2,
    p_status        IN VARCHAR2
) RETURN CLOB IS
    v_customer_id  customers.customer_id%TYPE;
    v_row_count    NUMBER;
BEGIN
    -- Find customer_id from customer_name
    SELECT customer_id
    INTO v_customer_id
    FROM customers
    WHERE name = p_customer_name;
    
    UPDATE customer_order_status
    SET status = p_status
    WHERE customer_id = v_customer_id
      AND order_number = p_order_number;

    v_row_count := SQL%ROWCOUNT;

    IF v_row_count = 0 THEN
        RETURN 'No matching record found to update.';
    ELSE
        RETURN 'Update successful.';
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        RETURN 'Error: ' || SQLERRM;
END;
</copy>
```

Test this function by updating an order for a specific customer and return a message on completion

```sql
<copy>
DECLARE
  v_result CLOB;
BEGIN
  v_result := update_customer_order_status('David Johnson', '3928', 'pending_return');

  DBMS_OUTPUT.PUT_LINE(v_result);
END;
</copy>
```

Verify the change by viewing the customer order update status table for that customer.

```sql
<copy>
select * from customer_order_statuswhere ORDER_NUMBER = 3928;
</copy>
```

## Task 4: Create a Task to Handle the Product Return

We use a task to specify the instructions for performing actions using tools. You'll define a task that your Select AI agent will perform. Each task has a unique name and a set of attributes that specify its behaviour when planning and performing the task. In this scenario, you'll describe the interaction with the customer and how to respond in various cases.

Create Handle_Product_Return_Task

Create a new file name task4_agent_create_product_return_task.py

```python
<copy>
import select_ai
from select_ai.agent import Task, TaskAttributes

from select_ai_agent_common import connect_to_adb

TASK_NAME = "Handle_Product_Return_Task"
TOOL_NAME = "Update_Order_Status_Tool"


def main():
    connect_to_adb()

    try:
        Task.delete_task(
            task_name=TASK_NAME,
            force=True,
        )

        task = Task(
            task_name=TASK_NAME,
            description="Handles customer product-return requests.",
            attributes=TaskAttributes(
                instruction="""
Process a product return request from a customer: {query}

1. Ask the customer for the reason for the return:
   no longer needed, arrived too late, box broken, or defective.

2. If no longer needed:
   a. Inform the customer to ship the product back at their expense.
   b. Update the order status to return_shipment_pending using
      Update_Order_Status_Tool.

3. If the product arrived too late:
   a. Ask the customer whether they want a refund.
   b. If the customer wants a refund, confirm that the refund is
      processed and update the order status to refund_completed.

4. If the product was defective or the box was broken:
   a. Ask the customer whether they want a replacement or a refund.
   b. For a replacement, inform the customer that the replacement is
      on its way and that they will receive a return shipping label.
      Update the order status to replaced.
   c. For a refund, instruct the customer to print the return shipping
      label, return the product, and update the order status to refund.

5. After completing a return, replacement, or refund, ask whether the
   customer needs any further help. End the task if they do not.
""",
                tools=[TOOL_NAME],
                enable_human_tool=True,
            ),
        )

        task.create(
            enabled=True,
            replace=True,
        )

        print(f"Created task: {task.task_name}")

    finally:
        select_ai.disconnect()


if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task4_agent_create_product_return_task.py
</copy>
```

Expected output:

```text
<copy>
Created task: Handle_Product_Return_Task
</copy>
```

## Task 5: Create OCI Credentials

In this task, you will create an AI provider credential in the database. This credential stores the authentication details required to access the AI provider.

This example uses OCI Generative AI and reads the OCI user, tenancy, fingerprint, and private key from the OCI config file.

Create a file named task5_agent_create_credential.py

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
    "my_oci_ai_profile_key"
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
        replace=True
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
python3.12 task5_agent_create_credential.py
</copy>
```

Expected output:

Created credential: my_oci_ai_profile_key

## Task 6: Create a Select AI Profile

You'll create an AI profile with LLM of your choice, to use in the next step to create an agent. In this task, you are using xAi's GROK 4.20 Multi Agent model with the OCI Gen AI credential.

Create a new file named task6_agent_create_ai_profile.py

```python
<copy>
import os
import sys

import select_ai

from select_ai_agent_common import connect_to_adb, required_env


connect_to_adb()

profile_name = os.getenv("SELECT_AI_PROFILE_NAME", " oci_ai_agent_profile")
credential_name = os.getenv("SELECT_AI_CREDENTIAL_NAME", " my_oci_ai_profile_key")

region = required_env("SELECT_AI_OCI_REGION")
model = os.getenv(
    "SELECT_AI_OCI_MODEL",
    "xai.grok-4.20-reasoning",
)
compartment_id = os.getenv("SELECT_AI_OCI_COMPARTMENT_ID")

try:
    provider_args = {
        "region": region,
        "oci_apiformat": "GENERIC",
        "model": model,
    }

    if compartment_id:
        provider_args["oci_compartment_id"] = compartment_id

    provider = select_ai.OCIGenAIProvider(**provider_args)

    profile_attributes = select_ai.ProfileAttributes(
        credential_name=credential_name,
        provider=provider,
        temperature=0.1,
        max_tokens=1024,
    )

    profile = select_ai.Profile(
        profile_name=profile_name,
        attributes=profile_attributes,
        description="AI profile for the Sales Return Agent.",
        replace=True,
    )

    print(f"Created agent AI profile: {profile_name}")

except Exception as error:
    print("Failed to create agent AI profile.")
    print(error)
    sys.exit(1)

finally:
    select_ai.disconnect()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task6_agent_create_ai_profile.py
</copy>
```

Expected output:

Created agent AI profile: oci_ai_agent_profile

## Task 7: Define Customer Agent

You'll create an agent, which specifies the LLM to use through an AI profile and the role the agent plays.

Create the Customer_Return_Agent.

Create a new file named task7_agent_create_customer_return_agent.py

```python
<copy>
import os
import sys
import select_ai
from select_ai.agent import Agent, AgentAttributes
from select_ai_agent_common import connect_to_adb
AGENT_NAME = "Customer_Return_Agent"
def main():
    connect_to_adb()

    profile_name = os.getenv(
        "SELECT_AI_PROFILE_NAME",
        "oci_ai_agent_profile",
    )

    try:
        Agent.delete_agent(
            agent_name=AGENT_NAME,
            force=True,
        )

        agent = Agent(
            agent_name=AGENT_NAME,
            description="Agent that handles customer product-return requests.",
            attributes=AgentAttributes(
                profile_name=profile_name,
                role=(
                    "You are an experienced customer return agent who "
                    "handles customer product-return requests."
                ),
                enable_human_tool=True,
            ),
        )

        agent.create(
            enabled=True,
            replace=True,
        )

        print(f"Created agent: {agent.agent_name}")

    except Exception as error:
        print("Failed to create customer return agent.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task7__agent_create_customer_return_agent.py
</copy>
```

Expected output:

Created agent: Customer_Return_Agent

## Task 8: Create the Agent Team

You'll define an agent team that uses the agent and tasks specified above. You specify agent-task pairs for defining a sequential workflow. If you have multiple agents or multiple tasks, these are specified within the JSON list. The same Agent can be specified multiple times with their associated tasks. In this scenario it is a single agent and task pair.

Create the Return_Agency_Team.

Create a new file named task8_agent_create_return_agency_team.py

```python
<copy>
import sys
import select_ai
from select_ai.agent import Team, TeamAttributes
from select_ai_agent_common import connect_to_adb
TEAM_NAME = "Return_Agency_Team"
AGENT_NAME = "Customer_Return_Agent"
TASK_NAME = "Handle_Product_Return_Task"


def main():
    connect_to_adb()

    try:
        Team.delete_team(
            team_name=TEAM_NAME,
            force=True,
        )

        team = Team(
            team_name=TEAM_NAME,
            description="Sequential team for handling customer product returns.",
            attributes=TeamAttributes(
                agents=[
                    {
                        "name": AGENT_NAME,
                        "task": TASK_NAME,
                    }
                ],
                process="sequential",
            ),
        )

        team.create(
            enabled=True,
            replace=True,
        )

        print(f"Created team: {team.team_name}")

    except Exception as error:
        print("Failed to create return agency team.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()


if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task8_agent_create_return_agency_team.py
</copy>
```

Expected output:

Created team: Return_Agency_Team

## Task 9: Interact with the Return Agency Agent

In this task, you will run the Return Agency Team and submit a customer return request. The agent uses the Sales Return Agent workflow to interact with the customer, collect the required return details, and update the order status when appropriate.

Create a new file name task9_agent_interact_with_return_agency.py

```python
<copy>
import datetime
import sys
import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    try:
        conversation = select_ai.Conversation(
            attributes=select_ai.ConversationAttributes(
                title="Sales Return Request",
                description=(
                    "Conversation between a customer and the "
                    "Sales Return Agent."
                ),
                retention_days=datetime.timedelta(days=7),
                conversation_length=20,
            )
        )

        conversation_id = conversation.create()

        print(f"Created conversation ID: {conversation_id}")

        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="I want to return a smartphone protective case.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("\nAgent Response:")
        print(response)

    except Exception as error:
        print("Failed to interact with the Return Agency Team.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task9_agent_interact_with_return_agency.py
</copy>
```

Expected output:

Created conversation ID: 5AF6DB61-BEA3-2BB0-E063-4BD5000A4BC6

Agent Response:

Thank you for contacting us. To process your return for the smartphone protective case, could you please tell me the reason? The options are: no longer needed, arrived too late, box broken, or defective.

Use the same conversation ID created in Task 9.

Create a new file named task9_agent_continue_return_conversation.py

```python
<copy>
import os
import sys

import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="The item is defective.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

Set the conversation ID returned by the previous task as an environment variable.

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=<conversation_id>
</copy>
```

```bash
<copy>
python3.12 task9_agent_continue_return_conversation.py
</copy>
```

Expected output:

Agent Response:

Thank you for informing us. Since the item is defective, would you like a replacement or a refund?

Create a new file named task9_agent_request_replacement.py

```python
<copy>
import os
import sys
import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="I'd like a replacement.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

Ensure the conversation ID is set from the earlier return request:

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=<conversation_id>
</copy>
```

```bash
<copy>
python3.12 task9_agent_request_replacement.py
</copy>
```

Expected output:

Agent Response:

Great, we'll send a replacement on its way, and you will receive a return shipping label shortly. To update the order status, could you please provide your customer name and order number?

Create a new file named task9_agent_provide_customer_order_details.py

```python
<copy>
import os
import sys
import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="I'm Bob Martinez and my order number is 7820.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

Ensure the conversation ID is set from the earlier return request:

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=<conversation_id>
</copy>
```

```bash
<copy>
python3.12 task9_agent_provide_customer_order_details.py
</copy>
```

Expected Output:

Agent Response:

Your replacement for order 7820 is on its way, and you will receive a return shipping label shortly. The order status has been updated to replaced. Do you need any further help?

Create a file named task9_agent_end_return_conversation.py

```python
<copy>
import sys
import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="No, thank you.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

Ensure the conversation ID is set from the earlier return request:

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=<conversation_id>
</copy>
```

```bash
<copy>
python3.12 task9_agent_end_return_conversation.py
</copy>
```

Expected output:

Agent Response:

Thank you for your feedback. Your order #7820 has been updated to 'replaced'. If you need anything else in the future, feel free to reach out. Have a great day!

## Task 10: Create a Function to Generate an Email

You'll create a function to return a confirmation email as a CLOB and test the output.

Create the build_return_email function.

Log in to Oracle Autonomous AI Database as ADB_USER, then run the following SQL statements.

```sql
<copy>
CREATE OR REPLACE FUNCTION build_return_email (
    p_order_number      IN  VARCHAR2,
    p_item_name         IN  VARCHAR2,
    p_customer_name     IN  VARCHAR2,
    p_reason_for_return IN  VARCHAR2,
    p_email_type        IN  VARCHAR2   -- 'refund' or 'replacement'
) RETURN CLOB IS
    l_subject VARCHAR2(4000);
    l_body    CLOB;
    l_nl      CONSTANT VARCHAR2(2) := CHR(10);
    l_type    VARCHAR2(20) := LOWER(TRIM(p_email_type));
    l_json    CLOB;
BEGIN
    IF l_type NOT IN ('refund', 'replacement') THEN
        RAISE_APPLICATION_ERROR(-20001,
            'email_type must be ''refund'' or ''replacement''');
    END IF;

    l_subject := 'Your Return Request for Order ' || p_order_number ||
                ' - ' || p_item_name;

    l_body := 'Dear ' || p_customer_name || ',' || l_nl || l_nl ||
            'Thank you for contacting us regarding your return request!' || l_nl || l_nl ||
            'This email confirms that we have received your request to return the following item from order number ' ||
            p_order_number || ':' || l_nl || l_nl ||
            'Item: ' || p_item_name || l_nl ||
            'Reason for Return: ' || p_reason_for_return || l_nl || l_nl;

    IF l_type = 'refund' THEN
        l_body := l_body ||
        'A full refund will be issued to the credit card on file. ' ||
        'Please allow 7-10 business days for the refund to reflect in your account.' || l_nl || l_nl;
    ELSE
        l_body := l_body ||
        'Your replacement for the ' || p_item_name || ' will be shipped within 3-5 business days. ' ||
        'You will receive a separate email with tracking information once your replacement has shipped.' || l_nl || l_nl;
    END IF;

    l_body := l_body ||
            'Please let me know if you have any questions or if there is anything else I can assist you with.' || l_nl || l_nl ||
            'Thank you, and have a great day!' || l_nl;

    -- Build JSON in PL/SQL to avoid RETURNING clause issues
    DECLARE
        jo JSON_OBJECT_T := JSON_OBJECT_T();
    BEGIN
        jo.put('subject', l_subject);
        jo.put('body',    l_body);
        l_json := TO_CLOB(jo.to_string);
    END;

    RETURN l_json;
END;
</copy>
```

Test the build_return_email function and verify the output.

```sql
<copy>
DECLARE
        v_json CLOB;
    BEGIN
        v_json := build_return_email(
            p_order_number      => '1234',
            p_item_name         => 'smartphone charging cord',
            p_customer_name     => 'Alice Thompson',
            p_reason_for_return => 'Item not compatible with my phone',
            p_email_type        => 'replacement'
        );

        DBMS_OUTPUT.PUT_LINE(v_json);
    END;
Result:
{"subject":"Your Return Request for Order 1234 - smartphone charging cord","body":"Dear Alice Thompson,\n\nThank you for contacting us regarding your return request!\n\nThis email confirms that we have received your request to return the following item from order number 1234:\n\nItem: smartphone charging cord\nReason for Return: Item not compatible with my phone\n\nYour replacement for the smartphone charging cord will be shipped within 3-5 business days. You will receive a separate email with tracking information once your replacement has shipped.\n\nPlease let me know if you have any questions or if there is anything else I can assist you with.\n\nThank you, and have a great day!\n"}
PL/SQL procedure successfully completed.
Elapsed: 00:00:00.023
</copy>
```

NOTE: The result is provided in JSON format and the result string includes \n for returns. Your application code can extract the subject and body and send using an email tool. Using the email tool is left as an unscripted exercise.

## Task 11: Create an Email Tool

You’ll create an email tool with clear instructions on what the tool should do.

Create a build_email_tool that specifies the build_return_email function.

Create a new file named task11_agent_create_build_email_tool.py

```python
<copy>
import sys
import select_ai
from select_ai.agent import Tool, ToolAttributes
from select_ai_agent_common import connect_to_adb
TOOL_NAME = "build_email_tool"
def main():
    connect_to_adb()

    try:
        Tool.delete_tool(
            tool_name=TOOL_NAME,
            force=True,
        )

        tool = Tool(
            tool_name=TOOL_NAME,
            description=(
                "Tool for building a customer email for a refund or replacement."
            ),
            attributes=ToolAttributes(
                instruction=(
                    "This tool constructs an email string using details from "
                    "the customer interaction about the product they want to "
                    "return, replace, or receive a refund for."
                ),
                function="build_return_email",
                tool_inputs=[
                    {
                        "name": "p_email_type",
                        "description": (
                            "Only supported values are 'refund' and "
                            "'replacement'."
                        ),
                    }
                ],
            ),
        )

        tool.create(
            enabled=True,
            replace=True,
        )

        print(f"Created tool: {tool.tool_name}")

    except Exception as error:
        print("Failed to create build email tool.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task11_agent_create_build_email_tool.py
</copy>
```

Expected output:

Created tool: build_email_tool

## Task 12: Update the Handle Product Return Task

You'll update the Handle_Product_Return_Task with a section to use the build_email_tool. The task gathers fields from the attributes, calls the tool, and returns the email text to review.

Create a new file named task12_agent_update_product_return_task.py

```python
<copy>
import sys
import select_ai
from select_ai.agent import Task, TaskAttributes
from select_ai_agent_common import connect_to_adb
TASK_NAME = "Handle_Product_Return_Task"
UPDATE_ORDER_STATUS_TOOL = "Update_Order_Status_Tool"
BUILD_EMAIL_TOOL = "build_email_tool"
def main():
    connect_to_adb()

    try:
        Task.delete_task(
            task_name=TASK_NAME,
            force=True,
        )

        task = Task(
            task_name=TASK_NAME,
            description=(
                "Handles product-return requests and generates a customer "
                "confirmation email."
            ),
            attributes=TaskAttributes(
                instruction="""
Process a product return request from a customer: {query}

1. Ask the customer for the reason for the return:
   no longer needed, arrived too late, box broken, or defective.

2. If no longer needed:
   a. Inform the customer to ship the product back at their expense.
   b. Update the order status to return_shipment_pending using
      Update_Order_Status_Tool.

3. If the product arrived too late:
   a. Ask the customer whether they want a refund.
   b. If the customer wants a refund, confirm that the refund is
      processed and update the order status to refund_completed.

4. If the product was defective or the box was broken:
   a. Ask the customer whether they want a replacement or a refund.
   b. For a replacement, inform the customer that the replacement is
      on its way and that they will receive a return shipping label.
      Update the order status to replaced.
   c. For a refund, instruct the customer to print the return shipping
      label, return the product, and update the order status to refund.

5. Use build_email_tool to generate a confirmation email for a refund
   or replacement request. Display the email to the customer and ask
   them to confirm that the information is correct.

6. After completing a return, replacement, or refund, ask whether the
   customer needs any further help. Say a friendly goodbye if they do not.
""",
                tools=[
                    UPDATE_ORDER_STATUS_TOOL,
                    BUILD_EMAIL_TOOL,
                ],
                enable_human_tool=True,
            ),
        )

        task.create(
            enabled=True,
            replace=True,
        )

        print(f"Updated task: {task.task_name}")

    except Exception as error:
        print("Failed to update the product return task.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task12_agent_update_product_return_task.py
</copy>
```

Expected output:

Updated task: Handle_Product_Return_Task

## Task 13: Update the Agent Team

Add the new task into your agent team, which now includes the email generation tool as part of the Handle_Product_Return_Task.

Create a new file named task13_agent_recreate_return_agency_team.py

```python
<copy>
import sys
import select_ai
from select_ai.agent import Team, TeamAttributes
from select_ai_agent_common import connect_to_adb
TEAM_NAME = "Return_Agency_Team"
AGENT_NAME = "Customer_Return_Agent"
TASK_NAME = "Handle_Product_Return_Task"
def main():
    connect_to_adb()

    try:
        Team.delete_team(
            team_name=TEAM_NAME,
            force=True,
        )

        team = Team(
            team_name=TEAM_NAME,
            description=(
                "Sequential team for handling customer product returns "
                "and generating confirmation emails."
            ),
            attributes=TeamAttributes(
                agents=[
                    {
                        "name": AGENT_NAME,
                        "task": TASK_NAME,
                    }
                ],
                process="sequential",
            ),
        )

        team.create(
            enabled=True,
            replace=True,
        )

        print(f"Created team: {team.team_name}")

    except Exception as error:
        print("Failed to recreate the Return Agency Team.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task13_agent_recreate_return_agency_team.py
</copy>
```

Expected output:

Created team: Return_Agency_Team

## Task 14: Interact with the Agent

Create a new file named task14_agent_run_return_agency_team.py to start a new Return Agency Team conversation

```python
<copy>
import datetime
import sys
import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    try:
        conversation = select_ai.Conversation(
            attributes=select_ai.ConversationAttributes(
                title="Sales Return Request with Email Confirmation",
                description=(
                    "Customer conversation with the Sales Return Agent."
                ),
                retention_days=datetime.timedelta(days=7),
                conversation_length=20,
            )
        )

        conversation_id = conversation.create()

        print(f"Created conversation ID: {conversation_id}")
        print(
            "Save it for follow-up prompts:\n"
            f"export SELECT_AI_CONVERSATION_ID={conversation_id}"
        )

        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="I want to return a smartphone protective case.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("\nAgent Response:")
        print(response)

    except Exception as error:
        print("Failed to run the Return Agency Team.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
python3.12 task14_agent_run_return_agency_team.py
</copy>
```

Expected output:

Created conversation ID: 5AF7E98D-1FFE-025F-E063-4BD5000A2B70

Save it for follow-up prompts:

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=5AF7E98D-1FFE-025F-E063-4BD5000A2B70
</copy>
```

Agent Response:

What is the reason for your return of the smartphone protective case? The options are: no longer needed, arrived too late, box broken, or defective.

Create a new file named task14_agent_reason_arrived_too_late.py

```python
<copy>
import sys

import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="It arrived too late.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
Run the script:
export SELECT_AI_CONVERSATION_ID=<conversation_id>
python3.12 task14_agent_reason_arrived_too_late.py
Expected output:
Agent Response:
Would you like a refund since the product arrived too late?
</copy>
```

Create a new file named task14_agent_confirm_refund.py

```python
<copy>
import sys
import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="Yes.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=<conversation_id>
</copy>
```

```bash
<copy>
python3.12 task14_agent_confirm_refund.py
</copy>
```

Expected output:

Agent Response:

To process your refund, could you please confirm your customer name and order number?

Create a new file named task14_agent_provide_refund_order_details.py

```python
<copy>
import sys
import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt="I am Carol Chen and my order number is 7635.",
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=<conversation_id>
</copy>
```

```bash
<copy>
python3.12 task14_agent_provide_refund_order_details.py
</copy>
```

Expected output:

Agent Response:

Thank you. To generate your refund confirmation email, what is the name of the item from order 7635? Also, please confirm the reason for return is 'arrived too late'.

Create a new file named task14_agent_confirm_refund_details.py

```python
<copy>
import sys

import select_ai
from select_ai.agent import Team
from select_ai_agent_common import connect_to_adb, required_env
TEAM_NAME = "Return_Agency_Team"
def main():
    connect_to_adb()

    conversation_id = required_env("SELECT_AI_CONVERSATION_ID")

    try:
        team = Team.fetch(TEAM_NAME)

        response = team.run(
            prompt=(
                "The item is smartphone backup storage, and yes, "
                "the reason for return is that it arrived too late."
            ),
            params={
                "conversation_id": conversation_id,
            },
        )

        print("Agent Response:")
        print(response)

    except Exception as error:
        print("Failed to continue the return conversation.")
        print(error)
        sys.exit(1)

    finally:
        select_ai.disconnect()
if __name__ == "__main__":
    main()
</copy>
```

Run the script:

```bash
<copy>
export SELECT_AI_CONVERSATION_ID=<conversation_id>
</copy>
```

```bash
<copy>
python3.12 task14_agent_confirm_refund_details.py
</copy>
```

Expected output:

Agent Response:

Here is the confirmation email for your refund:

Subject: Your Return Request for Order 7635 - smartphone backup storage

Body: Dear Carol Chen,

Thank you for contacting us regarding your return request!

This email confirms that we have received your request to return the following item from order number 7635:

Item: smartphone backup storage

Reason for Return: arrived too late

A full refund will be issued to the credit card on file. Please allow 7-10 business days for the refund to reflect in your account.

Please let me know if you have any questions or if there is anything else I can assist you with.

Thank you, and have a great day!

## Learn More

* [Select AI Agent Framework Interfaces](https://docs.oracle.com/en/database/oracle/oracle-database/26/selai/select-ai-agent-interfaces.html)
* [Examples of Using the Select AI Agent Framework](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/examples-using-select-ai-agent.html)
* [Multi-Agent Pattern for AI Agents](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/multi-agent-pattern-ai-agents.html)
* [`DBMS_CLOUD_AI_AGENT` Package](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/dbms-cloud-ai-agent-package.html)
* [Select AI for Python](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-py.html)

## Acknowledgements

- **Created By:** Chandrakanth Putha, Oracle AI Database Product Manager
- **Contributors:** Mark Hornick, Product Management for Select AI and Machine Learning
- **Last Updated By:** Dhanish Kumar, Senior Member of Technical Staff
