<!---
Purpose :
Test deploying container from VS Code because VS Code is much easier than Azure Portal

#################################################################
TODO

#################################################################
--->

# Create a Container Using Visual Studio Code and Deploy it to the DE10-Nano

This is part 2 of a 4-part tutorial series that shows you how to manage the DE10-Nano with Azure IoT Edge and use container-based virtualization to reprogram the onboard FPGA from the Azure Cloud. 

## Table of Contents

-  [About this Tutorial](#about-this-tutorial)
   -  [Objectives](#objectives)
   -  [Prerequisites](#prerequisites)
-  [Step 1: Install Visual Studio Code](#step-1-install-visual-studio-code)
-  [Step 2: Set up VS Code for Developing IoT Edge Containers](#step-2-set-up-vs-code-for-developing-iot-edge-containers)
   -  [Install IoT Tools Extension](#install-iot-tools-extension)
   -  [Create an Azure Container Registry](#create-an-azure-container-registry)
   -  [Log in to your Azure account from VS Code](#log-in-to-your-azure-account-from-vs-code)
   -  [Sign in to your IoT Hub](#sign-in-to-your-iot-hub)
-  [Step 3: Set up a Cross-compiling Environment](#step-3-set-up-a-cross-compiling-environment)
   -  [Install Docker on your Development PC](#install-docker-on-your-development-pc)
   -  [Install buildx](#install-buildx)
   -  [Set up buildx](#set-up-buildx)
-  [Step 4: Build and Deploy a Simulated Temperature Sensor Module from VS Code](#step-4-build-and-deploy-a-simulated-temperature-sensor-module-from-vs-code)
   -  [Create a Solution Template](#create-a-solution-template)
   -  [Add ACR Credentials to the Env File](#add-acr-credentials-to-the-env-file)
   -  [Change your Target Platform](#change-your-target-platform)
   -  [Log in to your ACR](#log-in-to-your-acr)
   -  [Build and Push a Sample Module to ACR](#build-and-push-a-sample-module-to-acr)
   -  [Create a Deployment Manifest](#create-a-deployment-manifest)
-  [Next Steps](#next-steps)


## About this Tutorial

This tutorial provides instructions for how to create a container application using Visual Studio Code (VS Code) and deploy it to the DE10-Nano.

<!-- where is the IoT Edge module / container application deployed from? From the Azure cloud? -->

<!-- 

### Audience

Software engineers who want to create an IoT Edge container application for SoC FPGA devices that integrate ARM 32-bit processors. Users can substitute the DE10-Nano for other devices that implement an ARMv7, such as Raspberry Pi 2 Model B.

-->

### Objectives

In this tutorial, you will learn you learn how to:

- Set up Visual Studio Code for developing an IoT Edge container application
- Set up a cross-compiling environment with buildx
- Build an IoT Edge container application with a sample SDK
- Store your container application in an Azure Container Registry (ACR)
- Deploy your container application from VS Code

### Prerequisites

* Complete the previous tutorial, [Run Azure IoT Edge on the DE10-Nano][LINK_module_01].
* An Azure IoT hub 

<!--- "de10-nano-iothub" --->

* An IoT Edge device (DE10-Nano) with IoT Edge runtime installed

  **Note**: If you completed the previous tutorial, the DE10-Nano should have IoT Edge runtime installed.

<!--- "de10-nano-iotedge" --->

# Step 1: Install Visual Studio Code

If you are not using Ubuntu on your development PC, install Visual Studio Code (VS Code) according to your development environment. Refer to [Visual Studio Code on Linux][Link_VSCode_Setup].

1. Set up apt to install VS Code.  

    ```
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
    sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/
    sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
    ```
    
2. Install VS Code via apt.  

    ```
    sudo apt-get install apt-transport-https
    sudo apt-get update
    sudo apt-get install code
    ```

# Step 2: Set up VS Code for Developing IoT Edge Containers

To develop an Azure IoT Edge container application, you will need to add the [IoT Tools Extension](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

## Install IoT Tools Extension

This extension enables you to manage IoT Edge devices. You can see device status, monitor messages sent from devices to the cloud, and build containers using VS Code.

1. Open VS Code.  

    ```
    code &
    ```

2. Open search form (Crtl + P) and install the extension.  

    ```
    ext install vsciot-vscode.azure-iot-tools
    ```

## Create an Azure Container Registry (ACR)

Before developing an IoT Edge container application, create an Azure Container Registry (ACR) to store container images on the cloud. By storing containers in ACR, you can securely deploy containers to IoT Edge Devices.  

**Note**: See [Quickstart: Create a private container registry using the Azure portal][LINK_Azure_ACR_getting_started] for Microsoft's tutorial on how to create an ACR. 

1. Open an Azure Portal, navigate to the search bar, and search "container". Click on **Container registries**.

2. Click **Create**.  

    ![Azure-ACR-Creation](picture/az-acr-creation-00.png)

3. Set project and instance details. Use the resource group you created in the previous tutorial (recommended). 

4. Enter a registry name (for example, de10nano). To avoid high costs, select **Basic** for SKU .

5. Click **Create**.
   
    ![Azure-ACR-Creation](picture/az-acr-creation-01.png)

6. Open the container registery you just made and view the login server name. Go to **Settings** > **Access keys**.

    ![Azure-ACR-Creation](picture/az-acr-creation-02.png)
    
    Remember the login server name. You will use this name for saving container applications and deploying container images on ACR to IoT Edge devices. In this example, the server name is **de10nano.azurecr.io**.

7. Make sure to Enable Admin user. 
   
## Log in to your Azure account from VS Code

1. Open a command palette (Ctrl + Shift + P) in VS Code.  

2. Type and enter the command **Azure: Sign in with Device Code**.  

    ![VSCode-Login-To-Azure](picture/az-azure-login-00.png)

3. Copy the device code.  

    ![Copy-Device-Code](picture/az-azure-login-01.png)

4. Open a browser, go to the [Azure Device Authentication][LINK_Azure_Device_login] page, and enter the device code.  

    ![Put-Device-Code](picture/az-azure-login-02.png)
    
    After you log in, go back to VS Code.  
    
    **Note :** If you get a pop up window about making a new password, proceed to do so and then continue tutorial

## Sign in to your IoT Hub from VS Code

After logging in to your Azure account from VS Code, you can select your IoT Hub.  

1. Open VS Code explorer.  

2. Click the "..." button on Azure IoT Hub explorer in the bottom left.  

3. Click **Select IoT Hub**.  

    ![VSCode-Select-Your-IoT-Hub](picture/az-select-iothub-00.png)

4. Select your subscription.  

    ![VSCode-Select-Your-IoT-Hub-Resource](picture/az-select-iothub-01.png)

5. Select your IoT Hub.  

    ![VSCode-Select-Your-IoT-Hub](picture/az-select-iothub-02.png)
    
    After registering your IoT Hub to VS Code, you can manage devices connected to your IoT Hub.  

# Step 3: Set up a Cross-compiling Environment

Before developing a container in VS Code, you need to set up a cross-compiling environment because x86 systems with Ubuntu often cannot create a container for arm32v7 devices.  

**Note**: If you build a container application on DE10-Nano, you do not need a cross-compiling environment. However, when compared to an x86 system, the build time is longer.  

## Install Docker on your Development PC

Install Docker version 19.03 or later on your development PC. buildx requires Docker version 19.03 or later.

To install Docker on you development PC, go to [Install Docker Engine][LINK_Docker_Installation].
  
**Note :** If you complete the post installation steps that can be found in the above link it will allow you to run docker commands as any user. Otherwise, need to run docker commands with root access moving forward.

## Install buildx

Set up your Docker config file to enable buildx.

1.  Create a new folder. 

    ```
    sudo su
    mkdir ~/.docker
    mkdir ~/.docker/cli-plugins  
    ```
    
2.  Open `config.json`.
    
    ```
    vim ~/.docker/config.json
    ```
3. Copy and paste the code below into the config file. Save your changes.

    ```
    {
        "experimental": "enabled"
    }
    ```

2. Download the latest buildx binary file from the [buildx release page on Github][LINK_Buildx_release]. Download the file to a specific folder, such as `Downloads`.

   **Note**: Currently, `buildx-v0.4.1` is the latest release. Update the commands below to reflect the latest release version.

3. Navigate to the folder where you downloaded the binary, move the binary to `~/.docker/cli-plugins` and rename it to `docker-buildx`.

    ```
    cd <Download folder>
    ```

    ```
    mv buildx-v0.4.1.linux-amd64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    ```

4. Confirm that buildx is installed.

    If the version displays, buildx is installed. 
    
    Input:
    
    ```
    docker buildx version
    ```
    
    Output:
    
    ```
    github.com/docker/buildx v0.4.1 6db68d029599c6710a32aa7adcba8e5a344795a7
    ```

## Set up buildx

1. Run a binfmt container.

    ```
    docker run --rm --privileged docker/binfmt:820fdd95a9972a5308930a2bdfb8573dd4447ad3
    ```

2. Create a builder.

    ```
    docker buildx create --name mybuilder
    docker buildx use mybuilder
    docker buildx inspect --bootstrap
    ```

    You have set up a cross-compiling environment and can now develop an arm32v7 container on your development PC.

    **Note**: When you reboot your development PC, run the following commands again to reenable the cross-compiling environment.

    ```
    docker run --rm --privileged docker/binfmt:820fdd95a9972a5308930a2bdfb8573dd4447ad3
    docker buildx inspect --bootstrap 
    ```

# Step 4: Build and Deploy a Simulated Temperature Sensor Module from VS Code

You are ready to use VS Code to build an IoT Edge module and deploy it to the DE10-Nano. In the previous tutorial, [Run Azure IoT Edge on DE10-Nano Development Kit][LINK_Run_Azure_IoT_Edge_on_DE10-Nano_Development_Kit], you deployed a simulated temperature sensor from the Azure Marketplace. In this step, you build the sensor from source code. 
  
**Note**: See [Tutorial: Develop a C IoT Edge module for Linux devices][LINK_Azure_VSCode_cmodule] for Microsoft's tutorial on how to deploy a sample module from VS Code.  

## Create a Solution Template

In this step, you create a solution template in C that you can customize with your own code.


1. Open a command palette in VS Code (Ctrl + Shift + P).  

2. Type and run the command **Azure IoT Edge: New IoT Edge solution**. 

    ![Azure-Template-Creation](picture/az-edge-template-00.png)

3. Select a folder on your development PC for VS Code to create the solution file. 

4. Name the solution. For example, **EdgeSolution**.  

    ![Azure-Template-Creation](picture/az-edge-template-01.png)

4. Choose **C module**.  

    ![Azure-Template-Creation](picture/az-edge-template-02.png)

5. Name your module. For example, **SampleModule**.  

    ![Azure-Template-Creation](picture/az-edge-template-03.png)

6. Provide your Docker image repository.
   
    Replace **localhost:5000** with the login server value of your ACR. You can find the login server from the Overview page of your container registry in the Azure portal. In this tutorial, the login server is **de10nano.azurecr.io**.  

    Your container image name is prepopulated from the name provided in the last step.
        
    ```
    <your login server>/<container image name>
    ```

    Example:  

    ```
    de10nano.azurecr.io/samplemodule
    ```

    ![Azure-Template-Creation](picture/az-edge-template-04.png)

    After a few minutes, VS Code will generate the solution template.  

## Add ACR Credentials to the Env File

1. From the VS Code explorer, open the .env file. In this example, the .env file is located under `edge-module/EdgeSolution/.env`.

2. Update the fields with the username and password for your ACR. You can find these values by navigating to **Settings** > **Access Keys** of your container registry in the
   Azure portal.

    ![Setup-Env](picture/az-acr-template-env-00.png)

    ```
    CONTAINER_REGISTRY_USERNAME_de10nano=<Your ACR Username>
    CONTAINER_REGISTRY_PASSWORD_de10nano=<Your ACR Password>
    ```

    Example:  

    ```
    CONTAINER_REGISTRY_USERNAME_de10nano=de10nano
    CONTAINER_REGISTRY_PASSWORD_de10nano=ydmavLBUw8KdKx/fsWYKD6rbHaVW4Cl6
    ```
    
3. Save your changes to this file.  

   **Note**: When you build a container image, the env file becomes the credential for accessing your ACR.

## Change your Target Platform

1. From VS Code, select the shortcut icon in the side bar at the bottom of the window.

2. Change the default platform to **arm32v7**.
   
    ![VSCode-Change-Default-Platform](picture/vscode-change-env-00.png)

## Log in to your ACR

1. Open a terminal (Ctrl + `) in VS Code.

2. From the terminal, log in to your ACR.

    ```
    sudo su

    docker login -u <your ACR username> --password <your ACR password> <your ACR server name>
    ```

    Below is an example; please provide your credentials.

    Input:

    ```
    docker login -u de10nano --password ydmavLBUw8KdKx/fsWYKD6rbHaVW4Cl6 de10nano.azurecr.io
    ```

    Output:

    ```
    ...
    Login Succeeded
    ```

## Build and Push a Sample Module to ACR

Before continuing, make sure you have done the following:
- Logged in to your ACR
- Opened a terminal in VS Code using sudo
- Enabled a cross-compiling environment, see instructions for [setting up buildx](#set-up-buildx)

1. Open VS Code explorer.

2. Right-click on the `deployment.template.json` file and select **Build and Push IoT Edge Solution**.

    ![VSCode-Build-And-Push-SampleModue](picture/vscode-build-container-00.png)

    The build and push command below is executed. This command builds the sample code, creates a container image, and pushes that image to your ACR.  

    ```
    docker build  --rm -f "<your folder place>/edge-module/EdgeSolution/modules/SampleModule/Dockerfile.arm32v7" -t de10nano.azurecr.io/samplemodule:0.0.1-arm32v7 "<your folder place>/edge-module/EdgeSolution/modules/SampleModule" && docker push de10nano.azurecr.io/samplemodule:0.0.1-arm32v7
    ```

    After the command completes, you will see the following output and can view the sample module in your ACR.
    ```
    ...
    ...
    ...
    Successfully built 617d12c76038
    Successfully tagged de10nanoej.azurecr.io/samplemodule:0.0.1-arm32v7
    The push refers to repository [de10nanoej.azurecr.io/samplemodule]
    c0cb4f87568a: Pushed 
    00bd9c2aaf2a: Pushed 
    1aba629ef948: Pushed 
    b934afb3d8bc: Pushed 
    d8e1ec74a40d: Pushed 
    0.0.1-arm32v7: digest: sha256:7b5b18c53c0bef088ed4c855b14e0b4197ebe563d18b8a567212c7251cbdd0af size: 1366
    ```

    ![Azure-ACR-See-SampleModule](picture/vscode-build-container-01.png)


## Create a Deployment Manifest

In this step, you create a JSON deployment manifest, and then use that file to push the deployment to the DE10-Nano. 

1. Open VS Code explorer.  

2. Right-click on the `deployment.template.json` file and select **Generate IoT Edge Deployment Manifest**.  

    ![VSCode-Create-Manifest](picture/vscode-create-manifest-00.png)

    In the `EdgeSolution/config` folder, a deployment.arm32v7.json file is generated.  

    **Note**:  See [Deploy Azure IoT Edge modules from Visual Studio Code][LINK_Azure_deploy_module_from_VSCode] for Microsoft's tutorial on how to deploy a container module from VS Code. 

3. From the `config` folder, right-click on `deployment.arm32v7.json` and select  **Create Deployment for Single Device**.

    ![VSCode-Deploy-Manifest](picture/vscode-create-manifest-01.png)  

4. Select your device.  

    ![VSCode-Deploy-Manifest](picture/vscode-create-manifest-02.png)

    From the terminal, you can see that the deployment succeeded. Both the SampleModule and SimulatedTemeratureSensor module are deployed to the DE10-Nano.  

    ![VSCode-Deploy-Manifest](picture/vscode-create-manifest-03.png)

    **Note**:  From a DE10-Nano console, you can use the `iotedge list` or `docker ps` command to check that the modules are running .
      
    **Note**:  May take up to 5 minutes for new module to be running and show up on `iotedge list` command .
    
## Next Steps

Congratulations! You have completed this tutorial. To continue to the next tutorial in this series, go to [Create a Container Application that uses the DE10-Nano G-Sensor][LINK_module_03]. You can delete the resource group unless you continue the next tutorial. It will delete all Azure services you associated with it. 


[LINK_Azure_cost]: https://docs.microsoft.com/en-us/azure/billing/billing-understand-your-bill

[LINK_Terasic_DE10_NANO_User_Manual]: https://www.terasic.com.tw/cgi-bin/page/archive_download.pl?Language=English&No=1046&FID=f1f656bb5f040121c36f2f93f6b107ff

[LINK_Terasic_getting_started_guide]: https://www.terasic.com.tw/attachment/archive/1046/Getting_Started_Guide.pdf

[LINK_Azure_Device_login]: https://microsoft.com/devicelogin

[LINK_Terasic_DE10-Nano-Purchase]: https://de10-nano.terasic.com/

[LINK_Terasic_bootable]: http://download.terasic.com/downloads/cd-rom/de10-nano/DE10-Nano-Cloud-Native.xz

[Link_VSCode_Setup]: https://code.visualstudio.com/docs/setup/linux

[LINK_Azure_ACR_getting_started]: https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal

[LINK_Docker_Installation]: https://docs.docker.com/engine/install/

[LINK_Azure_VSCode_cmodule]: https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-c-module

[LINK_Buildx_release]: https://github.com/docker/buildx/releases/latest

[LINK_Azure_deploy_module_from_VSCode]: https://docs.microsoft.com/en-us/azure/iot-edge/how-to-deploy-modules-vscode

<!---
###########################################################  
Fix following Link, after delete here  
###########################################################  
--->

<!---
[LINK_module_01]: https://TODO.link.to.module1
[LINK_module_03]: https://TODO.link.to.module3
--->
