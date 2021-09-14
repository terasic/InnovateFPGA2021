---
date : 9/1/2021
author : Daisuke Nakahara <daisuken@microsoft.com>
reviewer : Berry Tsai <betsai@microsoft.com>; Takehiro Hirai <takehiro.hirai@microsoft.com>
Maintainer : 
title : Azure IoT Sample Solution (PaaS) Deep Dive
---

# A demo IoT solution to connect DE10-Nano to Azure IoT : Technical Deep Dive

In this document, we will cover some of basic concepts, data flow, and data processing.  For an IoT Solution to be valuable, multiple things must happen :

- Provision DE10-Nano
- Connect DE10-Nano
- Send data from DE10-Nano to Cloud
- Receive data in cloud
- Route data to IoT application
- Process data in IoT application

## IoT Solution Pattern

Typical IoT Solutions have 4 building blocks :

![Solution Pattern](/images/IoTSolutionPattern.png)

- Things  

  Sensors and gateways. Often called devices, edge.  DE10-Nano Cloud Connectivity Kit is an edge device with sensor.  It also act as a gateway.

- Cloud Gateway  

  Entry point to Azure cloud.  Azure IoT Hub is used in many solutions.  DE10-Nano Cloud Connectivity Kit connects to Azure IoT Hub to send telemetry.

- Insights

  Sensors and gateways sends data to cloud application through Cloud Gateway.  Data is ingested, processed, and analyzed to create business value.  For examples, temperature data of an engine is sent from a sensor, a cloud application monitors temperature of the engine.  When the temperature stays too high, the cloud application determines `Critical Condition`.

  Sending data to cloud may take time and can be costly.  You may run `Insights`, or process data at edge to reduce latency/delay.  Azure IoT Edge provides application runtime to bring computing logic down to the edge.

- Action  

  Based on insight, the cloud application takes an action.  For example, sending a text message to notify driver that the engine is overheating.  
  Action can also be taken at the edge.  For example, a gateway may send alert/command to the engine to shutdown to prevent overheat.

## Domains in the solution

The sample IoT solution is consist of multiple technology domains.

- Security domain  

  Typically solutions requires multiple security aspects.  To keep the sample solution simple, the sample solution implements minimum security.  

  - Device Authentication via DPS and IoT Hub.
  - No user authentication.
  - Azure IoT Services are protected via connection strings or keys

  > [!IMPORTANT]  
  > If you plan to make the web site available, consider adding Azure Active Directory (AAD) based Role Based Access Control (RBAC) to the web app.  
  >  
  > <https://docs.microsoft.com/azure/azure-app-configuration/concept-enable-rbac>

- Business and Industrial domain

  Solutions are typically designed and built for a specific use cases.  Business requirements as well as industrial requirements determines additional requirements to be implemented.  For example, monitoring engine's temperature for tracks and factory machineries may have different set of requirements.  The sample solution does not implement specific business nor industrial requirements to keep generic enough as a template.  Consider adding business or industrial requirements to your solution for your use case.  For example, requirements for animal tracking/monitoring solution may be very different from energy conservation solution.

- Device Management domain

  In order for a cloud application to ingest data from devices, devices must be provisioned and connected to cloud.  A solution may require updating device firmware/device application.  For example, solution administrators need to :  

  - Connect only devices that is known and trusted
  - Monitor and manage devices remotely
  - Capable of disconnecting devices if they are tampered

  The sample solution implements device provisioning through Azure Device Provisioning Service (DPS) using Symmetric Key with Individual Enrollment.  For broader provisioning (e.g. provision hundreds of devices), you may want to consider using X.509 certificate with Group Enrollment.  DPS works closely with IoT Hub to register and allow connection to IoT Hub.

- IoT Data domain

  For an IoT solution to create business value, data must be made available to cloud application(s).  This domain covers how communication between `Things` and `Cloud Application` through Cloud Gateway.  This domain determines how data flows into cloud, then distributed to IoT application.  

- Application domain

  Once data is made available in cloud, a cloud application processes data. This domain is where you make value from data, which is usually determined by :

  - Business problem to be solved
  - Requirements from use cases, business, and/or industry

  For example, a super simple use case (or business problem) may be "how do I visualize large amount of data?".  Visualizing millions of data points with a personal computer may take very long time, however, cloud can provide enough computing power to draw charts and graphs for millions of data points.

  The sample solution simply displays data from the device in real-time.  To keep the solution simple (and be cost effective), it does not store data.

  ![Solution Diagram](/images/solution-diagram.png)

## IoT Device and Data domain

The core of IoT Solution is provisioning, connecting, and interacting with IoT devices.  IoT Devices (DE10-Nano) sends data as well as accepts settings (Threshold) from Cloud.  Depending on your use cases, you may want to model more functionalities so you can send command from your solution, or trigger alerts based on the computation results from FPGA.

IoT Plug and Play provides a language to define device interaction model.

In typical IoT use cases involves following steps :

- Provision device
- Connect device
- Accept incoming data (Data Ingestion)
- Process data
- Take action

