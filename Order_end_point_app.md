# Order End Point App


Order End Point App is the entry point application which will mimic the current function of GPS to generate orders. It will constitute multiple azure functions. The input of the app will be Order XML/JSON files uploaded through POST operation. It will be responsbile to send the Orders through POST operation to OTM and and share the Order data with the Consignment App. 


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

### Azure Http Trigger Function

Once the input XML is received through POST operation, the file will be persisted into storage area with a unique order ID. The function will return HTTP status code 200 if the order is successfully created. In case of failure, an error response will be returned with error code 
* 4xx If the issue is with the input data for example : "Order ID duplication exception" or "Mandatory field not present"
* 5xx For internal server or technical errors, 5xx error code will be returned. 

### Azure Logic App

Once the order is saved, it will be picked up by the Azure Logic App. This Logic app will be backed by a scheduler which will run at scheduled repeat interval and pick up all newly saved orders. These orders will be sent to OTM cloud using POST operation. 

-- TODO
Cover flow from Order end point app to Consigmnment App
(Question - will Logic app take care of publishing to topic for the Consignment App to pick up) 

### Database

The order data will be persisted into NOSQL Azure Cosmos DB against the unique Order ID. 
(Question - Should we maintain a new field "Status" as a metadata for Order to track if the Order is succesfully published. In case of failure, the Logic app can pick failed Orders to be created in OTM again) 


### Other Components 
-- TODO
Capture the Service bus component here.

## Logical Boundary
The application processing completes when the Order object is sent through POST operation in OTM and HTTP success code is received.  
-- TODO Handle failure scenario, Capture the flow of Order from Order End Point App to Consignment App



