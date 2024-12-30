# Azure Container Registry (ACR): Building Custom Images from OCI Artifacts

An OCI (Open Container Initiative) artifact is a standardised, portable and secure way to package and distribute software. It defines a common format for container images and supporting files, ensuring consistent execution across various operating systems and container runtimes. Azure Container Registry (ACR) natively supports the creation and management of OCI artifacts.

This repository demonstrates how to use the Azure CLI and ORAS CLI tools to build a customised Docker image of an Nginx web service. It includes a sample tarball and Dockerfile to facilitate end-to-end testing, from creating an OCI artifact to deploying a fully functional web site.

> [!NOTE]
> This step-by-step guide assumes you are using Windows 11 on your development machine.

## Table of Contents
* [Pre-requisites](#pre-requisites)
* [Step 1: Create an OCI Artifact](#step-1-create-an-oci-artifact)
* [Step 2: Create an ACR Agent Pool](#step-2-create-an-acr-agent-pool)
* [Step 3: Build a Docker Image](#step-3-build-a-docker-image)
* [Step 4: Deploy a Web site](#step-4-deploy-a-web-site)

## Pre-requisites
1. To build this demo, you'll need:
    - An Azure subscription with an active Entra ID account,
    - An Azure Container Registry (ACR),
    - Docker installed on your development machine.
2. Once you have all the above resources deployed, set relevant environment variables to support execution of CLI commands in the steps below.
``` shell
set MyRegistry=<YOUR_ACR_RESOURCE>
set MyResourceGroup=<RESOURCE_GROUP_OF_ACR>
set MyImage=<TARGET_ACR_IMAGE_NAME>
set MyStorage=<YOUR_STORAGE_RESOURCE>
set MyContainer=<STORAGE_CONTAINER_NAME>
set MyBlob=<TARBALL_FILE_NAME>
```
3. Use the ```az login``` command to authenticate with your Azure subscription using Entra ID credentials.

## Step 1: Operations with Azure Storage account
1. The following command lists all the blobs in the specified Azure Storage container. You can verify the presence of your tarball (_%MyBlob%_) here.
``` PowerShell
az storage blob list --account-name %MyStorage% --container-name %MyContainer% --output table --auth-mode login
```
> [!NOTE]
> For demo purposes, you can re-use the _tarball_ provided with this repo.
2. If the tarball exists, download the specified blob (_%MyBlob%_) from your storage account and place it in the _./src_ directory on your development machine.
``` PowerShell
az storage blob download --account-name %MyStorage% --container-name %MyContainer% --name %MyBlob% --file ./src/%MyBlob% --auth-mode login
```

## Step 2: Operations with Azure Container Registry
1. Use the ```az acr build``` command to build a Docker image using the _Dockerfile_ and the downloaded tarball from the local ```src``` directory. Replace _%MyImage%_ with the desired image name for your development project.
``` PowerShell
az acr build --registry %MyRegistry% --image %MyImage%:latest --file Dockerfile ./src
```
> [!NOTE]
> For demo purposes, you can re-use the _Dockerfile_ provided with this repo.

## Step 3: Testing customised Nginx Web service
1. Verify that your custom Docker image is listed in the ACR repository.
``` PowerShell
az acr repository list --name %MyRegistry% --output table
```
2. You can use the ```show-tags``` command to check the available tags for your image.
``` PowerShell
az acr repository show-tags --name %MyRegistry% --repository %MyImage% --output table
```
3. Now you are ready to run the image as a Docker container. This command maps port 8080 on your local machine to port 80 within the container.
``` PowerShell
docker run -d --name %MyTask% -p 8080:80 %MyRegistry%.azurecr.io/%MyImage%:latest
```
4. You can verify the results by opening the Web page http://localhost:8080/ in your local browser.
![Nginx_site](images/ACR_Tarball.gif)
