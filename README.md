# Kogito - Coffee Shop 

This repository showcases some of the key capabilities of Kogito specifically on how it:

1. Provides runtime persistence for workflows to preserve the state of the process instances across restarts
2. Supports first class citizen support for events and enables integration with third party systems with both events and external REST API calls
3. Enables the tracking of the process instances progress from the Kogito process management console
4. Integrates with an OpenId Connect Server to provide security to the the Kogito application endpoints 

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
 
Alternatively, the following commands could be run on a terminal: `code <<project-name>>` where project names are `apps/coffee-shop` and `apps/barista` to open both these projects in VSCode.

## Start the required containers

All the relevant containers that are required to run these applications `coffee-shop` and `barista` are contained in `config/all-in-one-docker-compose.yaml` and these containers could be started by using docker-compose.

`$ cd config && docker-compose -f all-in-one-docker-compose.yaml up -d`

This will start up 6 containers namely - Kafka, ZooKeeper, Infinispan, Keycloak, Kogito Data Index Engine and Kogito Management Console. 

## Coffee-shop project - A quick overview

The drink-order-process (_located at `coffee-shop/src/main/resource/org/bala/drink/process`_) depicts the typical drink ordering steps at a coffee shop where a customer would walk in, place an order at the counter desk and then make payment at the payment desk either by cash or by a credit card. If the payment type is credit card, the payment processing would be handled by an external payment gateway. Once the payment processing goes through successfully, the barista would be notified to get started with the preparation of the required coffee. 

![Drink Order Process](config/2.png?raw=true "Title")


## Barista project - A quick overview

The barista process has been designed to run as a seperate Kogito application and would be reacting to the coffee drink order taking process. Again, it is a very simple one that shows that the barista would prepare the coffee and the customer would collect it once it is ready.

![Barista Process](config/3.png?raw=true "Title")


## Process persistence

Kogito provides process persistence through Infinispan, which is an in-memory data grid. Process persistence can be enabled in Kogito by adding the extensions `quarkus-infinispan-client` and `infinispan-persistence-addon` to the Kogito project. The dependencies that are required for adding/enabling these extensions are:

    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-infinispan-client</artifactId>
    </dependency>
    <dependency>
      <groupId>org.kie.kogito</groupId>
      <artifactId>infinispan-persistence-addon</artifactId>
    </dependency>
    
The connection configurations using which the Kogito application needs to connect to a Infinispan data-grid has been specified in the application.properties. 

    quarkus.infinispan-client.server-list=localhost:11222
    quarkus.infinispan-client.auth-server-name=infinispan
    quarkus.infinispan-client.auth-username=infinispan
    quarkus.infinispan-client.auth-password=infinispan
    quarkus.infinispan-client.auth-realm=default
    quarkus.infinispan-client.client-intelligence=BASIC

### Data Containers for processes

The Infinispan management console can be accessed at `https://localhost:11222` with the credentials `infinispan\infinispan`

![Infinispan Console](config/4.png?raw=true "Title")

The infinispan console would display the list of all available data stores. Only three data containers will be listed in the console - jobs, processinstances and usertaskinstances that were created by kogito data index service and jobs service. There will not be any other process specific persistent stores available in the data grid at this moment. 

### Running the applications

In order to see process persistence in action, execute the commands listed below in two different terminals to start the applications `coffee-shop` and `barista` 

**Terminal 1:**
`$cd apps/coffee-shop && mvn clean compile quarkus:dev`

**Terminal 2:** 
`$cd apps/barista && mvn clean compile quarkus:dev`

Now, go back to the Infinispan management console and you'll see that the the application specific backing stores (`drink_order_process_store` and `barista_process_store`) would have got created on the fly in the data-grid. 

## Process instance execution

Once the applications have been started, the process specific REST APIs would get generated automatically and Swagger UI could be used to invoke these APIs and interact with the applications. In order to add the open API & swagger capabilities to a Kogito application, the following dependency has been added to the pom.xml of the application.

    <dependency>
	    <groupId>io.quarkus</groupId>
	    <artifactId>quarkus-smallrye-openapi</artifactId>
	</dependency>

The Swagger UI for the coffee-shop application can be accessed at `http:localhost:8080/swagger-ui`. 

![Swagger UI](config/5.png?raw=true "Title")

From the Swagger UI, the drink order process can be started by invoking the end point `POST
/drink_order_process`. Alternatively, the process instance could be started from the terminal using the command

    curl -i -X POST "http://localhost:8080/drink_order_process" -H "accept: application/json" -H "Content-Type: application/json" -d "{}"

