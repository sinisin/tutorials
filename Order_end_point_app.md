# Order End Point App


Order End Point App is the entry point application which will mimic the current function of GPS to generate orders. It will constitute multiple azure functions. The input of the app will be Order XML/JSON files uploaded through POST operation. It will be responsible to send these Orders through POST operation to OTM and and share the Order data with the Consignment App. 


## Order Attributes

The Order XML received must have the following details as these are the key attributes to create consignments later in the work flow. 
* Source
* Destination
* Payment Terms
* Currency
* Pick up date
* Delivery date
* Order Configuration 
	* Item, Weight, Volume, THU
* Date Emphasis
	
## Backend Processing

The processing can be split into following steps : 

* Save the uploaded XML/JSON into Azure storage area
* Invoke OTM POST operation to send Order object
* Post event in Service bus with Order ID and Order storage area file path
* Listen to the event and fetch the Order data from specified file path in the event
* Save the Order XML/JSON into Cosmos DB

### Azure Http Trigger Function

Once the input XML/JSON is received through POST operation, the XML/JSON object will be saved into storage area. The function will return HTTP status code 200 if the order is successfully created. In case of failure, an error response will be returned with error code 
* 4xx If the issue is with the input data for example : "Order ID duplication exception" or "Mandatory field not present"
* 5xx For internal server or technical errors, 5xx error code will be returned. 

### Azure Logic App

Once the order is saved, it will be picked up by the Azure Logic App Function. This Logic app will be backed by a scheduler which will run at scheduled repeat interval and pick up all newly saved orders. These orders will be sent to OTM cloud by invoking OTM's POST operation. Once the successful response is received, the function will post an event in the service bus with the Order's storage area filepath and the orderID of the order as attributes.

### Azure Persistence Function

This function will be trigerred once the event is posted by the service bus in the above function. The app will read the XML/JSON file from file path received in the event and persist into Cosmos DB against unique Order ID. 

### Database

The order data will be persisted into NOSQL Azure Cosmos DB against the unique Order ID. 

### Logging and Monitoring

Function App built in feature Application Insights can be used as it collects log, performance, and error data. It automatically detects performance anomalies. It can be configured be enabling this feature in the function app. 

## Logical Boundary
The application processing completes when the Order object is persisted into Cosmos DB. 



