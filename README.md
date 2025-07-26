# CSP451-Checkpoint9

## Set up Azure VM for Docker Workload
1.	Create an Ubuntu VM and open necessary ports on Azure portal
VM Public IP: 20.63.24.20

2.	Install Docker and Docker Compose on the VM

3.	Clone s sample multi-service Docker app including frontend, backend, and mongodb
4.	Run command docker-compose up -d to start the applications
   
## Expose Services Securely
1.	Configure NGINX reverse proxy or expose services individually using host ports
•	Modify compose.yaml file to add new NGINX service
2.	Restart compose.yaml for the update
3.	Validate access to the app from a browser
•	Access http://20.63.24.20 to test frontend access

## Build a Serverless Integration Layer
1.	Create Azure Function App with an HTTP trigger
•	Run command to create and configure storage account on Azure
2.	Run command to create and configure Azure Function App
3.	Create a HTTP trigger in Azure Function App
4.	Include a script to define trigger
•	Define the script with the following:
•	If the stock quantity is lower than threshold 10, it will trigger “low stock alert” to Slack for product restock and display live product data
•	Log to blob account and display message “Stock update processed” for any update

*Script content:
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
};**

5.	Slack configuration
•	Create a new Slack account and configure a new Workplace called CSP451
•	Add integration Stock to this channel
•	Stock will send alert to #all-csp451 for incoming webhooks
6.	Add variables for Azure Function App under Environment Variables tab
•	It requires Name and Value to add variables
•	Add variables for Webhooks and Blob for the script
•	Acquire Webhooks URL by enabling Incoming Webhooks
•	Acquire Blob connection string under Storage account > Security + Networking > Access keys
7.	On UbuntuVM, add the Azure Function key in compose.yaml to link the function with Docker
## Create a Balanced Architecture
Present final application flow
1.	Run docker ps on Ubuntu VM
2.	Confirm the frontend communicates with the backend API
•	Append the following script to server.js file in backend directory to visualize the frontend-to-backend communication
•	Access the web page http://20.63.24.20/api/ping
•	Run docker-compose logs -f backend to capture the backend logs in real-time
•	The log shows “Ping received from frontend!” to confirm frontend communicates with backend API
3.	Conform Azure Function reacts to events like "low stock"
•	Execute Azure Function on trigger
•	Upon triggering, it shows custom message “Stock update processed” to update the inventory
4.	Confirm Azure Function execution will trigger an alert for API result 
•	Execute the Function to run command that stock inventory is below 10. It send “low stock alert” to Slack Stock page and display real-time product inventory information
