# deets
# https://github.com/Contrast-Security-OSS/azure-aks-example

# login
az login
az acr login -n kramericaindustries
docker login


# update files, then build, tag to docker
docker build -f Dockerfile . -t netflicks:latest
docker tag netflicks netflicks:latest
docker login kramericaindustries.azurecr.io
docker push netflicks:latest

# azure variables
ACR=kramericaindustries
RG=app-sec-testing-rg
AKS=netflicks-aks

# Run the following line to create an Azure Container Registry if you do not already have one
az acr create -n $ACR -g $RG --sku basic

# Create an AKS cluster with ACR integration
az aks create -n $AKS -g $RG --generate-ssh-keys --attach-acr $ACR

# deploy netflicks image from ACR to AKS
az aks get-credentials -g $RG -n $AKS
kubectl create secret generic contrast-security --from-file=./contrast_security.yaml
cd ./kubernetes/manifests
kubectl apply -f web-deployment-updated.yaml,web-service.yaml,database-deployment.yaml,database-service.yaml,volume-claim.yaml
