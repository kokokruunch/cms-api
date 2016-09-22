# Customer Management System API #

This application implements an API that allows the client to manage customer information for a Customer Management System. [RAML](http://raml.org/) was used to design the API and was then subsequently mapped to an implementation of an API in Mule. The RAML can be found on (src/main/api/). The backend implementation has been done using Java.

### Use Cases ###

The API has been designed to support the following consumer use cases:

1. A client can periodically (every 5 minutes) consume the API to enable it (the client) to maintain a copy of the provider API's customers (the API represents the system of record)
2. A mobile application used by customer service representatives that uses the API to retrieve and update the customers details
3. Simple extension of the API to support future resources such as orders and products 

### Assumptions ###

1. Filtering and searching are not in scope.
2. Customer addresses are denormalised, and have therefore been stored as a single *String* object eg. *'4 Wallaby Way, Sydney, NSW 2000 Australia'*
3. Updates to the customer information are made in 'full', as opposed to 'partial' updates. This means that the entire customer data is updated.

### Design Considerations and Decisions ###

1. Caching has been implemented on the retrieval of the customers (/GET customers) to reduce the processing load and improve performance. This is particularly notable due to high frequency of calls made by the client (Use Case 1). Tuning up by updating the [Thread Profiles](https://docs.mulesoft.com/mule-user-guide/v/3.6/tuning-performance) was also considered but not implemented due to not having the necessary volumetrics to identify the right values of the thread pool parameters.
2. Security has been implemented through the use of access tokens. This has been added on the 'write' methods to disallow unauthorised users to modify the customer data. Other security implementation such as specifying OAuth [security schemes](https://github.com/raml-org/raml-spec/blob/master/versions/raml-08/raml-08.md#security) on the RAML has been considered but not added due to the need to set-up additional artifacts.
3. The message format chosen is JSON. This provides the benefits of being easier to work with for a mobile application due to it being lightweight and portable. The need to optimise bandwidth usage is important to ensure that API calls from mobile applications over the network are efficient (Use Case 2). The methods defined at the customerId resource (PUT, GET, DELETE), have been made at that level to reduce the bandwidth usage . A compression strategy has been considered, but not implemented due to the current nature of the customer data.
4. The RAML has been factorised to allow for future resources such as orders and products (Use Case 3) to be added. This has been done by defining *resourceTypes* to handle the retrieval, creation, updates and deletion of the customer, which will most likely be the same use case for orders and products.
5. The Java implementation was also modularised by having a separate Customer class to handle Customer information. Following this pattern, future resources will be handled in their own respective classes (Use Case 3). CustomerMgmtSystem will be the main gateway to these objects. Subsequently, the Customer flow implementation was separated from the Main flow.
6. Exception strategy utilises the APIKit's generated exception handlers. Additional exception messages have been added and can be seen on the stack trace, some examples can be seen below. The stack trace was not added on the error response due to its verbosity.

### Exceptional Scenarios ###

1. Customer ID does not exist in the system:
		An error is thrown to indicate that the customer ID does not exist. This is applicable to the update, deletion and retrieval of a customer.

		Response:
		"message": "Resource not found"

		 Root Exception stack trace:
			org.mule.module.apikit.exception.NotFoundException: Customer 10 does not exist.

2. Incomplete customer data:
		In the event that the customer does not have a particular data eg. last name, an empty string will be added instead of marking it as 'null'.

3. No customer data:
		An error is thrown to indicate that there are no customers

		Response:
		"message": "Resource not found"

		Root Exception stack trace:
			org.mule.module.apikit.exception.NotFoundException: No customers are found.

Studio automatically generates several global exception strategy mappings that the Main flow references to send error responses in HTTP-status-code-friendly format. Defined at a global level within the project's XML config, this standard set of exception strategy mappings ensure that anytime a backend flow throws an exception, the API responds to the caller with an HTTP-status code and corresponding plain-language message.

### Set Up and Run the Example ###

Follow the procedure below to run and test the functionality of this example application in Anypoint Studio.

1. Open the **cms-api** project in Anypoint Studio by Importing it as an *Anypoint Studio Project*.
2. In your application in Studio, click the **Global Elements** tab. Double-click the HTTP Listener global element to open its **Global Element Properties** panel. Change the contents of the **port** field to the required HTTP port e.g. 8081
3. In the Package Explorer pane in Studio, right-click the project name, then select Run As > Mule Application.
4. Anypoint Studio starts the application and automatically opens an API console below the canvas.
5. Click **GET** for the **/customers** resource.
The console opens a details panel for the GET method for this resource that allows you to view details about the request format and expected responses, as well as a **Try It** section in the right.
When using the **POST**, **DELETE** and **PUT** methods, an *accessToken* needs to be added as a query parameter. eg. http://localhost:8081/api/customers?accessToken=AABBCCDD
6. In the **Try It** section, then click **GET**. *Note: due to the caching strategy applied, an updated customer list will appear after 10 seconds.*
7. Scroll down to view the response. The API should return a list of customers.
8. Click through the other resources and methods to test out other API calls.

#### Main Flow

The main flow is standard for an APIkit project. It contains an inbound endpoint and an APIkit Router. The exception strategies are explained separately, below.

#### Backend Flows ####

The backend flows in this application are stubs, with placeholder backend implementation defined with Spring beans.