## Provision Device

Before you connect your DE10-Nano to the solution, you must prepare the device.  Device provisioning involves :

- Registering device to the solution
- Configure device authentication mechanism
- Configure the device with required settings

Provisioning IoT devices is similar to registering your smartphone to your carrier.  Your phone must be known to the carrier network, associated with your contracts, enabled/disabled features based on your contract, and unique phone number must be assigned.  These information may be programmed in your phone or through SIM card.  In the backend, your carrier manages contract information, phone number etc.

Device Provisioning Service (DPS) provides functionalities to provision your IoT devices to your IoT solution.  DPS uses `enrollment` information to identify the device, authenticate, then assign to the right Azure IoT Hub.

In the sample solution portal, you can manage enrollment with :

1. Click `Device Management` menu, then click `Device Provisioning (DPS)`  

    ![App 01](/images/App-01.png)

1. Click `Add New Enrollment`

    ![App 02](/images/App-02.png)

1. Enter `Enrollment Name`
1. Select Individual or Group enrollment  
  
    If you are planning to provision a single device, select `Individual`.  If you are provisioning multiple devices, select `Group` to manage as a group or create multiple `Individual` enrollments.

1. Turn on `IoT Edge` if you are provisioning Azure IoT Edge device

In Azure Portal, you can manage enrollments with :

1. Navigate to <https://portal.azure.com> and sign in with your Microsoft account.
1. Select `Resource groups` from the menu, then select your resource group  

    ![Portal 01](/images/Portal-01.png)

1. Select `Device Provisioning Service` instance from the list  

    ![Portal 02](/images/Portal-02.png)

1. Select `Manage Enrollments`  

    ![Portal 03](/images/Portal-03.png)

Learn more about managing enrollments : <https://docs.microsoft.com/azure/iot-dps/how-to-manage-enrollments>

## Connect Device

Once your device is provisioned, the device can now connect to Azure IoT Hub.  Azure IoT Hub act as a gateway between Cloud (Azure) and public internet.  IoT Hub provides bi-directional communication over MQTT protocol for secure communication.  

In order to connect DE10-Nano to IoT Hub, the device firmware/app must provide credential information to DPS.  The sample application supports `Symmetric Key` authentication.

More on symmetric key attestation : <https://docs.microsoft.com/azure/iot-dps/concepts-symmetric-key-attestation?tabs=linux>

In order to provision with DPS, the device app must provide 3 pieces of information :  

1. [ID Scope](https://docs.microsoft.com/azure/iot-dps/concepts-service#id-scope)
1. [Enrollment Name](https://docs.microsoft.com/azure/iot-dps/concepts-service#registration-id) (a.k.a. Registration ID)
1. [Symmetric Key](https://docs.microsoft.com/azure/iot-dps/concepts-symmetric-key-attestation?tabs=linux)

In the sample solution, you can retrieve above information.  Use `Copy` buttons to copy ID Scope and Symmetric Key.  

The reference device application takes these information through environment settings.  Set these variables with :

```bash
export IOTHUB_DEVICE_SECURITY_TYPE="DPS"
export IOTHUB_DEVICE_DPS_ENDPOINT="global.azure-devices-provisioning.net"
export IOTHUB_DEVICE_DPS_DEVICE_ID=<Enrollment Name>
export IOTHUB_DEVICE_DPS_ID_SCOPE=<ID Scope>
export IOTHUB_DEVICE_DPS_DEVICE_KEY=<Symmetric Key>
```

## IoT Data

The device can send data such as Sensor data from RFS, as well as receive data from Cloud.  Upstream data is often called Device to Cloud messages (D2C).  Downstream data has 3 types.  1) Cloud to Device message (C2D), 2) Device Twin, and 3) Device Command (or method).

Device Twin is used to communicate settings from Cloud to devices, which is called `Desired Property`, or `Writable Property` in Digital Twin/IoT Plug and Play world.  Device Twin can also be used to communicate properties of the device to cloud, which is called `Reported Property`, or `Property` in Digital Twin/IoT Plug and Play world.  IoT Hub provides storage space for each device so that these settings are not lost.  The communication is asynchronous way.  Properties are communicated when the device connects to IoT Hub, and the device will receive any changes if it is online.

Sometimes Cloud needs to communicate with the device and needs to ensure the communication is received.  This type of activities require `synchronous` communication.  In other words, when the sender (Cloud application) sends data, and the receiver (device) must respond.  If the device is offline, the command will fail with timeout.  The requester needs to know if the receiver received data, and processed it or not.  This communication can be achieved using Device Method, or Command in Digital Twin/IoT Plug and Play world.

