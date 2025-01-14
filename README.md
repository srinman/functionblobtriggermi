# Step by Step Guide to use Blob trigger with managed identity in Azure Function


## Create function app with managed identity
az functionapp create --resource-group functestrg --consumption-plan-location eastus2 --runtime python --runtime-version "3.11" --functions-version 4 --name srinmanfun08 --os-type linux --storage-account srinmanstgfun01 

## Create an identity for the function app
az identity create --resource-group functestrg --name srinmanfun08identity --location eastus2

## Assign the identity to the function app
export IDENTITY_ID=$(az identity show --resource-group functestrg --name srinmanfun08identity --query id -o tsv)
az functionapp identity assign --name srinmanfun08 --resource-group functestrg --identities $IDENTITY_ID

## Get the client id of the identity
IDENTITY_CLIENT_ID=$(az identity show --resource-group functestrg --name srinmanfun08identity --query clientId -o tsv)

## Configure the function app to use the managed identity (for blob trigger)
az functionapp config appsettings set --name srinmanfun08 --resource-group functestrg --settings AppBlobTriggerStorage__accountName=srinmanblobtriggertest
az functionapp config appsettings set --name srinmanfun08 --resource-group functestrg --settings AppBlobTriggerStorage__clientId=$IDENTITY_CLIENT_ID
az functionapp config appsettings set --name srinmanfun08 --resource-group functestrg --settings AppBlobTriggerStorage__credential=managedidentity


## Assign role to the managed identity
Storage Blob Data Reader role assignment to the managed identity
Sample command:
az role assignment create --role "Storage Blob Data Reader" --assignee $IDENTITY_ID --scope "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/functestrg/providers/Microsoft.Storage/storageAccounts/srinmanblobtriggertest"


## Deploy the function app
func azure functionapp publish srinmanfun08


## Review the code 

```python
@app.blob_trigger(arg_name="myblob", path="inbound",
                               connection="AppBlobTriggerStorage") 
def srinmanBlobTrigger(myblob: func.InputStream):
    logging.info(f"Python blob trigger function processed blob"
                f"Name: {myblob.name}"
                f"Blob Size: {myblob.length} bytes")
```

