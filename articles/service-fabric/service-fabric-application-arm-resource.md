---
title: Развертывание и обновление с помощью Azure Resource Manager
description: Узнайте, как развертывать приложения и службы в кластере Service Fabric с помощью шаблона Azure Resource Manager.
ms.topic: conceptual
ms.date: 12/06/2017
ms.openlocfilehash: ed6bc7d96cb3ea0934929e6543c5e637a9f42c1f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97930843"
---
# <a name="manage-applications-and-services-as-azure-resource-manager-resources"></a>Управление приложениями и службами как ресурсами Azure Resource Manager

Вы можете развертывать приложения и службы в кластере Service Fabric с помощью Azure Resource Manager. Это означает, что теперь вместо развертывания приложений и управления ими с помощью PowerShell или интерфейса командной строки после ожидания готовности кластера можно определить эти приложения и службы в формате JSON и развертывать их с помощью того же шаблона Resource Manager, что и кластер. Процесс регистрации, подготовки и развертывания приложения происходит за один шаг.

Это рекомендуемый способ развертывания любых приложений установки, систем управления или приложений управления кластером, требуемых в кластере. К ним относятся [приложение для управления исправлениями](service-fabric-patch-orchestration-application.md), модули наблюдения и любые приложения, которые нужно запустить в кластере перед развертыванием других приложений или служб. 

Если это возможно, следует управлять приложениями как ресурсами Resource Manager, чтобы улучшить:
* Журнал аудита: Resource Manager выполняется аудит каждой операции и хранит подробный *журнал действий*, позволяющий отслеживать все изменения, внесенные в эти приложения и кластер.
* Управление доступом на основе ролей Azure (Azure RBAC). Управление доступом к кластерам, а также приложениям, развернутым в кластере, можно выполнить с помощью одного и того же шаблона диспетчер ресурсов.
* Azure Resource Manager (посредством портала Azure) становится единым центром управления кластером и развертываниями важных приложений.

В следующем фрагменте кода показаны различные ресурсы, которыми можно управлять с помощью шаблона.

```json
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications/services",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
    "location": "[variables('clusterLocation')]"
}
```

## <a name="add-a-new-application-to-your-resource-manager-template"></a>Добавление нового приложения в шаблон Resource Manager

