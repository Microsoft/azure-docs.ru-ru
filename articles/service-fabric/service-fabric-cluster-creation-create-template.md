---
title: Создание шаблона кластера Azure Service Fabric
description: Сведения о создании шаблона Resource Manager для кластера Service Fabric. Настройка безопасности, Azure Key Vault и Azure Active Directory (Azure AD) для проверки подлинности клиентов.
ms.topic: conceptual
ms.date: 08/16/2018
ms.openlocfilehash: b028b5c1f32733146c071e50b24202e5bad945c0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86260498"
---
# <a name="create-a-service-fabric-cluster-resource-manager-template"></a>Создание шаблона Resource Manager для кластера Service Fabric

[Кластер Azure Service Fabric](service-fabric-deploy-anywhere.md) — это подключенный к сети набор виртуальных машин, в котором вы развертываете микрослужбы и управляете ими. Кластер Service Fabric, работающий в Azure, — это ресурс Azure, который развертывается, управляется и отслеживается с помощью Resource Manager.  В этой статье описывается создание шаблона Resource Manager для кластера Service Fabric, запущенного в Azure.  После создания шаблона можно [развернуть кластер в Azure](service-fabric-cluster-creation-via-arm.md).

Безопасность кластера настраивается при его начальной настройке. Позже ее невозможно будет изменить. Перед настройкой кластера ознакомьтесь со статьей [Сценарии защиты кластера Service Fabric][service-fabric-cluster-security]. В Azure для защиты кластера и его конечных точек, проверки подлинности клиентов и шифрования данных Service Fabric требует использования сертификата x509. Azure Active Directory также рекомендуется использовать для обеспечения безопасного доступа к конечным точкам управления. Перед созданием кластера необходимо создать клиенты и пользователей Azure AD.  Дополнительные сведения см.в статье [Set up Azure Active Directory for client authentication](service-fabric-cluster-creation-setup-aad.md) (Настройка Azure AD для проверки подлинности клиентов).

Прежде чем развертывать производственный кластер для запуска рабочих нагрузок, обязательно ознакомьтесь со статьей [Контрольный список готовности рабочей среды](service-fabric-production-readiness-checklist.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-the-resource-manager-template"></a>Создание шаблона Resource Manager
Примеры шаблонов Resource Manager доступны в [образцах Azure на сайте GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). Их можно использовать в качестве отправной точки для создания шаблона кластера.

В этой статье используется пример шаблона [защищенного кластера с пятью узлами][service-fabric-secure-cluster-5-node-1-nodetype] и его параметры. Загрузите файлы *azuredeploy.json* и *azuredeploy.parameters.json* на компьютер и откройте их в текстовом редакторе.

> [!NOTE]
> Для национальных облаков (Azure для государственных организаций, Azure для Китая и Azure для Германии) также необходимо добавить в шаблон следующие параметры `fabricSettings`: `AADLoginEndpoint`, `AADTokenEndpointFormat` и `AADCertEndpointFormat`.

## <a name="add-certificates"></a>Добавление сертификатов
Сертификаты добавляются в шаблон Resource Manager кластера с помощью ссылок на хранилище ключей, которое содержит ключи сертификата. Добавьте эти параметры и значения хранилища ключей в файл параметров шаблона диспетчер ресурсов (*azuredeploy.parameters.jsв*).

### <a name="add-all-certificates-to-the-virtual-machine-scale-set-osprofile"></a>Добавление всех сертификатов в osProfile набора масштабирования виртуальных машин
Каждый сертификат, установленный в кластере, должен быть настроен в разделе **osProfile** ресурса масштабируемого набора (Microsoft. COMPUTE/virtualMachineScaleSets). Это указывает, что поставщик ресурсов должен установить сертификат на виртуальных машинах. Эта установка содержит сертификат кластера, а также все сертификаты безопасности приложений, которые планируется использовать для приложений.

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "osProfile": {
      ...
      "secrets": [
        {
          "sourceVault": {
            "id": "[parameters('sourceVaultValue')]"
          },
          "vaultCertificates": [
            {
              "certificateStore": "[parameters('clusterCertificateStorevalue')]",
              "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
            },
            {
              "certificateStore": "[parameters('applicationCertificateStorevalue')",
              "certificateUrl": "[parameters('applicationCertificateUrlValue')]"
            },
            ...
          ]
        }
      ]
    }
  }
}
```

### <a name="configure-the-service-fabric-cluster-certificate"></a>Настройка сертификата кластера Service Fabric

Сертификат аутентификации кластера должен быть настроен в ресурсе кластера Service Fabric (Microsoft.ServiceFabric/clusters) и расширении Service Fabric для масштабируемых наборов виртуальных машин в ресурсе масштабируемого набора виртуальных машин. Это позволяет поставщику ресурсов Service Fabric настроить его, чтобы использовать для аутентификации кластера и сервера на конечных точках управления.

#### <a name="add-the-certificate-information-the-virtual-machine-scale-set-resource"></a>Добавьте информацию о сертификате для ресурса масштабируемого набора виртуальных машин

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt_',variables('vmNodeType0Name'))]",
            "properties": {
              ...
              "settings": {
                ...
                "certificate": {
                  "commonNames": ["[parameters('certificateCommonName')]"],
                  "x509StoreName": "[parameters('clusterCertificateStoreValue')]"
                },
                ...
              }
            }
          }
        ]
      }
    }
  }
}
```

