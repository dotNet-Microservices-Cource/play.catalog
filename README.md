## Play.Catalog
Play Economy Catalog microservice

## Create and publish package
```powershell
$version="1.0.3"
$owner="dotNet-Microservices-Cource"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Catalog.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.catalog -o..\packages

dotnet nuget push ..\packages\Play.Catalog.Contracts.$version.nupkg --api-key $gh_pat --source "github"
```

## Build the docker image
```powershell
$env:GH_OWNER="dotNet-Microservices-Cource"
$env:GH_PAT="[PAT HERE]"
$appname="playeconomyapp"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$appname.azurecr.io/play.catalog:$version" .
```

## Run the docker image
```powershell
$cosmosDbConnString="[CONN STRING HERE]"
$serviceBusConnString="[CONN STRING HERE]"
docker run -it --rm -p 5000:5000 --name catalog -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" play.catalog:$version
```

## Publishing the Docker image
```powershell
az acr login --name $appname
docker push "$appname.azurecr.io/play.catalog:$version"
```

## Creating the pod managed identity
```powershell
$resourcegroup="playeconomy"
$managedname="catalog"
az identity create --resource-group $resourcegroup --name $managedname
$IDENTITY_RESOURCE_ID=az identity show -g $resourcegroup -n $managedname --query id -otsv

az aks pod-identity add --resource-group $resourcegroup --cluster-name $appname --namespace $managedname --name $managedname --identity-resource-id $IDENTITY_RESOURCE_ID
```

## Granting access to Key Vault secrets
```powershell
$IDENTITY_CLIENT_ID=az identity show -g $resourcegroup -n $managedname --query clientId -otsv
az keyvault set-policy -n $appname --secret-permissions get list --spn $IDENTITY_CLIENT_ID
```