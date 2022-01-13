# Parse documents with Azure Form Recognizer and Logic Apps and display the results in a PowerApp

This application enables an end user to upload a PDF and see the parsed results in the UI. The UI is built with PowerApps and the background processing is achieved with Azure Logic Apps and Azure Form Recognizer

![image](https://user-images.githubusercontent.com/35609369/149397062-715dd270-b5c1-465e-9233-e9d9f53fa453.png)

## How it works

There are 2 sequences that are available to the user: upload PDF and view parsed results. 

To upload a PDF, the user must click upload PDF from the PowerApp UI and provide a name for the file. When this happens, this triggers a Logic App. This Logic App will call the Form Recognizer to analyze the PDF and then store the results as a .json file in Blob Storage. 

To view the parsed results, the user must click on a PDF file from the PowerApp UI. This will display the PDF within the UI in the middle of the page. This will also display the parsed results in the right of the page (if they have been processed by the Logic App). A refresh button is available to refresh the parsed results until the Logic App has processed the PDF (depending on the recurrence of the Logic App).


**Upload PDF** | ![Untitled Diagram drawio (9)](https://user-images.githubusercontent.com/35609369/149404268-f5931dba-4ef1-4eba-b9a4-0e90a9aae5b2.png)
--- | ---
**View parsed results** | ![Untitled Diagram drawio (10)](https://user-images.githubusercontent.com/35609369/149404252-bab2aa77-d785-441f-8f39-4d8291bfaf61.png)

<br/>

## How to build it

Table of contents:
1. [Logic App A: Process uploaded documents from Blob Storage with Form Recognizer and store results to Blob Storage](#1-logic-app-a-process-uploaded-documents-from-blob-storage-with-form-recognizer-and-store-results-to-blob-storage)
2. [Logic App B: Upon POST request, retrieve content of parsed results from Blob Storage and respond to request with content in body](#2-logic-app-b-upon-post-request-retrieve-content-of-parsed-results-from-blob-storage-and-respond-to-request-with-content-in-body)
3. [PowerApp: Display folders and files from Blob Storage, display PDFs and call Logic App B using custom connector to display parsed results](#3-powerapp-display-folders-and-files-from-blob-storage-display-pdfs-and-call-logic-app-b-using-custom-connector-to-display-parsed-results)

### 1. Logic App A: Process uploaded documents from Blob Storage with Form Recognizer and store results to Blob Storage

![Untitled Diagram drawio (11)](https://user-images.githubusercontent.com/35609369/149404534-e1ec18fa-fbd7-466e-93c4-b4dd3f0e5b08.png)

#### Deploy Azure resources

1. In the Azure Portal, create new resource group for project. Mine will be called `document-parsing-backend`
2. In the Azure Portal, create a storage account in your resource group for this project. Mine will be called `documentstoragebackend`. Performance standard and redundancy LRS are sufficient for this proof-of-concept.
3. In the Azure Portal, create a logic app in your resource group for this project. Mine will be called `documentprocessingbackend`. The type is `consumption` for this proof-of-concept. 
4. In the Azure Portal, create a Form Recognizer resource in your resource group for this project. Mine will be called `documentparsingformrecog`. The pricing tier will be `free` for this proof-of-concept.
5. Create 2 folders in the storage account: `documents` and `documents-parsed`. `documents` will store the PDFs and `documents-parsed` will store the Form Recognizer results.

#### Implement Logic App Logic


1. In the Logic App, open the Logic App Designer (logic app from blank).
2. Search for Blob Storage and select the `When a blob is added or modified (properties only) (V2)` trigger 
    <details>
    <summary>Substeps</summary>
    <p>
        # Hello
        1. Connection name: `documentstoragebackend` (your choice of a connection name)
        2. Authentication type: `Access Key`
        3. Azure Storage Account name: the name of your storage account, in my case `documentstoragebackend`
        4. Azure Storage Account Access Key: in another tab fetch your storage account access key as such ![](/images/2022-01-13-15-59-35.png)
        ![](/images/2022-01-13-16-00-20.png)
    </p>
    </details>


3. 

### 2. Logic App B: Upon POST request, retrieve content of parsed results from Blob Storage and respond to request with content in body

![Untitled Diagram drawio (12)](https://user-images.githubusercontent.com/35609369/149404614-26dccead-c265-4072-8288-5900ae873996.png)

### 3. PowerApp: Display folders and files from Blob Storage, display PDFs and call Logic App B using custom connector to display parsed results 

![Untitled Diagram drawio (13)](https://user-images.githubusercontent.com/35609369/149404706-26091316-10ca-4689-a1f8-7fdbc9849664.png)


