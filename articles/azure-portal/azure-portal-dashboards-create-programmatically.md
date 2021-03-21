---
title: Создание панелей мониторинга Azure программными средствами
description: Используйте панель мониторинга в портал Azure в качестве шаблона для программного создания панелей мониторинга Azure. Включает Справочник по JSON.
ms.topic: how-to
ms.date: 12/4/2020
ms.openlocfilehash: bd56dc1c729c5aa7a77e79aa3af3366166fdcfea
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101095171"
---
# <a name="programmatically-create-azure-dashboards"></a>Создание панелей мониторинга Azure программными средствами

В этой статье описывается процесс создания и публикации панелей мониторинга Azure программными средствами. Ниже представлена панель мониторинга, которая используется в этой статье.

![пример панели мониторинга](./media/azure-portal-dashboards-create-programmatically/sample-dashboard.png)

## <a name="overview"></a>Обзор

Общие панели мониторинга в [портал Azure](https://portal.azure.com) являются [ресурсами](../azure-resource-manager/management/overview.md) , такими как виртуальные машины и учетные записи хранения. Управлять ресурсами можно программным путем с помощью [Azure Resource Manager интерфейсов api](/rest/api/) [Azure CLI](/cli/azure)и [команд Azure PowerShell](/powershell/azure/get-started-azureps).

Многие функции создают на этих API, что упрощает управление ресурсами. Каждый из этих API-интерфейсов и средств предлагает способы создания, перечисления, извлечения, изменения и удаления ресурсов. Так как панели мониторинга — это ресурсы, вы можете выбрать предпочтительный API или инструмент для использования.

Независимо от используемого средства для создания панели мониторинга программным путем создается представление JSON объекта панели мониторинга. Этот объект содержит сведения о плитках на панели мониторинга. Сюда входят размеры, позиции, ресурсы, к которым они привязаны, и любые пользовательские настройки.

Самый практичный способ создать этот документ JSON — использовать портал Azure. Вы можете в интерактивном режиме добавлять и позиционировать плитки. Затем экспортируйте JSON и создайте шаблон из результата для последующего использования в скриптах, программах и средствах развертывания.

## <a name="create-a-dashboard"></a>Создание панели мониторинга

Чтобы создать панель мониторинга, выберите в меню [портал Azure](https://portal.azure.com) пункт **панель мониторинга** , а затем выберите пункт **создать панель мониторинга**.

![команда "Создать панель мониторинга"](./media/azure-portal-dashboards-create-programmatically/new-dashboard-command.png)

Используйте коллекцию плиток для поиска и добавления плиток. Чтобы добавить плитки, вам нужно просто перетащить их. Некоторые плитки поддерживают изменение размера с помощью маркера перетаскивания.

![изменить размер маркера перетаскивания](./media/azure-portal-dashboards-create-programmatically/drag-handle.png)

Другие имеют фиксированные размеры, которые можно выбрать в контекстном меню.

![Размер контекстного меню для изменения размера](./media/azure-portal-dashboards-create-programmatically/sizes-context-menu.png)

## <a name="share-the-dashboard"></a>Общий доступ к панели мониторинга

После настройки панели мониторинга следующим шагом является публикация панели мониторинга с помощью команды **Share** .

![Совместное использование панели мониторинга](./media/azure-portal-dashboards-create-programmatically/share-command.png)

При выборе команды **поделиться** предлагается выбрать подписку и группу ресурсов для публикации. Необходимо иметь доступ на запись к выбранной подписке и группе ресурсов. Дополнительные сведения см. в статье [Назначение ролей Azure с помощью портала Azure](../role-based-access-control/role-assignments-portal.md).

![внесение изменений в общий доступ и доступ](./media/azure-portal-dashboards-create-programmatically/sharing-and-access.png)

## <a name="fetch-the-json-representation-of-the-dashboard"></a>Получение представления JSON панели мониторинга

Публикация занимает несколько секунд. После этого следующий шаг — получение JSON с помощью команды **скачать** .

![Скачать представление JSON](./media/azure-portal-dashboards-create-programmatically/download-command.png)

## <a name="create-a-template-from-the-json"></a>Создание шаблона из документа JSON

Следующим шагом является создание шаблона из этого JSON. Используйте этот шаблон программно с соответствующими API-интерфейсами управления ресурсами, программами командной строки или на портале.

Вам не нужно полностью понимать структуру JSON панели мониторинга, чтобы создать шаблон. В большинстве случаев необходимо сохранить структуру и конфигурацию каждой плитки. Затем параметризовать набор ресурсов Azure, на которые указывает плитка. Просмотрите экспортированную панель мониторинга JSON и найдите все вхождения идентификаторов ресурсов Azure. Наш пример панели мониторинга содержит несколько плиток, которые указывают на одну виртуальную машину Azure. Это связано с тем, что наша панель мониторинга рассматривает только один ресурс. При поиске в образце JSON, который включается в конец документа, для "/субскриптионс" можно найти несколько вхождений этого идентификатора.

`/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1`

Чтобы опубликовать эту панель мониторинга для любой виртуальной машины в будущем, следует параметризовать каждое вхождение этой строки в JSON.

Существует два подхода к интерфейсам API, которые создают ресурсы в Azure:

* Императивные API создают по одному ресурсу за раз. Дополнительные сведения см. в статье о [ресурсах DSC](/rest/api/resources/resources).
* Система развертывания на основе шаблона, которая создает несколько зависимых ресурсов с помощью одного вызова API. Дополнительные сведения см. в статье  [развертывание ресурсов с помощью шаблонов диспетчер ресурсов и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md).

Развертывание на основе шаблона поддерживает параметризацию и создание шаблонов. Мы используем этот подход в этой статье.

## <a name="programmatically-create-a-dashboard-from-your-template-using-a-template-deployment"></a>Программное создание панели мониторинга из шаблона с помощью развертывания шаблона

Azure предлагает возможность оркестрации развертывания нескольких ресурсов. Создайте шаблон развертывания, который выражает набор развертываемых ресурсов и связи между ними.  Для каждого ресурса используется единый формат JSON, как и при их последовательном создании. Разница заключается в том, что язык шаблона включает несколько понятий, например переменные, параметры, базовые функции и другие понятия. Этот расширенный синтаксис поддерживается только в контексте развертывания шаблона. Он не работает при использовании с императивными API, которые обсуждались ранее. Дополнительные сведения см. [в разделе сведения о структуре и синтаксисе шаблонов Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

Параметризация следует выполнять с помощью синтаксиса параметров шаблона.  Замените все экземпляры идентификатора ресурса, который мы нашли ранее, как показано ниже.

Пример свойства JSON с жестко запрограммированным ИДЕНТИФИКАТОРом ресурса:

```json
id: "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
```

Пример свойства JSON, преобразованный в параметризованную версию на основе параметров шаблона

```json
id: "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
```

Объявите требуемые метаданные шаблона и параметры в верхней части шаблона JSON следующим образом:

```json

{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualMachineName": {
            "type": "string"
        },
        "virtualMachineResourceGroup": {
            "type": "string"
        },
        "dashboardName": {
            "type": "string"
        }
    },
    "variables": {},

    ... rest of template omitted ...
```
После настройки шаблона разверните его с помощью любого из следующих методов.

* [REST API](/rest/api/resources/deployments);
* [PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
* [Azure CLI](/cli/azure/group/deployment#az-group-deployment-create)
* [Страница развертывания шаблона портал Azure](https://portal.azure.com/#create/Microsoft.Template)

Далее вы увидите две версии нашего примера панели мониторинга JSON. Первая — версия, экспортированная с портала, которая уже была привязана к ресурсу. Вторая — это версия шаблона, которая может быть программно привязана к любой виртуальной машине и развернута с помощью Azure Resource Manager.

### <a name="json-representation-of-our-example-dashboard-before-templating"></a>Представление JSON нашего примера панели мониторинга перед созданием шаблонов

В этом примере показано, что можно увидеть, если вы следовали этой статье. В инструкциях экспортировано представление JSON панели мониторинга, которая уже развернута. Жестко запрограммированные идентификаторы ресурсов показывают, что эта панель мониторинга указывает на определенную виртуальную машину Azure.

```json

{
    "properties": {
        "lenses": {
            "0": {
                "order": 0,
                "parts": {
                    "0": {
                        "position": {
                            "x": 0,
                            "y": 0,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [],
                            "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                            "settings": {
                                "content": {
                                    "settings": {
                                        "content": "## Azure Virtual Machines Overview\r\nNew team members should watch this video to get familiar with Azure Virtual Machines.",
                                        "title": "",
                                        "subtitle": ""
                                    }
                                }
                            }
                        }
                    },
                    "1": {
                        "position": {
                            "x": 3,
                            "y": 0,
                            "rowSpan": 4,
                            "colSpan": 8
                        },
                        "metadata": {
                            "inputs": [],
                            "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                            "settings": {
                                "content": {
                                    "settings": {
                                        "content": "This is the team dashboard for the test VM we use on our team. Here are some useful links:\r\n\r\n1. [Getting started](https://www.contoso.com/tsgs)\r\n1. [Troubleshooting guide](https://www.contoso.com/tsgs)\r\n1. [Architecture docs](https://www.contoso.com/tsgs)",
                                        "title": "Test VM Dashboard",
                                        "subtitle": "Contoso"
                                    }
                                }
                            }
                        }
                    },
                    "2": {
                        "position": {
                            "x": 0,
                            "y": 2,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [],
                            "type": "Extension[azure]/HubsExtension/PartType/VideoPart",
                            "settings": {
                                "content": {
                                    "settings": {
                                        "title": "",
                                        "subtitle": "",
                                        "src": "https://www.youtube.com/watch?v=YcylDIiKaSU&list=PLLasX02E8BPCsnETz0XAMfpLR1LIBqpgs&index=4",
                                        "autoplay": false
                                    }
                                }
                            }
                        }
                    },
                    "3": {
                        "position": {
                            "x": 0,
                            "y": 4,
                            "rowSpan": 3,
                            "colSpan": 11
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Percentage CPU",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "4": {
                        "position": {
                            "x": 0,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Disk Read Operations/Sec",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            },
                                            {
                                                "name": "Disk Write Operations/Sec",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "5": {
                        "position": {
                            "x": 3,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Disk Read Bytes",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            },
                                            {
                                                "name": "Disk Write Bytes",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "6": {
                        "position": {
                            "x": 6,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Network In",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            },
                                            {
                                                "name": "Network Out",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "7": {
                        "position": {
                            "x": 9,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 2
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "id",
                                    "value": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Compute/PartType/VirtualMachinePart",
                            "asset": {
                                "idInputName": "id",
                                "type": "VirtualMachine"
                            },
                            "defaultMenuItemId": "overview"
                        }
                    }
                }
            }
        },
        "metadata": { }
    },
    "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/dashboards/providers/Microsoft.Portal/dashboards/aa9786ae-e159-483f-b05f-1f7f767741a9",
    "name": "aa9786ae-e159-483f-b05f-1f7f767741a9",
    "type": "Microsoft.Portal/dashboards",
    "location": "eastasia",
    "tags": {
        "hidden-title": "Created via API"
    }
}

```

### <a name="template-representation-of-our-example-dashboard"></a>Представление шаблона примера панели мониторинга

В версии шаблона панели мониторинга определены три параметра с именами `virtualMachineName` , `virtualMachineResourceGroup` и `dashboardName` .  Они позволяют настроить эту панель мониторинга для другой виртуальной машины Azure при каждом развертывании. Эту панель мониторинга можно настроить и развернуть программно, чтобы она указывала на любую виртуальную машину Azure. Чтобы протестировать эту функцию, скопируйте следующий шаблон и вставьте его на [страницу развертывания шаблона портал Azure](https://portal.azure.com/#create/Microsoft.Template).

В этом примере панель мониторинга развертывается сама по себе, но язык шаблонов позволяет развертывать несколько ресурсов и в то же время объединять одну или несколько панелей мониторинга.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualMachineName": {
            "type": "string"
        },
        "virtualMachineResourceGroup": {
            "type": "string"
        },
        "dashboardName": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "properties": {
                "lenses": {
                    "0": {
                        "order": 0,
                        "parts": {
                            "0": {
                                "position": {
                                    "x": 0,
                                    "y": 0,
                                    "rowSpan": 2,
                                    "colSpan": 3
                                },
                                "metadata": {
                                    "inputs": [],
                                    "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                                    "settings": {
                                        "content": {
                                            "settings": {
                                                "content": "## Azure Virtual Machines Overview\r\nNew team members should watch this video to get familiar with Azure Virtual Machines.",
                                                "title": "",
                                                "subtitle": ""
                                            }
                                        }
                                    }
                                }
                            },
                            "1": {
                                "position": {
                                    "x": 3,
                                    "y": 0,
                                    "rowSpan": 4,
                                    "colSpan": 8
                                },
                                "metadata": {
                                    "inputs": [],
                                    "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                                    "settings": {
                                        "content": {
                                            "settings": {
                                                "content": "This is the team dashboard for the test VM we use on our team. Here are some useful links:\r\n\r\n1. [Getting started](https://www.contoso.com/tsgs)\r\n1. [Troubleshooting guide](https://www.contoso.com/tsgs)\r\n1. [Architecture docs](https://www.contoso.com/tsgs)",
                                                "title": "Test VM Dashboard",
                                                "subtitle": "Contoso"
                                            }
                                        }
                                    }
                                }
                            },
                            "2": {
                                "position": {
                                    "x": 0,
                                    "y": 2,
                                    "rowSpan": 2,
                                    "colSpan": 3
                                },
                                "metadata": {
                                    "inputs": [],
                                    "type": "Extension[azure]/HubsExtension/PartType/VideoPart",
                                    "settings": {
                                        "content": {
                                            "settings": {
                                                "title": "",
                                                "subtitle": "",
                                                "src": "https://www.youtube.com/watch?v=YcylDIiKaSU&list=PLLasX02E8BPCsnETz0XAMfpLR1LIBqpgs&index=4",
                                                "autoplay": false
                                            }
                                        }
                                    }
                                }
                            },
                            "3": {
                                "position": {
                                    "x": 0,
                                    "y": 4,
                                    "rowSpan": 3,
                                    "colSpan": 11
                                },
                                "metadata": {
                                    "inputs": [
                                        {
                                            "name": "queryInputs",
                                            "value": {
                                                "timespan": {
                                                    "duration": "PT1H",
                                                    "start": null,
                                                    "end": null
                                                },
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Percentage CPU",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    }
                                                ]
                                            }
                                        }
                                    ],
                                    "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                                    "settings": {}
                                }
                            },
                            "4": {
                                "position": {
                                    "x": 0,
                                    "y": 7,
                                    "rowSpan": 2,
                                    "colSpan": 3
                                },
                                "metadata": {
                                    "inputs": [
                                        {
                                            "name": "queryInputs",
                                            "value": {
                                                "timespan": {
                                                    "duration": "PT1H",
                                                    "start": null,
                                                    "end": null
                                                },
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Disk Read Operations/Sec",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    },
                                                    {
                                                        "name": "Disk Write Operations/Sec",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    }
                                                ]
                                            }
                                        }
                                    ],
                                    "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                                    "settings": {}
                                }
                            },
                            "5": {
                                "position": {
                                    "x": 3,
                                    "y": 7,
                                    "rowSpan": 2,
                                    "colSpan": 3
                                },
                                "metadata": {
                                    "inputs": [
                                        {
                                            "name": "queryInputs",
                                            "value": {
                                                "timespan": {
                                                    "duration": "PT1H",
                                                    "start": null,
                                                    "end": null
                                                },
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Disk Read Bytes",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    },
                                                    {
                                                        "name": "Disk Write Bytes",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    }
                                                ]
                                            }
                                        }
                                    ],
                                    "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                                    "settings": {}
                                }
                            },
                            "6": {
                                "position": {
                                    "x": 6,
                                    "y": 7,
                                    "rowSpan": 2,
                                    "colSpan": 3
                                },
                                "metadata": {
                                    "inputs": [
                                        {
                                            "name": "queryInputs",
                                            "value": {
                                                "timespan": {
                                                    "duration": "PT1H",
                                                    "start": null,
                                                    "end": null
                                                },
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Network In",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    },
                                                    {
                                                        "name": "Network Out",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    }
                                                ]
                                            }
                                        }
                                    ],
                                    "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                                    "settings": {}
                                }
                            },
                            "7": {
                                "position": {
                                    "x": 9,
                                    "y": 7,
                                    "rowSpan": 2,
                                    "colSpan": 2
                                },
                                "metadata": {
                                    "inputs": [
                                        {
                                            "name": "id",
                                            "value": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                        }
                                    ],
                                    "type": "Extension/Microsoft_Azure_Compute/PartType/VirtualMachinePart",
                                    "asset": {
                                        "idInputName": "id",
                                        "type": "VirtualMachine"
                                    },
                                    "defaultMenuItemId": "overview"
                                }
                            }
                        }
                    }
                }
            },
            "metadata": { },
            "apiVersion": "2015-08-01-preview",
            "type": "Microsoft.Portal/dashboards",
            "name": "[parameters('dashboardName')]",
            "location": "westus",
            "tags": {
                "hidden-title": "[parameters('dashboardName')]"
            }
        }
    ]
}
```

Теперь, когда вы увидели пример использования параметризованного шаблона для развертывания панели мониторинга, можно попытаться развернуть шаблон с помощью [Azure Resource Manager интерфейсов API](/rest/api/), [Azure CLI](/cli/azure)или [команд Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="programmatically-create-a-dashboard-by-using-azure-cli"></a>Программное создание панели мониторинга с помощью Azure CLI

Подготовьте среду к работе с Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- В этих примерах используется следующая панель мониторинга: [portal-dashboard-template-testvm.jsв](https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-portal/portal-dashboard-template-testvm.json). Замените содержимое в угловых скобках вашими значениями.

Выполните команду [AZ Portal Dashboard](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_create) , чтобы создать панель мониторинга:

```azurecli
az portal dashboard create --resource-group myResourceGroup --name 'Simple VM Dashboard' \
   --input-path portal-dashboard-template-testvm.json --location centralus
```

Панель мониторинга можно обновить с помощью команды [az portal dashboard update](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_update).

```azurecli
az portal dashboard update --resource-group myResourceGroup --name 'Simple VM Dashboard' \
--input-path portal-dashboard-template-testvm.json --location centralus
```

Просмотрите сведения о панели мониторинга, выполнив команду [AZ Portal панель мониторинга Показать](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_show) :

```azurecli
az portal dashboard show --resource-group myResourceGroup --name 'Simple VM Dashboard'
```

Чтобы просмотреть все панели мониторинга для текущей подписки, используйте [az portal dashboard list](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_list).

```azurecli
az portal dashboard list
```

Вы также можете просмотреть все панели мониторинга для группы ресурсов.

```azurecli
az portal dashboard list --resource-group myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о настольных компьютерах см. в разделе [Управление параметрами и](set-preferences.md)настройками портал Azure.

Дополнительные сведения о поддержке Azure CLI для панелей мониторинга см. в описании команды [az portal dashboard](/cli/azure/ext/portal/portal/dashboard).
