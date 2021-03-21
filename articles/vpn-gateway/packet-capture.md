---
title: 'VPN-шлюз Azure: Настройка записи пакетов'
description: Узнайте о функциях записи пакетов, которые можно использовать на VPN-шлюзах, чтобы упростить причину проблемы.
services: vpn-gateway
author: anzaman
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alzam
ms.openlocfilehash: 0983139d1c9af235eba4c9f99da7bc9dea3f231b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101726620"
---
# <a name="configure-packet-capture-for-vpn-gateways"></a>Настройка записи пакетов для VPN-шлюзов

Проблемы с подключением и производительностью часто сложны. Чтобы ограничить причину проблемы, может потребоваться значительное время и усилия. Запись пакетов помогает ограничить область проблемы определенными частями сети. Она поможет определить, находится ли проблема на стороне клиента сети, в сети Azure или в другом месте. После того как проблема будет устранена, отладка будет более эффективной.

Существуют некоторые распространенные средства записи пакетов. Получение релевантных данных о пакетах с помощью этих средств может оказаться громоздким, особенно в сценариях с большим объемом трафика. Возможности фильтрации, предоставляемые записью пакетов VPN-шлюза Azure, являются основными отличиями. Вы можете использовать запись пакетов VPN-шлюза вместе с часто доступными средствами записи пакетов.

## <a name="vpn-gateway-packet-capture-filtering-capabilities"></a>Возможности фильтрации записи пакетов VPN-шлюза

Вы можете запустить запись пакетов VPN-шлюза на шлюзе или в определенном подключении в зависимости от ваших потребностей. Кроме того, запись пакетов можно выполнять одновременно для нескольких туннелей. Вы можете записывать однонаправленный или двунаправленный трафик, трафик IKE и ESP, а также внутренние пакеты с фильтрацией на VPN-шлюзе.

При изоляции проблем с большим объемом трафика полезно использовать фильтр с пятью кортежами (исходная подсеть, конечная подсеть, порт источника, порт назначения, протокол) и флаги TCP (SYN, ACK, FIN, УРГ, КОМАНДНОМ PSH, RST).

В следующих примерах JSON и схемы JSON содержится объяснение каждого свойства. Ниже приведены некоторые ограничения, которые следует учитывать при выполнении записи пакетов.

- В приведенной здесь схеме фильтр является массивом, но в настоящее время можно использовать только один фильтр.
- Одновременное выполнение нескольких захватов пакетов на уровне шлюза невозможно.
- Невозможно одновременно запустить несколько записей пакетов для одного подключения. Одновременно можно запустить несколько записей пакетов для разных подключений.
- Для каждого шлюза можно параллельно выполнять не более пяти записей пакетов. Эти записи пакетов могут представлять собой сочетание захвата пакетов на уровне шлюза и записи пакетов для каждого подключения.
- Единица для Макспаккетбуфферсизе — байты, а MaxFileSize — МБ.

