# kogito

This repository showcases some of the key capabilities of Kogito specifically on how Kogito:

1. Provides runtime persistence for workflows to preserve the state of the process instances across restarts. 
2. Supports first class citizen support for events and enables integration with third party systems with both events and external REST API calls. 
3. Enables the tracking of the process instances progress from the Kogito process management console. 

To start with clone this repo:

git clone https://github.com/bbalakriz/kogito

Then move to the apps folder.

cd apps/coffee-shop

Open the drink-order-process.bpmn2 using either the online BPMN editor - https://kiegroup.github.io/kogito-online/#/ (by dragging and dropping the bpmn file) or do the following to open up the coffee-shop project in VSCode. 

1. First, install the Kogito Extension in VSCode.
2. Download the latest Visual Studio plugin from the project page: https://github.com/kiegroup/kogito-tooling/releases
3. Select the latest version
4. From asset section download the file vscode_extension_kogito_kie_editors_n.n.n.vsix
5. Open Visual Studio Code
6. Select the Extensions pane on the left
7. Click the... icon on the top right
8. Select Install from VSIX...

This process depicts the typical drink ordering steps at a coffee shop where a customer would walk in, place an order at the counter desk and then make payment at the payment desk either by cash or by a credit card - and when it is a credit card payment, the payment processing would be handled by an external payment gateway. Once the payment is successful, the barista would be notified to get started with the preparation of the required coffee. 

In here, the barista process has been designed to run as a seperate Kogito application and would be reacting to the coffee drink order taking process. Take a look at the barista process, it is a very simple one that shows that the barista would prepare the coffee and the customer would collect it once it is ready..
