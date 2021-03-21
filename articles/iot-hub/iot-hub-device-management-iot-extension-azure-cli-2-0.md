---
title: Управление устройствами Интернета вещей Azure с помощью расширения Интернета вещей для Azure CLI | Документы Майкрософт
description: С помощью расширения Интернета вещей для Azure CLI можно управлять устройствами центра Интернета вещей Azure, используя прямые методы и возможности управления требуемыми свойствами двойника.
author: chrissie926
manager: ''
keywords: управление устройствами Интернета вещей Azure, управление устройствами в Центре Интернета вещей Azure, управление устройствами, Интернет вещей, управление устройствами в Центре Интернета вещей
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 01/16/2018
ms.author: menchi
ms.openlocfilehash: 499c0c065cb38313c98ed9412ab1f08d70dbc654
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102199532"
---
# <a name="use-the-iot-extension-for-azure-cli-for-azure-iot-hub-device-management"></a>Управление устройствами центра Интернета вещей Azure с помощью расширения Интернета вещей для Azure CLI

![Комплексная схема](media/iot-hub-get-started-e2e-diagram/2.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

[Расширение IOT для Azure CLI](https://github.com/Azure/azure-iot-cli-extension) — это расширение IOT с открытым кодом, которое добавляет к возможностям [Azure CLI](/cli/azure/overview). Azure CLI включает команды для взаимодействия с конечными точками Azure Resource Manager и управления. Например, Azure CLI можно использовать для создания центра Интернета вещей или виртуальной машины Azure. Расширение CLI включает службу Azure для ускорения работы Azure CLI, за счет чего вы получаете доступ к дополнительным определенным функциям службы. Расширение IoT предоставляет разработчикам Интернета вещей доступ для командной строки ко всем центрам Интернета вещей, IoT Edge и возможностям службы подготовки устройств для центра Интернета вещей.

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

| Возможность управления          | Задача  |
|----------------------------|-----------|
| Прямые методы             | Выполнение на устройстве таких действий, как запуск и остановка отправки сообщений или перезагрузка устройства.                                        |
| Требуемые свойства двойников    | Перевод устройства в определенные состояния, например включение зеленого светодиодного индикатора или установка 30-минутного интервала отправки данных телеметрии.         |
| Сообщаемые свойства двойника   | Получение зарегистрированного состояния устройства, например данных о том, что сейчас на устройстве мигает индикатор.                                    |
| Теги двойников                  | Хранение метаданных для конкретного устройства в облаке. Например, расположение развертывания торгового автомата.                         |
| Запросы двойника устройства        | Запросите все двойникови устройств, чтобы получить эти двойников с произвольными условиями, такими как идентификация доступных устройств для использования. |

Более подробное объяснение различий и рекомендации по использованию этих параметров см. в статьях [Руководство по обмену данными между устройством и облаком](iot-hub-devguide-d2c-guidance.md) и [Руководство по обмену данными между облаком и устройством](iot-hub-devguide-c2d-guidance.md).

Двойники устройств — это документы JSON, хранящие сведения о состоянии устройства (метаданные, конфигурации и условия). Центр Интернета вещей сохраняет двойник устройства для каждого устройства, подключаемого к нему. Дополнительные сведения о двойниках устройства см. в статье [Начало работы с двойниками устройств](iot-hub-node-node-twin-getstarted.md).

## <a name="what-you-learn"></a>Что вы узнаете

Вы узнаете, как использовать расширение Интернета вещей для Azure CLI с различными вариантами управления на компьютере разработки.

## <a name="what-you-do"></a>Что нужно сделать

Запустите Azure CLI и расширение Интернета вещей для него с разными параметрами управления.

## <a name="what-you-need"></a>Необходимые элементы

* Пройдите одно из руководств по [использованию симулятора Raspberry Pi в подключенном режиме](iot-hub-raspberry-pi-web-simulator-get-started.md), например [Подключение онлайн-симулятора Raspberry Pi к Центру Интернета вещей Azure (Node.js)](iot-hub-raspberry-pi-kit-node-get-started.md). Эти элементы охватывают следующие требования.

  - Активная подписка Azure.
  - Центр Интернета вещей Azure в подписке;
  - клиентское приложение, которое отправляет сообщения в Центр Интернета вещей Azure.

* Во время работы с этим руководством на устройстве должно работать клиентское приложение.

* [Python 2.7x или Python 3.x](https://www.python.org/downloads/)

* Azure CLI. Если вам необходимо выполнить установку, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). Вам понадобится как минимум Azure CLI версии 2.0.70 или более поздней. Для проверки используйте `az –version`.

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

* Установите расширение Интернета вещей. Проще всего запустить `az extension add --name azure-iot`. В [файле сведений расширения Интернета вещей](https://github.com/Azure/azure-iot-cli-extension/blob/master/README.md) описывается несколько способов установки расширения.

## <a name="sign-in-to-your-azure-account"></a>Вход в учетную запись Azure

Войдите в свою учетную запись Azure с помощью следующей команды:

```azurecli
az login
```

## <a name="direct-methods"></a>Прямые методы

```azurecli
az iot hub invoke-device-method --device-id <your device id> \
  --hub-name <your hub name> \
  --method-name <the method name> \
  --method-payload <the method payload>
```

## <a name="device-twin-desired-properties"></a>Требуемые свойства двойника устройства

Установите для требуемого свойства интервал 3000, выполнив следующую команду:

```azurecli
az iot hub device-twin update -n <your hub name> \
  -d <your device id> --set properties.desired.interval = 3000
```

Устройство может прочитать это свойство.

## <a name="device-twin-reported-properties"></a>Сообщаемые свойства двойника устройства

Получите сообщаемые свойства устройства, выполнив следующую команду:

```azurecli
az iot hub device-twin show -n <your hub name> -d <your device id>
```

Одно из сообщаемых свойств двойника — $metadata. $lastUpdated, в котором показано время последнего обновления приложением устройства набора сообщаемых свойств.

## <a name="device-twin-tags"></a>Теги двойника устройства

Отобразите теги и свойства устройства, выполнив следующую команду:

```azurecli
az iot hub device-twin show --hub-name <your hub name> --device-id <your device id>
```

Добавьте на устройство поле role = temperature&humidity, выполнив следующую команду:

```azurecli
az iot hub device-twin update \
  --hub-name <your hub name> \
  --device-id <your device id> \
  --set tags = '{"role":"temperature&humidity"}}'
```

## <a name="device-twin-queries"></a>Запросы двойника устройства

Отправьте запрос на предоставление данных устройств с тегом роли 'temperature&humidity', выполнив следующую команду:

```azurecli
az iot hub query --hub-name <your hub name> \
  --query-command "SELECT * FROM devices WHERE tags.role = 'temperature&humidity'"
```

Отправьте запрос на предоставление данных всех устройств, кроме тех, для которых задан тег роли 'temperature&humidity', выполнив следующую команду:

```azurecli
az iot hub query --hub-name <your hub name> \
  --query-command "SELECT * FROM devices WHERE tags.role != 'temperature&humidity'"
```

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали, как отслеживать сообщения, отправляемые из устройства Интернета вещей в облако Центра Интернета вещей, и отправлять сообщения из этого облака на устройство.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]