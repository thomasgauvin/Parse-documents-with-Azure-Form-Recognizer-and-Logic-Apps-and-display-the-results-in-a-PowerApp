# Parse documents with Azure Form Recognizer and Logic Apps and display the results in a PowerApp

This application enables an end user to upload a PDF and see the parsed results in the UI. The UI is built with PowerApps and the background processing is achieved with Azure Logic Apps and Azure Form Recognizer | ![image](https://user-images.githubusercontent.com/35609369/149397062-715dd270-b5c1-465e-9233-e9d9f53fa453.png)
--- | ---

## How it works

There are 2 sequences that are available to the user: upload PDF and view parsed results. 

To upload a PDF, the user must click upload PDF from the PowerApp UI and provide a name for the file. When this happens, this triggers a Logic App. This Logic App will call the Form Recognizer to analyze the PDF and then store the results as a .json file in Blob Storage. 

To view the parsed results, the user must click on a PDF file from the PowerApp UI. This will display the PDF within the UI in the middle of the page. This will also display the parsed results in the right of the page (if they have been processed by the Logic App). A refresh button is available to refresh the parsed results until the Logic App has processed the PDF (depending on the recurrence of the Logic App).


**Upload PDF** | ![Untitled Diagram drawio (8)](https://user-images.githubusercontent.com/35609369/149403292-ca218433-941a-42cf-a88f-ed1fb79c7e4c.png)
--- | ---
**View parsed results** | ![Untitled Diagram drawio (7)](https://user-images.githubusercontent.com/35609369/149402262-21b359be-305a-44a4-8291-29e27cac41ac.png)

## How to build it