_**Note**: Note down the process instance id that's comes back as a part of the response to this API call. The following sections will refer to this id as {puid}_

After starting the process instances, the status/progress of the process instance could be tracked from the process management console that is available at `http://localhost:8380/`. 

![Kogito Management Console](config/1.png?raw=true "Title")

In the process management console, select the `Drink Order Process` instance and you can see that the process instance would be waiting at the `place-order` step. Since the persistence has been enabled, even if the `coffee-shop` kogito application goes down, the process instance would remain intact in its current state. You may try to terminate the `coffee-shop` application (_by pressing ctrl+c in the terminal in which it was started_) and you would see that process instance would still remaining in its current state in the Kogito management console. (_if you terminate the application, remember to start it back with the command `cd apps/coffee-shop && mvn compile quarkus:dev`_)

Complete the place order by using the step specific REST APIs. First, get the task instance id for the process instance {puid} and by invoking the API

    curl -i -X GET "http://localhost:8080/drink_order_process/{puid}/tasks" -H "accept: application/json"
    
_**Note**: Note down the task instance id that's comes back as a part of the response to this API call. The following sections will refer to this id as {t1uid}_

Then, complete the `place-order` task by invoking the step specific API along with the order details. 

    curl -X POST "http://localhost:8080/drink_order_process/{puid}/place_order/{t1uid}" -H "accept: application/json" -H "Content-Type: application/json" -d "{\"order_out\":{\"cupSize\":\"SMALL\",\"drinkType\":\"MOCHA\"}}"    

Now that the `place-order` step has been completed, re-invoke the API to get the task id  for the task that's next in the line of process execution. 

    curl -i -X GET "http://localhost:8080/drink_order_process/{puid}/tasks" -H "accept: application/json"    

_**Note**: Note down the task instance id that's comes back as a part of the response to this API call. The following sections will refer to this id as {t2uid}_

Complete the `make-payment` by using the {puid} and {t2uid} as shown below.

    curl -X POST "http://localhost:8080/drink_order_process/{puid}/make_payment/{t2uid}" -H "accept: application/json" -H "Content-Type: application/json" -d "{\"order_out\":{\"cardPayment\":{\"cardNumber\":\"6432648726424467\",\"expDate\":\"11-2024\",\"nameOnCard\":\"John Dow\"},\"cupSize\":\"SMALL\",\"drinkType\":\"MOCHA\",\"orderPrice\":15}}"    

Once the payment details have been specified in the make payment step, and since the payment method has been specified as "credit card", the payment processing automated step would get triggered which in turn has been configured to invoke a payment gateway API. Looking at the process payment step and it can seen that the service that's linked to this step is Payments Service and payment processing service operation in that service will be invoked. 

![Payments Service](config/6.png?raw=true "Title")

In the implementation of the PaymentService.java, there's not much of business logic except for the invocation of the external payment gateway API. The external API invocation has been made simple with the help of a Rest Client extension (_`quarkus-rest-client`_) and the extension has been added to the pom.xml of the application. 


        @ApplicationScoped
    public class PaymentsService {
    
        @Inject
        @RestClient
        PaymentsGateway paymentsGateway;
    
        public boolean process(DrinkOrder order) {
            // invoke an external payments gateway service
            Response response = paymentsGateway.makePayment(order.getCardPayment());
            System.out.println("*******************************************************");
            System.out.println("Payment processed successfully by the payment gateway");
            System.out.println("*******************************************************");
            return response.getStatus() == Response.Status.OK.getStatusCode();
        }
    }    

With that in place, an interface for the rest client (PaymentsGateway.java) has been  registered with the relevant HTTP path and method details as shown below. 

    @Path("/")
    @Produces(MediaType.APPLICATION_JSON)
    @RegisterRestClient
    public interface PaymentsGateway {
    
        @POST
        @Path("/post")
        Response makePayment(CardPayment payment);
    }

The actual endpoint url of the payment gateway has been externalized and added to the application configuration properties file. With that, the payment gateway API will be invoked and the barista would be notified of the order. 

## Kogito Reactive Messaging

From the Kogito management console, it can seen that the barista process have got started. This was made possible by the native reactive messaging support provided by Kogito. In the drink-order-process BPMN model, you can see that the step `Inform barista` has been modeled as an End Message Event and it has been configured to send order details in the form of a message (_with message name as barista-process_).

