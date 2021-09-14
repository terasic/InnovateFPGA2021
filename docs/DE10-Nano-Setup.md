<!---
date : 9/1/2021
author : Daisuke Nakahara <daisuken@microsoft.com>
reviewer : Berry Tsai <betsai@microsoft.com>; Takehiro Hirai <takehiro.hirai@microsoft.com>
Maintainer : 
title : DE10-Nano setup instruction for Azure IoT Edge
--->

# Setting up DE10-Nano to run Azure IoT Edge

The reference device application can be run as a docker container application.  Azure IoT Edge provides runtime environment to host containerized application.

Learn more about Azure IoT Edge : <https://azure.microsoft.com/services/iot-edge/>

> [!IMPORTANT]  
> Please make sure to run Azure IoT Edge runtime version 1.2.4 or above.

## Upgrade to Ubuntu 18.04

In order to run Azure IoT Edge, please upgrade to Ubuntu 18.04.

Learn more about supported OS <https://docs.microsoft.com/azure/iot-edge/support?view=iotedge-2020-11#operating-systems>

1. Burn SD Card with <http://download.terasic.com/downloads/cd-rom/de10-nano/DE10-Nano-Cloud-Native.zip>
1. Increase partition  

    ```bash
    root@de10nano:~# fdisk /dev/mmcblk0
    
    Welcome to fdisk (util-linux 2.27.1).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.
    
    
    Command (m for help): p
    Disk /dev/mmcblk0: 7.4 GiB, 7904165888 bytes, 15437824 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xbdf7f996
    
    Device         Boot  Start     End Sectors  Size Id Type
    /dev/mmcblk0p1        2048  206848  204801  100M  b W95 FAT32
    /dev/mmcblk0p2      227331 7395332 7168002  3.4G 83 Linux
    /dev/mmcblk0p3      206849  227330   20482   10M a2 unknown
    
    Partition table entries are not in disk order.
    
    Command (m for help): d
    Partition number (1-3, default 3): 2
    
    Partition 2 has been deleted.
    
    Command (m for help): n
    Partition type
        p   primary (2 primary, 0 extended, 2 free)
        e   extended (container for logical partitions)
    Select (default p): p
    Partition number (2,4, default 2): 2
    First sector (227331-15437823, default 229376): 227331
    Last sector, +sectors or +size{K,M,G,T,P} (227331-15437823, default 15437823): 15437823
    
    Created a new partition 2 of type 'Linux' and of size 7.3 GiB.
    
    Command (m for help): p
    Disk /dev/mmcblk0: 7.4 GiB, 7904165888 bytes, 15437824 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xbdf7f996
    
    Device         Boot  Start      End  Sectors  Size Id Type
    /dev/mmcblk0p1        2048   206848   204801  100M  b W95 FAT32
    /dev/mmcblk0p2      227331 15437823 15210493  7.3G 83 Linux
    /dev/mmcblk0p3      206849   227330    20482   10M a2 unknown
    
    Partition table entries are not in disk order.
    
    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Re-reading the partition table failed.: Device or resource busy
    
    The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8).
    ```

1. Reboot the device

    ```bash
    root@de10nano:~# reboot now
    ```

1. Expand partition

    ```bash
    root@de10nano:~# resize2fs /dev/mmcblk0p2
    resize2fs 1.42.13 (17-May-2015)
    Filesystem at /dev/mmcblk0p2 is mounted on /; on-line resizing required
    old_desc_blocks = 1, new_desc_blocks = 1
    The filesystem on /dev/mmcblk0p2 is now 1901311 (4k) blocks long.
    ```

1. Update package list  

    ```bash
    apt update && \
    apt upgrade -y && \
    apt dist-upgrade -y && \
    apt autoremove -y && \
    apt install -y update-manager-core
    ```

1. Start upgrade  

    If prompted, select `install the package maintainer's version`

    ```bash
    do-release-upgrade
    ```

1. Check OS Version  

  ```bash
  lsb_release -a
  ```

1. Update package list after reboot  

    ```bash
    apt update && \
    apt upgrade -y && \
    apt autoremove -y     
    ```

