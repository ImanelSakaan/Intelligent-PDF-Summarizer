<!--
---
description: This end-to-end sample shows how implement an intelligent PDF summarizer using Durable Functions. 
page_type: sample
products:
- azure-functions
- azure
urlFragment: durable-func-pdf-summarizer
languages:
- python
- bicep
- azdeveloper
---
-->

# 🧠 Intelligent PDF Summarizer using Azure Durable Functions

This project is a serverless PDF summarization pipeline built with **Azure Durable Functions** and **Azure Cognitive Services**. It takes a PDF document as input, extracts text from each page, summarizes the content using AI, and returns a final summary to the user.
![Architecture Diagram](./media/architecture_v2.png)

> 📌 Built as part of Lab 2 for the Full-Stack Cloud Developer program.

---

## 1. 📸 Demo Video

🎥 Watch the 10-minute demo here:  
**[▶️ YouTube Video Link](https://youtu.be/e5V9-HNniXs)**

---
## 2. 🚀 Features

- Upload a PDF via HTTP
- Extracts and processes each page using Durable Functions (fan-out/fan-in pattern)
- Uses Azure AI (Text Analytics or OpenAI) to summarize page content
- Aggregates all summaries into one final result
- Monitors orchestration via Durable Functions status endpoint
<img width="556" alt="Screenshot 2025-06-17 in-out" src="https://github.com/user-attachments/assets/ebf399a4-4d8e-412f-8dfc-bb413b7f9308" />

---

## 3. 🧰 Tech Stack and Azure Resource Setup

- Python 3.10+
- Azure Durable Functions
- Azure Blob Storage
- Azure Cognitive Services (OpenAI or Text Analytics)
- PyMuPDF / Azure Form Recognizer (for PDF text extraction)

### Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/intelligent-pdf-summarizer.git
cd intelligent-pdf-summarizer
```

### Set Up Python Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate    # For macOS/Linux
.venv\Scripts\activate       # For Windows
pip install -r requirements.txt

```

### ✅ 3.1. Create a Resource Group (optional but recommended)

Create a resource group named `pdf-summary-rg` in the `canadacentral` region:

```bash
az group create --name pdf-summary-rg --location canadacentral
```
### ✅ 3.2. Create a Storage Account

Create a storage account named `pdfsummarystorage` in the `canadacentral` region:

```bash
aaz storage account create --name pdfsummarystorage --location canadacentral --resource-group pdf-summary-rg --sku Standard_LRS

```
### ✅ Make Note of the Storage Account Name and Connection String

Run the following command to retrieve the connection string:

```bash
az storage account show-connection-string --name pdfsummarystorage --resource-group pdf-summary-rg

```
### ✅ 3.3. Create a Blob Container for PDFs

Create a private blob container named `pdf-uploads` in your storage account:

```bash
aaz storage container create --name pdf-uploads --account-name pdfsummarystorage --public-access off
```

### ✅ 3.4. Create an Azure Function App (Python & Durable Enabled)

Create a Function App named `pdfsummarizerfunc` using the Python runtime and Durable Functions support:

```bash
az functionapp create --resource-group pdf-summary-rg --consumption-plan-location canadacentral --runtime python --functions-version 4 --name pdfsummarizerfunc --storage-account pdfsummarystorage --os-type Linux
```
### ✅ 3.5. Create and Configure Azure Form Recognizer

1. Go to the [Azure Portal](https://portal.azure.com)
2. Click **"Create a resource"** → Search for **Form Recognizer**
3. Select **Form Recognizer** and create it in the `pdf-summary-rg` resource group
4. Choose a name like `pdfsummarizerform`
5. Choose a pricing tier  
   > 💡 **Free F0** is available and works well for testing

---

Once created, copy the following values for use in your application:

### 📌 Form Recognizer Configuration Values

- **FORM_RECOGNIZER_ENDPOINT**  
   https://pdfsummarizerform.cognitiveservices.azure.com/

- **FORM_RECOGNIZER_KEY**  
   FBZ8uMGMZGeJl7vLW4e56OBEQdwnJJfO3kOcybTd6m8mJHyj06ouJQQJ99BGACBsN54XJ3w3AAALACOGVV7C
---
### ✅ 3.6. Set Up Azure OpenAI (or Cognitive Services for Summarization)

1. Go to the [Azure Portal](https://portal.azure.com)
2. Search for **Azure OpenAI** and create a resource  
   
3. Once created, go to the resource and **deploy a model**  
   - Recommended: `gpt-35-turbo` or `gpt-4`
---

Once the model is deployed, copy the following values:

- **OPENAI_API_KEY**  
   1z14G32NmMC2nBCUdi6O0wuaAlczkPfdDfyx0IPldBgo9WmC1d8mJQQJ99BGACYeBjFXJ3w3AAAAACOGw45w
- **OPENAI_ENDPOINT**  
   [https://pdfcogopenai.openai.azure.com/](https://elsa0-mcxnbjbl-eastus.cognitiveservices.azure.com/openai/deployments/gpt-35-turbo-instruct/chat/completions?api-version=2025-01-01-preview)

- **OPENAI_DEPLOYMENT_NAME**  
   gpt-35-turbo-instruct
  
<img width="1109" height="641" alt="image" src="https://github.com/user-attachments/assets/a67c10ff-3d7e-40b5-b9fb-9d1e6559299b" />

### ✅ 3.7. Set Local Environment Variables

Create a file named `local.settings.json` in the root of your Azure Function project with the following content:

```json
{
  {
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=pdfsummarystorage;AccountKey=+9V4tJMZPDcNqHWIOYIN2ehXYdK8BEr9FQGajTrtqYXpJ15a6BJI/zwtI+9BixvMd7V/DNL2aFNF+AStU7Mzrw==;EndpointSuffix=core.windows.net",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "FORM_RECOGNIZER_ENDPOINT": "https://pdfsummarizerform.cognitiveservices.azure.com/",
    "FORM_RECOGNIZER_KEY": "FBZ8uMGMZGeJl7vLW4e56OBEQdwnJJfO3kOcybTd6m8mJHyj06ouJQQJ99BGACBsN54XJ3w3AAALACOGVV7C",
    "OPENAI_API_KEY": "1z14G32NmMC2nBCUdi6O0wuaAlczkPfdDfyx0IPldBgo9WmC1d8mJQQJ99BGACYeBjFXJ3w3AAAAACOGw45w",
    "OPENAI_DEPLOYMENT_NAME": "gpt-35-turbo-instruct",
    "OPENAI_ENDPOINT": "https://elsa0-mcxnbjbl-eastus.cognitiveservices.azure.com/openai/deployments/gpt-35-turbo-instruct/chat/completions?api-version=2025-01-01-preview"
  }
}
  }
}
```

## 4. 🛠️ How to Deploy and Run the App
### 4.1. Drop a PDF file into the input container (use Storage Explorer or code).

<img width="1255" height="336" alt="image" src="https://github.com/user-attachments/assets/c9b8671a-a1ad-485b-a183-5eaa9ed86cb4" />

### 4.2. Run the Application Locally

Start the Azure Functions runtime:

```bash
func start
````
<img width="934" height="350" alt="image" src="https://github.com/user-attachments/assets/b1abbea4-1a39-4147-8487-1098d11ad12c" />

<img width="929" height="38" alt="image" src="https://github.com/user-attachments/assets/8b613bb2-06b2-4b7f-909b-16c0524d31d0" />

This will launch your Azure Function app locally at:

```
http://localhost:7071
```
<img width="817" height="609" alt="image" src="https://github.com/user-attachments/assets/3a45413e-5f66-4254-a0d7-be8f13cf647a" />


The endpoint to trigger the PDF summarization process:

```bash
POST http://localhost:7071/api/orchestrators/OrchestratorFunction
```

You can use tools like **Postman** or **curl** to test the PDF upload:

```bash
curl -X POST http://localhost:7071/api/orchestrators/OrchestratorFunction \
  -F "file=@your-pdf-file.pdf"
```
<img width="925" height="368" alt="image" src="https://github.com/user-attachments/assets/11bf6d6b-9d34-416b-8b98-af0e57d4fcf7" />

A response will be returned with several status URLs to monitor the Durable Function instance:

* `statusQueryGetUri`: Get the current progress/status
* `terminatePostUri`: Manually terminate the orchestration
* `sendEventPostUri`: Send custom events to the orchestration (if used)




Thanks a lot
Iman Elsakaan


![image](https://github.com/user-attachments/assets/a46eafc6-b414-4382-bf19-c38f40fd46ec)


``












The application's workflow is as follows:
1.	PDFs are uploaded to a blob storage input container.
2.	A durable function is triggered upon blob upload.
- - Downloads the blob (PDF).
- - Utilizes the Azure Cognitive Service Form Recognizer endpoint to extract the text from the PDF.
- - Sends the extracted text to Azure Open AI to analyze and determine the content of the PDF.
- - Save the summary results from Azure Open AI to a new file and upload it to the output blob container.

Below, you will find the instructions to set up and run this app locally..

## Prerequsites
- [Create an active Azure subscription](https://learn.microsoft.com/en-us/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing).
- [Install the latest Azure Functions Core Tools to use the CLI](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local)
- Python 3.9 or greater
- Access permissions to [create Azure OpenAI resources and to deploy models](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/role-based-access-control).
- [Start and configure an Azurite storage emulator for local storage](https://learn.microsoft.com/azure/storage/common/storage-use-azurite).

## local.settings.json
You will need to configure a `local.settings.json` file at the root of the repo that looks similar to the below. Make sure to replace the placeholders with your specific values.

```json
{
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "AzureWebJobsFeatureFlags": "EnableWorkerIndexing",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "BLOB_STORAGE_ENDPOINT": "<BLOB-STORAGE-ENDPOINT>",
    "COGNITIVE_SERVICES_ENDPOINT": "<COGNITIVE-SERVICE-ENDPOINT>",
    "AZURE_OPENAI_ENDPOINT": "AZURE-OPEN-AI-ENDPOINT>",
    "AZURE_OPENAI_KEY": "<AZURE-OPEN-AI-KEY>",
    "CHAT_MODEL_DEPLOYMENT_NAME": "<AZURE-OPEN-AI-MODEL>"
  }
}
```

## Running the app locally
1. Start Azurite: Begin by starting Azurite, the local Azure Storage emulator.

2. Install the Requirements: Open your terminal and run the following command to install the necessary packages:

```bash
python3 -m pip install -r requirements.txt
```
3. Create two containers in your storage account. One called `input` and the other called `output`. 

4. Start the Function App: Start the function app to run the application locally.

```bash
func start --verbose
```

5. Upload PDFs to the `input` container. That will execute the blob storage trigger in your Durable Function.

6. After several seconds, your appliation should have finished the orchestrations. Switch to the `output` container and notice that the PDFs have been summarized as new files. 

>Note: The summaries may be truncated based on token limit from Azure Open AI. This is intentional as a way to reduce costs. 

## Inspect the code
This app leverages Durable Functions to orchestrate the application workflow. By using Durable Functions, there's no need for additional infrastructure like queues and state stores to manage task coordination and durability, which significantly reduces the complexity for developers. 

Take a look at the code snippet below, the `process_document` defines the entire workflow, which consists of a series of steps (activities) that need to be scheduled in sequence. Coordination is key, as the output of one activity is passed as an input to the next. Additionally, Durable Functions handle durability and retries, which ensure that if a failure occurs, such as a transient error or an issue with a dependent service, the workflow can recover gracefully.

![Orchestration Code](./media/code.png)

## Deploy the app to Azure

Use the [Azure Developer CLI (`azd`)](https://aka.ms/azd) to easily deploy the app. 

1. In the root of the project, run the following command to provision and deploy the app:

    ```bash
    azd up
    ```

1. When prompted, provide:
   - A name for your [Azure Developer CLI environment](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/faq#what-is-an-environment-name).
   - The Azure subscription you'd like to use.
   - The Azure location to use.

Once the azd up command finishes, the app will have successfully provisioned and deployed. 

# Using the app
To use the app, simply upload a PDF to the Blob Storage `input` container. Once the PDF is transferred, it will be processed using document intelligence and Azure OpenAI. The resulting summary will be saved to a new file and uploaded to the `output` container.
