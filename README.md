# Kogito - Coffee Shop 

This repository showcases some of the key capabilities of Kogito specifically on how it:

1. Provides runtime persistence for workflows to preserve the state of the process instances across restarts
2. Supports first class citizen support for events and enables integration with third party systems with both events and external REST API calls
3. Enables the tracking of the process instances progress from the Kogito process management console

## Pre-requisites

To start with, ensure that the following are installed in your machine.

-   [OpenJDK 8+](https://computingforgeeks.com/how-to-install-java-11-openjdk-11-on-rhel-8)
-   [Visual Studio Code 1.41.1](https://code.visualstudio.com/docs/setup/linux)
-   [Kogito Visual Studio Code extension (latest)](https://github.com/kiegroup/kogito-tooling/releases)
-   [Red Hat Java VSCode extension (latest)](https://marketplace.visualstudio.com/items?itemName=redhat.java)
-   [Maven 3.6.0+](https://maven.apache.org/install.html)
-   [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
-   [Docker Engine (atleast 1.13) and Docker Compose (atleast 1.25)](https://download.docker.com/)

## Installing Kogito tooling on Visual Studio Code

1. Download the latest Visual Studio plugin from the project page: https://github.com/kiegroup/kogito-tooling/releases
2. Select the latest version
3. From the 'Assets' section, download the file `vscode_extension_kogito_kie_editors_n.n.n.vsix`
4. Open Visual Studio Code
  a. Select the Extensions pane on the left
  b. Click the... icon on the top right
  c. Select Install from VSIX...
5. Exit VSCode 

## Importing & opening the projects into Visual Studio Code

 1. Open Visual Studio Code 
 2. Go to "File", "Add Folder to Workspace"
 3. Select the folder `coffee-shop` in your file system 
 4. Click "Add" 
 5. Repeat the same steps for the `barista` project as well. 
 
Alternatively, you could run the following command on your terminal: `code <<project-name>>` where project names are `apps/coffee-shop` and `apps/barista` for both these projects

## Coffee-shop Project - Quick Overview

The drink-order-process (located at coffe-shop/src/main/resource/org/bala/drink/process) depicts the typical drink ordering steps at a coffee shop where a customer would walk in, place an order at the counter desk and then make payment at the payment desk either by cash or by a credit card - and when it is a credit card payment, the payment processing would be handled by an external payment gateway. Once the payment is successful, the barista would be notified to get started with the preparation of the required coffee. 

## Barista Project - Quick Overview

The barista process has been designed to run as a seperate Kogito application and would be reacting to the coffee drink order taking process. Again, it is a very simple one that shows that the barista would prepare the coffee and the customer would collect it once it is ready..
