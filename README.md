<!---
date : 9/1/2021
author : Daisuke Nakahara <daisuken@microsoft.com>
reviewer : Berry Tsai <betsai@microsoft.com>; Takehiro Hirai <takehiro.hirai@microsoft.com>
Maintainer : 
title : Azure IoT Sample Solution for InnovateFPGA 2021
--->

# InnovateFPGA2021 Design Contest

![InnovateFPGA Logo](images/Logo-Banner.png)

## Requirements

- Microsoft Account  

  <https://account.microsoft.com/account>

- Azure Subscription  

    If you do not have Azure Subscription, please create a new account for free.  

    <https://azure.microsoft.com/free/>  

    If you are a student, you may sign up for Azure for Students.

    <https://azure.microsoft.com/free/students>  

- A PC with Web Browser
- Terasic DE10-Nano Cloud Connectivity Kit

## IoT Solutions for InnovateFPGA design contest

This page describes how to deploy IoT solution in Azure using Azure Resource Manager (ARM) template.  
Microsoft provides Platform as a Service (PaaS) as well as Software as a Service (SaaS).  Depending on your needs, you may choose to deploy one of two or both.  

- [Sign up for Azure Subscription](docs/AzureSignup.md)  
- [Deploy PaaS solution](docs/PaaS-Deploy.md) : IoT Hub based sample IoT solution  
  - [Provision DE10-Nano](docs/PaaS-Provision.md) to the Sample IoT Solution
  - [Provision DE10-Nano as Azure IoT Edge](docs/DE10-Nano-IoTEdge.md)
  - [Deploy Reference Application as IoT Edge module](docs/DE10-Nano-IoTEdge-Deploy.md)
  - [Technical Deep Dive](docs/PaaS-DeepDive.md)

- [Deploy SaaS solution](docs/SaaS-Deploy.md) : Azure IoT Central pre-configured application
  - [Provision DE10-Nano](docs/SaaS-Provision.md) to your IoT Central Application

## Resources

- InnovateFPGA 2021 Homepage : <https://innovatefpga.com>
- Intro video : <https://youtu.be/OXPPFblUhtc>
- Source code
  - Reference Device Application : <https://github.com/intel-iot-devkit/terasic-de10-nano-kit/tree/master/azure-de10nano-document/sensor-aggregation-reference-design-for-azure>
  - Sample IoT Solution Web App : <https://github.com/microsoft/InnovateFPGA2021-WebApp>
  - Sample IoT Solution Functions App : <https://github.com/microsoft/InnovateFPGA2021-Functions>
