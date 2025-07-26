# CSP451-Checkpoint9

## Objective
- Set up an Azure Ubuntu VM
- Deploy multi-container Docker application by using `docker-compose`
- Integrate Dockerized app with Azure Serverless components - Azure Functions
- Create and configure a hybrid cloud-native architecture with both IaaS and PaaS


## Task1: Set up Azure VM for Docker Workload
### 1.	Create an Ubuntu VM in Azure portal
- VM Public IP: 20.63.24.20
  
### 2.	Open necessary ports for the associated Security Group
- open ports for HTTP(80), HTTPS(443), port 3000, port 5000

### 3.	Install Docker and Docker Compose on the VM
- run `sudo apt update`
- run `sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin`

### 4.	Clone s sample multi-service Docker app including frontend, backend, and mongodb
- run `git clone https://github.com/docker/awesome-compose.git`
- cd into *awesome-compose/react-express-mongodb** and run `ls` to confirm frontend, backend, and mongodb directory
  
### 5. Start the applications
- run `docker-compose up -d` to start up application 
   
## Task2: Expose Services Securely
### 1.	Configure NGINX reverse proxy
- remain in the directory *react-express-mongodb** and run ls to find *compose.yaml** file
- run `nano compose.yaml` and add NGINX service into the file
- add contents: *nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx:/etc/nginx
    depends_on:
      - frontend
      - backend
    networks:
      - react-express**

### 2.	Restart compose.yaml for the update
- run `docker-compose down && docker-compose up -d`

### 3.	Validate access to the app from a browser
- navigate to http://20.63.24.20 to display the webpage and test frontend access
- navigate to http://20.63.24.20/api/ to display webpage and test backend access

## Task3: Build a Serverless Integration Layer
### 1.	Create an Azure Function App with an HTTP trigger
- create Storage Account by running `az storage account create --name yang334storageaccount --resource-group Student-RG-1727501 --location canadacentral --sku Standard_LRS`

### 2.	Run command to create and configure Azure Function App
- create Azure Function app by running `az functionapp create --resource-group Student-RG-1727501 --consumption-plan-location canadacentral --runtime node --functions-version 4 --name stockAlertFuncApp --storage-account yang334storageaccount`
  
### 3.	Create a HTTP trigger in Azure Function App
- navigate to Function App > Overview page > create HTTP function
  
### 4.	Include a script to define trigger
- define the script in trigger
- if the stock quantity is lower than threshold 10, it will trigger *low stock alert** to Slack for product restock and display live product data
- Log to blob account and display message *Stock update processed** for any update
- *script content:
module.exports = async function (context, req) {
  const stock = req.body;
  if (!stock || !stock.productId || stock.quantity === undefined) {
    context.res = { status: 400, body: "Invalid stock payload" };
    return;
  }
  const threshold = 10;
  if (stock.quantity < threshold) {
    // Send alert (Slack example)
    await fetch(process.env.SLACK_WEBHOOK_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        text: `⚠️ Low stock alert for ${stock.productId}: ${stock.quantity} units left`
      })
    });
  }
  // Log to Blob (simplified example)
  context.log(`Audit log: ${JSON.stringify(stock)}`);
  context.res = {
    status: 200,
    body: { message: "Stock update processed" }
  };
};*

### 5.	Slack configuration
- create a new Slack account and configure a new Workplace called *CSP451*
- Add integration *Stock* to this channel
- configure the settings for *Stock* page so it will send alert to #all-csp451 for incoming webhooks when the Function is triggered
  
### 6.	Add variables for Azure Function App under Environment Variables tab
- variable syntax: Name and Value
- add variables for Webhooks and Blob for the script
- navigate to Slack page> under configuration setting for Stock > click Webhooks > click Enable Incoming Webhooks > copy Webhooks URL
-	navigate to storage account > Security + Networking > Access keys > copy blob connection string
-	copy and paste the URL and string for values to create variables
  
### 7.	On UbuntuVM, add the Azure Function key in compose.yaml to link the function with Docker
- *environment:
      - AZURE_FUNCTION_URL=https://stockalertfuncapp.azurewebsites.net/api/StockTrigger?code=xxxxxxx*

## Task4: Create a Balanced Architecture
### 1.	Run `docker ps` on Ubuntu VM
### 2.	Confirm the frontend communicates with the backend API
- append the following script to *server.js* file in backend directory to visualize the frontend-to-backend communication
- *module.exports = app;
  app.get("/api/ping", (req, res) => {
  console.log("Ping received from frontend!");
  res.json({ status: "Backend connection confirmed!" });
  });*
  
- access the web page http://20.63.24.20/api/ping
-	then run `docker-compose logs -f backend` to capture the backend logs in real-time
-	the log shows “Ping received from frontend!” to confirm frontend communicates with backend API
  
### 3.	Conform Azure Function reacts to events like "low stock"
- Execute Azure Function on trigger
- Upon triggering, it shows custom message “Stock update processed” to update the inventory
  
### 4.	Confirm Azure Function execution will trigger an alert for API result 
- Execute the Function to run command that stock inventory is below 10
- the function will be triggered and send “low stock alert” to Slack Stock page
- display real-time product inventory information "Low stock alert for widget-123: 2 units left"
