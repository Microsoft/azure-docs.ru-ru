---
title: Azure Service Fabric CLI — развертывание сетки sfctl
description: Сведения о sfctl, интерфейсе командной строки Azure Service Fabric. Содержит список команд для создания Service Fabricных ресурсов сетки.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: fb2adafab88eb1d3855cdec8268601fb4e15dcbb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86257281"
---
# <a name="sfctl-mesh-deployment"></a>sfctl mesh deployment
Создание ресурсов службы "Сетка Service Fabric".

## <a name="commands"></a>Команды

|Команда|Описание|
| --- | --- |
| create | Создает развертывание ресурсов Сетки Service Fabric. |

## <a name="sfctl-mesh-deployment-create"></a>sfctl mesh deployment create
Создает развертывание ресурсов Сетки Service Fabric.

### <a name="arguments"></a>Аргументы

|Аргумент|Описание|
| --- | --- |
| --input-yaml-files [обязательный] | Относительные или абсолютные пути к файлам всех файлов YAML или относительный или абсолютный путь к каталогу (с рекурсией), который содержит файлы YAML. |
| --parameters | Относительный или абсолютный путь к YAML-файлу или объекту JSON, который содержит параметры, которые необходимо переопределить. |

### <a name="global-arguments"></a>Глобальные аргументы

|Аргумент|Описание|
| --- | --- |
| --debug | Повышение уровня детализации журнала для включения всех журналов отладки. |
| --help -h | Отображение этого справочного сообщения и выход. |
| --output -o | Формат вывода.  Допустимые значения\: json, jsonc, table, tsv.  Значение по умолчанию\: json. |
| --query | Строка запроса JMESPath. Дополнительные сведения и примеры см. на веб-сайте http\://jmespath.org/. |
| --verbose | Повышение уровня детализации журнала. Чтобы включить полные журналы отладки, используйте параметр --debug. |

### <a name="examples"></a>Примеры

Объединение и развертывание всех ресурсов в кластер путем переопределения параметров, указанных в YAML-файле
``` 
sfctl mesh deployment create --input-yaml-files ./app.yaml,./network.yaml --parameters  
./param.yaml    
```

Объединение и развертывание всех ресурсов каталога в кластер путем переопределения параметров, указанных в YAML-файле

``` 
sfctl mesh deployment create --input-yaml-files ./resources --parameters ./param.yaml
```

Объединяет и развертывает все ресурсы в каталоге в кластере путем переопределения параметров, передаваемых непосредственно в виде объекта JSON.
``` 
sfctl mesh deployment create --input-yaml-files ./resources --parameters "{ 'my_param' :    
{'value' : 'my_value'} }"   
```

## <a name="next-steps"></a>Дальнейшие действия
- [Настройте](service-fabric-cli.md) Service Fabric CLI.
- Узнайте, как использовать интерфейс командной строки Service Fabric, с помощью [примеров сценариев](./scripts/sfctl-upgrade-application.md).
