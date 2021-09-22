<!---
date : 9/1/2021
author : Daisuke Nakahara <daisuken@microsoft.com>
reviewer : Berry Tsai <betsai@microsoft.com>; Takehiro Hirai <takehiro.hirai@microsoft.com>
Maintainer : 
title : Azure IoT Sample Solution provisioning DE10-nano
--->

# Connecting DE10-Nano to your IoT Solution

In order to connect your DE10-Nano to Azure IoT Hub, you must provision your DE10-Nano.  Provisioning is done through Device Provisioning Service (DPS).
Once your DE10-Nano is provisioned, it can connect to IoT Hub.  Steps involved in connecting DE10-Nano are :

1. Flash SD Card with Ubuntu 18 image
1. Boot DE10-Nano
1. Connect DE10-Nano over UART or SSH
1. Clone Reference Application
1. Set up DE10-Nano for Python application
1. Create DPS Enrollment for DE10-Nano
1. Configure your DE10-Nano with DPS provisioning information
1. Run reference device application

Reference device application provides 2 versions of runtime.

- Python application
- Azure IoT Edge module (written in Python)

This document describes how to run Python application to provision to the sample IoT solution.

## 1. Flash SD Card with Ubuntu 18 Image

Download Ubuntu 18 disk image from <http://xxxx.com>

> [!NOTE]
> TBD Add link when image is available

## 2. Assemble DE10-Nano

Assemble DE10-Nano and connect cables.

- Ethernet cable for Internet connection
- Mini-USB to Type-A USB Cable for Serial Console
- HDMI cable (Optional)

![DE10-Nano](/images/DE10-Nano-Cabling.png)

## 3. Connect Serial Terminal

- Connect Mini-USB cable to your PC.  
- Install Serial Terminal such as Putty or Teraterm on your PC.
- Configure Serial Port with :

    | Setting      | Value              |
    |--------------|--------------------|
    | Port         | Serial Port Number |
    | Baud rate    | 115200             |
    | Data         | 8 bit              |
    | Parity       | None               |
    | Stop Bit     | 1 bit              |
    | Flow Control | None               |

If you do not have Mini-USB cable, use SSH to connect to DE10-Nano.

> [!TIP]  
> DE10-Nano does not have hardware MAC address.  As a result,
> IP Address may change on every boot.  You may configure DE10-Nano with
> static IP.

## 4. Clone reference application

Clone reference application with :

```bash
git clone https://github.com/intel-iot-devkit/terasic-de10-nano-kit.git
```

## 5. Set up DE10-Nano for Python application

Install Python 3.7 (or above) and libraries with :

```bash
cd ~/terasic-de10-nano-kit/<TBD Path> && \
apt update && \
apt install -y python3.7 python3-pip && \
python3.7 -m pip install -r ../requirements.txt
```

## 6. Create DPS Enrollment

An [enrollment](https://docs.microsoft.com/azure/iot-dps/concepts-service#enrollment) is a way to register your DE10-Nano to your IoT solution.  With DPS, your DE10-Nano can auto-provision to IoT Hub.

1. Navigate to your InnovateFPGA Azure Demo web site
1. Click `Device Management` menu in the navigation bar  
1. Click `Device Provisioning (DPS)`

    ![DPS01](/images/DPS-01.png)

1. Click `Add New Enrollment` to expand menu

    ![DPS02](/images/DPS-02.png)

1. Provide enrollment information  

|  | Setting         | Description                                                                                                                                         | Example     |
|--|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
|  | Enrollment Name | Name of new enrollment.  This name becomes Device ID in IoT Hub.                                                                                    | DE10-Nano   |
|  | Enrollment Type | Enrollment can be individual or group.  Group enrollment is used when you are provisioning more than 1 devices to IoT solution.  Select individual. | Individual. |
|  | Capability      | If you are running Azure IoT Edge module, enable IoT Edge.  In this instruction, we will use Python App.                                            |             |

    ![DPS03](/images/DPS-03.png)

1. Click `Create` to create a new individual enrollment

    You should see a new enrollment in the list box above.
  
    ![DPS04](/images/DPS-04.png)

## 7. Configure your DE10-Nano with DPS provisioning information

In order to provision your DE10-Nano through DPS, the device application requires following information.  

| Information     | Description                                                                                                                                                                                                                                        | Example     |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| Registration ID | Name of the enrollment.  By default, Registration ID is used as Device ID in IoT Hub.                                                                                                                                                              | DE10-Nano   |
| ID Scope        | ID Scope is an identifier of your DPS instance.  With ID Scope, device application can communicate to your DPS instance.                                                                                                                           | 0ne00123456 |
| Symmetric Key   | DPS support 3 authentication method, or [Attestation Mechanisms](https://docs.microsoft.com/azure/iot-dps/concepts-service#attestation-mechanism).  1) Symmetric Key, 2) X.509, and 3) TPM.  This IoT solution is configured to use Symmetric Key. |             |

You can pass these information through environment variables to Intel's reference device application.

Click `Copy` buttons for each item and set environment variables.

Example :

```bash
export IOTHUB_DEVICE_SECURITY_TYPE='DPS'
export IOTHUB_DEVICE_DPS_ID_SCOPE='0ne0037CE3D'
export IOTHUB_DEVICE_DPS_DEVICE_ID='DE10-Nano'
export IOTHUB_DEVICE_DPS_DEVICE_KEY='L2MD3xTyzPTJsdxw8/BAd+0ylYmT3QblLfgzlooriLjMN6UcFXQ8KPw/zTACdQhNE/uxWmHFzixcsDhhX5A2KdfdafdQ=='
```

Configure FPGA by copying `.dtbo` and `.rbf` files to `/lib/firmware` folder

Example :

```bash
overlay_dir="/sys/kernel/config/device-tree/overlays/socfpga"
overlay_dtbo="rfs-overlay.dtbo"
overlay_rbf="Module5_Sample_HW.rbf"

if [ -d $overlay_dir ];then
  rmdir $overlay_dir
fi

cp $overlay_dtbo /lib/firmware/
cp $overlay_rbf /lib/firmware/

mkdir $overlay_dir
```

> [!TIP]  
> If you are interested in running Azure IoT Edge, follow [this](DE10-Nano-IoTEdge.md) instruction

## 3. Run reference device application

Execute main.py in DE10-Nano with :

```bash
python3.7 main.py
```

Once your DE10-Nano is provisioned and connected, you should see device events such as device created and connected, as well as Telemetry from your DE10-Nano.

![DPS05](/images/DPS-05.png)

> [!TIP]  
> You can see details of events and telemetry by clicking the blue button on the left of each row.

Learn more about Device Events from IoT Hub : <https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-event-grid#event-types>

> [!CAUTION]  
> The reference application sends 2 telemetry messages every 10 seconds.   You will hit daily quota of 8000 messages per day in 11 hours.  
> Please make sure you turn off the device or stop the app when you are not using it.

## Completed

Congratulations.  You are now connect to IoT Hub and you can visualize device events from IoT Hub and telemetry from your DE10-Nano.

## Next Step

- [Deep Dive on sample IoT solution](PaaS-DeepDive.md)
- [Back to README](../README.md)
