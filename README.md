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

## How to build it from scratch

If you are interested in building this application from scratch, I have documented every step in the [HowToBuildIt.md](HowToBuildIt.md) file.