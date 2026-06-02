# Azure Resume 

This is my Cloud Resume built in Azure. The resume is a static website hosted on Azure Storage visually presented using files built with HTML, javascript and CSS.


Additionally here is the architecture diagram for my project:

<p align = "center">
<img src ="https://github.com/khadijahW/CyberFlash/blob/main/Azure.png" width ="650">
  </p>



# Frontend
- The frontend folder contains the files used to host the static website
- The api folder contains the dotnet API deployed to Azure functions
- The main.js file contains the visitor counter code

# Backend 
 - This is where the HTTP trigger function resides. When the functions is triggered it retrieves the CosmosDB item and adds 1 to it and returns this value to the GetResumeCounter


# Workflow
- This is the CI/CD that utilizes Azure CLI to push necesfile sary changes to Azure Storage 

# Issues 
As I progressed through the project, I faced a few challenges and realizations as noted below:
I decided to document this issues for my own recording and to help those who may attempt this project.
<br>
</br>
>[Issue] The project Function was missing as I prepared to deploy my function

            Why: Copying files from other repositories I had created rather than the individual code
<br>
</br>

>[Issue] Resume counter not working

           Why: Issue with main.js formating and displaying counter in html file
            
             <!--body holds the function counter and diplays it -->
                Hi, I'm Khadijah<br>
                <body><script src="main.js"></script>
                </h1> <p>This site has been viewed: <span id="counter"></span> times</p>                           
                </body>
                           
           Additionally: The funtion(host key) must be used in the function API of the main.js file 
           
           Additionally: The function will fail to initially display the visitor count, this is because the function is on a consumption plan which cause a delay in the count being present on the website.
           - In this sutation check that the host key url opens and properly displays the count in JSON format 
<br>
</br>

>[Issue] Error:This XML file does not appear to have any style information associated with it. The document tree is shown below. InvalidQueryParameterValue Value for one of the query parameters specified in the request URI is invalid. RequestId:349475fe-801e-003f-7333-cc2d80000000 Time:2024-07-02T03:56:37.3972391Zcomp

          Why: Fronted end files not updated and necessary CORS orgin missing 
          
<br>
</br>

>[Issue] Backend workflow Failure

          Why: This is because the backend has not been implemented yet 

<br>
</br>

>[Issue] Github and linkedin icons not displaying in html

          Why: need to add
          <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.m
<br>
</br>


>[Issue] Can't determine project language from files. Please use one of [--dotnet-isolated, --dotnet, --javascript, --typescript, --java, --python, --powershell, --custom] 

          Why: Accidental deletion of local.settings.json file- this needed to be re-added to the api folder 

<br>
</br>

>[Issue] Frontend git actions: Upload to Blob storage issue

        Why: Authorization required inorder for git to access the blob storage
        
        Fix: In the frontend.main.yml file use --account-key and add the account key
