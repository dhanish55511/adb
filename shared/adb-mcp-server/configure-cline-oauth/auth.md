# Configure Cline in Visual Studio Code to Use an MCP Server with OAuth Authentication

## Introduction

In the previous lab 6, you used Cline which is a Visual Studio Code extension to call database tools exposed through the MCP Server using bearer token.

In this lab, you will learn how to set up and use Visual Studio Code with the Cline extension as an MCP client using OAuth authentication. In OAuth authentication you don’t need to manually generate and paste a token. Instead, you’ll configure MCP connectivity to use a browser-based OAuth sign-in flow, then verify that Cline can load and invoke the same database tools created earlier.

This lab demonstrates how different AI clients connect to the same MCP-enabled database instance.

Estimated Time:20 Minutes

### Objectives

In this lab, you will:

* Install Visual Studio Code (if not already installed)
* Install and configure the Cline extension
* Configure authentication for the MCP Server using either:
  * OAuth (browser-based sign-in), or
  * a bearer token (generated using cURL)
* Configure Cline to securely connect to your Autonomous AI Database MCP Server
* Verify that Cline loads available database tools
* Select and manage tool access through the Cline extension

## Task 1:  Install Visual Studio Code 

If you already have Visual Studio Code installed on your system, proceed to Task 2.

If Visual Studio Code is not installed, follow the instructions below to install it. Install Visual Studio Code on your computer based on your operating system — Windows or Mac. Choose from:

#### For Windows:

1. Go to the official [download page](https://code.visualstudio.com/download)
2. Download the Windows installer (.exe file).
3. Run the installer and follow the instructions in the setup wizard.
4. Once installed, launch Visual Studio Code.

#### For macOS:

1. Go to the official [download page](https://code.visualstudio.com/download)
2. Download the macOS installer (`.dmg` file).
3. Open the `.dmg` file and drag Visual Studio Code into your **Applications** folder.
4. Launch Visual Studio Code from **Applications**.

## Task 2: Install the Cline Extension

In this task, you will install the Cline extension in Visual Studio Code.

1. In Visual Studio Code, click the Extensions icon. 
  ![Cline main window](../configure-cline/images/cline-vs-extension.png)
2. In the **Activity Bar** (or use Ctrl+Shift+X) search for **Cline**.
  ![Cline main window](../configure-cline/images/cline-install.png)
3. Click **Install** next to the Cline extension.
4. If prompted, click **Trust Publisher & Install**. 
  ![Cline main window](../configure-cline/images/cline-trust-yes.png)
5. Once installed, the Cline extension icon appears in the Activity Bar. 
  ![Cline main window](../configure-cline/images/cline-after-installation.png)
6. Click the Cline icon and confirm the interface loads without error.
7. Optionally, if this is your first time using Cline, open the extension to complete creating a Cline account by following the prompts on the screen.

## Task 3: Authenticate to the MCP Server (OAuth or Bearer Token)

To authenticate with the MCP server, choose **one** of the following methods:
