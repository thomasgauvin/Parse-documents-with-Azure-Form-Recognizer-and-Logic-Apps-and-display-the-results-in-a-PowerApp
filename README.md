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

    1. Connection name: `documentstoragebackend` (your choice of a connection name)
    2. Authentication type: `Access Key`
    3. Azure Storage Account name: the name of your storage account, in my case `documentstoragebackend`
    4. Azure Storage Account Access Key: in another tab fetch your storage account access key & click create
        <details>
            <summary>Screenshot</summary>

        ![](/images/2022-01-13-15-59-35.png)
        </details>
    
    5. Select your storage account name as previously configured (`Use connection settings`)
    6. Select container `/documents`
    7. Set Number of blobs to return to `10`
    8. How often do you want to check for items: once per minute

    <details>
    <summary>Screenshots</summary>

    ![](/images/2022-01-13-16-00-20.png)
    ![](/images/2022-01-13-16-11-14.png)
    </details>
3. Add a new step, search for `Parse JSON`
    1. Set the Content as List of File (from the dynamic content modal)
    2. Set the Schema as follows ([as documented](https://docs.microsoft.com/en-us/connectors/azureblobconnector/#blobmetadata)):
    <details>
    <summary>Schema</summary>

    ```
    {
        "properties": {
            "DisplayName": {
                "type": "string"
            },
            "ETag": {
                "type": "string"
            },
            "FileLocator": {
                "type": "string"
            },
            "Id": {
                "type": "string"
            },
            "IsFolder": {
                "type": "boolean"
            },
            "LastModified": {
                "type": "string"
            },
            "LastModifiedBy": {},
            "MediaType": {
                "type": "string"
            },
            "Name": {
                "type": "string"
            },
            "Path": {
                "type": "string"
            },
            "Size": {
                "type": "integer"
            }
        },
        "type": "object"
    }
    ```
    </details>

    <details>
    <summary>Screenshot</summary>

    ![](/images/2022-01-13-16-14-59.png)
    </details>
4. Add a new step, `Get blob Content (V2)`
    1. For the Storage Account name, select `Use connection settings`
    2. To specify the blob, select Id from the Dynamic content modal
    3. Set Infer content type to `yes`
    <details>
    <summary>Screenshot</summary>

    ![](/images/2022-01-13-16-25-02.png)
    </details>
5. Add a new step, searching for Analyze Layout from the Form Recognizer service.
    1. Enter a connection name for your form recognizer, in my case `documentparsingformrecog`
    2. In another tab, retrieve your Form Recognizer service Endpoint URL and Account key in the Azure Portal.
    3. Enter your Endpoint URL, in my case `https://documentparsingformrecog.cognitiveservices.azure.com/`
    4. Enter your Account Key & click create
    5. Set Document/Image file content as `File Content` from the dynamic content modal
    <details>
    <summary>Screenshots</summary>

    ![](/images/2022-01-13-16-29-54.png)
    ![](/images/2022-01-13-16-29-06.png)
    ![](/images/2022-01-13-16-31-15.png)
    </details>
6. Add a new step, searching for `Create blob V2`

### 2. Logic App B: Upon POST request, retrieve content of parsed results from Blob Storage and respond to request with content in body

![Untitled Diagram drawio (12)](https://user-images.githubusercontent.com/35609369/149404614-26dccead-c265-4072-8288-5900ae873996.png)

### 3. PowerApp: Display folders and files from Blob Storage, display PDFs and call Logic App B using custom connector to display parsed results 

![Untitled Diagram drawio (13)](https://user-images.githubusercontent.com/35609369/149404706-26091316-10ca-4689-a1f8-7fdbc9849664.png)