1. Install a few tools and locate  

    ```bash
    apt install -y less file locate nano && \
    updatedb
    ```

### (Option) Static IP  

DE10-Nano does not have hardware MAC address, therefore, IP address assigned by DHCP may change every boot.  Depending on your environment, static IP may be more convenient for your development work.

1. Open /etc/network/interfaces` file with your favorite editor  

    ```bash
    nano /etc/network/interfaces
    ```

1. Add following block to the end of the file  

    e.g. Setting Static IP to 192.168.1.2

    ```bash
    auto eth0
      iface eth0 inet static
      address 192.168.1.2
      netmask 255.255.255.0
      gateway 192.168.1.1
      dns-nameserver 192.168.1.1
    ```

1. Update network interface  

    ```bash
    ifup eth0
    ```

1. Check new IP Address  

    ```bash
    ifconfig
    ```

### (Option) Create disk image  

```bash
sudo dd bs=1M if=/dev/sdb of=/home/Ubuntu18_1.img
```

## Install Azure IoT Edge

1. Open `/overlay/fpgaoverlay.sh` with your favorite editor

    e.g. with Nano

    ```bash
    nano /overlay/fpgaoverlay.sh
    ```

1. Comment out all lines after `mkdir`

    ```bash
    echo "creating $overlay_dir"
    mkdir $overlay_dir
    
    #echo "Doing Device Tree Overlay"
    #echo overlay.dtbo > $overlay_dir/path
    
    #echo "Successfully Device Tree Overlay Done"
    
    #echo "Loading altvipfb"
    #modprobe altvipfb
    #echo "Successfully altvipfb is loaded"
    ```

1. Save the change and exit the editor

1. Add package list  

    > [!NOTE]  
    > To Be Updated after the fix (v1.2.4) is released

    ```bash
    mkdir -p ~/tmp && \
    cd ~/tmp && \
    curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list && \
    cp ./microsoft-prod.list /etc/apt/sources.list.d/ && \
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg && \
    cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
    ```

1. Install Moby  

    ```bash
    apt-get update && \
    apt-get install -y moby-engine
    ```

1. Install IoT Edge Runtime

    > [!NOTE]  
    > To Be Updated after the fix (v1.2.4) is released

    ```bash
    curl -L https://github.com/Azure/azure-iotedge/releases/download/1.2.3/aziot-identity-service_1.2.2-1_ubuntu18.04_armhf.deb -o aziot-identity-service.deb && \
    curl -L https://github.com/Azure/azure-iotedge/releases/download/1.2.3/aziot-edge_1.2.3-1_ubuntu18.04_armhf.deb -o aziot-edge.deb && \
    sudo apt-get install -y ./aziot-identity-service.deb ./aziot-edge.deb
    ```

1. Copy and open IoT Edge Runtime configuration file with your favorite editor  

    E.g.

    ```bash
    cp /etc/aziot/config.toml.edge.template /etc/aziot/config.toml && \
    nano /etc/aziot/config.toml 
    ```

1. Navigate to the sample solution's web dashboard

1. Create `Individual Enrollment` with following parameters

    | Setting         | Description                                                                                           | Example        |
    |-----------------|-------------------------------------------------------------------------------------------------------|----------------|
    | Enrollment Name | A new name of enrollment.  This name is used for Device ID.  Alphanumeric characters and hyphen only. | DE10-Nano-Edge |
    | Enrollment Type | Type of enrollment.  Select individual.                                                               | Individual.    |
    | Capacity        | Creating enrollment for IoT Edge device.  Turn on IoT Edge.                                           |                |

    ![App 03](/images/App-03.png)

1. Click `Create` to create a new individual enrollment.

    If a new enrollment is successfully created, you should see the new enrollment in the top half of the dialog.

    ![App 04](/images/App-04.png)

1. Navigate to `DPS provisioning with symmetric key` section and uncomment

    ```toml
    ## DPS provisioning with symmetric key
    [provisioning]
    source = "dps"
    global_endpoint = "https://global.azure-devices-provisioning.net"
    id_scope = "0ab1234C5D6"
    #
    [provisioning.attestation]
    method = "symmetric_key"
    registration_id = "my-device"
    #
    symmetric_key = { value = "YXppb3QtaWRlbnRpdHktc2VydmljZXxhemlvdC1pZGVudGl0eS1zZXJ2aWNlfGF6aW90LWlkZW50aXR5LXNlcg==" } # inline key (base64), or...
    # symmetric_key = { uri = "file:///var/secrets/device-id.key" }                                                          # file URI, or...
    # symmetric_key = { uri = "pkcs11:slot-id=0;object=device%20id?pin-value=1234" }                                         # PKCS#11 URI
    
    ```

1. Copy and paste `ID Scope`, `Enrollment Name`, and `Symmetric Key` from web UI to config.toml file using `copy` buttons.

    ```bash
    ## DPS provisioning with symmetric key
    [provisioning]
    source = "dps"
    global_endpoint = "https://global.azure-devices-provisioning.net"
    id_scope = "0ne00378E84"
    #
    [provisioning.attestation]
    method = "symmetric_key"
    registration_id = "DE10-Nano-Edge"
    #
    symmetric_key = { value = "vHVWBvpxFkyTUHY0Grm8bj3MlYcVim2TpZeBcyXxjNe6jGmdfvQ2I5yH3C2Oun1JRYrHfoHjlRHfMKkDwl5Vag==" }
    ```

1. Save the edit and exit the editor

   With Nano editor, press `CTRL+X`, then enter `Y` key to save.

1. Update IoT Edge runtime with :

    ```bash
    iotedge config apply
    ```

    e.g.

    ```bash
    root@de10nano:/etc/aziot# iotedge config apply
    Note: Symmetric key will be written to /var/secrets/aziot/keyd/device-id
    Azure IoT Edge has been configured successfully!
    
    Restarting service for configuration to take effect...
    Stopping aziot-edged.service...Stopped!
    Stopping aziot-identityd.service...Stopped!
    Stopping aziot-keyd.service...Stopped!
    Stopping aziot-certd.service...Stopped!
    Stopping aziot-tpmd.service...Stopped!
    Starting aziot-edged.mgmt.socket...Started!
    Starting aziot-edged.workload.socket...Started!
    Starting aziot-identityd.socket...Started!
    Starting aziot-keyd.socket...Started!
    Starting aziot-certd.socket...Started!
    Starting aziot-tpmd.socket...Started!
    Starting aziot-edged.service...Started!
    Done.
    root@de10nano:/etc/aziot#
    ```

## Verify successful installation

Follow the steps described [here](https://docs.microsoft.com/azure/iot-edge/how-to-auto-provision-symmetric-keys?view=iotedge-2020-11&tabs=linux#verify-successful-installation) to validate successful installation.

<https://docs.microsoft.com/azure/iot-edge/how-to-auto-provision-symmetric-keys?view=iotedge-2020-11&tabs=linux#verify-successful-installation>

The sample solution dashboard should show `DeviceCreated` device event from IoT Hub.

![App 05](/images/App-05.png)

> [!TIP]  
> It can take several minutes to initialize Azure IoT Edge, depending on your network speed.

After a few minutes, IoT Edge runtime module `edgeAgent` should be in the running status.

```bash
root@de10nano:~# iotedge list
NAME             STATUS           DESCRIPTION      CONFIG
edgeAgent        running          Up 6 minutes     mcr.microsoft.com/azureiotedge-agent:1.2
```

The dashboard should show `DeviceConnected` device event.

![App 06](/images/App-06.png)

> [!CAUTION]  
> The reference application sends 2 telemetry messages every 10 seconds.   You will hit daily quota of 8000 messages per day in 11 hours.  
> Please make sure you turn off the device or stop IoT Edge when you are not using it.
>
> To stop IoT Edge, run
>
> ```bash
> iotedge system stop
> ```

## Completed

Congratulations!  You are now connect to IoT Hub and you can deploy IoT Edge module to DE10-Nano.

## Next Step

- [Deep Dive on sample IoT solution](PaaS-DeepDive.md)
- [Back to README](../README.md)
