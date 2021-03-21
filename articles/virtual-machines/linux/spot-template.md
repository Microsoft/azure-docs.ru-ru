---
title: Использование шаблона для развертывания виртуальных машин Azure на месте
description: Узнайте, как использовать шаблон для развертывания виртуальных машин Azure на месте для экономии затрат.
author: cynthn
ms.service: virtual-machines
ms.subservice: spot
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 03/25/2020
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: 04319066c59dda5d240f527d86894674a505eaed
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101669347"
---
# <a name="deploy-azure-spot-virtual-machines-using-a-resource-manager-template"></a>Развертывание виртуальных машин Azure с помощью шаблона диспетчер ресурсов

Используя [виртуальные машины Azure](../spot-vms.md) , вы можете использовать преимущества нашей неиспользуемой емкости с значительной экономией затрат. В любой момент, когда Azure нужна емкость, инфраструктура Azure будет выключать виртуальные машины Azure. Таким образом, виртуальные машины Azure для виртуальных машин отлично подходят для рабочих нагрузок, которые могут обрабатывать прерывания, такие как задания пакетной обработки, среды разработки и тестирования, большие вычислительные рабочие нагрузки и многое другое.

Цены на виртуальные машины Azure для машинного хранения являются переменными на основе региона и SKU. Дополнительные сведения см. на страницах с информацией о ценах на виртуальные машины [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) и [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/).

Вы можете задать максимальную цену, которую вы хотите платить, в час, для виртуальной машины. Максимальная цена для виртуальной машины точки Azure может быть задана в долларах США (USD), в которой используется до 5 десятичных разрядов. Например, значение `0.98765` определяет максимальную цену 0,98765 долларов США в час. Если вы укажете для максимальной цены значение `-1`, виртуальная машине не будет вытесняться по критерию цены. Цена на виртуальную машину будет соответствовать текущей цене для виртуальных машин Azure, а также цена для стандартной виртуальной машины, которая когда-либо будет меньше, если есть доступная емкость и квота. Дополнительные сведения о настройке максимальной цены см. в статье о [ценах на виртуальные машины Azure](../spot-vms.md#pricing).


## <a name="use-a-template"></a>Использование шаблона

Для развертывания шаблона виртуальной машины Azure выберите `"apiVersion": "2019-03-01"` или более поздней версии. Добавьте `priority` `evictionPolicy` Свойства и в `billingProfile` шаблон:

```json
"priority": "Spot",
"evictionPolicy": "Deallocate",
"billingProfile": {
    "maxPrice": -1
}
```

Ниже приведен пример шаблона с добавленными свойствами для виртуальной машины в точке Azure. Замените имена ресурсов собственными и `<password>` паролем для учетной записи локального администратора на виртуальной машине.

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2019-03-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
    },
    "variables": {
        "vnetId": "/subscriptions/ec9fcd04-e188-48b9-abfc-abcd515f1836/resourceGroups/spotVM/providers/Microsoft.Network/virtualNetworks/spotVM",
        "subnetName": "default",
        "networkInterfaceName": "spotVMNIC",
        "publicIpAddressName": "spotVM-ip",
        "publicIpAddressType": "Dynamic",
        "publicIpAddressSku": "Basic",
        "virtualMachineName": "spotVM",
        "osDiskType": "Premium_LRS",
        "virtualMachineSize": "Standard_D2s_v3",
        "adminUsername": "azureuser",
        "adminPassword": "<password>",
        "diagnosticsStorageAccountName": "diagstoragespot2019",
        "diagnosticsStorageAccountId": "Microsoft.Storage/storageAccounts/diagstoragespot2019",
        "diagnosticsStorageAccountType": "Standard_LRS",
        "diagnosticsStorageAccountKind": "Storage",
        "subnetRef": "[concat(variables('vnetId'), '/subnets/', variables('subnetName'))]"
    },
    "resources": [
        {
            "name": "spotVM",
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2019-03-01",
            "location": "eastus",
            "dependsOn": [
                "[concat('Microsoft.Network/publicIpAddresses/', variables('publicIpAddressName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            },
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIpAddress": {
                                "id": "[resourceId(resourceGroup().name, 'Microsoft.Network/publicIpAddresses', variables('publicIpAddressName'))]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "name": "[variables('publicIpAddressName')]",
            "type": "Microsoft.Network/publicIpAddresses",
            "apiVersion": "2019-02-01",
            "location": "eastus",
            "properties": {
                "publicIpAllocationMethod": "[variables('publicIpAddressType')]"
            },
            "sku": {
                "name": "[variables('publicIpAddressSku')]"
            }
        },
        {
            "name": "[variables('virtualMachineName')]",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2019-03-01",
            "location": "eastus",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', variables('networkInterfaceName'))]",
                "[concat('Microsoft.Storage/storageAccounts/', variables('diagnosticsStorageAccountName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[variables('virtualMachineSize')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "createOption": "fromImage",
                        "managedDisk": {
                            "storageAccountType": "[variables('osDiskType')]"
                        }
                    },
                    "imageReference": {
                        "publisher": "Canonical",
                        "offer": "UbuntuServer",
                        "sku": "18.04-LTS",
                        "version": "latest"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('networkInterfaceName'))]"
                        }
                    ]
                },
                "osProfile": {
                    "computerName": "[variables('virtualMachineName')]",
                    "adminUsername": "[variables('adminUsername')]",
                    "adminPassword": "[variables('adminPassword')]"
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true,
                        "storageUri": "[concat('https://', variables('diagnosticsStorageAccountName'), '.blob.core.windows.net/')]"
                    }
                },
                "priority": "Spot",
                "evictionPolicy": "Deallocate",
                "billingProfile": {
                    "maxPrice": -1
                }
            }
        },
        {
            "name": "[variables('diagnosticsStorageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "location": "eastus",
            "properties": {},
            "kind": "[variables('diagnosticsStorageAccountKind')]",
            "sku": {
                "name": "[variables('diagnosticsStorageAccountType')]"
            }
        }
    ],
    "outputs": {
        "adminUsername": {
            "type": "string",
            "value": "[variables('adminUsername')]"
        }
    }
}
```

## <a name="simulate-an-eviction"></a>Имитация вытеснения

Вы можете [имитировать вытеснение](/rest/api/compute/virtualmachines/simulateeviction) виртуальной машины в машинном коде Azure, чтобы проверить, насколько хорошо ваше приложение будет ответил внезапного вытеснения. 

Замените следующие сведения: 

- `subscriptionId`
- `resourceGroupName`
- `vmName`


```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете создать виртуальную машину Azure с помощью [Azure PowerShell](../windows/spot-powershell.md) или [Azure CLI](spot-cli.md).

Запросите текущие сведения о ценах с помощью [API розничных цен Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) , чтобы получить сведения о ценах на виртуальные машины Azure для машинного характера. Объект `meterName` и `skuName` будет содержать `Spot` .

Если возникает ошибка, см. раздел [коды ошибок](../error-codes-spot.md).