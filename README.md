# Parse documents with Azure Form Recognizer and Logic Apps and display the results in a PowerApp

This application enables an end user to upload a PDF and see the parsed results in the UI. The UI is built with PowerApps and the background processing is achieved with Azure Logic Apps and Azure Form Recognizer

![image](https://user-images.githubusercontent.com/35609369/149397062-715dd270-b5c1-465e-9233-e9d9f53fa453.png)

## How it works

There are 2 sequences that are available to the user: upload PDF and view parsed results. 

To upload a PDF, the user must click upload PDF from the PowerApp UI and provide a name for the file. When this happens, this triggers a Logic App. This Logic App will call the Form Recognizer to analyze the PDF and then store the results as a .json file in Blob Storage. 

To view the parsed results, the user must click on a PDF file from the PowerApp UI. This will display the PDF within the UI in the middle of the page. This will also display the parsed results in the right of the page (if they have been processed by the Logic App). 


**Upload PDF** | ![Untitled Diagram drawio (9)](https://user-images.githubusercontent.com/35609369/149404268-f5931dba-4ef1-4eba-b9a4-0e90a9aae5b2.png)
--- | ---
**View parsed results** | ![Untitled Diagram drawio (10)](https://user-images.githubusercontent.com/35609369/149404252-bab2aa77-d785-441f-8f39-4d8291bfaf61.png)

<br/>

## How to deploy the current project

1. Clone this project locally. This will facilitate accessing files.
2. Deploy the Resources to Azure. You may do so with the below Deploy to Azure Button or by uploading the ArmTemplate/template.json file to the Azure Studio.


    [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fthomasgauvin%2FParse-documents-with-Azure-Form-Recognizer-and-Logic-Apps-and-display-the-results-in-a-PowerApp%2Fmain%2FArmTemplate%2Ftemplate.json)

3. In the Logic Apps, configure the connection strings to the Blob Storage and the Form Recognizer if need be. You may refer to [HowToBuildIt.md](HowToBuildIt.md) for more details.

4. In the PowerApps studio, create the Custom Connector. 
    1. Under the Data tab, select Custom Connectors. 
    2. Click New Custom Connector. Select Import from OpenAPI file.
    3. Select the CustomConnector/fetch-documents.swagger.json file

5. In the PowerApps Studio ([make.powerapps.com](make.powerapps.com)). 
    1. In the Create tab on the left, select Create from other Sources. Select Open on the Left, and then Browse Files. Select the File PowerAppExport/Microsoft.PowerApps/apps.958791291695911168/N963c2303-a452-45a7-b3e3-66a6448bb50a-document.msapp from this repository.
    2. You may have to reconfigure the Blob Storage and Custom Connector fetch-documents Data connections. Refer to [HowToBuildIt.md](HowToBuildIt.md) for more details. 


## How to build it from scratch

If you are interested in building this application from scratch, I have documented every step in the [HowToBuildIt.md](HowToBuildIt.md) file.