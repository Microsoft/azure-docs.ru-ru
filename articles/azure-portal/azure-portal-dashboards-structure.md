---
title: Структура панелей мониторинга Azure
description: Пройдите по структуре JSON панели мониторинга Azure, используя пример панели мониторинга. Содержит ссылку на свойства ресурсов.
ms.topic: conceptual
ms.date: 12/20/2019
ms.openlocfilehash: d37e2fd9c9f6ef6e7ddea6dea002f26f20cd66a7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96745967"
---
# <a name="the-structure-of-azure-dashboards"></a>Структура панелей мониторинга Azure
В этом документе описывается структура панели мониторинга Azure. В качестве примера используется следующая панель мониторинга Azure:

![пример панели мониторинга](./media/azure-portal-dashboards-structure/sample-dashboard.png)

Так как [панели мониторинга Azure являются ресурсами](../azure-resource-manager/management/overview.md), панель мониторинга можно представить в виде кода JSON.  Следующий код JSON представляет панель мониторинга, которая визуализирована выше.

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
        "metadata": {
            "model": {
                "timeRange": {
                    "value": {
                        "relative": {
                            "duration": 24,
                            "timeUnit": 1
                        }
                    },
                    "type": "MsPortalFx.Composition.Configuration.ValueTypes.TimeRange"
                }
            }
        }
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

## <a name="common-resource-properties"></a>Общие свойства ресурса

Давайте разобьем соответствующие разделы JSON.  Свойства верхнего уровня __id__, __name__, __type__, __location__ и __tags__ совместно используются во всех типах ресурсов Azure. Таким образом, они практически не относятся к содержимому панели мониторинга.

### <a name="the-id-property"></a>Свойство ID

Идентификатор ресурса Azure в соответствии с [соглашениями об именовании ресурсов Azure](/azure/architecture/best-practices/resource-naming). Когда портал создает панель мониторинга, она обычно выбирает идентификатор в виде идентификатора GUID, но при создании программными средствами можно использовать любое допустимое имя. 

### <a name="the-name-property"></a>Свойство name
Имя — это сегмент идентификатора ресурса, который не включает в себя подписку, тип ресурса или сведения о группе ресурсов. По сути, это последний сегмент идентификатора ресурса.

### <a name="the-type-property"></a>Свойство type
Все панели мониторинга имеют тип __Microsoft.Portal/dashboards__.

### <a name="the-location-property"></a>Свойство location
В отличие от других ресурсов, панели мониторинга не имеют компонента времени выполнения.  Для панелей мониторинга расположение указывает основное географическое местоположение, где хранится представление JSON панели мониторинга. Значение должно быть одним из кодов расположения, который можно извлечь с помощью [API расположений в ресурсе подписок](/rest/api/resources/subscriptions).

### <a name="the-tags-property"></a>Свойство tags
Теги — это основная возможность ресурсов Azure, которая позволяет организовать ресурс с помощью произвольных пар "имя — значение". Для панелей мониторинга есть один специальный тег, называемый __hidden-title__. Если это свойство панели мониторинга заполнено, оно используется в качестве отображаемого имени панели мониторинга на портале. Идентификаторы ресурсов Azure нельзя переименовать, но теги — можно. Этот тег дает возможность изменять отображаемое имя для панели мониторинга.

`"tags": { "hidden-title": "Created via API" }`

### <a name="the-properties-object"></a>Объект properties
Объект свойств содержит два свойства: __lenses__ и __metadata__. Свойство __lenses__ содержит сведения о плитках на панели мониторинга.  Свойство __metadata__ предназначено для потенциальных будущих возможностей.

### <a name="the-lenses-property"></a>Свойство lenses
Свойство __lenses__ содержит панель мониторинга. Обратите внимание, что объект lenses (групп связанных элементов) в этом примере содержит одно свойство, называемое 0. Группы связанных элементов — это концепция группирования, которая в настоящее время не реализована в панелях мониторинга. На данном этапе все панели мониторинга имеют это одно свойство в объекте lens, называемое 0.

### <a name="the-lens-object"></a>Объект lens
Объект 0 содержит два свойства — __order__ и __parts__.  В текущей версии панелей мониторинга __order__ всегда равен 0. Свойство __Parts__ содержит объект, определяющий отдельные части (также называемые плитками) на панели мониторинга.

Объект __parts__ содержит свойство для каждой части, и именем свойства является номер. Это число не имеет значения. 

### <a name="the-part-object"></a>Объект part
Каждый отдельный объект part имеет свойства __position__ и __metadata__.

