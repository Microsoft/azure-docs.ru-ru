---
title: Настройка шлюза для маршрутизации запросов
description: Сведения о настройке шлюза, который обрабатывает входящий трафик для приложений, работающих на платформе "Сетка Service Fabric"
author: georgewallace
ms.topic: conceptual
ms.date: 11/28/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: 40bba03c35568c013e383c08f0a13f765c1cc8bd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99625860"
---
# <a name="configure-a-gateway-resource-to-route-requests"></a>Настройка шлюза для маршрутизации запросов

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Ресурс шлюза используется для перенаправления входящего трафика в сеть, в которой находится приложение. Настройте шлюз, указав правила направления запросов к конкретным службам или конечным точкам в зависимости от структуры запроса. В статье [Introduction to networking in Service Fabric Mesh applications](service-fabric-mesh-networks-and-gateways.md) (Общие сведения о сетевых подключениях в приложениях службы "Сетка Service Fabric") вы найдете дополнительные сведения о сетях и шлюзах в Сетке Service Fabric. 

Ресурсы шлюза объявляются в шаблоне развертывания (JSON или YAML) и зависят от ресурса сети. В этом документе описаны различные свойства, которые можно задать для шлюза, и приведен пример конфигурации шлюза.

## <a name="options-for-configuring-your-gateway-resource"></a>Параметры для настройки ресурса шлюза

Фактически ресурс шлюза выполняет роль моста между сетью приложения и сетью базовой инфраструктуры (сетью `open`). Вам остается лишь настроить шлюз (в предварительной версии Сетки Service Fabric для каждого приложения можно настроить только один шлюз). Объявление ресурса состоит из двух основных частей: метаданных и свойств. 

### <a name="gateway-resource-metadata"></a>Метаданные ресурса шлюза

Шлюз объявляется со следующими метаданными:
* `apiVersion` — необходимо указать "2018-09-01-preview" (в будущем — более позднюю версию);
* `name` — имя для этого шлюза в строковом формате;
* `type` — значение "Microsoft.ServiceFabricMesh/gateways";
* `location` — расположение приложения или сети, обычно в виде ссылки на параметр location в развертывании;
* `dependsOn` — сеть, для которой этот шлюз будет служить точкой входящего трафика.

Примерно так это выглядит в шаблоне развертывания Azure Resource Manager (JSON): 

```json
{
  "apiVersion": "2018-09-01-preview",
  "name": "myGateway",
  "type": "Microsoft.ServiceFabricMesh/gateways",
  "location": "[parameters('location')]",
  "dependsOn": [
    "Microsoft.ServiceFabricMesh/networks/myNetwork"
  ],
  "properties": {
    [...]
  }
}
```

### <a name="gateway-properties"></a>Свойства шлюза

Раздел свойств используется для определения сетей, между которыми размещается шлюз, а также правил для маршрутизации запросов. 

#### <a name="source-and-destination-network"></a>Исходная и целевая сети 

Для каждого шлюза требуются сети `sourceNetwork` и `destinationNetwork`. Исходной считается та сеть, из которой приложение будет получать входящие запросы. Для нее свойство имени должно всегда иметь значение "Open" (Открытая). Целевой считается сеть, в которую направляются запросы. Значение имени для нее должно содержать имя ресурса локальной сети приложения (со ссылкой на ресурс в полном формате). Ниже представлен пример файла конфигурации для подобного развертывания в сети с именем "myNetwork".

```json 
"properties": {
  "description": "Service Fabric Mesh Gateway",
  "sourceNetwork": {
    "name": "Open"
  },
  "destinationNetwork": {
    "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'myNetwork')]"
  },
  [...]
}
```

#### <a name="rules"></a>Правила 

Шлюз может содержать несколько правил маршрутизации, которые определяют способ обработки входящего трафика. Правило маршрутизации задает связь между портом прослушивания и целевой конечной точкой определенного приложения. В правилах маршрутизации для протокола TCP применяется сопоставление 1:1 между портами и конечными точками. Для протокола HTTP можно создать более сложные правила маршрутизации запросов, которые учитывают путь запроса и (необязательно) заголовки. 

Правила маршрутизации задаются отдельно для каждого порта. С каждым входящим портом сопоставляется собственный массив правил, описанный в разделе свойств в конфигурации шлюза. 

#### <a name="tcp-routing-rules"></a>Правила маршрутизации TCP 

Правило маршрутизации TCP состоит из следующих свойств: 
* `name` — ссылка на правило, которая может содержать произвольную строку. 
* `port` — порт, на котором прослушиваются входящие запросы. 
* `destination` — спецификация конечной точки, которая содержит `applicationName`, `serviceName` и имя конечной точки `endpointName`, на которую нужно направлять запросы.

Вот пример правила маршрутизации для протокола TCP:

```json
"properties": {
  [...]
  "tcp": [
    {
      "name": "web",
      "port": 80,
      "destination": {
        "applicationName": "myApp",
        "serviceName": "myService",
        "endpointName": "myListener"
      }
    }
  ]
}
```


