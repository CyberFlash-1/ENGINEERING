# This is the Backend

GetResumeCounter.cs 
- This is where you are adding the COSMOSDB Bindings
 > [!IMPORTANT] Ensure that the correct nuget package is installed 

Copy the primary connection string from CosmosDB to local.settings.json to the AzureResumeConnectionString parameter

Next step is to create the function 
1. Select the function icon
    -   Browse to the location and esure that the backed/api folder is selected as the location to create the function in
    -   net8.0 LTS isolated 
> [!NOTE] You must create the function in order to have a local project folder otherwise you wont be able to deploy the function to azure
2. The Counter.cs file stores the Id and count values originally created in azure functions
3. Then run func host start from api folder
- update html to include counter(this is the variable that stores the count information when the site is visited)

body holds the function counter and diplays it as seen below 

- <body>
    <script src="main.js"></script>
    </h1>
    <p>This site has been viewed: <span id="counter"></span> times</p>                                       
- </body>




Deploy the function to azure by creating a new function           





Can't determine project language from files. Please use one of [--dotnet-isolated, --dotnet, --javascript, --typescript, --java, --python, --powershell, --custom]
Can't determine project language from files. Please use one of [--dotnet-isolated, --dotnet, --javascript, --typescript, --java, --python, --powershell, --custom]


- had to add local.setting/json to api folder, accidentally deleted 


had to add accountkey so github workflow can push changes to storage account 
