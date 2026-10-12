# Set up Your Workshop Environment

## Introduction

This lab guides you through setting up an Autonomous AI Database Serverless environment and installing and configuring Oracle Select AI for Python on an OCI Compute instance. Together, these services let you build AI-powered applications that answer natural-language questions about your data.

### Objectives

In this lab, you will:

- Provision an Autonomous AI Database instance.
- Generate SSH keys and provision an OCI Compute instance.
- Install and configure Oracle Select AI for Python.

## Task 1: Create the Autonomous AI Database Instance

On the **Autonomous AI Databases** page in the OCI Console, select your region and compartment, then click **Create Autonomous AI Database**.

### Choose the workload type

In **Workload type**, select **Lakehouse** for this workshop. The other available types are:

- **Transaction Processing**: For transaction processing, mixed transaction and analytics workloads, IoT, and JSON document applications.
- **JSON**: For NoSQL-style applications that use JSON documents. You can store up to 20 GB of non-JSON data.
- **APEX**: A low-cost Oracle Cloud service for building and deploying low-code applications.

Lakehouse is designed for analytics and AI across your data.

![Choose the Lakehouse workload type](./images/adb-workload-type.png =70%x*)

### Configure the database

In **Database configuration**, set the following values:

- **Always Free**: Leave the slider disabled unless you are using an Always Free account. An Always Free database must be deployed in your tenancy's home region, and you can choose only its database version. Always Free databases cannot be scaled up or down.
- **Developer**: Leave the slider disabled. Developer databases are a low-cost option for development and can be upgraded to a full paid version later.
- **Database version**: Select **26ai**.
- **ECPU count**: Select **2 ECPUs**. This setting is not available for an Always Free database.
- **Compute auto scaling**: Keep the default enabled setting.
- **Storage**: Select **1 TB**. An Always Free database includes 20 GB.
- **Storage auto scaling**: Keep the default disabled setting.

You can open **Advanced options** to review elastic pools or bring your own license options.

![Configure the database](./images/adb-database-configuration.png =70%x*)

### Set backup options

In **Backup**, accept the default automatic backup retention period of 60 days and leave immutable backups disabled.

![Set backup options](./images/adb-backup-retention.png =70%x*)

### Set administrator credentials

The administrator username is **ADMIN**. Record the password you create; you will use these credentials later.

The password must be 12 to 30 characters long and include at least one uppercase letter, one lowercase letter, and one number. It cannot contain the username, a double quote (`"`), or a single quote (`'`). It must also differ from your last four passwords and from any password you set in the last 24 hours.

![Set the administrator password](./images/adb-administrator-credentials.png =70%x*)

### Set network access

For this lab, keep **Secure access from everywhere** selected. To restrict database access, choose **Secure access from allowed IPs and VCNs only** or **Private endpoint access only**. If you select **Require mutual TLS (mTLS) authentication**, clients must use mTLS to connect. TLS connections without a wallet are supported with JDBC thin driver and JDK 8 or later; see the Oracle documentation for the available network options.

![Choose network access](./images/adb-network-access.png =70%x*)

### Add a contact and create the database

In **Contacts for operational notifications and announcements**, enter an email address for service notices, announcements, and unplanned maintenance notifications.

![Add an operational contact](./images/adb-operational-contact.png =70%x*)

Click **Create**. The database details page opens with the status **Provisioning**. If the guided tour appears, click **Skip tour**, then confirm by clicking **Skip**.

![Skip the database guided tour](./images/adb-provisioning-tour.png =70%x*)

Wait for the instance status to change to **Available**. Review the displayed name, database name, database version, ECPU count, and storage size. The display name and database name in your tenancy may differ from the sample.

![Review the available database details](./images/adb-available-details.png =70%x*)

To return to the list of databases, click **Autonomous AI Databases** at the top of the page.

![Return to the Autonomous AI Databases page](./images/adb-database-list-navigation.png =70%x*)

Confirm that your new database appears in the list.

![Find the new database in the list](./images/adb-database-list.png =70%x*)

OCI also sends a confirmation email to the contact address you provided. It includes links to Database Actions and other database resources.

![Database provisioning confirmation email](./images/adb-provisioning-email.png =70%x*)

## Task 2: Create an Autonomous AI Database User

1. If you are not signed in to OCI, open the cloud menu, select **Oracle AI Database**, and then select **Autonomous AI Database**.

   ![Open Autonomous AI Database](./images/select-adb.png =70%x*)

2. Select the database you created for this workshop. The example uses **Training-Database**.

   ![Select the workshop database](./images/click-dbinstance.png =70%x*)

3. On the database details page, click **Database Actions**, then select **Database Users**.

   ![Open Database Users](./images/db-user.png =70%x*)

4. If prompted to sign in before the Database Actions Launchpad opens, enter **ADMIN** and the administrator password you created.

5. Click **Create User**.

   ![Open the Create User dialog](./images/db-actions-create-user.png =70%x*)

