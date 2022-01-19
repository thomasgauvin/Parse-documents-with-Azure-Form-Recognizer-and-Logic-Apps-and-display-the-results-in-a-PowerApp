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

<br/>

### 1. Logic App A: Process uploaded documents from Blob Storage with Form Recognizer and store results to Blob Storage

Logic App A is responsible for processing uploaded forms in the background. Upon a PDF being uploaded to Blob Storage, a Logic App will be triggered and will call a Form Recognizer to analyze the contents. The output of the Form Recognizer will be stored in Blob Storage by the Logic App.

*Depending on your use case, it may be better to store the output of the Form Recognizer in a structured database such as Cosmos DB. For the sake of this Demo, we wanted to display only the transcript as received by the Form Recognizer so we decided to store it in Blob Storage.*

![Untitled Diagram drawio (11)](https://user-images.githubusercontent.com/35609369/149404534-e1ec18fa-fbd7-466e-93c4-b4dd3f0e5b08.png)

#### Deploy Azure resources

1. In the Azure Portal, create new resource group for project. Mine will be called `document-parsing-backend`
2. In the Azure Portal, create a storage account in your resource group for this project. Mine will be called `documentstoragebackend`. Performance standard and redundancy LRS are sufficient for this proof-of-concept.
3. In the Azure Portal, create a logic app in your resource group for this project. Mine will be called `documentprocessingbackend`. The type is `consumption` for this proof-of-concept. 
4. In the Azure Portal, create a Form Recognizer resource in your resource group for this project. Mine will be called `documentparsingformrecog`. The pricing tier will be `free` for this proof-of-concept.
5. Create 2 folders in the storage account: `documents` and `documents-parsed`. `documents` will store the PDFs and `documents-parsed` will store the Form Recognizer results.

#### Implement Logic App A Logic

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
    1. Set Storage account name as `Use connection settings`
    2. Set Folder path as `/documents-parsed`
    3. Set display name as `DisplayName.json`
    4. Set Blob content as `analyzeResult` from the dynamic content modal

        <details>
        <summary>Screenshot</summary>

        ![](/images/2022-01-18-15-32-05.png)
        </details>
7. Save the logic app. 
8. Test that the logic app is working properly
    1. From your resource group (in my case, `document-parsing-backend`), go to your storage account (in my case, `document-parsing-backend`)
    2. Navigate to the containers. In the `/documents/ container, upload a sample PDF from your computer.
    3. Return to the Logic App, in my case `documentprocessingbackend`
    4. In the Logic App Overview, you should see a run has started in the Runs History tab (this may take up to 1 minute to be triggered). Click on it to see the details. From here, you can see each step in the Logic App, their inputs and their outputs.
    5. Lastly, return to your storage account and verify that a blob with the parsed data has been stored in the `/documents-parsed` container as show in the screenshot. You can also download the file to ensure the contents are as expected as well. You should see a json object with the transcript as parsed by Form Recognizer.

        ```
        {"version":"2.1.0","readResults":[{"page":1,"angle":0,"width":8.2639,"height":11.6944,"unit":"inch","lines":[{"boundingBox":[2.7009,0.7449,5.5057,0.7449,5.5057,1.0947,2.7009,1.0947],"text":"James Anderson",
        ...
        }
        ```

        <details>
        <summary>Screenshots</summary>

        ![](/images/2022-01-18-15-34-23.png)
        ![](/images/2022-01-18-15-35-40.png)
        ![](/images/2022-01-18-15-37-52.png)
        ![](/images/2022-01-18-15-38-56.png)
        </details>

<br/>


### 2. Logic App B: Upon POST request, retrieve content of parsed results from Blob Storage and respond to request with content in body

Logic App B will provide an API to retrieve the contents of the Blob Storage file as an HTTP Response. This will allow our PowerApp to display the JSON as content. This is necessary because PowerApps does not have a native connector to display Txt/JSON data within an app.

*This step could be reconsidered for your use case. If you decided to store your data within a database such as Cosmos DB, you could connect to it directly from within your PowerApp and this extra API would not be necessary.

![Untitled Diagram drawio (12)](https://user-images.githubusercontent.com/35609369/149404614-26dccead-c265-4072-8288-5900ae873996.png)

#### Deploy Azure resources

1. Deploy a new Logic App resource to your existing resource group (in my case `document-parsing-backend`)
    1. Set Resource-group as the one you created for this project
    2. Set the Type to `Consumption`
    3. Set the Logic App name to anything, in my case `documents-parsed-fetching`
    4. Select any Region
    5. Set Enable log analytics to `No`
    6. Click Review + Create
        <details>
        <summary>Screenshot</summary>

        ![](/images/2022-01-18-15-53-21.png)
        </details>

#### Implement Logic App B logic

1. Navigate to the Logic App you just created and select `Blank Logic App`

    <details>
    <summary>Screenshot</summary>

    ![](/images/2022-01-18-15-55-03.png)
    </details>

2. Search for `When a HTTP request is received` as your trigger
    1. Set the Request Body JSON Schema as follows (when calling this endpoint, we will pass an object with a key filePath and a value representing the filePath of the document-parsed we want):
        ```
        {
            "properties": {
                "filePath": {
                    "type": "string"
                }
            },
            "type": "object"
        }
        ```
3. Add a new step, searching for `Get blob content using path (V2)`
    1. Set the connection name to represent a connection to your Storage account, in my case `documentparsedfetching`
    2. Set the Authentication type as `Access Key`
    3. Set the Azure Storage Account name to the name of your storage account, in my case `documentstoragebackend`
    4. Set the Azure Storage Account Access Key to the Access Key of your Storage Account (retrieve it as done in the previous step)
    5. Click create
    6. Set Storage account name to `Use connection strings`
    7. Set Blob path to filePath from the Dynamic Content modal
    8. Set Infer content type to No (we just want to extract the raw content)

        <details>
        <summary>Screenshots</summary>

        ![](/images/2022-01-18-16-00-47.png)
        ![](/images/2022-01-18-16-02-47.png)
        </details>
4. Add a new step, searching for `Response`
    1. Set Status Code as `200`
    2. Set Headers as `Content-Type` `application/json`
    3. Set Body as follows (as screenshot) **(note that quotes ("") must surround the `File Content` so that it is returned as a string)**:
        ```
        {
            "fileContent": "[INSERT FILE CONTENT FROM DYNAMIC CONTENT MODAL]"
        }
        ```
        <details>
        <summary>Screenshot</summary>

        ![](/images/2022-01-18-16-06-10.png)
        </details>
5. Save the Logic App. 
6. From the Logic App designer, click `Run Trigger` > `Run with payload` to test your logic app.
    1. Leave URL as default
    2. Set Method as `POST`
    3. Set body as follows, replacing the path of the filePath for the filePath of your parsed document from the documents-parsed container in your storage account. In my case, the body looks as follows:
        ```
        {
            "filePath": "/documents-parsed/Resume 24.pdf.json"
        }
        ```
    4. Press Run. You should see a Response Body with as an object, with a key fileContent and a value consisting of the contents of the JSON file (see screenshot)

        <details>
        <summary>Screenshots</summary>

        ![](/images/2022-01-18-16-10-02.png)
        ![](/images/2022-01-18-16-11-58.png)
        </details>

<br/>

### 3. PowerApp: Display folders and files from Blob Storage, display PDFs and call Logic App B using custom connector to display parsed results 

![Untitled Diagram drawio (13)](https://user-images.githubusercontent.com/35609369/149404706-26091316-10ca-4689-a1f8-7fdbc9849664.png)

1. Go to make.powerapps.com and make a new Blank Canvas App, making sure to select the `Tablet` format

    <details>
    <summary>Screenshot</summary>

    ![](/images/2022-01-18-16-19-04.png)
    </details>

2. Implement the folder view of documents from the Blob Storage following this [tutorial](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/connections/connection-azure-blob-storage)
    1. Connect your PowerApp to the Azure Blob Storage account ([as documented](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/connections/connection-azure-blob-storage#create-canvas-app-with-azure-blob-storage-connection))
        1. On the left, select the Data tab.
        2. Click the Add data button, searching for Azure Blob Storage (see screenshot)
        3. To configure the Blob Storage connection, select `Authentication type` as `Access Key`, `Azure Storage Account name` as the name of your storage account in the Azure Portal, `Azure Storage Account Access Key` as the access key of your storage account (as retrieved previously).
        4. Click connect
            <details>
            <summary>Screenshots</summary>

            ![](/images/2022-01-19-09-38-39.png)
            ![](/images/2022-01-19-09-40-23.png)
            </details>
    2. Implement the view to see Blob Storage containers and folders [as documented](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/connections/connection-azure-blob-storage#view-containers-and-files)
        1. From the Insert tab, insert a Vertical Gallery
        2. Set the data source for the Vertical Gallery to be Azure Blob Storage
        3. Set the Items of the Vertical Gallery `Gallery1` to `AzureBlobStorage.ListRootFolderV2().value`
        4. Set the layout of the Vertical Gallery to be `Title`
        5. From the Insert tab, insert another Vertical Gallery
        6. Set the Items of the Vertical Gallery `Gallery 2` to `AzureBlobStorage.ListFolderV2(Gallery1.Selected.Id).value`
        7. Set the Layout as `Title`
        8. For both Vertical Galleries `Gallery 1` and `Gallery 2`, set the Template Fill attribute to: 
            ```
            If(ThisItem.IsSelected,
            RGBA(0,0,0,0.3), 
            RGBA(0,0,0,0)
            )       
            ```
        9. Reduce the size of the Font to 12 and the size of the arrow
            <details>
            <summary>Screenshots</summary>

            ![](/images/2022-01-19-09-44-58.png)
            ![](/images/2022-01-19-09-45-43.png)
            ![](/images/2022-01-19-09-47-13.png)
            ![](/images/2022-01-19-09-49-37.png)
            ![](/images/2022-01-19-09-50-09.png)
            ![](/images/2022-01-19-09-59-46.png)
            </details>

    3. Implement the file upload [as documented](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/connections/connection-azure-blob-storage#upload-files-to-azure-blob-storage)
        1. From the Insert tab, search for Add Picture and add it to your PowerApp
        2. Set the `Text` to `Select PDF`
        3. Resize the button to fit below the folder view
        4. From the Insert tab, search for Button and add it to your PowerApp
        5. Set the `Text` to `Upload`
        6. From the Insert tab, search for `Text input` and add it to your PowerApp
        7. Set the `Text` to `Filename`
        8. Set the `OnSelect` property of the Button to `AzureBlobStorage.CreateFile(Gallery1.Selected.Name, TextInput1.Text, UploadedImage1.Image)`
        9. Test the application to see if you can upload a new file. Note that when uploading, select any type of file to have the option to upload PDFs when selecting your file. You will have to click out of your parent folder and back to see the contents of the folder refresh.

            <details>
            <summary>Screenshots</summary>

            ![](/images/2022-01-19-10-03-22.png)
            ![](/images/2022-01-19-10-04-32.png)
            ![](/images/2022-01-19-10-05-41.png)
            ![](/images/2022-01-19-10-09-59.png)
            </details>

        10. (Optimization, skip if you are content with the workaround noted above to refresh the folder contents) Set the `Items` of `Gallery 2` to `SecondLevelList`. `SecondLevelList` is a global variable which will be set later.
        11. Add the following code to the `OnSelect` of the upload Button
        `ClearCollect(SecondLevelList, AzureBlobStorage.ListFolderV2(Gallery1.Selected.Id).value);`
        12. Set the `OnSelect` property of the `NextArrow` of `Gallery 1` to `Select(Parent); ClearCollect(SecondLevelList, AzureBlobStorage.ListFolderV2(Gallery1.Selected.Id).value);`
        13. Test your application, uploading 

            <details>
            <summary>Screenshots</summary>

            ![](/images/2022-01-19-10-23-35.png)
            ![](/images/2022-01-19-10-23-19.png)
            ![](/images/2022-01-19-10-24-54.png)
            </details>

    4. Display the selected PDF
        1. From the Insert tab, insert the `PDF viewer (experimental)`
        2. Set the `Document` attribute to `AzureBlobStorage.GetFileContent(Gallery2.Selected.Id)`
        3. Test that the PDF viewer is working. It may take some time to load the PDF or you may have to click out of the currently selected PDF to another one to see the PDF viewer in effect.

            <details>
            <summary>Screenshot</summary>

            ![](/images/2022-01-19-10-26-38.png)
            ![](/images/2022-01-19-10-28-55.png)
            </details>