Then, in the application config file, the destination topic to which this message has to be published has been specified along with the connection details to an event stream processing engine like Kafka, and the Kogito event publisher would take care of publishing the event to the specified destination. 

    mp.messaging.outgoing.barista-process.bootstrap.servers=localhost:9092
    mp.messaging.outgoing.barista-process.connector=smallrye-kafka
    mp.messaging.outgoing.barista-process.topic=barista-process
    mp.messaging.outgoing.barista-process.value.serializer=org.apache.kafka.common.serialization.StringSerializer

All of this has been done without any coding but just with few configurations and by enabling the kogito reactive messaging add-ons in the pom.xml 

        <dependency>
          <groupId>io.quarkus</groupId>
          <artifactId>quarkus-smallrye-reactive-messaging</artifactId>
        </dependency>
        <dependency>
          <groupId>io.quarkus</groupId>
          <artifactId>quarkus-smallrye-reactive-messaging-kafka</artifactId>
        </dependency>
        <dependency>
          <groupId>org.kie.kogito</groupId>
          <artifactId>kogito-events-reactive-messaging-addon</artifactId>
        </dependency>

On the receiving side, the barista process has been configured to listen to the same topic (_Start Message Event with the message name as barista-process_) in Kafka and the Kogito event receiver would then receive the event and start the barista process. And this is how we have got the new instance of the barista process created automatically when the drink order process published an event.

Besides that, the Kogito applications can be configured to publish process and user task events at various stages of the process execution, so that any other external systems that need to react to the process instance execution can do so by listening to those events that are emitted at various stages of process execution. 

For instance, the inbuilt kogito-processinstances-events and kogito-usertaskinstances-events publishing has been enabled in this application using these configurations. 

    mp.messaging.outgoing.kogito-usertaskinstances-events.bootstrap.servers=localhost:9092
    mp.messaging.outgoing.kogito-usertaskinstances-events.connector=smallrye-kafka
    mp.messaging.outgoing.kogito-usertaskinstances-events.topic=kogito-usertaskinstances-events
    mp.messaging.outgoing.kogito-usertaskinstances-events.value.serializer=org.apache.kafka.common.serialization.StringSerializer
    
    mp.messaging.outgoing.kogito-processinstances-events.bootstrap.servers=localhost:9092
    mp.messaging.outgoing.kogito-processinstances-events.connector=smallrye-kafka
    mp.messaging.outgoing.kogito-processinstances-events.topic=kogito-processinstances-events
    mp.messaging.outgoing.kogito-processinstances-events.value.serializer=org.apache.kafka.common.serialization.StringSerializer
    
