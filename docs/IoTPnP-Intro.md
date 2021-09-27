# IoT Plug and Play

IoT Plug and Play enables solution builders to integrate IoT devices with their solutions without any manual configuration. At the core of IoT Plug and Play, is a device model that a device uses to advertise its capabilities to an IoT Plug and Play-enabled application. This model is structured as a set of elements that define:

- Properties that represent the read-only or writable state of a device or other entity. For example, a device serial number may be a read-only property and a target temperature on a thermostat may be a writable property.
- Telemetry that's the data emitted by a device, whether the data is a regular stream of sensor readings, an occasional error, or an information message.
- Commands that describe a function or operation that can be done on a device. For example, a command could reboot a gateway or take a picture using a remote camera.
- Components are the way to structure IoT Plug and Play models.  You can reuse or reference components from another device model.  

## Requirements for IoT Plug and Play

In order for a device to act as IoT Plug and Play, followings are required :

- IoT Plug and Play device model definition file  

    JSON based documents written in [Digital Twin Definition Language (DTDL)](https://aka.ms/dtdl)

- Model ID Announcement

    During provisioning, the device application must provide Digital Twin Model ID (DTMI) to Device Provisioning Service (DPS).  
    DTMI is passed to DPS as a part of payload during provisioning.

- Model Repository  

    Based on DTMI, IoT Solution references Device Model definition files (JSON files), therefore, definition files must be accessible.
    You may use existing infrastructure, or embed definition files to IoT solution, or set up your own repository.  

- Device Code to adhere IoT Plug and Play convention  

    IoT Plug and Play convention defines how IoT device code and IoT solution application interact.  Data must be sent in an agreed format.  Please refer to [online document ](https://docs.microsoft.com/azure/iot-develop/concepts-convention)for more details.

## Reference Application

Reference Application uses componentized structure.

```json

DE10-Nano Device Model (ID = dtmi:Terasic:FCC:DE10_Nano;1)
    |- Onboard G-Sensor Component (ID = dtmi:Terasic:FCC:DE10_Nano_Sensor;1)
    |- RFS Daughter Card Component (ID = dtmi:Terasic:FCC:RFS;1)
    |- Sensor Threshold Setting Component (ID = dtmi:Terasic:FCC:DE10_Nano_Threshold;1)

```

[DE10-Nano IoT Plug and Play model definition file](https://github.com/Azure/iot-plugandplay-models/tree/main/dtmi/terasic/fcc)

## Customization ideas

There are multiple ways you can add IoT Plug and Play interfaces.  Example are :

- Add more telemetry and properties for additional sensors connected to DE10-Nano
- Send command from Cloud to perform tasks on DE10-Nano
- Report events when DE10-Nano detected specific conditions
- Manage binaries for FPGA remotely from Cloud

Depending on your use cases, you may add a new component for additional sensors, properties, and/or commands.  You can always start with device model prepared for the reference application.

## Resources

- [IoT Plug and Play](https://aka.ms/iotpnp)
- [Digital Twin Definition Language](https://aka.ms/dtdl)  
- [Back to README](../README.md)
