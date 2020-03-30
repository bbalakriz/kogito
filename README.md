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

## Start the required containers

All the relevant containers that are required to run `apps/coffee-shop` and `apps/barista` are contained in `config/all-in-one-docker-compose.yaml`. So, these containers can be easily started by using docker-compose.

`cd config
 docker-compose -f all-in-one-docker-compose.yaml up -d`

This will start up 6 containers namely - Kafka, ZooKeeper, Infinispan, Keycloak, Kogito Data Index Engine and Kogito Management Console. 

## Coffee-shop Project - Quick Overview

The drink-order-process (located at coffe-shop/src/main/resource/org/bala/drink/process) depicts the typical drink ordering steps at a coffee shop where a customer would walk in, place an order at the counter desk and then make payment at the payment desk either by cash or by a credit card - and when it is a credit card payment, the payment processing would be handled by an external payment gateway. Once the payment is successful, the barista would be notified to get started with the preparation of the required coffee. 

## Barista Project - Quick Overview

The barista process has been designed to run as a seperate Kogito application and would be reacting to the coffee drink order taking process. Again, it is a very simple one that shows that the barista would prepare the coffee and the customer would collect it once it is ready.

## Process persistence

Kogito provides process persistence through Infinispan, which is an in memory data grid. Process persistence can be enabled in Kogito by simply adding the extensions `quarkus-infinispan-client` and `infinispan-persistence-addon` to the Kogito project. The exact dependency details that are to be added in the pom.xml for these extensions are:

    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-infinispan-client</artifactId>
    </dependency>
    <dependency>
      <groupId>org.kie.kogito</groupId>
      <artifactId>infinispan-persistence-addon</artifactId>
    </dependency>
    
The connection configuration using which the Kogito application can connect to Infinispan, needs to be specified in the application.properties. 

    quarkus.infinispan-client.server-list=localhost:11222
    quarkus.infinispan-client.auth-server-name=infinispan
    quarkus.infinispan-client.auth-username=infinispan
    quarkus.infinispan-client.auth-password=infinispan
    quarkus.infinispan-client.auth-realm=default
    quarkus.infinispan-client.client-intelligence=BASIC

### Data Containers for processes

Open the Infinispan management console in the browser by navigating to `https://localhost:11222` with the credentials `infinispan\infinispan`


Let's now run this Kogito app and see the persistence in action. This is the infinispan console that would display the list of all data stores. It's empty now but the application specific backing store will get created once the application in started. So, let me start the application. The application is up and running and we can see that the drink order persistent data store has got created on the fly. 

Once the application is started, the process specific rest APIs are automatically generated and we'll be using the swagger UI to invoke these APIs and interact with the application. The open API & swagger capabilities gets automatically enabled in the application by adding the quarkus open API extension. 

Let's start the drink order process from the Swagger UI. The process instance has been started and we can track the process instance from the process management console. As we can see here, the proces instance is waiting at the place order step. Since the persistence is enabled, even if the kogito application is taken down, the process instance would remain intact in its current state. So, let me terminate the application and we can see that process instance is still remaining in its current state. Now, let me bring back the application.

Let's complete the place order and make payment steps by using the step specific REST APIs. The place order step is complete now and before we complete the make payment step, let's take a look at the how the payment processing happens. Once the payment details are specified in the make payment step, and if the payment method is specified as credit card, the payment processing automated step would get triggered which in turn is configured to invoke a payment gateway API. Let's have a quick look at the process payment step and we can see that the service that's linked to this step is Payments Service and payment processing service operation will be invoked in the payments service. Loooking at the implementation of the payment service, we don't see much of business logic in here except for the invokation of the external payment gateway API. The external API invokation has been made simple with the help of a Rest Client extension. The extension can be directly added in the pom.xml as can be seen here. With that in place, an interface for the rest client just needs to be registered with the relevant HTTP path and method details. The actual endpoint url of the payment gateway has been externalized and added to the application configuratoin properties file. With that let me continue the exection of the process instance by completing the make payment step with the credit card details and we will see that the payment gateway API wil be invoked and the barista would be notified of the order. 

From the management console we can see that barista process has started. Now, how this happened. This was made possible using the native reactive messaging support provided by Kogito. Going back to the process model, we can see that the step Inform barista has been modeled to message event end that has been configured to send order details to the specified barista-process topic in a stream processing engine like Kafka. And all of this has been done without any coding and with just few configurations. In the application config file, the destination topic to which the message has to be published needs to be specified along with the connection details, and the Kogito event publisher would take care of publishing the event to the specified topic. Again all of this is done without any coding but just with few configurations and by enabling the kogito reactive messaging add ons in the pom.xml

On the recieving side, the barista process has been configured to listen to the same topic and the KOgito event reciever would then recieve the event and start the barista process. And this is how we have got the new instance of barista process created automaitcally when the drink order process published an event.

Besides that, the Kogito applications can be configured to publish process and user task events at various stages of the process execution, so that any other external systems that needs to react to the process instance execution can do so by listening to those events that are emitted at various stages of process execution. For instance, the inbuilt  kogito-processinstances-events and userevents publishing has been enabled in this application. Let's connnect to these configured topics using a kafka client and see what events has got published. 

