---
title: Развертывание на выделенном узле
description: Использование выделенного узла для достижения действительной изоляции на уровне узла для рабочих нагрузок службы "экземпляры контейнеров Azure"
ms.topic: article
ms.date: 01/17/2020
author: macolso
ms.author: macolso
ms.openlocfilehash: 68b9b31cdfb55e8150b05e3efd35389320905cdc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98034277"
---
# <a name="deploy-on-dedicated-hosts"></a>Развертывание на выделенных узлах

"Выделенный" — это номер SKU для службы "экземпляры контейнеров Azure" (ACI), предоставляющий изолированную и выделенную среду вычислений для безопасного выполнения контейнеров. Использование выделенного номера SKU в каждой группе контейнеров имеет выделенный физический сервер в центре обработки данных Azure, обеспечивая полную изоляцию рабочей нагрузки для обеспечения безопасности и соответствия требованиям вашей организации. 

Выделенный номер SKU подходит для рабочих нагрузок контейнера, требующих изоляции рабочей нагрузки с точки зрения физического сервера.

## <a name="prerequisites"></a>Предварительные условия

> [!NOTE]
> Из-за некоторых текущих ограничений не все запросы на увеличение лимита гарантированно утверждаются.

* Ограничение по умолчанию для любой подписки, использующей выделенный номер SKU, равно 0. Если вы хотите использовать этот номер SKU для развертывания рабочих контейнеров, создайте [Поддержка Azure][azure-support] , чтобы увеличить это ограничение.

## <a name="use-the-dedicated-sku"></a>Использовать выделенный номер SKU

> [!IMPORTANT]
> Использование выделенного номера SKU доступно только в последней версии API (2019-12-01), которая находится в процессе развертывания. Укажите версию API в шаблоне развертывания.
>

Начиная с API версии 2019-12-01, `sku` в разделе свойств группы контейнеров шаблона развертывания, который требуется для развертывания ACI, существует свойство. В настоящее время это свойство можно использовать как часть шаблона развертывания Azure Resource Manager для ACI. Дополнительные сведения о развертывании ресурсов ACI с помощью шаблона см. в [руководстве по развертыванию группы с несколькими контейнерами с использованием шаблона диспетчер ресурсов](./container-instances-multi-container-group.md). 

`sku`Свойство может иметь одно из следующих значений:
* `Standard` — Стандартный вариант развертывания ACI, который по-прежнему гарантирует безопасность на уровне гипервизора. 
* `Dedicated` — используется для изоляции уровня рабочей нагрузки с выделенными физическими узлами для группы контейнеров.

## <a name="modify-your-json-deployment-template"></a>Изменение шаблона развертывания JSON

В шаблоне развертывания измените или добавьте следующие свойства:
* В разделе `resources` задайте `apiVersion` для значение `2019-12-01` .
* В свойствах группы контейнеров добавьте `sku` свойство со значением `Dedicated` .

Ниже приведен пример фрагмента для раздела Resources шаблона развертывания группы контейнеров, который использует выделенный номер SKU:

```json
[...]
"resources": [
    {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2019-12-01",
        "location": "[resourceGroup().location]",    
        "properties": {
            "sku": "Dedicated",
            "containers": {
                [...]
            }
        }
    }
]
```

Ниже приведен полный шаблон, который развертывает пример группы контейнеров, выполняющей один экземпляр контейнера:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "containerGroupName": {
        "type": "string",
        "defaultValue": "myContainerGroup",
        "metadata": {
          "description": "Container Group name."
        }
      }
    },
    "resources": [
        {
            "name": "[parameters('containerGroupName')]",
            "type": "Microsoft.ContainerInstance/containerGroups",
            "apiVersion": "2019-12-01",
            "location": "[resourceGroup().location]",
            "properties": {
                "sku": "Dedicated",
                "containers": [
                    {
                        "name": "container1",
                        "properties": {
                            "image": "nginx",
                            "command": [
                                "/bin/sh",
                                "-c",
                                "while true; do echo `date`; sleep 1000000; done"
                            ],
                            "ports": [
                                {
                                    "protocol": "TCP",
                                    "port": 80
                                }
                            ],
                            "environmentVariables": [],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    }
                ],
                "restartPolicy": "Always",
                "ipAddress": {
                    "ports": [
                        {
                            "protocol": "TCP",
                            "port": 80
                        }
                    ],
                    "type": "Public"
                },
                "osType": "Linux"
            },
            "tags": {}
        }
    ]
}
```

## <a name="deploy-your-container-group"></a>Развертывание группы контейнеров

Если вы создали и редактировали файл шаблона развертывания на рабочем столе, его можно передать в каталог Cloud Shell, перетащив в него файл. 

Создайте группу ресурсов с помощью команды [az group create][az-group-create].

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Разверните шаблон с помощью команды [AZ Deployment Group Create][az-deployment-group-create] .

```azurecli-interactive
az deployment group create --resource-group myResourceGroup --template-file deployment-template.json
```

В течение нескольких секунд вы должны получить исходный ответ Azure. Успешное развертывание выполняется на выделенном узле.

<!-- LINKS - Internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