6. Enter the user details and click **Create User**.

   ![Create ADB_USER](./images/create_adb_user.png =70%x*)

   - **User Name**: Enter `ADB_USER`.
   - **Password**: Enter a password that is 12 to 30 characters long and contains at least one uppercase letter, one lowercase letter, and one number. It cannot contain a double quote or the username.
   - **Confirm Password**: Re-enter the password.
   - **Quota of tablespace data**: Select a quota. For a typical Always Free database, select **UNLIMITED**.
   - **Password Expired**: Select this option if the user must reset their password.
   - **Account is locked**: Select this option to lock the account.
   - **OML**: Select this required option to allow access to Oracle Machine Learning.
   - **Graph**: Optionally, select this option to enable Graph.
   - **Web Access**: Optionally, select this option to allow this user to access Web and Database Actions.
   - **Show Code**: Optionally, click this to view or copy the user creation and role-grant SQL.

7. Confirm that the success message says `User ADB_USER created successfully`.

## Task 3: Grant Privileges

1. Open your database instance and select the **Tool Configuration** tab.
2. Click **Copy** next to the **Database Actions** URL.

   ![Copy the Database Actions URL](./images/db-actions-url.png =70%x*)

3. Paste the URL into your browser and sign in as **ADMIN**.
4. Open the **Development** tab, then select **SQL** in the left navigation to open the SQL worksheet.
5. Run the following commands to grant the required privileges to the schema user:

   <copy>
   ```sql
   GRANT EXECUTE ON DBMS_CLOUD_AI TO ADB_USER;
   GRANT EXECUTE ON DBMS_CLOUD_AI_AGENT TO ADB_USER;
   GRANT EXECUTE ON DBMS_CLOUD_PIPELINE TO ADB_USER;
   GRANT EXECUTE ON DBMS_CLOUD TO ADB_USER;
   ```
   </copy>

Replace `ADB_USER` with your schema user name if you chose a different name.

## Task 4: Generate SSH Keys with PuTTYgen

Generate an SSH key pair to connect to your OCI Compute instance.

1. Open PuTTYgen on your local machine.
2. Select **EdDSA** and confirm that **Ed25519** is selected.
3. Click **Generate**, then move your mouse in the blank area until key generation finishes.
4. Save the private key as a `.ppk` file. You can set an optional passphrase.
5. Copy the public key text and save it for the OCI Console. It should begin with `ssh-ed25519`.

## Task 5: Create an OCI Compute Instance

1. In the OCI Console, open the navigation menu, select **Compute**, then **Instances**.

   ![Open the Compute instances page](./images/compute-instances.png =70%x*)

2. Click **Create instance**.
3. In **Basic Information**, enter an instance name and select a compartment.

   ![Enter the instance name and compartment](./images/compute-instance-basic-information.png =70%x*)

4. Accept the defaults and click **Next** until you reach the **Networking** screen.
5. Under **Primary VNIC**, enter a VNIC name.

   ![Configure the primary VNIC](./images/compute-primary-vnic.png =70%x*)

6. Select **Create new virtual cloud network**. Enter a VCN name and select its compartment.
7. Under **Subnet**, keep **Create new public subnet** selected. Enter a subnet name and select its compartment.
8. Under **Add SSH keys**, select **Paste public key** and paste the public key you generated with PuTTYgen.

   ![Add the SSH public key](./images/compute-ssh-key.png =70%x*)

9. Continue through the remaining screens with the default values. Review the details and click **Create**.
10. Wait for the instance state to change to **Running**. Copy its public IP address for the SSH connection.

    ![Confirm the Compute instance is running](./images/compute-instance-running.png =70%x*)

## Task 6: Install and Configure Oracle Select AI for Python

### Connect to the Compute instance

1. Open PuTTY and enter the Compute instance's public IP address. You can save the session for later.

   ![Configure the PuTTY session](./images/putty-session-configuration.png =70%x*)

2. Optionally, set the keepalive interval in seconds.

   ![Set the PuTTY keepalive interval](./images/putty-keepalive.png =70%x*)

3. In PuTTY, open **Connection > SSH > Auth > Credentials** and browse to the private key `.ppk` file you saved.

   ![Load the private key in PuTTY](./images/putty-private-key.png =70%x*)

4. Save the session, connect to the instance, and sign in as `opc`.

### Install Python and Select AI for Python

Enable the Oracle Linux 9 Developer repository:

```bash
<copy>
sudo dnf config-manager --enable ol9_developer
</copy>
```

Install Python 3.12, its development libraries, and pip:

```bash
<copy>
sudo dnf install -y python3.12 python3.12-devel python3.12-pip
python3.12 --version
</copy>
```

Create and activate a virtual environment named `.venv`:

```bash
<copy>
python3.12 -m venv .venv
source .venv/bin/activate
</copy>
```

Upgrade pip and install Select AI for Python:

```bash
<copy>
python3.12 -m pip install --upgrade pip
python3.12 -m pip install --upgrade select_ai
</copy>
```