### <a name="the-position-object"></a>Объект position
Свойство __position__ содержит сведения о размере и расположении части, выраженные значениями __x__, __y__, __rowSpan__ и __colSpan__. Значения указаны в единицах сетки. Эти единицы сетки отображаются, когда панель мониторинга находится в режиме настройки, как показано ниже. Если вы хотите, чтобы плитка имела ширину двух единиц сетки, высоту одной единицы сетки и местоположение в верхнем левом углу панели инструментов, то объект position будет выглядеть следующим образом:

`location: { x: 0, y: 0, rowSpan: 2, colSpan: 1 }`

![На снимке экрана показано близкое к сетке, в которой выделена одна квадратная единица сетки.](./media/azure-portal-dashboards-structure/grid-units.png)

### <a name="the-metadata-object"></a>Объект metadata
Каждая часть имеет свойство metadata, а объект имеет только одно требуемое свойство под названием __type__. Эта строка указывает порталу, какие плитки отображать. В нашем примере панели мониторинга используются такие типы плиток:


1. `Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart` — для показа метрик мониторинга.
1. `Extension[azure]/HubsExtension/PartType/MarkdownPart` — для показа текста или изображений с базовым форматированием для ссылок, списков и т. д.
1. `Extension[azure]/HubsExtension/PartType/VideoPart` — для показа видео с YouTube, Channel9 и любых других типов видео, которые работают в теге HTML video.
1. `Extension/Microsoft_Azure_Compute/PartType/VirtualMachinePart` — для показа имени и состояния виртуальной машины Azure.

Каждый тип части имеет собственную конфигурацию. Возможные свойства конфигурации: __inputs__, __settings__ и __asset__. 

### <a name="the-inputs-object"></a>Объект inputs
Объект inputs обычно содержит сведения, которые привязывают плитку к экземпляру ресурса.  Часть виртуальной машины на нашем примере панели мониторинга содержит один вход, который использует идентификатор ресурса Azure для выражения привязки.  Этот формат идентификатора ресурса согласован во всех ресурсах Azure.

```json
"inputs":
[
    {
        "name": "id",
        "value": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
    }
]

```
Часть диаграммы метрик имеет один тип входных данных, которые выражают ресурс, к которому выполняется привязка, а также информацию об отображаемых метриках. Вот входные данные для плитки, на которой отображаются метрики входящего и исходящего сетевого трафика.

```json
“inputs”:
[
    {
        "name": "queryInputs",
        "value": 
        {
            "timespan": 
            {
                "duration": "PT1H",
                "start": null,
                "end": null
           },
            "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
            "chartType": 0,
            "metrics": 
            [
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
]

```

### <a name="the-settings-object"></a>Объект settings
Объект settings содержит настраиваемые элементы части.  В нашем примере панели мониторинга часть Markdown использует параметры для хранения пользовательского содержимого markdown, а также настраиваемый заголовок и подзаголовок.

```json
"settings": 
{
    "content": 
    {
        "settings": 
        {
            "content": "This is the team dashboard for the test VM we use on our team. Here are some useful links:\r\n\r\n1. [Getting started](https://www.contoso.com/tsgs)\r\n1. [Troubleshooting guide](https://www.contoso.com/tsgs)\r\n1. [Architecture docs](https://www.contoso.com/tsgs)",
            "title": "Test VM Dashboard",
            "subtitle": "Contoso"
        }
    }
}

```

Аналогично плитка видео имеет собственные параметры, которые содержат указатель на воспроизводимое видео, параметр автоматического воспроизведения и дополнительную информацию о заголовке.

```json
"settings": 
{
   "content": 
    {
        "settings": 
        {
            "title": "",
            "subtitle": "",
            "src": "https://www.youtube.com/watch?v=YcylDIiKaSU&list=PLLasX02E8BPCsnETz0XAMfpLR1LIBqpgs&index=4",
            "autoplay": false
        }
    }
}

```

### <a name="the-asset-object"></a>Объект asset
Для плиток, связанных с управляемыми объектами портала первого класса (ресурсами), связь выражается через объект asset.  В нашем примере панели мониторинга плитка виртуальной машины содержит такое описание ресурса.  Свойство __идинпутнаме__ сообщает порталу, что входные данные идентификатора содержат уникальный идентификатор для ресурса, в данном случае идентификатор ресурсов. Для большинства типов ресурсов Azure ресурсы определены на портале.

`"asset": {    "idInputName": "id",    "type": "VirtualMachine"    }`