### <a name="example-json"></a>Пример JSON
```JSON-interactive
{
  "TracingFlags": 11,
  "MaxPacketBufferSize": 120,
  "MaxFileSize": 200,
  "Filters": [
    {
      "SourceSubnets": [
        "20.1.1.0/24"
      ],
      "DestinationSubnets": [
        "10.1.1.0/24"
      ],
      "SourcePort": [
        500
      ],
      "DestinationPort": [
        4500
      ],
      "Protocol": [
        6
      ],
      "TcpFlags": 16,
      "CaptureSingleDirectionTrafficOnly": true
    }
  ]
}
```
### <a name="json-schema"></a>Схема JSON
```JSON-interactive
{
    "type": "object",
    "title": "The Root Schema",
    "description": "The root schema input JSON filter for packet capture",
    "default": {},
    "additionalProperties": true,
    "required": [
        "TracingFlags",
        "MaxPacketBufferSize",
        "MaxFileSize",
        "Filters"
    ],
    "properties": {
        "TracingFlags": {
            "$id": "#/properties/TracingFlags",
            "type": "integer",
            "title": "The Tracingflags Schema",
            "description": "Tracing flags that customer can pass to define which packets are to be captured. Supported values are CaptureESP = 1, CaptureIKE = 2, CaptureOVPN = 8. The final value is OR of the bits.",
            "default": 11,
            "examples": [
                11
            ]
        },
        "MaxPacketBufferSize": {
            "$id": "#/properties/MaxPacketBufferSize",
            "type": "integer",
            "title": "The Maxpacketbuffersize Schema",
            "description": "Maximum buffer size of each packet. The capture will only contain contents of each packet truncated to this size.",
            "default": 120,
            "examples": [
                120
            ]
        },
        "MaxFileSize": {
            "$id": "#/properties/MaxFileSize",
            "type": "integer",
            "title": "The Maxfilesize Schema",
            "description": "Maximum file size of the packet capture file. It is a circular buffer.",
            "default": 100,
            "examples": [
                100
            ]
        },
        "Filters": {
            "$id": "#/properties/Filters",
            "type": "array",
            "title": "The Filters Schema",
            "description": "An array of filters that can be passed to filter inner ESP traffic.",
            "default": [],
            "examples": [
                [
                    {
                        "Protocol": [
                            6
                        ],
                        "CaptureSingleDirectionTrafficOnly": true,
                        "SourcePort": [
                            500
                        ],
                        "DestinationPort": [
                            4500
                        ],
                        "TcpFlags": 16,
                        "SourceSubnets": [
                            "20.1.1.0/24"
                        ],
                        "DestinationSubnets": [
                            "10.1.1.0/24"
                        ]
                    }
                ]
            ],
            "additionalItems": true,
            "items": {
                "$id": "#/properties/Filters/items",
                "type": "object",
                "title": "The Items Schema",
                "description": "An explanation about the purpose of this instance.",
                "default": {},
                "examples": [
                    {
                        "SourcePort": [
                            500
                        ],
                        "DestinationPort": [
                            4500
                        ],
                        "TcpFlags": 16,
                        "SourceSubnets": [
                            "20.1.1.0/24"
                        ],
                        "DestinationSubnets": [
                            "10.1.1.0/24"
                        ],
                        "Protocol": [
                            6
                        ],
                        "CaptureSingleDirectionTrafficOnly": true
                    }
                ],
                "additionalProperties": true,
                "required": [
                    "SourceSubnets",
                    "DestinationSubnets",
                    "SourcePort",
                    "DestinationPort",
                    "Protocol",
                    "TcpFlags",
                    "CaptureSingleDirectionTrafficOnly"
                ],
                "properties": {
                    "SourceSubnets": {
                        "$id": "#/properties/Filters/items/properties/SourceSubnets",
                        "type": "array",
                        "title": "The Sourcesubnets Schema",
                        "description": "An array of source subnets that need to match the Source IP address of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                "20.1.1.0/24"
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/SourceSubnets/items",
                            "type": "string",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": "",
                            "examples": [
                                "20.1.1.0/24"
                            ]
                        }
                    },
                    "DestinationSubnets": {
                        "$id": "#/properties/Filters/items/properties/DestinationSubnets",
                        "type": "array",
                        "title": "The Destinationsubnets Schema",
                        "description": "An array of destination subnets that need to match the Destination IP address of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                "10.1.1.0/24"
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/DestinationSubnets/items",
                            "type": "string",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": "",
                            "examples": [
                                "10.1.1.0/24"
                            ]
                        }
                    },
                    "SourcePort": {
                        "$id": "#/properties/Filters/items/properties/SourcePort",
                        "type": "array",
                        "title": "The Sourceport Schema",
                        "description": "An array of source ports that need to match the Source port of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                500
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/SourcePort/items",
                            "type": "integer",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": 0,
                            "examples": [
                                500
                            ]
                        }
                    },
                    "DestinationPort": {
                        "$id": "#/properties/Filters/items/properties/DestinationPort",
                        "type": "array",
                        "title": "The Destinationport Schema",
                        "description": "An array of destination ports that need to match the Destination port of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                4500
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/DestinationPort/items",
                            "type": "integer",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": 0,
                            "examples": [
                                4500
                            ]
                        }
                    },
                    "Protocol": {
                        "$id": "#/properties/Filters/items/properties/Protocol",
                        "type": "array",
                        "title": "The Protocol Schema",
                        "description": "An array of protocols that need to match the Protocol of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                6
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/Protocol/items",
                            "type": "integer",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": 0,
                            "examples": [
                                6
                            ]
                        }
                    },
                    "TcpFlags": {
                        "$id": "#/properties/Filters/items/properties/TcpFlags",
                        "type": "integer",
                        "title": "The Tcpflags Schema",
                        "description": "A list of TCP flags. The TCP flags set on the packet must match any flag in the list of flags provided. FIN = 0x01,SYN = 0x02,RST = 0x04,PSH = 0x08,ACK = 0x10,URG = 0x20,ECE = 0x40,CWR = 0x80. An OR of flags can be provided.",
                        "default": 0,
                        "examples": [
                            16
                        ]
                    },
                    "CaptureSingleDirectionTrafficOnly": {
                        "$id": "#/properties/Filters/items/properties/CaptureSingleDirectionTrafficOnly",
                        "type": "boolean",
                        "title": "The Capturesingledirectiontrafficonly Schema",
                        "description": "A flags which when set captures reverse traffic also.",
                        "default": false,
                        "examples": [
                            true
                        ]
                    }
                }
            }
        }
    }
}
```

