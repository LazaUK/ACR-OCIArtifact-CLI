# Azure Container Registry: Building Custom Images from OCI Artifacts

An OCI (Open Container Initiative) artifact is a standardised, portable and secure way to package and distribute software. It defines a common format for container images and supporting files, ensuring consistent execution across various operating systems and container runtimes. Azure Container Registry (ACR) natively supports the creation and management of OCI artifacts.

This repository demonstrates how to use the **Azure CLI** and **ORAS CLI** tools to build a customised Docker image of an Nginx web service. It includes a sample tarball and Dockerfile to facilitate end-to-end testing, from creating an OCI artifact to deploying a fully functional web site.

> [!NOTE]
> This step-by-step guide assumes you are using Windows 11 on your development machine.

## Table of Contents
* [Pre-requisites](#pre-requisites)
* [Step 1: Create an OCI Artifact](#step-1-create-an-oci-artifact)
* [Step 2: Create an ACR Agent Pool](#step-2-create-an-acr-agent-pool)
* [Step 3: Build a Docker Image](#step-3-build-a-docker-image)
* [Step 4: Deploy a Web site](#step-4-deploy-a-web-site)

## Pre-requisites
1. Resources:
    - **Azure Subscription**: Ensure you have an active Azure subscription with an Entra ID account.
    - **Azure Container Registry**: Create an ACR instance.
    - **Docker**: Install Docker on your development machine.
2. Environment Variables:
    - Once you have these resources, set the following environment variables for the CLI commands in the upcoming steps. Replace the placeholders with your actual values:
    ``` shell
    set MyRegistry=<YOUR_ACR_RESOURCE>
    set MyRegistryFQDN=<YOUR_ACR_RESOURCE>.azurecr.io
    set MyResourceGroup=<RESOURCE_GROUP_OF_ACR>
    set MyAgentPool=<ACR_POOL_NAME>
    set MyTask=<CUSTOM_ACR_TASK_NAME>
    set MyImage=<TARGET_IMAGE_NAME>
    set MyBlob=<TARBALL_FILE_NAME>
    ```

## Step 1: Create an OCI Artifact
1. Install ORAS CLI tool as described [here](https://oras.land/docs/installation).
2. Logic to ACR with your Entra ID credentials:
``` PowerShell
az acr login --name %MyRegistry%
```
3. Push tarball and Dockerfile to ACR create an OCI artifact:
``` PowerShell
oras.exe push %MyRegistryFQDN%/demotar:v1 demopage.tar.gz:application/x-tar Dockerfile:text/plain
```
> [!NOTE]
> _application/x-tar_ and _text/plain_ provide metadata, to describe file types of uploaded tarball and Dockerfile.
4. Check the manifest of created OCI artifact to verify its structure:
``` PowerShell
oras manifest fetch --pretty %MyRegistryFQDN%/demotar:v1
```

## Step 2: Create an ACR Agent Pool
1. Use the ```az acr build``` command to build a Docker image using the _Dockerfile_ and the downloaded tarball from the local ```src``` directory. Replace _%MyImage%_ with the desired image name for your development project.
``` PowerShell
az acr build --registry %MyRegistry% --image %MyImage%:latest --file Dockerfile ./src
```
> [!NOTE]
> For demo purposes, you can re-use the _Dockerfile_ provided with this repo.

## Step 3: Build a Docker Image

## Step 4: Deploy a Web site
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
