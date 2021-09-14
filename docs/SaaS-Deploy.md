<!---
date : 9/1/2021
author : Daisuke Nakahara <daisuken@microsoft.com>
reviewer : Berry Tsai <betsai@microsoft.com>; Takehiro Hirai <takehiro.hirai@microsoft.com>
Maintainer : 
title : Deploy IoT Central for InnovateFPGA 2021
--->

# SaaS IoT Solution : Azure IoT Central

IoT Central is an IoT application platform that reduces the burden and cost of developing, managing, and maintaining enterprise-grade IoT solutions.  To learn more about IoT Central, please visit [IoT Central documentation](https://docs.microsoft.com/azure/iot-central/core/overview-iot-central).

Each instance of IoT Central deployment is called `IoT Central Application`.  For your convenience, we prepared a customized IoT Central application, and made it available through [IoT Central Application Template](https://docs.microsoft.com/azure/iot-central/core/concepts-app-templates).

## 1. Start deploying Azure IoT Central

Click **Deploy to Azure** button below to deploy IoT Central Application

> [!TIP]  
> Right click the button below and select **Open link in new tab** or **Open lin in new window**

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdaisukeiot%2FInnovateFPGA2021%2Fmain%2Fazuredeployiotc.json)

## 2. Provide parameters for IoT Central deployment

![SaaS01](/images/SaaS-01.png)

| Setting        | Description  | Example    |
|----------------|--------------|------------|
| Subscription   | Select your Azure subscription from the list. |            |
| Resource Group | Resource group is a way to organize resources (services).  To learn more visit [here](/azure/azure-resource-manager/management/manage-resource-groups-portal).  Create new resource group by clicking `Create new`.| MyInnovateFPGAIoTC           |
| Region         | Select nearest location.  To learn more, visit [here](https://azure.microsoft.com/global-infrastructure/geographies/#overview).                                                                                                                                                       | West US 2 for Pacific Timezone           |
| Iot Central Application Nam      | Default name given is with 'InnovateFPGA-` prefix and random strings (E.g. InnovateFPGA-&lt;abx73&gt;).  Accept default name or specify your own unique name.  Since IoT Central uses this name for URL, please make sure you provide unique name without space or special characters. | MyInnovateFPGA-IoTC01 |
| IoT Central Location    | Location (region/country) to deploy IoT Central application.  |            |

> [!NOTE]  
> For IoT Central, `Region` setting does not really matter.  The new IoT Central application will be deployed based on `IoT Central Location` setting.

## 3. Review and start IoT Central deployment

Click `Review + create` button for Azure Portal to validate the settings.  Start deployment by clicking `Create` button.

![SaaS02](/images/SaaS-02.png)

## 4. Confirm successful IoT Central deployment

Deployment should finish in 1 minute or so.  Please wait for the deployment to complete.
Once the deployment is completed, navigate to the sample Web Application to confirm your solution instance is up and running.

![SaaS03](/images/SaaS-03.png)

1. Click `Outputs` in the left pane, then copy Web app's URL by clicking the blue button on the right.

    ![PaaS04](/images/SaaS-04.png)

1. Open a new browser tab to access your new IoT Central application

    ![SaaS05](/images/SaaS-05.png)

> [!TIP]  
> You can navigate to your IoT Central application from <https://azureiotcentral.com>

## Next Step

- [Provision DE10-Nano](SaaS-Provision.md) to your IoT Central to interact.  
- [Back to README](../README.md)
