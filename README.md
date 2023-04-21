<!---
Purpose :

Develop a container which uses unique device on DE10-Nano.
It's pretty much practical than implementing the SimulatedTemperatureSensor container which we created in previous tutorial.

Bullet Points for What to Do:
-  Download Sample codes, g-sensor from Terasic, Azure IoT Edge sample codes from Microsoft
-  Create the Development Container on DE10-Nano
-  Setup VSCode for Remote File Access
-  Create g-sensor container, test and push 

#################################################################
TODO

Proofread
Link check

#################################################################
--->

# Create a Container Application that uses the DE10-Nano G-Sensor

This is part 3 of a 4-part tutorial series that shows you how to manage the DE10-Nano with Azure IoT Edge and use container-based virtualization to reprogram the onboard FPGA from the Azure Cloud.

## Table of Contents
- [About this Tutorial](#about-this-tutorial)
  - [Objectives](#objectives)
  - [Prerequisites](#prerequisites)
- [Step 1: Before you Begin](#step-1-before-you-begin)
- [Step 2: Download Source Code from Terasic and GitHub](#step-2-download-source-code-from-terasic-and-github)
  - [Determine the DE10-Nano Board Revision](#determine-the-de10-nano-board-revision)
  - [Download the CD-ROM for your Board Revision](#download-the-cd-rom-for-your-board-revision)
- [Step 3: Compile and Test the G-sensor Executable](#step-3-compile-and-test-the-g-sensor-executable)
  - [Send the G-Sensor Code to your DE10-Nano](#send-the-g-sensor-code-to-your-de10-nano)
  - [Patch the G-sensor Code](#patch-the-g-sensor-code)
  - [Run the G-Sensor Executable](#run-the-g-sensor-executable)
- [Step 4: Download the Azure IoT Device SDK for C](#step-4-download-the-azure-iot-device-sdk-for-c)
- [Step 5: Create a Development Container](#step-5-create-a-development-container)
  - [Build a Development Container with buildx](#build-a-development-container-with-buildx)
  - [Send the Container Image to the DE10-Nano](#send-the-container-image-to-the-de10-nano)
- [Step 6: Set up VS Code for Remote File Access](#step-6-set-up-vs-code-for-remote-file-access)
  - [Connect to the DE10-Nano](#connect-to-the-de10-nano)
  - [Send a Public Key to the DE10-Nano (Optional)](#send-a-public-key-to-the-de10-nano-optional)
- [Step 7: Patch the Source Code](#step-7-patch-the-source-code)
  - [Copy Source Code to a Workspace](#copy-source-code-to-a-workspace)
  - [Apply a Patch to your Workspace](#apply-a-patch-to-your-workspace)
  - [Changes Made by the Patch File](#changes-made-by-the-patch-file)
- [Step 8: Create and Test the G-sensor Executable](#step-8-create-and-test-the-g-sensor-executable)
- [Step 9: Build and Push the G-sensor Module to Azure Container Registries](#step-9-build-and-push-the-g-sensor-module-to-azure-container-registries)
  - [Update the API](#update-the-api)
  - [Build the Image](#build-the-image)
  - [Set up buildx](#set-up-buildx)
  - [Build and Push the Module to ACR](#build-and-push-the-module-to-acr)
  - [Deploy the Module to DE10-Nano](#deploy-the-module-to-de10-nano)

## About this tutorial

This tutorial provides instructions for creating an Azure IoT Edge container application that sends accelerometer data from the DE10-Nano G-Sensor to the Azure Cloud.

<!-- which step do we "develop an application which uses Azure IoT Edge API"?  And testing the application on the development container? --> 


### Objectives

In this tutorial, you will learn how to:

* Gather and patch G-Sensor and IoT Edge sample source code
* Build and push your IoT Edge Module to ACR
* Deploy your IoT Edge Module to the DE10-Nano

### Prerequisites

* Complete the previous tutorial, [Create a Container Using Visual Studio Code and Deploy it to the DE10-Nano][LINK_module_02].
* An Azure IoT hub 
* An IoT Edge device (DE10-Nano) with IoT Edge runtime installed
* An Azure Container Registry (ACR)

## Step 1: Before you Begin

The DE10-Nano board has a built-in 3-axis accelerometer, known as the G-sensor. Before you begin, test out the G-sensor on the DE10-Nano.

1. Open a console on DE10-Nano.

2. Input the following command:

    Input: 
    ```
    /root/gsensor
    ```
    
    Output:
    
    ```
    [1]X=32 mg, Y=-40 mg, Z=916 mg
    [2]X=20 mg, Y=-40 mg, Z=968 mg
    ```

    Move the board to change the X, Y, Z accelerometer data. This g-sensor executable reads raw data from the sensor via I2C, processes that data, and then displays it as X, Y, and Z-axis values. 

<!-- How is this G-sensor different from the one in Step #3? -->

## Step 2: Download Source Code from Terasic and GitHub

You will develop a container application that uses the DE10-Nano G-sensor by leveraging source code provided by Terasic.

1. Open the [DE10-Nano Kit resource page][LINK_Terasic_resource] from Terasic's site and navigate to **Resources**.  

    ![TerasicResource](picture/gsensor-terasic-00.png)

### Determine the DE10-Nano Board Revision

The DE10-Nano has different board revisions, and you need to download content according to your board's revision.  

1. From the **Resources** page, navigate to **Documents** and open the *How to distinguish the board revision and what's different* pdf.  

   ![TerasicRevision](picture/gsensor-terasic-01.png)
   
2. Read the pdf to determine your board's revision. 

### Download the CD-ROM for your Board Revision

1. From the **Resources** page, navigate to **CD-ROM**, and download the CD-ROM for your board's revision.
  
    In this example, the DE10-Nano board revision is **B**, and **DE10-Nano CD-ROM (rev. A/B Hardware)** is the appropriate download.  

    ![TerasicCDROM](picture/gsensor-terasic-02.png)

2. Open a console on your development PC.  

3. Unzip the CD-ROM.

    ```
    cd ~/Downloads
    mkdir de10nano
    unzip DE10-Nano_v.1.2.4_HWrevAB_SystemCD.zip -d de10nano/
    ```

4. Verify the contents in `de10nano`.  

    ```
    ls de10nano/

    Datasheet  Demonstrations  Manual  Schematic  Tools  Verify.md5  Verify.sfv
    ```

    The G-sensor code is located in the `Demonstrations` folder.

### Get Source Codes from GitHub

Open a terminal to get source codes. 

```
sudo apt install git 
cd ~/Download
git clone https://github.com/intel-iot-devkit/terasic-de10-nano-kit
```
This repository has not only files for these tutorials(Module 3 and Module 4) but other tutorial files.


## Step 3: Compile and Test the G-sensor Executable

Before creating a container application that uses the G-sensor, you need to compile and test the G-sensor executable. If you cannot run the executable, you cannot create a container that performs the same function.  


### Send the G-Sensor Code to your DE10-Nano

1. Open a console on your DE10-Nano and find the DE10-Nano IP address.

    Input: 

    ```
    ip addr
    ```

    Output:

    ```
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
        link/ether f2:36:44:ac:08:7b brd ff:ff:ff:ff:ff:ff
        inet 192.168.100.145/24 brd 192.168.100.255 scope global dynamic eth0
          valid_lft 600184sec preferred_lft 600184sec
    ```

    In this example, the address is **192.168.100.145**.

2. Open a console on your development PC and send the folder that contains the G-sensor executable via scp.

    Input:

    ```
    cd ~/Downloads/de10nano/Demonstrations/SoC
    scp -r hps_gsensor root@<DE10-Nano IP address>:/root/Downloads
    ```

    Output:

    ```
    gsensor                          100%   11KB  11.3KB/s   00:00
    ADXL345.h                        100% 3551     3.5KB/s   00:00
    Makefile                         100%  546     0.5KB/s   00:00
    ADXL345.c                        100% 2106     2.1KB/s   00:00
    main.c                           100% 3121     3.1KB/s   00:00  
    ```

### Patch the G-sensor Code

The G-sensor code has unnecessary dependencies, and you need to patch the source code before compiling.  

1. Send the patch file to the DE10-Nano.

    ```
    scp ~/Downloads/terasic-de10-nano-kit/azure-de10nano-document/module03-gsensor-deploy-guide/hps_gsensor.patch root@<DE10-Nano IP address>:~/Downloads/hps_gsensor
    ```

2. Open a console from DE10-Nano and patch the code.

    ```
    cd ~/Downloads/hps_gsensor
    patch < hps_gsensor.patch
    ```

### Run the G-Sensor Executable

1. Generate the executable.

    ```
    make
    ```

   You should see a G-sensor executable in the `hps_gsensor` folder. 

2. Run the executable.

   Input:
   
    ```
    ./gsensor
    ```
    
    Ouput:
    
    ```
    ===== gsensor test =====
    id=E5h
    [1]X=20 mg, Y=-16 mg, Z=964 mg
    [2]X=20 mg, Y=-16 mg, Z=972 mg
    ```

3. Clean the files (optional).

    ```
    make clean
    rm hps_gsensor.patch
    ```

## Step 4: Download the Azure IoT Device SDK for C

The Microsoft Azure IoT Device SDKs contain code that help you build applications that connect to Azure IoT Hub services. For details, see [Understand and use Azure IoT Hub SDKs][LINK_Azure_IoT_Edge_Sample_Module].  
In this step, you prepare sample codes for using the IoT Edge API.

1. Open a console on the DE10-Nano and use git to download the Azure IoT Device SDK for C.

    ```
    cd ~/Downloads
    git clone https://github.com/Azure/azure-iot-sdk-c.git
    ```

    **Note**: 
    We verified the below commit and you may need to checkout it.  
    *Commit ID:86c0884e9a0c4332aafba70ca201384495915d7f*
    ```
    git checkout 86c0884e9a0c4332aafba70ca201384495915d7f
    ```

    **Note**: This example uses the SDK for the C language but Microsoft provides SDKs for other languages. For details, see [Azure IoT Hub Device SDK][LINK_Azure_IoT_Device_SDK_another_language].  

2. View the contents.  

    Input:

    ```
    cd azure-iot-sdk-c
    ls
    ```

    Output:
    
    ```
    build               iothub_client                SECURITY.MD
    build_all           iothub_service_client        serializer
    certs               jenkins                      testtools
    CMakeLists.txt      LICENSE                      thirdpartynotice.txt
    configs             lts_branches.png             tools
    c-utility           provisioning_client          uamqp
    dependencies.cmake  provisioning_service_client  umqtt
    deps                readme.md                    version.txt
    doc                 samples
    ```

3. Navigate to the `samples` folder under `iothub_client`.  
   
   This folder contains sample code for messaging IoT Edge modules or IoT Device applications to Azure IoT Hub.  

    ```
    cd iothub_client/samples
    ```

4. View the contents.

    Input:

    ```
    ls
    ```

    Output:

    ```
    CMakeLists.txt
    ios
    iotedge_downstream_device_sample
    iothub_client_device_twin_and_methods_sample
    iothub_client_sample_amqp_shared_methods
    iothub_client_sample_module_filter
    iothub_client_sample_module_method_invoke
    iothub_client_sample_module_sender
    iothub_client_sample_mqtt_dm
    iothub_client_sample_mqtt_esp8266
    iothub_client_sample_upload_to_blob
    iothub_client_sample_upload_to_blob_mb
    iothub_convenience_sample
    iothub_ll_c2d_sample
    iothub_ll_client_sample_amqp_shared
    iothub_ll_client_shared_sample
    iothub_ll_client_x509_sample
    iothub_ll_telemetry_sample
    readme.md
    ```

    This tutorial uses sample code from **iothub_client_sample_module_sender**.

## Step 5: Create a Development Container

 Here, you use buildx to create a development container on your development PC and then send the container image to the DE10-Nano.
 
 **Note**: You can immediately build the container on the DE10-Nano but this takes a lot of time.

### Build a Development Container with buildx

1. Open a console on your Development PC.  

   **Note**: Make sure your cross-compiling environment is enabled before completing this step. To enable cross-compiling, run the binfmt and inspect commands.

    ```
    sudo su
    docker run --privileged docker/binfmt:820fdd95a9972a5308930a2bdfb8573dd4447ad3
    docker buildx inspect --bootstrap
    ```

2. Navigate to Downloads and edit `Dockerfile.arm32v7`.  

    ```
    cd ~/Downloads
    vim Dockerfile.arm32v7
    ```

3. Copy the code below into `Dockerfile.arm32v7`.  

    ```
    FROM arm32v7/ubuntu:xenial AS base
    RUN apt-get update && \
        apt-get install -y --no-install-recommends software-properties-common && \
        add-apt-repository -y ppa:aziotsdklinux/ppa-azureiot && \
        apt-get update && \
        apt-get install -y azure-iot-sdk-c-dev && \
        rm -rf /var/lib/apt/lists/*

    FROM base AS build-env
    RUN apt-get update && \
        apt-get install -y --no-install-recommends cmake gcc g++ make && \
        rm -rf /var/lib/apt/lists/*
    WORKDIR /app
    ```

4. Build the development container with buildx.  

    ```
    sudo su
    docker build --rm -f $PWD/Dockerfile.arm32v7 -t de10nano/iotedgedev:arm32v7 $PWD
    ```
    
    Build output:
    ```
    Sending build context to Docker daemon  23.55kB

    Step 1/5 : FROM arm32v7/ubuntu:xenial AS base
    ---> f46cdaad2749
    Step 2/5 : RUN apt-get update &&   apt-get install -y --no-install-recommends software-properties-common &&   add-apt-repository -y ppa:aziotsdklinux/ppa-azureiot &&   apt-get update &&   apt-get install -y azure-iot-sdk-c-dev &&   rm -rf /var/lib/apt/lists/*
    ...
    ...
    Step 3/5 : FROM base AS build-env
    ---> ebcff6fd852a
    Step 4/5 : RUN apt-get update &&   apt-get install -y --no-install-recommends cmake gcc g++ make &&   rm -rf /var/lib/apt/lists/*
    Step 5/5 : WORKDIR /app
    ---> Running in cde1ca5c79e1
    Removing intermediate container cde1ca5c79e1
    ---> 7b77b7910054
    Successfully built 7b77b7910054
    Successfully tagged de10nano/iotedgedev:arm32v7
    ```

### Send the Container Image to the DE10-Nano

1. Save the container image as a tar file on your development PC.

    ```
    docker save de10nano/iotedgedev:arm32v7 -o de10nano-container.tar
    ```

2. Send the tar file to DE10-Nano via scp.

    ```
    scp de10nano-container.tar root@<DE10-Nano IP address>:~/Downloads/
    ```

    **Note**: You may need to change the file permissions of the tar file to send it to the DE10-Nano.

<!-- will other users have the same permission issue with sending the file to the de10-nano? -->

3. Delete the files on your development PC (optional).

    ```
    rm de10nano-container.tar 
    docker rmi de10nano/iotedgedev:arm32v7
    rm ~/Downloads/Dockerfile.arm32v7
    ```

4. Open a console on the DE10-Nano.  

5. Load the image.  

    ```
    cd ~/Downloads
    docker load -i ~/Downloads/de10nano-container.tar 
    ```

    Loading the image takes a few minutes. When the operation completes, use `docker images` to see the container image.

    Output:
    ```
    210d4e9a7717: Loading layer    110MB/110MB
    cce307bcd8be: Loading layer  15.87kB/15.87kB
    a7eb8edada8b: Loading layer  11.78kB/11.78kB
    d9837397634c: Loading layer  3.072kB/3.072kB
    79e261a9d107: Loading layer  92.38MB/92.38MB
    26385008c393: Loading layer  117.2MB/117.2MB
    286f79fd2b0e: Loading layer  2.048kB/2.048kB
    Loaded image: de10nano/iotedgedev:arm32v7

    ```
    
    Input:

    ```
    docker images
    ```

    Output:
    
    ```
    de10nano/iotedgedev arm32v7 7acdd7debb6b    2 days ago  298MB
    ```

<!-- image name: de10nano/iotedgedev:arm32v7 -->

6. You can remove the tar file from the `Downloads` folder on the DE10-Nano (optional).

    ```
    rm de10nano-container.tar
    ```

## Step 6: Set up VS Code for Remote File Access

Here, you will set up VS Code for remote file access, enabling you to use your development PC to develop code on the DE10-Nano. It is easier to modify code via VS Code rather than using vim on DE10-Nano.  

**Note**: See [Remote Development using SSH][LINK_VSCode_remote_SSH] for Microsoft's tutorial on how to use the remote SSH extension.

To complete this step, the DE10-Nano and development PC must be on the same network. Also, make sure you can SSH into the DE10-Nano from your development PC. For instructions on how to set up an SSH connection, see [Run Azure IoT Edge on the DE10-Nano][LINK_module_01].

1. Open a command palette (Ctrl+Shift+P) in VS Code.  

2. To install the [Remote Development extension pack][LINK_VSCode_Remote_Extension], type and enter the following:

    ```
    ext install ms-vscode-remote.vscode-remote-extensionpack
    ```

    ![VSCodeSSHExtensionInstallaion](picture/gsensor-vscode-extension-01.png)

### Connect to the DE10-Nano

1. Open a command palette (Ctrl+Shift+P) in VS Code. 

2. Type and enter the command **Remote-SSH: Connect to Host...**.  

    ![VSCodeRemoteSSH](picture/gsensor-vscode-remote-ssh-00.png)

3. Put the **\<username\>@\<ip address\>** of your DE10-Nano.  

    This example uses `root@192.168.100.145`.  

    **Note**: If you set a static IP for DE10-Nano in [the previous tutorial][LINK_module_01], use it here.

    ![VSCodeRemoteSSH](picture/gsensor-vscode-remote-ssh-01.png)

4. Enter your root password.

    ![VSCodeRemoteSSH](picture/gsensor-vscode-remote-ssh-02.png)

5. Open a **gsensor-module** folder by clicking **Open Folder** and selecting the path `/home/root/Downloads/gsensor-module`.  

    ![VSCodeRemoteSSH](picture/gsensor-vscode-remote-ssh-03.png)

  **Note**: The gsensor-module folder refers to the folder that you SCP in [Step 3: Compile and Test the G-sensor Executable](#step-3-compile-and-test-the-g-sensor-executable).
  
### Send a Public Key to the DE10-Nano (Optional) 

1. Open a console on your development PC and type:

    ```
    ssh-copy-id root@<de10-nano IP address>
    ```

## Step 7: Patch the Source Code

### Copy Source Code to a Workspace

1. Open a console on your DE10-Nano.

2. Create a workspace.

    ```
    mkdir ~/Downloads/gsensor-module
    ```

3. Copy files in `iothub_client_sample_module_sender` to your workspace.

    ```
    cp ~/Downloads/azure-iot-sdk-c/iothub_client/samples/iothub_client_sample_module_sender/* ~/Downloads/gsensor-module/
    ```

4. Copy files in `hps_gsensor` too.

    ```
    cp ~/Downloads/hps_gsensor/* ~/Downloads/gsensor-module/
    ```

   The G-sensor code and IoT Edge sample code are now located in the `gsensor-module` folder.

    Input:

    ```
    cd ~/Downloads/gsensor-module
    ls
    ```

    Output:

    ```
    ADXL345.c  
    ADXL345.h  
    CMakeLists.txt  
    iothub_client_sample_module_sender.c  
    iothub_client_sample_module_sender.h  
    main.c  
    Makefile
    ```

    You just gathered Iot Edge sample code from Microsoft and G-sensor code from Terasic. You will use these codes to develop an Azure IoT Edge container application.

    | IoT Edge sample code                 | G-sensor code |
    |--------------------------------------|---------------|
    | iothub_client_sample_module_sender.c | ADXL345.c     |
    | iothub_client_sample_module_sender.h | ADXL345.h     |
    | CMakeLists.txt                       | main.c        |  
    |                                      | Makefile      |   

### Apply a Patch to your Workspace

Apply the `gsensor-module.patch` to your workspace folder.

1. Send the patch to DE10-Nano by scp.

    ```
    scp ~/Downloads/terasic-de10-nano-kit/azure-de10nano-document/module03-gsensor-deploy-guide/gsensor-module.patch root@<DE10-Nano IP address>:/root/Downloads/gsensor-module
    ```

2. Open VS Code and connect DE10-Nano by a remote SSH extension.  

3. Open a console on DE10-Nano.

    ```
    cd ~/Downloads/gsensor-module
    patch < gsensor-module.patch
    ```

    After patching the code, you need to set your IoT Edge connection string in `iothub_client_sample_module_sender.c`.

    Patch Output:
    ```
    patching file CMakeLists.txt
    patching file iothub_client_sample_module_sender.c
    ```

5. Open `iothub_client_sample_module_sender.c` and replace `<Your IoT Edge Connection String>` with your connection string.

    **Before**:

    ```
    static const char* connectionString = "<Your IoT Edge Connection String>";
    ```

    **After**:

    ```
    static const char* connectionString = "HostName=de10nano-iothub.azure-devices.net;DeviceId=de10-nano-iotedge;SharedAccessKey=rPiy9a15CM4WQ54EAwXq6/XQ07diE0zUi0NXTCBmuic=";
    ```

### Changes Made by the Patch File

The patch makes three major changes.

1. Integrates G-sensor and IoT Edge API sample codes
   
   The patch first integrates `main.c` (G-sensor code) into `iothub_client_sample_module_sender.c`. The key processes of `main.c`: open I2C for accessing the G-sensor, initialize the G-sensor, and read the G-sensor and convert to standard output. To read the G-sensor data, these steps must be ported to the `iothub_client_sample_module_sender.c`.
 
2. Modifies the Azure IoT Edge API function
   
   The patch changes Azure IoT Edge API in `iothub_client_sample_module_sender.c`. Specifically, it changes `IoTHubModuleClient_LL_CreateFromEnvironment(MQTT_Protocol)` function to `IoTHubModuleClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol)` with your IoT Edge connection string.

   **Before**:

    ```
    else if ((iotHubModuleClientHandle = IoTHubModuleClient_LL_CreateFromEnvironment(MQTT_Protocol)) == NULL)
    ```

   **After**:

    ```
    else if ((iotHubModuleClientHandle = IoTHubModuleClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol)) == NULL)
    ```

   **Note**: `CreateFromConnectionString` is used because `CreateFromEnvironment` cannot be easily used in the development container. Specifically, the `CreateFromEnvironment` function uses environment variables which are generated when the IoT Edge Runtime deploys the container from Azure cloud.  If we use the API in the development container, an error will occur because the environment variables that are supposed to be provided by IoT Edge Runtime don't exist. Thus, the patch replaces it with the `CreateFromConnectionString` API. Because this connection string is unique to a certain IoT Edge, it is not portable. After testing and debugging, it is necessary to return to the original `CreateFromEnvironment` API. 

3. Updates CMakeLists.txt
   
   `CMakeLists.txt`, originally in the `iothub_client_sample_module_sender` folder is supposed to be built in a higher folder as
   the [Azure IoT SDK for C Github][LINK_IoT_SDK_compile] page describes. Therefore, you will get an error if you build the 
   `CMakeLists.txt` only due to lacking some library dependencies. In "SampleModule" that you created in previous tutorial, 
   there is CMakeLists.txt which works fine, we used the most of it.  

<!-- improve wording. confusing -->

## Step 8: Create and Test the G-sensor Executable

In this step, you create a G-sensor binary using your development container.  

1. Open VS Code and connect to the DE10-Nano.  

2. Open a DE10-Nano terminal in VS Code and navigate to the `gsensor-module` folder. 

    ```
    cd ~/Downloads/gsensor-module
    ```

3. Run the development container.

    ```
    docker run -it -v $PWD:/app de10nano/iotedgedev:arm32v7
    ```

4. Create the G-sensor executable.  

    ```
    cmake .
    make
    ```

    **Note**: The `cmake` and `make` commands work because IoT Edge API packages are installed in the development container. If you try to generate the executable without running the container, an error will occur.  

    cmake output:

    ```
    -- The C compiler identification is GNU 5.4.0
    -- The CXX compiler identification is GNU 5.4.0
    -- Check for working C compiler: /usr/bin/cc
    -- Check for working C compiler: /usr/bin/cc -- works
    -- Detecting C compiler ABI info
    -- Detecting C compiler ABI info - done
    -- Detecting C compile features
    -- Detecting C compile features - done
    -- Check for working CXX comipler: /usr/bin/c++
    -- Check for working CXX comipler: /usr/bin/c++ -- works
    -- Detecting CXX compiler ABI info
    -- Detecting CXX compiler ABI info - done
    -- Detecting C compile features
    -- Detecting C compile features - done
    -- Configuring done
    -- Generating done
    -- Build files have been written to: /app
    ```
    make output:

    ![VSCodeRemoteSSH](picture/gsensor-vscode-remote-ssh-06.png)

5. Exit the container.

    ```
    exit
    ```

6. Test the executable.  

    Input:

    ```
    ./iothub_client_sample_module_sender
    ```
    **Note**: Run *sudo apt-get install libcurl3 -y* if error "version `CURL_OPENSSL_3' not found  occurs.
   
    Output:

    ```
    ===== gsensor test =====
    id=E5h
    [1]X=8 mg, Y=-88 mg, Z=964 mg
    IoTHubModuleClient_LL_SendEventAsync accepted message [0] for transmission to IoT Hub.
    [2]X=16 mg, Y=-96 mg, Z=956 mg
    IoTHubModuleClient_LL_SendEventAsync accepted message [1] for transmission to IoT Hub.
    ```

    You can see that the messages are correctly sent to Azure IoT Hub . In the previous tutorial, you used the Azure CLI to monitor messages. Here, you use VS Code to monitor messages from your IoT Hub.

7. Open a new window in VS Code.  
   
   **Note**: Make sure that this window does not connect to the DE10-Nano. If the window connects to DE10-Nano, the Azure IoT Hub tab will 
   not appear. 
    
8. Click the "..." button on the **Azure IoT Hub** tab and then click **Start Monitoring Built-in Event Endpoint**.

    ![VSCodeMonitor](picture/gsensor-vscode-monitor-00.png)

    You can see that the IoT Hub is receiving accelerometer data.

    ![VSCodeMonitor](picture/gsensor-vscode-monitor-01.png)

## Step 9: Build and Push the G-sensor Module to Azure Container Registries

### Update the API


1. From a DE10-Nano terminal in VS Code, open `iothub_client_sample_module_sender.c`.

    ```
    cd ~/Downloads/gsensor-module
    vim iothub_client_sample_module_sender.c
    ```

2. Comment out the connection string variable declaration.  

    **Before**:

    ```
    static const char* connectionString = "HostName=de10nano-iothub.azure-devices.net;DeviceId=de10-nano-iotedge;SharedAccessKey=rPiy9a15CM4WQ54EAwXq6/XQ07diE0zUi0NXTCBmuic=";
    ```

    **After**:

    ```
    //static const char* connectionString = "HostName=de10nano-iothub.azure-devices.net;DeviceId=de10-nano-iotedge;SharedAccessKey=rPiy9a15CM4WQ54EAwXq6/XQ07diE0zUi0NXTCBmuic=";
    ```

3. Update the API.

   You need to change the API from IoTHubModuleClient_LL_**CreateFromConnectionString()** to IoTHubModuleClient_LL_**CreateFromEnvironment()**.  

    **Before**:

    ```
    else if ((iotHubModuleClientHandle = IoTHubModuleClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol)) == NULL)
    ```

    **After**:

    ```
    else if ((iotHubModuleClientHandle = IoTHubModuleClient_LL_CreateFromEnvironment(MQTT_Protocol)) == NULL)
    ```

### Build the Image

1. From a VS Code explorer, navigate to your `modules` folder.  

2. Open a terminal in VS Code and copy the `gsensor-module` folder to `modules`.

    ```
    scp -r root@<DE10-Nano IP address>:/root/Downloads/gsensor-module <path to modules folder on your development PC>
    ```
    Example:
     
    ```
    scp -r root@<DE10-Nano IP address>:/root/Downloads/gsensor-module ~/Documents/EdgeSolution/modules
    ```

    **Note**: Make sure that the `gsensor-module` folder is located at the same level as `SampleModule`.

    ![VSCodeCopyGSensorToSampleModule](picture/gsensor-vscode-copy-folder-00.png)

3. Create `Dockerfile.arm32v7` in the `gsensor-module` folder.

    ```
    cd <your SampleModule Path>/EdgeSolution/modules/gsensor-module
    vim Dockerfile.arm32v7
    ```

4. Copy and paste the following code into `Dockerfile.arm32v7`. 

    ```
    FROM arm32v7/ubuntu:xenial AS base
    RUN apt-get update && \
        apt-get install -y --no-install-recommends software-properties-common && \
        add-apt-repository -y ppa:aziotsdklinux/ppa-azureiot && \
        apt-get update && \
        apt-get install -y azure-iot-sdk-c-dev && \
        rm -rf /var/lib/apt/lists/*

    FROM base AS build-env
    RUN apt-get update && \
        apt-get install -y --no-install-recommends cmake gcc g++ make && \
        rm -rf /var/lib/apt/lists/* 
    WORKDIR /app
    COPY . ./
    RUN cmake . 
    RUN make

    FROM base
    WORKDIR /app
    COPY --from=build-env /app ./
    CMD ["./iothub_client_sample_module_sender"]
    ```
<!-- copy and paste the following code into `Dockerfile.arm32v7`? Save the file. -->

6. Create `module.json` in the same folder.

    ```
    vim module.json
    ```

7. Copy and paste the following code into `module.json`.

    **Note**: Be sure to substitute the repository name with the name you used during set up. Besides the respository, you can keep everything else the same.
    
    ```
    {
      "$schema-version": "0.0.1",
      "description": "",
      "image": {
        "repository": "<your ACR repository address>/gsensor-module",
        "tag": {
          "version": "0.0.1",
          "platforms": {
            "arm32v7": "./Dockerfile.arm32v7"
          }
        },
        "buildOptions": []
      },
      "language": "c"
    }
    ```
    

8. Update `deployment.template.json` to build the G-sensor module.

    **Before**:

    ```
            "modules": {
              "SampleModule": {
                "version": "1.0",
                "type": "docker",
                "status": "running",
                "restartPolicy": "always",
                "settings": {
                  "image": "${MODULES.SampleModule}",
                  "createOptions": {}
                }
              },
              "SimulatedTemperatureSensor": {
                "version": "1.0",
                "type": "docker",
                "status": "running",
                "restartPolicy": "always",
                "settings": {
                  "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
                  "createOptions": {}
                }
              }
            }
    ```

    **After**:

    ```
            "modules": {
              "gsensor-module": {
                "version": "1.0",
                "type": "docker",
                "status": "running",
                "restartPolicy": "always",
                "settings": {
                  "image": "${MODULES.gsensor-module}",
                  "createOptions": {
                    "HostConfig": {
                      "Privileged": true
                    }
                  }
                }
              }
            }
    ```

9. Change message routes.

    **Before**:

    ```
        "$edgeHub": {
          "properties.desired": {
            "schemaVersion": "1.0",
            "routes": {
              "SampleModuleToIoTHub": "FROM /messages/modules/SampleModule/outputs/* INTO $upstream",
              "sensorToSampleModule": "FROM /messages/modules/SimulatedTemperatureSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/SampleModule/inputs/input1\")"
            },
            "storeAndForwardConfiguration": {
              "timeToLiveSecs": 7200
            }
          }
        }
    ```

    **After**:

    ```
        "$edgeHub": {
          "properties.desired": {
            "schemaVersion": "1.1",
            "routes": {
              "SampleModuleToIoTHub": "FROM /messages/modules/* INTO $upstream"
            },
            "storeAndForwardConfiguration": {
              "timeToLiveSecs": 7200
            }
          }
        }
    ```

### Set up buildx

Enable your cross-compiling environment to prepare for using buildx. 

Open a console on your development PC and type:

```
sudo su
docker run -it --privileged docker/binfmt:820fdd95a9972a5308930a2bdfb8573dd4447ad3 
docker buildx inspect --bootstrap
```

### Build and Push the Module to ACR

Right-click on `deployment.template.json` and then click **Build and Push IoT Edge Solution**.

   ![VSCodeBuildAndPush](picture/vscode-build-container-00.png)

### Deploy the Module to DE10-Nano

1. To generate a manifest, right-click on `deployment.template.json` and then click **Generate IoT Edge Deployment Manifest**.

   ![VSCodeBuildManifest](picture/vscode-create-manifest-00.png)


2. Navigate to the config folder, right-click on `deployment.arm32v7.json`, and then click **Create Deployment for Single  Device**

   ![VSCodeDeployManifest](picture/vscode-create-manifest-01.png)


  Output

  ![VSCodeDeployOutput](picture/vscode-deployment-output.png)

After a few minutes(potentially it will go up to 10minutes depending on the internet speed), the module will be deployed to the DE10-Nano and the G-sensor data will appear on the VS Code window when you click the "..." button on the **Azure IoT Hub** tab and then click **Start Monitoring Built-in Event Endpoint**.
  
  ![VSCodeAzureIotHub](picture/vscode-azureiothub.png) 

  ![VSCodeAzureIotHubMonitor](picture/vscode-azureiothub-monitor.png)


**Note**: If you cannot see any messages, you may have reached the maximum number of messages that can be sent per a day. Try redeploying the module. Open a console on your DE10-Nano and use the command `iotedge restart <module name>`.

## Next Steps

Congratulations! You have completed this tutorial. To continue to the next tutorial in this series, go to [Create a Container Application that Reconfigures the FPGA on the DE10-Nano][LINK_module_04]. You can delete the resource group unless you plan to continue the next tutorial. It will delete all Azure services you associated with it.


[LINK_Terasic_DE10-Nano-Purchase]: https://de10-nano.terasic.com/

[LINK_Terasic_DE10_NANO_User_Manual]: https://www.terasic.com.tw/cgi-bin/page/archive_download.pl?Language=English&No=1046&FID=f1f656bb5f040121c36f2f93f6b107ff

[LINK_Terasic_getting_started_guide]: https://www.terasic.com.tw/attachment/archive/1046/Getting_Started_Guide.pdf

[LINK_Terasic_resource]: de10-nano.terasic.com/cd

[LINK_VSCode_remote_SSH]: https://code.visualstudio.com/docs/remote/ssh

[LINK_VSCode_Remote_Extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack

[LINK_IoT_SDK_compile]: https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md#linux

[LINK_Azure_IoT_Edge_Sample_Module]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-sdks#azure-iot-device-sdks

[LINK_Azure_IoT_Device_SDK_another_language]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-sdks#azure-iot-device-sdks


<!--
###########################################################  
Fix following Link, after delete here  
###########################################################  

[LINK_module_01]: https://TODO.link.to.module1

[LINK_module_02]: https://TODO.link.to.module2

[LINK_module_04]: https://TODO.link.to.module4

-->

