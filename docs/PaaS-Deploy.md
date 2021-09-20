<!---
date : 9/1/2021
author : Daisuke Nakahara <daisuken@microsoft.com>
reviewer : Berry Tsai <betsai@microsoft.com>; Takehiro Hirai <takehiro.hirai@microsoft.com>
Maintainer : 
title : Azure IoT Sample Solution Deployment
--->

# Deploy PaaS IoT Solution

This sample solution is built with 6 Azure IoT Platform services.

- Azure [IoT Hub](https://azure.microsoft.com/services/iot-hub/) : To establish secure bi-directional communication with DE10-Nano.
- Azure IoT Hub [Device Provisioning Service](https://docs.microsoft.com/azure/iot-dps/about-iot-dps) : To provision DE10-Nano to IoT solution.
- Azure [App Service](https://azure.microsoft.com/services/app-service/) : To host Web site.
- Azure [Storage Service](https://azure.microsoft.com/services/storage/blobs/) : Storage space used for the web application.
- Azure [Functions](https://azure.microsoft.com/services/functions/) : To process device data and push to Web application.
- Azure [SignalR](https://azure.microsoft.com/services/signalr-service/) : To feed real-time data to the web application.
- [IoT Plug and Play](https://aka.ms/iotpnp) enabled

Click [here](PaaS-DeepDive.md) to learn more in details.

With this sample solution, you can perform basic IoT operations:

- Basic Device Management  
  - Provision a device
  - Connects a device to Azure IoT
  - Manage device identity
- Visualize telemetry (messages) from the device in Web UI
- Visualize device events such as device connect and disconnect in Web UI  

## 1. Start deploying Azure IoT Services

> [!NOTE]  
> TBD Update link to ARM template after converting the repo to public.

Click **Deploy to Azure** button below.  The button will take you to Azure Portal and loads [Azure Resource Manager (ARM) template](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview).

> [!TIP]  
> <https://portal.azure.com> is called **Azure Portal** which is a web-based, unified GUI tool where you can monitor and manage Azure services.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdaisukeiot%2FInnovateFPGA2021%2Fmain%2Fazuredeploy.json)

> [!TIP]  
> Right click the button below and select **Open link in new tab** or **Open link in new window**

## 2. Basic settings for your solution

Each solution requires a few unique settings and parameters.  Please provide information to customize your solution.

| Setting        | Description                                                                                                                                                                                                                                           | Example                        |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| Subscription   | Select your Azure subscription from the list.  If you do not have one, please sign up [free subscription](https://azure.microsoft.com/free/).                                                                                                         |                                |
| Resource Group | Resource group is a way to organize resources (services).  To learn more visit [here](/azure/azure-resource-manager/management/manage-resource-groups-portal).  Create new resource group by clicking `Create new`.                                   | MyInnovateFPGAGroup            |
| Region         | Select nearest location.  To learn more, visit [here](https://azure.microsoft.com/global-infrastructure/geographies/#overview).                                                                                                                       | West US 2 for Pacific Timezone |
| Unique ID      | Default names given to all resources are InnovateFPGA-&lt;ServiceName&gt;-&lt;Your Unique ID&gt;.  (E.g. InnovateFPGA-IoTHub-&lt;Unique Id&gt;)  Since some services require globally unique names so please make it unique, or accept random string. | YourName01                     |
| IoT Hub Sku    | If you plan to send more than 8000 messages per day, please select S1.  S1 will cost you $25/month.  Please see [Azure IoT Hub pricing page](https://azure.microsoft.com/pricing/details/iot-hub/) for more details.                                  |                                |

![PaaS02](/images/PaaS-02.png)

## 3. Review and start deployment

Click `Review + create` button for Azure Portal to validate the settings.  Start deployment by clicking `Create` button.

![PaaS03](/images/PaaS-03.png)

> [!TIP]  
> You can see the progress of deployment as well as results of operations.
>
> ![PaaS04](/images/PaaS-04.png)

## 4. Confirm successful deployment

Deployment should finish in about 10 minutes.  Please wait for the deployment to complete.
Once the deployment is completed, navigate to the sample Web Application to confirm your solution instance is up and running.

![PaaS05](/images/PaaS-05.png)

1. Click `Outputs` in the left pane, then copy Web app's URL by clicking the blue button on the right.

    ![PaaS06](/images/PaaS-06.png)

1. Open a new browser tab to access your solution

    ![PaaS07](/images/PaaS-07.png)

> [!TIP]  
> You can find Web App's URL in `App Service` instance in Azure Portal.
>
> ![PaaS08](/images/PaaS-08.png)

## Next Step

- [Provision DE10-Nano](PaaS-Provision.md) to your IoT Solution to interact with DE10-Nano.  
- [Back to README](../README.md)