Communication is done in [IoT Plug and Play convention](https://docs.microsoft.com/azure/iot-develop/concepts-convention).

### Ingesting Data

DE10-Nano reference application sends telemetry with Sensor Data every 2 seconds.

You should see these telemetry from DE10-Nano :

Telemetry with sensor data (RFS)

```json
{
  "lux": 0.6660000085830688,
  "humidity": 43.93310546875,
  "temperature": 22.831707000732422,
  "mpu9250": {
    "ax": 0.2968810200691223,
    "ay": 0.3064578175544739,
    "az": 9.96945571899414,
    "gx": 0.037247881293296814,
    "gy": 0.011706477031111717,
    "gz": -0.004256900865584612,
    "mx": 16.892578125,
    "my": 27.33926010131836,
    "mz": -1.9142580032348633
  }
}
```

Telemetry with gSensor data (onboard)

```json
{
  "gSensor": {
    "x": -0.028,
    "y": -0.024,
    "z": 1.012
  }
}

```

IoT Hub can manage communication and receives data from DE10-Nano.  However, IoT Hub does not do anything with data.  Data has to be routed to the appropriate destination for processing.

### Types of device data

There are 2 types of device data :

- Telemetry  

  Also called messages, or Device-to-Cloud message (D2C).  Telemetry usually contains timestamp and sent in periodic fashion.  A single data point usually does not provide enough information to make a decision.  Telemetry data is often processed with other data points, and/or as a series of data.

  For example DE10-Nano's reference application sends sensor data every 2 seconds.  A single data point, such as `temperature`=`22` at October 1st, 2021 at 0:00 simply shows the fact.  However, by monitoring sensor data over 60 minutes, we can tell if the temperature is trending up or down, or had spikes and/or dips.

- Events  

  Events happens in sporadic fashion.  Events typically states `Something happened`.  For example, IoT Hub generates events when DE10-Nano is connected or disconnected.  Events often carries enough information about what happened, but usually you cannot tell why it happened.

## Message Routing

IoT Hub receives data from authenticated devices, however, IoT Hub does not process data at all.  For example, if you do not save data, data from DE10-Nano will be removed.  

> [!NOTE]  
> IoT Hub has so called `built-in Event Hubs` which will keep messages for one day (default), up to 7 days.  
> This is called `Message Retention`.
> <https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-read-builtin>

Using IoT Hub's message routing, you can route any incoming messages to other services and applications.  The destination receives data with Publisher-Subscriber (PUB-SUB) fashion.  In a traditional communication (e.g. Server - Client), both ends must be available at the time of communication.  PUB-SUB does not require both ends to be available.  The middleman (called broker) will manage communication.  The sender (IoT Hub as Publisher) can send data and the receiver (Event Hubs as Subscriber) picks up data at will.  Azure Event Hubs is a broker service that keeps data (messages) for 1 day (by default) for the subscriber(s) to retrieve.

> [!TIP]  
> For solution with small number of devices, Built-in Event Hubs is enough to process data.  However, for the purpose of demo, it deploys Azure Event Hubs.

> [!NOTE]  
> IoT Hub's Built-in Event Hubs provides very similar functionalities as Azure Event Hubs, with a different set of limitations and quotas.  
> The sample solution uses Free SKU of IoT Hub to keep the cost down.  Free SKU supports one additional endpoint and 5 routings.
> <https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-quotas-throttling>

![IoTHub-EventHub](/images/IoTHub-EventHub.png)

If you would like to add another subscriber to consume telemetry, you can create a separate Custom Endpoint or subscribe from `DeviceTelemetryToEventHub-DP` endpoint.

> [!TIP]  
> The sample solution also routes Device Twin Change events and Device Lifecycle events to `DeviceTelemetryToEventHub-EP`.  
> The solution simply forwards events to Web app.  You may add additional processing as required.
> See OnDeviceTwinChange(), On OnDeviceLifecycleChanged() functions.

## Event Distribution

The sample solution implements Azure Event Grid to distribute events.  Similar to Telemetry, events are distributed in PUB-SUB fashion, while Event Grid is being a broker.  The publisher is IoT Hub, and the subscriber is WebApp.

![IoTHub-EventGrid](/images/IoTHub-EventGrid.png)

If you are interested in listening to events from IoT Hub and/or DE10-Nano, you can add a new `Event Subscription` to IoT Hub.

## Data Processing

Device Events are simply sent to Web Application through Event Grid.  Device Telemetry is sent to Azure Functions for further processing.  [Azure Functions](https://azure.microsoft.com/services/functions/) is a serverless application written in .Net.  It listens to telemetry through Event Hubs, then format data for [Azure SignalR](https://azure.microsoft.com/services/signalr-service/) message.  The sample solution does not monitor data but you can add your business logic to OnTelemetryReceived() function.  

For example, if you connect GPS to DE10-Nano and send location data to track movement of animals or shipment, you can update browser's view without having users to press refresh button by sending GPS telemetry data to WebApp in SignalR messages.

![SignalR](/images/SignalR.png)

## Actions

The sample solution does not take any actions.  The action it performs is re-formatting telemetry into SignalR message and send SignalR messages to SinglR service.  Depending on your use cases, you may want to add actions based on data and events from your DE10-Nano application.  For example, you can add sending SMS or email when AI model running in DE10-Nano found an animal. Or send warning message to Web UI when sensor data monitoring ocean temperature goes above or below threshold.

## Next Step

- [Back to README](../README.md)
