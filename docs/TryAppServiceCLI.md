# Using the Azure CLI with Try App Service (Linux Web Apps)

This tutorial illustrates how to get started with the [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/overview), in order to manage a Linux Web App that was created via [Try App Service](tryappservice.azure.com). This way, in addition to being able to evaluate the Azure Portal, Git deployment, etc., you can check out the CLI as well, which is an important aspect of the Azure developer experience.

## Pre-Requisites

In order to ensure you can spend the entire trial period dedicated to Linux Web Apps, make sure to complete the following two steps (if you haven't already) before actually creating the app via `Try App Service`:

1. [Install](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) the Azure CLI 2.0 and it's dependencies (depending on your platform). 

2. Login to the Azure CLI 2.0, using the Microsoft account that you intend to use when authenticating with `Try App Service`:

    ```shell
    az login
    ```

    > Note: Follow the instructions displayed by the above command in order to complete the authentication process.

## Getting Started

1. Launch a browser and navigate to [Try App Service](tryappservice.azure.com), select the `Linux Web App` app type, and then create a trial app using whichever template you'd prefer (e.g. Node.js). Remember to login with the same Microsoft account you used when logging in with the Azure CLI.

2. Return to your terminal, and run the following command in order to configure your environment to default to the web app that was created by `Try App Service`. This ensures that all subsequent CLI commands are appropriately scoped, and will make it much simpler for you to experiment with the CLI:

    ```shell
    az account set -s $(az account list --query "[?starts_with(@.name, 'TryLinux')].name" -otsv) && \
        az configure -d group=$(az group list --query "[].name" -otsv) web=$(az webapp list --query "[].name" -otsv)
    ```

3. Launch your newly deployed web app to ensure that everything is setup correctly:

    ```shell
    az webapp browse
    ```

4. That's it! You can now explore the various commands you can run to manage your web app (e.g. setting env vars, restarting, deploying a Docker container). Just add `-h` after a command group (e.g. `az webapp config`) or command (e.g. `az webapp restart`) in order to get more details about it.

    ```shell
    # View details about all web commands and groups
    az webapp -h

    # View details about the "config" group
    az webapp config -h
    ```

Whenever you're done, or the Try App Service instance expires (after 30 minutes), you can run the following commands in order to reset your CLI environment:

```shell
az configure -d group='' web=''

az logout
```

> Note: If you'd like, you can skip the call to `az logout`, provision another Linux Web App trial, and repeat step #2 above in order to manage another trial web app instance. 
