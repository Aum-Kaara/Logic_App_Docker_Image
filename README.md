# How to create Docker Image of Azure Logic App Standard and Deploy in Azure Kubernetes service (AKS)
This articles gives you overview of 
- How to create Docker Image of Azure Logic app 
- How to push image into azure Container Registry (ACR) and 
- How to deploy ACR image into AKS 

### Pre-requisites
- Docker for Desktop
- Docker extension for VS code
- Logic App (Standard) Extension
- C# Extension
- Azure Account
- Stoarge account 

### Create a stateless workflow locally in Visual studio Code

i am using VS code for developing Azure Logic App Standard .

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

make sure to tag like <acr name>.azurecr.io\imagename to push image into ACR

```
docker build --tag <ImageName> .
```

Run the container locally by using the below command
```
docker run -p 8080:80 <ImageName>
```


you can see images and instances in Docker extension 

![image](https://user-images.githubusercontent.com/6815990/155837407-789954ad-145f-43ed-9c79-e7bf04c47a66.png)


Navigate to the Azure storage account provided in the docker definition and open blob containers. You will see the following folders

![image](https://user-images.githubusercontent.com/6815990/155837426-72ea344a-c0b6-4e1a-abcc-56114768c470.png)


### Push Docker Image to Azure Container Registry

Create Azure Container registry manually on Azure Portal

![image](https://user-images.githubusercontent.com/6815990/155837509-03d30b4a-8d28-49af-8eda-bfa5ebfc19ca.png)

### Enable Access key to login into ACR 
 
 this helps to login into acr and push images 

![image](https://user-images.githubusercontent.com/6815990/155837633-2af41bd5-0684-43a9-bc76-7018075e0a22.png)

### Login into Docker Registry

```
docker login laacr.azurecr.io
```

![image](https://user-images.githubusercontent.com/6815990/155837729-d241390e-05f4-4b1a-8151-93b93c626979.png)


### Tag and Push Image to ACR

Use docker tag to create an alias of the image with the fully qualified path to your registry. 

```
docker tag laimage laacr.azurecr.io/laimage
```

```
docker push laacr.azurecr.io/laimage
```

![image](https://user-images.githubusercontent.com/6815990/155837776-e24a25f0-bd1e-4146-b6f0-005ea6d5aac9.png)

![image](https://user-images.githubusercontent.com/6815990/155840267-aa740023-8252-4e7c-aa81-31e0027f6ba9.png)

### Create Azure Kubernetes cluster - AKS

![image](https://user-images.githubusercontent.com/6815990/155880829-85a5786f-8fbc-496c-8f6c-8145e8fd5611.png)

### Link ACR with AKS

```
az aks update -n laaks -g acr --attach-acr laacr
```

### Deploy Logic App Image from ACR into AKS 

Go to AKS - > 
```
- apiVersion: v1
  kind: Namespace
  metadata:
    name: azure-la
  spec:
    finalizers:
      - kubernetes
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: azure-la-front
    namespace: azure-la
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: azure-la-front
    template:
      metadata:
        labels:
          app: azure-la-front
      spec:
        nodeSelector:
          beta.kubernetes.io/os: linux
        containers:
          - name: azure-la-front
            image: laacr.azurecr.io/laimage
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            ports:
              - containerPort: 80
            env:
              - name: AzureWebJobsStorage
                value: <<connectionstring>>
- apiVersion: v1
  kind: Service
  metadata:
    name: azure-la-front
    namespace: azure-la
  spec:
    type: LoadBalancer
    ports:
      - port: 80
    selector:
      app: azure-la-front

```

![image](https://user-images.githubusercontent.com/6815990/155884836-449339eb-6a77-4ae9-afd6-a5e4600bbdbc.png)

check if app service deployed 

![image](https://user-images.githubusercontent.com/6815990/155884890-a3601a65-7f87-4f15-93e5-744fb179c295.png)


### Fetch logic app URL using callback url 

Fetch master key from blob storage which will help in fetching callback url 

![image](https://user-images.githubusercontent.com/6815990/155885243-11635446-1a2a-46c0-b015-4de39b9c86a6.png)


Fetch callback url using url 

http://20.76.240.221:80/runtime/webhooks/workflow/api/management/workflows/Stateful1/triggers/manual/listCallbackUrl?api-version=2020-05-01-preview&code=<amsterkey>

![image](https://user-images.githubusercontent.com/6815990/155879905-afc54d46-4ba0-42bf-9f00-4d255d8e99a7.png)

you can use postman to test logic app 

Logic App url - http://20.76.240.221:80/api/Stateful1/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=<key>
![image](https://user-images.githubusercontent.com/6815990/155885042-441e3d21-2d45-4f67-adfe-f955f489ab5d.png)