## <a name="packet-capture---portal"></a>Запись пакетов — портал

Запись пакетов можно настроить в портал Azure.

:::image type="content" source="./media/packet-capture/portal.jpg" alt-text="Снимок экрана: запись пакетов на портале." lightbox="./media/packet-capture/portal.jpg":::

## <a name="packet-capture---powershell"></a>Запись пакетов — PowerShell

В следующих примерах показаны команды PowerShell, которые запускают и останавливают запись пакетов. Дополнительные сведения о параметрах параметров см. в разделе [Start-азвиртуалнетворкгатевайпаккеткаптуре](/powershell/module/az.network/start-azvirtualnetworkgatewaypacketcapture).

### <a name="start-packet-capture-for-a-vpn-gateway"></a>Запуск записи пакетов для VPN-шлюза

```azurepowershell-interactive
Start-AzVirtualnetworkGatewayPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayName"
```

Для применения фильтра можно использовать необязательный параметр `-FilterData` .

### <a name="stop-packet-capture-for-a-vpn-gateway"></a>Завершение записи пакетов для VPN-шлюза

```azurepowershell-interactive
Stop-AzVirtualNetworkGatewayPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayName" -SasUrl "YourSASURL"
```

### <a name="start-packet-capture-for-a-vpn-gateway-connection"></a>Запуск записи пакетов для подключения VPN-шлюза

```azurepowershell-interactive
Start-AzVirtualNetworkGatewayConnectionPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayConnectionName"
```

Для применения фильтра можно использовать необязательный параметр `-FilterData` .

### <a name="stop-packet-capture-on-a-vpn-gateway-connection"></a>Прерывать запись пакетов при подключении к VPN-шлюзу

```azurepowershell-interactive
Stop-AzVirtualNetworkGatewayConnectionPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayConnectionName" -SasUrl "YourSASURL"
```

## <a name="key-considerations"></a>Основные рекомендации

- Выполнение записи пакетов может повлиять на производительность. Не забудьте прерывать запись пакетов, если она не нужна.
- Рекомендуемая минимальная длительность записи пакетов составляет 600 секунд. Из-за проблем синхронизации между несколькими компонентами в пути более короткие записи пакетов могут не предоставлять полные данные.
- Файлы данных записи пакетов создаются в формате ПКАП. Используйте Wireshark или другие часто доступные приложения для открытия файлов ПКАП.
- Записи пакетов не поддерживаются на шлюзах, основанных на политиках.
- Если `SASurl` параметр настроен неправильно, трассировка может завершиться с ошибками хранилища. Примеры правильного создания `SASurl` параметра см. в разделе командлета [азвиртуалнетворкгатевайпаккеткаптуре](/powershell/module/az.network/stop-azvirtualnetworkgatewaypacketcapture).



## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о VPN-шлюзе см. в статье [что такое VPN-шлюз?](vpn-gateway-about-vpngateways.md).
