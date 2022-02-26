# Logic_App_Docker_Image
Create a Docker Image of logic app

# Pre-requisites
- Docker for Desktop
- Docker extension for VS code

# Create a stateless workflow locally in Visual studio Code

![image](https://user-images.githubusercontent.com/6815990/155837169-a34430b5-e331-4a6f-b728-438371cc9dc0.png)

![image](https://user-images.githubusercontent.com/6815990/155837205-49e4dd0b-3a99-4d9f-a509-a872fc4f793e.png)

it will create skeleton of workflow

![image](https://user-images.githubusercontent.com/6815990/155837221-849e10c9-45b4-4919-9ebc-1abd7f1567ef.png)

UseDevelopmentStorage=true property lets us run logic apps locally using Azure Storage Emulator.

Right click on the workflow.json file and select Open in Designer to view the workflow designer

![image](https://user-images.githubusercontent.com/6815990/155837247-39607915-c8b4-4d57-a822-bcd9bfc52cee.png)

![image](https://user-images.githubusercontent.com/6815990/155837277-b1a2e976-852a-464d-8baa-b7cb2643e88b.png)

Select Run->Start Debugging or F5 to start the logic apps locally, finally the execution displays the following to show the logic apps is up and running.

### Add a docker file to your project as below named “DockerFile”

```
FROM mcr.microsoft.com/azure-functions/node:3.0

 

ENV AzureWebJobsStorage DefaultEndpointsProtocol=<BlobStorageConnectionString>

ENV AzureWebJobsScriptRoot=/home/site/wwwroot AzureFunctionsJobHost__Logging__Console__IsEnabled=true FUNCTIONS_V2_COMPATIBILITY_MODE=true

ENV WEBSITE_HOSTNAME localhost

ENV WEBSITE_SITE_NAME weatherforecast

ENV AZURE_FUNCTIONS_ENVIRONMENT Development

COPY . /home/site/wwwroot
```

And modify the local.settings.json to have the storage account as the

Build the docker container image by using the docker file by running the below command

```
docker build --tag <ImageName> .
```

Run the container locally by using the below command
```
docker run -p 8080:80 <ImageName>
```

you can see Docker extension 

![image](https://user-images.githubusercontent.com/6815990/155837407-789954ad-145f-43ed-9c79-e7bf04c47a66.png)


Navigate to the Azure storage account provided in the docker definition and open blob containers. You will see the following folders

![image](https://user-images.githubusercontent.com/6815990/155837426-72ea344a-c0b6-4e1a-abcc-56114768c470.png)