To install the optional command-line interface, run:

```bash
<copy>
python3.12 -m pip install --upgrade "select_ai[cli]"
</copy>
```

Install the OCI Python SDK:

```bash
<copy>
python3.12 -m pip install --upgrade oci
</copy>
```

### Install and configure the OCI CLI

Install the OCI CLI package:

```bash
<copy>
sudo dnf install python39-oci-cli
</copy>
```

When prompted, enter `y`. Verify the installation:

```bash
<copy>
oci --version
</copy>
```

Start the OCI CLI configuration wizard:

```bash
<copy>
oci setup config
</copy>
```

Respond to the prompts as follows:

- For the config file location, press Enter to accept `/home/opc/.oci/config`.
- Enter your **User OCID** and **Tenancy OCID**.
- Enter the region ID for your Generative AI service, such as `<your-gen-ai-region>`.
- When asked whether to generate a new API signing RSA key pair, enter `Y`.
- Press Enter to accept the default directory `/home/opc/.oci` and key name `oci_api_key`.
- For the private key passphrase, enter `N/A`, then enter `N/A` again to confirm.

Display the public key:

```bash
<copy>
cat /home/opc/.oci/oci_api_key_public.pem
</copy>
```

In the OCI Console, open your profile, select your name or email address, then open **Tokens and Keys**. Under **API Keys**, click **Add API Key**, select **Paste a public key**, paste the public key, and click **Add**.

### Download and configure the database wallet

Use the OCI CLI to download the Autonomous AI Database wallet. Replace the database OCID and wallet password with your own values.

```bash
<copy>
oci db autonomous-database generate-wallet --autonomous-database-id <database-ocid> --file wallet_adb.zip --password <wallet-password>
</copy>
```

![Download the database wallet](./images/wallet-download.png =70%x*)

Create a wallet directory and move the downloaded archive into it:

```bash
<copy>
mkdir wallet
mv wallet_adb.zip wallet/
</copy>
```

![Create the wallet directory](./images/wallet-directory.png =70%x*)

Extract the wallet files:

```bash
<copy>
cd wallet
unzip wallet_adb.zip
</copy>
```

![Extract the wallet files](./images/wallet-extraction.png =70%x*)

From the wallet directory, run `vi sqlnet.ora`. Edit the file so `WALLET_LOCATION` points to `/home/opc/wallet`, and keep `SSL_SERVER_DN_MATCH=yes`.

```text
<copy>
WALLET_LOCATION = (SOURCE = (METHOD = file) (METHOD_DATA = (DIRECTORY="/home/opc/wallet")))
SSL_SERVER_DN_MATCH=yes
</copy>
```

![Update sqlnet.ora](./images/wallet-sqlnet-configuration.png =70%x*)

### Test the Select AI database connection

Set the database connection environment variables in the active shell. Replace the example values with your schema credentials, a TNS service name from `tnsnames.ora`, and your wallet password.

```bash
<copy>
export SELECT_AI_USER="ADB_USER"
export SELECT_AI_PASSWORD="<database-user-password>"
export SELECT_AI_DB_CONNECT_STRING="<tns-service-name>"
export SELECT_AI_WALLET_DIR="/home/opc/wallet"
export SELECT_AI_WALLET_PASSWORD="<wallet-password>"
</copy>
```

Create a file named `select_ai_connection_test.py` with the following code:

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


user = required_env("SELECT_AI_USER")
password = required_env("SELECT_AI_PASSWORD")
dsn = required_env("SELECT_AI_DB_CONNECT_STRING")
wallet_dir = required_env("SELECT_AI_WALLET_DIR")
wallet_password = required_env("SELECT_AI_WALLET_PASSWORD")
wallet_path = Path(wallet_dir)

for filename in ("tnsnames.ora", "ewallet.pem"):
    if not (wallet_path / filename).exists():
        print(f"{filename} not found in {wallet_dir}")
        sys.exit(1)

try:
    select_ai.connect(
        user=user,
        password=password,
        dsn=dsn,
        wallet_location=wallet_dir,
        config_dir=wallet_dir,
        wallet_password=wallet_password,
    )
    print("Connected to the Database")
except Exception as exc:
    print("Connection failed:")
    print(exc)
    sys.exit(1)
</copy>
```

Run the connection test:

```bash
<copy>
python3.12 select_ai_connection_test.py
</copy>
```

If the connection succeeds, the script displays `Connected to the Database`. You may now proceed to the next lab.

## Learn More

* [Provision an Autonomous AI Database Instance](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/autonomous-provision.html)
* [Creating an OCI Compute Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
* [Connect Python Applications to Autonomous AI Database](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/connecting-python.html)
* [Select AI for Python](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-py.html)

## Acknowledgements

- **Created By:** Chandrakanth Putha, Oracle AI Database Product Manager
- **Contributors:** Mark Hornick, Product Management for Select AI and Machine Learning
- **Last Updated By:** Dhanish Kumar, Senior Member of Technical Staff