#### <a name="add-the-certificate-information-to-the-service-fabric-cluster-resource"></a>Добавьте информацию о сертификате в кластерный ресурс Service Fabric

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  "location": "[parameters('clusterLocation')]",
  "dependsOn": [
    "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
  ],
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
  }
}
```

## <a name="add-azure-ad-configuration-to-use-azure-ad-for-client-access"></a>Добавление конфигурации Azure AD для клиентского доступа через Azure AD

Добавьте конфигурации Azure AD в шаблон Resource Manager кластера с помощью ссылок на хранилище ключей, которое содержит ключи сертификата. Добавьте эти параметры и значения Azure AD в файл параметров шаблона диспетчер ресурсов (*azuredeploy.parameters.jsв*). 

> [!NOTE]
> В Linux клиенты и пользователи Azure AD должны быть созданы перед созданием кластера.  Дополнительные сведения см.в статье [Set up Azure Active Directory for client authentication](service-fabric-cluster-creation-setup-aad.md) (Настройка Azure AD для проверки подлинности клиентов).

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
    "azureActiveDirectory": {
      "tenantId": "[parameters('aadTenantId')]",
      "clusterApplication": "[parameters('aadClusterApplicationId')]",
      "clientApplication": "[parameters('aadClientApplicationId')]"
    },
    ...
  }
}
```

## <a name="populate-the-parameter-file-with-the-values"></a>Указание значений в файле параметров

Наконец, используйте полученные значения в результате выполнения команд PowerShell для хранилища ключей и Azure AD, чтобы заполнить файл параметров.

Если вы планируете использовать модули Azure Service Fabric RM для PowerShell, вам не нужно вводить информацию о сертификате кластера. Если же вы хотите, чтобы система сгенерировала самозаверяющий сертификат для обеспечения безопасности кластера, просто задайте нулевые значения. 

> [!NOTE]
> Для сбора и заполнения этих пустых значений параметров модулями RM названия параметров должны совпадать с названиями ниже.

```json
"clusterCertificateThumbprint": {
    "value": ""
},
"certificateCommonName": {
    "value": ""
},
"clusterCertificateUrlValue": {
    "value": ""
},
"sourceVaultvalue": {
    "value": ""
},
```

Если вы используете сертификаты приложения или существующий кластер, отправленный в хранилище ключей, необходимо получить и указать эти сведения.

Модули RM не могут создавать конфигурации Azure AD. Поэтому если вы планируете использовать Azure AD для клиентского доступа, необходимо указать значения.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        ...
        "clusterCertificateStoreValue": {
            "value": "My"
        },
        "clusterCertificateThumbprint": {
            "value": "<thumbprint>"
        },
        "clusterCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myclustercert/4d087088df974e869f1c0978cb100e47"
        },
        "applicationCertificateStorevalue": {
            "value": "My"
        },
        "applicationCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myapplicationcert/2e035058ae274f869c4d0348ca100f08"
        },
        "sourceVaultvalue": {
            "value": "/subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault"
        },
        "aadTenantId": {
            "value": "<guid>"
        },
        "aadClusterApplicationId": {
            "value": "<guid>"
        },
        "aadClientApplicationId": {
            "value": "<guid>"
        },
        ...
    }
}
```

## <a name="test-your-template"></a>Тестирование шаблона
Используйте следующую команду PowerShell для тестирования шаблона Resource Manager с помощью файла параметров.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json
```

Если возникают проблемы и приходят зашифрованные сообщения, в качестве решения попробуйте использовать параметр -Debug (отладка).

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -Debug
```

На приведенной ниже схеме показано, как конфигурация хранилища ключей и Azure AD помещается в шаблон Resource Manager.

![Сопоставление зависимостей Resource Manager][cluster-security-arm-dependency-map]

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда кластер обладает шаблоном, можно [развернуть его в Azure](service-fabric-cluster-creation-via-arm.md).  Перед развертыванием рабочего кластера необходимо ознакомиться со статьей [Контрольный список готовности рабочей среды](service-fabric-production-readiness-checklist.md).

Дополнительные сведения о синтаксисе и свойствах JSON для ресурсов, развертываемых в этой статье:

* [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/clusters)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Network/loadBalancers](/azure/templates/microsoft.network/loadbalancers)
* [Microsoft.Compute/virtualMachineScaleSets](/azure/templates/microsoft.compute/virtualmachinescalesets)

<!-- Links -->
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[service-fabric-secure-cluster-5-node-1-nodetype]: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure
[resource-group-template-deploy]: ../azure-resource-manager/templates/deploy-powershell.md

<!-- Images -->
[cluster-security-arm-dependency-map]: ./media/service-fabric-cluster-creation-create-template/cluster-security-arm-dependency-map.png