#### <a name="http-routing-rules"></a>Правила маршрутизации для HTTP 

Правило маршрутизации HTTP состоит из следующих свойств: 
* `name` — ссылка на правило, которая может содержать произвольную строку. 
* `port` — порт, на котором прослушиваются входящие запросы. 
* `hosts` — массив политик, которые применяются к запросам, поступающим на указанный выше порт для разных "узлов". Узлами считаются приложения и службы, которые выполняются в сети и могут обслуживать входящие запросы, например веб-приложения. Политики для узлов оцениваются в строгом порядке, а значит следующие свойства нужно создавать в порядке убывания точности.
    * `name` — DNS-имя узла, к которому применяются описанные далее правила маршрутизации. Если вы укажете здесь символ "*", правила маршрутизации будут применяться ко всем узлам.
    * `routes` — массив политик для конкретного узла.
        * `match` — описание структуры входящих запросов, которое будет применяться к этому правилу, на основе `path`.
            * `path` — содержит `value` (входящий URI), `rewrite` (способ переадресации запроса) и `type` (сейчас может быть только "Prefix" (Префикс)).
            * `header` — необязательный массив значений заголовков для сопоставления с заголовками запроса, который соответствует указанному выше пути.
              * Каждая запись содержит `name` (строковое имя сопоставляемого заголовка), `value` (строковое значение этого заголовка в запросе) и `type` (сейчас может быть только "Exact" (Точный)).
        * `destination` — соответствующий условиям запрос будет направлен в место назначения, которое задается здесь с помощью `applicationName`, `serviceName` и `endpointName`.

Вот пример правила маршрутизации для HTTP, которое применяется к запросам, поступающим на порт 80, для любых узлов, на которых работают приложения в этой сети. Если структура URL-адреса запроса соответствует указанной спецификации пути (например, `<IPAddress>:80/pickme/<requestContent>`), такой запрос будет направлен на конечную точку `myListener`.  

```json
"properties": {
  [...]
  "http": [
    {
      "name": "web",
      "port": 80,
      "hosts": [
        {
          "name": "*",
          "routes": [
            {
              "match": {
                "path": {
                  "value": "/pickme",
                  "rewrite": "/",
                  "type": "Prefix"
                }
              },
              "destination": {
                "applicationName": "meshApp",
                "serviceName": "myService",
                "endpointName": "myListener"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

## <a name="sample-config-for-a-gateway-resource"></a>Пример конфигурации для ресурса шлюза 

Здесь представлен полный пример конфигурации для ресурса шлюза (это адаптированный фрагмент из примера для входящего трафика, размещенного в [репозитории примеров для Сетки](https://github.com/Azure-Samples/service-fabric-mesh/blob/2018-09-01-preview/templates/ingress/meshingress.linux.json)):

```json
{
  "apiVersion": "2018-09-01-preview",
  "name": "ingressGatewayLinux",
  "type": "Microsoft.ServiceFabricMesh/gateways",
  "location": "[parameters('location')]",
  "dependsOn": [
    "Microsoft.ServiceFabricMesh/networks/meshNetworkLinux"
  ],
  "properties": {
    "description": "Service Fabric Mesh Gateway for Linux mesh samples.",
    "sourceNetwork": {
      "name": "Open"
    },
    "destinationNetwork": {
      "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'meshNetworkLinux')]"
    },
    "http": [
      {
        "name": "web",
        "port": 80,
        "hosts": [
          {
            "name": "*",
            "routes": [
              {
                "match": {
                  "path": {
                    "value": "/hello",
                    "rewrite": "/",
                    "type": "Prefix"
                  }
                },
                "destination": {
                  "applicationName": "meshAppLinux",
                  "serviceName": "helloWorldService",
                  "endpointName": "helloWorldListener"
                }
              },
              {
                "match": {
                  "path": {
                    "value": "/counter",
                    "rewrite": "/",
                    "type": "Prefix"
                  }
                },
                "destination": {
                  "applicationName": "meshAppLinux",
                  "serviceName": "counterService",
                  "endpointName": "counterServiceListener"
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```

Для этого шлюза настраивается приложение Linux "meshAppLinux", содержащее по меньшей мере две службы, helloWorldService и counterService, которые прослушивают порт 80. В зависимости от структуры URL-адреса входящего запроса этот шлюз перенаправит запрос к одной из этих служб. 
* " \<IPAddress> : 80/helloWorld/" приведет к тому, что \<request\> запрос будет направлен в "хелловорлдлистенер" в helloWorldService. 
* " \<IPAddress> : 80/Counter/" приведет к тому, что \<request\> запрос будет направлен в "каунтерлистенер" в каунтерсервице. 

## <a name="next-steps"></a>Дальнейшие действия
* Разверните [пример для входящих данных](https://github.com/Azure-Samples/service-fabric-mesh/tree/2018-09-01-preview/templates/ingress), чтобы увидеть шлюзы в действии.