The set of events that has been published to these topics (_kogito-processinstances-events and kogito-usertaskinstances-events_) could be viewed with the help of any Kafka client it would have the event details published as per the cloudevents specification. A sample message that's been published to the topic kogito-processinstances-events has been shown below. 

    {
      "specversion": "0.3",
      "id": "e7a9669b-5b31-428a-81a2-1055ff07312d",
      "source": "/drink_order_process",
      "type": "ProcessInstanceEvent",
      "time": "2020-03-30T14:20:53.624+08:00",
      "data": {
        "id": "1a9022dd-06a3-4263-b360-d0671b44a82e",
        "parentInstanceId": "",
        "rootInstanceId": "",
        "processId": "drink_order_process",
        "rootProcessId": "",
        "processName": "Drink Order Process",
        "startDate": "2020-03-30T14:11:41.519+08:00",
        "endDate": "2020-03-30T14:20:53.615+08:00",
        "state": 2,
        "nodeInstances": [
          {
            "id": "c56e9d2c-a4be-48d9-afd0-b74d2330014c",
            "nodeId": "2",
            "nodeDefinitionId": "_F5372E8C-7723-493E-A523-B4B9FB4AA2AF",
            "nodeName": "Inform Barista",
            "nodeType": "EndNode",
            "triggerTime": "2020-03-30T14:20:53.605+08:00",
            "leaveTime": "2020-03-30T14:20:53.613+08:00"
          },
          {
            "id": "6506fd23-fb45-4e03-bd4b-f4b503f10d33",
            "nodeId": "1",
            "nodeDefinitionId": "_B9944B1B-FB77-4D0D-8EBF-97DE201FC267",
            "nodeName": "Join",
            "nodeType": "Join",
            "triggerTime": "2020-03-30T14:20:53.604+08:00",
            "leaveTime": "2020-03-30T14:20:53.605+08:00"
          },
          {
            "id": "a0dd09e0-fb64-4be7-9c57-82ea4d498349",
            "nodeId": "4",
            "nodeDefinitionId": "_8644C4EC-BCCA-4718-B8A1-6D6C66413074",
            "nodeName": "Split",
            "nodeType": "Split",
            "triggerTime": "2020-03-30T14:20:53.604+08:00",
            "leaveTime": "2020-03-30T14:20:53.604+08:00"
          },
          {
            "id": "5a325f87-bc85-423f-8b18-ed0d600fc4c8",
            "nodeId": "5",
            "nodeDefinitionId": "_A710B96A-6EE1-43BB-A2EE-ACEE59F7479E",
            "nodeName": "Process payment",
            "nodeType": "WorkItemNode",
            "triggerTime": "2020-03-30T14:20:51.402+08:00",
            "leaveTime": "2020-03-30T14:20:53.604+08:00"
          },
          {
            "id": "b1edd96d-ca01-4f13-9aa1-96cfc2cc1780",
            "nodeId": "6",
            "nodeDefinitionId": "_DE7D069C-0E57-4CDF-872A-4D1266D7A6C9",
            "nodeName": "Split",
            "nodeType": "Split",
            "triggerTime": "2020-03-30T14:20:51.401+08:00",
            "leaveTime": "2020-03-30T14:20:51.402+08:00"
          },
          {
            "id": "169f848c-7960-4c64-abfd-d926aa300b0a",
            "nodeId": "7",
            "nodeDefinitionId": "_EF90EB52-7E8D-434C-BA36-AE5A80668DC1",
            "nodeName": "Make Payment",
            "nodeType": "HumanTaskNode",
            "triggerTime": "2020-03-30T14:14:20.785+08:00",
            "leaveTime": "2020-03-30T14:20:51.401+08:00"
          }
        ],
        "variables": {
          "process_state": true,
          "order": {
            "drinkType": "MOCHA",
            "cupSize": "SMALL",
            "cardPayment": {
              "nameOnCard": "John Doe",
              "cardNumber": "6432648726424467",
              "expDate": "11-2024"
            },
            "orderPrice": 15
          }
        },
        "error": null,
        "roles": null
      },
      "kogitoProcessinstanceId": "1a9022dd-06a3-4263-b360-d0671b44a82e",
      "kogitoParentProcessinstanceId": "",
      "kogitoRootProcessinstanceId": "",
      "kogitoProcessId": "drink_order_process",
      "kogitoRootProcessId": "",
      "kogitoProcessinstanceState": "2",
      "kogitoReferenceId": null,
      "kogitoAddons": "infinispan-persistence"
    }

## Securing the Kogito REST API endpoints

The interactions with the REST APIs of the Kogito application and the process instances has been done in an unsecured fashion so far. These APIs could be quickly secured with the help of an Open ID Connect (OIDC) server, for instance using Keycloak. Shown below are the OIDC server connection details that are to be added to the application.properties to secure the root context path of the Kogito application.

    quarkus.oidc.auth-server-url=http://localhost:8280/auth/realms/kogito
     quarkus.oidc.client-id=kogito-app
     quarkus.oidc.credentials.secret=secret
    
     quarkus.http.auth.permission.authenticated.paths=/*
     quarkus.http.auth.permission.authenticated.policy=authenticated

Once these configurations are added to the cofiguration file, a hot reload of the Kogito application could be triggered by accessing `http://localhost:8080/swagger-ui` in the browser and you'll find that the swagger-ui would be inaccessible. 

The client in the OIDC server has been configured with the client credentials flow, and so the bearer token can be first obtained and the API can be invoked. 

The access token could be obtained by using the client credentials and stored it in an env variable using the following command.

    export access_token=$(\
        curl -X POST http://localhost:8280/auth/realms/kogito/protocol/openid-connect/token \
        --user kogito-app:secret \
        -H 'content-type: application/x-www-form-urlencoded' \
        -d 'username=john&password=john&grant_type=password' | jq --raw-output '.access_token' \
     )
 
With that the drink-order-process could be started from a terminal using the command 

    curl -i -H "Authorization: Bearer "$access_token -X POST "http://localhost:8080/drink_order_process" -H "accept: application/json" -H "Content-Type: application/json" -d "{}"

You'll notice that a process instance will get created.

Trying to invoke the API without an access token will result in an `401 Unauthorized` error.

    curl -i -H "Authorization: Bearer "$access_token -X POST "http://localhost:8080/drink_order_process" -H "accept: application/json" -H "Content-Type: application/json" -d "{}"
 



