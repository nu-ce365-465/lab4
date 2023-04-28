# Download and Install Intel (R) SoC Embedded Development Suite Software

- [Download Intel(R) SoC FPGA Embedded Development Suite](#download-intelr-soc-fpga-embedded-development-suite)
- [Install Intel(R) SoC FPGA Embedded Development Suite](#install-intelr-soc-fpga-embedded-development-suite)



## Download Intel(R) SoC FPGA Embedded Development Suite

Intel(R) SoC FPGA Embedded Development Suite is necessary for compiling DTS.  

1. Download [Intel(R) SoC FPGA Embedded Development Suite][LINK_SoCEDS_20.1_Standard_download].

2. Choose "Standard", "20.1", and "Linux". Download it.  

    ![image-20210316145224141](picture/soceds_download.png)

## Install Intel(R) SoC FPGA Embedded Development Suite

1. When the download completes, install the software.  

    ```
    cd ~/Downloads
    chmod +x SoCEDSSetup-20.1.0.711-linux.run 
    ./SoCEDSSetup-20.1.0.711-linux.run 
    ```

2. When a graphical installation appears, click **Next**.  

    ![image-20210316145816392](picture/soceds_20.1_install_01.png)

3. When prompted to provide a folder path, change the folder path to the folder where you installed Quartus Prime.  

    ```
    /home/<your_username>/intelFPGA/20.1
    ```
    
    Then, click **Next**.

    ![image-20210316150327338](picture/soceds_20.1_install_04.png)

4. When "Select Componets", you may un-check Programmer and Tolls installation to save your disk drive capacity.   Click **Next**.  

    ![image-20210316150621231](picture/soceds_20.1_install_02.png)

5. Once the software is installed, click **Finish**.  

    ![image-20210316154720491](picture/soceds_20.1_install_03.png)




[LINK_SoCEDS_20.1_Standard_download]: https://fpgasoftware.intel.com/soceds/20.1/?edition=pro&platform=linux&download_manager=direct