1. Подготовьте шаблон Resource Manager кластера для развертывания. Дополнительные сведения об этом приведены в разделе [Создание кластера Service Fabric в Azure с помощью Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).
2. Подумайте, какие приложения планируется развернуть в кластере. Возможно, среди них есть приложения, которые будут всегда работать, и другие приложения будут зависеть от них? Планируется ли развертывание систем управления кластером или приложений установки? Такими приложениями лучше всего управлять с помощью шаблона Resource Manager, как описано выше. 
3. После выяснения того, какие приложения необходимо развернуть таким образом, приложения должны быть упакованы, помещены в ZIP-файл и размещены в общей папке хранилища. Этот файловый ресурс должен быть доступен через конечную точку REST, чтобы Azure Resource Manager мог использовать его во время развертывания. Дополнительные сведения см. [в разделе Создание учетной записи хранения](service-fabric-concept-resource-model.md#create-a-storage-account) .
4. В шаблоне Resource Manager под объявлением кластера опишите свойства каждого приложения. Эти свойства включают в себя количество реплик или экземпляров, а также все цепочки зависимостей между ресурсами (другие приложения или службы). Обратите внимание на то, что это не заменяет манифесты приложений или служб. Просто часть их содержимого указывается в шаблоне Resource Manager кластера. Ниже приведен пример шаблона, который включает в себя развертывание службы без отслеживания состояния *Service1* и службы с отслеживанием состояния *Service2* в качестве части приложения *Application1*.

   ```json
   {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "clusterName": {
        "type": "string",
        "defaultValue": "Cluster",
        "metadata": {
          "description": "Name of your cluster - Between 3 and 23 characters. Letters and numbers only."
        }
      },
      "applicationTypeName": {
        "type": "string",
        "defaultValue": "ApplicationType",
        "metadata": {
          "description": "The application type name."
        }
      },
      "applicationTypeVersion": {
        "type": "string",
        "defaultValue": "1",
        "metadata": {
          "description": "The application type version."
        }
      },
      "appPackageUrl": {
        "type": "string",
        "metadata": {
          "description": "The URL to the application package sfpkg file."
        }
      },
      "applicationName": {
        "type": "string",
        "defaultValue": "Application1",
        "metadata": {
          "description": "The name of the application resource."
        }
      },
      "serviceName": {
        "type": "string",
        "defaultValue": "Application1~Service1",
        "metadata": {
          "description": "The name of the service resource in the format of {applicationName}~{serviceName}."
        }
      },
      "serviceTypeName": {
        "type": "string",
        "defaultValue": "Service1Type",
        "metadata": {
          "description": "The name of the service type."
        }
      },
      "serviceName2": {
        "type": "string",
        "defaultValue": "Application1~Service2",
        "metadata": {
          "description": "The name of the service resource in the format of {applicationName}~{serviceName}."
        }
      },
      "serviceTypeName2": {
        "type": "string",
        "defaultValue": "Service2Type",
        "metadata": {
          "description": "The name of the service type."
        }
      }
    },
    "variables": {
      "clusterLocation": "[resourcegroup().location]"
    },
    "resources": [
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [],
        "properties": {
          "provisioningState": "Default"
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "appPackageUrl": "[parameters('appPackageUrl')]"
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applications",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'), '/versions/', parameters('applicationTypeVersion'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "typeName": "[parameters('applicationTypeName')]",
          "typeVersion": "[parameters('applicationTypeVersion')]",
          "parameters": {},
          "upgradePolicy": {
            "upgradeReplicaSetCheckTimeout": "01:00:00.0",
            "forceRestart": "false",
            "rollingUpgradeMonitoringPolicy": {
              "healthCheckWaitDuration": "00:02:00.0",
              "healthCheckStableDuration": "00:05:00.0",
              "healthCheckRetryTimeout": "00:10:00.0",
              "upgradeTimeout": "01:00:00.0",
              "upgradeDomainTimeout": "00:20:00.0"
            },
            "applicationHealthPolicy": {
              "considerWarningAsError": "false",
              "maxPercentUnhealthyDeployedApplications": "50",
              "defaultServiceTypeHealthPolicy": {
                "maxPercentUnhealthyServices": "50",
                "maxPercentUnhealthyPartitionsPerService": "50",
                "maxPercentUnhealthyReplicasPerPartition": "50"
              }
            }
          }
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applications/services",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applications/', parameters('applicationName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "serviceKind": "Stateless",
          "serviceTypeName": "[parameters('serviceTypeName')]",
          "instanceCount": "-1",
          "partitionDescription": {
            "partitionScheme": "Singleton"
          },
          "correlationScheme": [],
          "serviceLoadMetrics": [],
          "servicePlacementPolicies": []
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applications/services",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName2'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applications/', parameters('applicationName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "serviceKind": "Stateful",
          "serviceTypeName": "[parameters('serviceTypeName2')]",
          "targetReplicaSetSize": "3",
          "minReplicaSetSize": "2",
          "replicaRestartWaitDuration": "00:01:00.0",
          "quorumLossWaitDuration": "00:02:00.0",
          "standByReplicaKeepDuration": "00:00:30.0",
          "partitionDescription": {
            "partitionScheme": "UniformInt64Range",
            "count": "5",
            "lowKey": "1",
            "highKey": "5"
          },
          "hasPersistedState": "true",
          "correlationScheme": [],
          "serviceLoadMetrics": [],
          "servicePlacementPolicies": [],
          "defaultMoveCost": "Low"
        }
      }
    ]
   }
   ```

   > [!NOTE] 
   > Дополнительные сведения о свойствах отдельных шаблонов см. в [справочнике](/azure/templates/microsoft.servicefabric/clusters/applicationtypes) по Service Fabric Azure Resource Manager.

5. Разверните шаблон. 

## <a name="remove-service-fabric-resource-provider-application-resource"></a>Удаление ресурса приложения поставщика ресурсов Service Fabric
Следующий триггер приведет к отмене подготовки пакета приложения из кластера, что приведет к очистке используемого места на диске:
```powershell
Get-AzureRmResource -ResourceId /subscriptions/{sid}/resourceGroups/{rg}/providers/Microsoft.ServiceFabric/clusters/{cluster}/applicationTypes/{apptType}/versions/{version} -ApiVersion "2019-03-01" | Remove-AzureRmResource -Force -ApiVersion "2017-07-01-preview"
```
Простое удаление Microsoft. ServiceFabric/Clusters/Application из шаблона ARM не приведет к отмене подготовки приложения.

>[!NOTE]
> После завершения удаления не следует видеть версию пакета в SFX или ARM. Нельзя удалить ресурс версии типа приложения, с которым выполняется приложение; ARM/SFRP сделает это невозможным. Если вы попытаетесь отменить подготовку выполняющегося пакета, то среда выполнения будет препятствовать этому.


## <a name="manage-an-existing-application-via-resource-manager"></a>Управление существующим приложением с помощью Resource Manager

Если кластер уже запущен и в нем развернуто несколько приложений, которыми требуется управлять как ресурсами Resource Manager, то вместо удаления этих приложений и их повторного развертывания можно использовать вызов PUT с использованием тех же интерфейсов API, чтобы эти приложения были подтверждены как ресурсы Resource Manager. Дополнительные сведения см. в статье [что такое модель ресурсов приложения Service Fabric?](./service-fabric-concept-resource-model.md)

> [!NOTE]
> Чтобы позволить обновлению кластера игнорировать неработоспособные приложения, клиент может указать "maxPercentUnhealthyApplications: 100" в разделе "upgradeDescription/healthPolicy". Подробные описания всех параметров приведены в [документации по политике обновления REST API для кластера Service Fabrics](/rest/api/servicefabric/sfrp-model-clusterupgradepolicy).

## <a name="next-steps"></a>Дальнейшие действия

* Используйте [интерфейс командной строки Service Fabric](service-fabric-cli.md) или [PowerShell](service-fabric-deploy-remove-applications.md) для развертывания других приложений в кластере. 
* [Обновите кластер Service Fabric](service-fabric-cluster-upgrade.md).
