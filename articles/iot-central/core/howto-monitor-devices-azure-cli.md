---
title: Мониторинг подключения устройств с помощью обозревателя Azure IoT Central
description: Отслеживайте сообщения с устройств и изменения двойников устройств с помощью интерфейса командной строки обозревателя IoT Central.
author: viv-liu
ms.author: viviali
ms.date: 03/27/2020
ms.topic: how-to
ms.service: iot-central
ms.custom: devx-track-azurecli, device-developer
services: iot-central
manager: corywink
ms.openlocfilehash: 62981686c7aadc713c4abc78075be8613fe0af45
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102199312"
---
# <a name="monitor-device-connectivity-using-azure-cli"></a>Мониторинг подключения устройств с помощью Azure CLI

*Этот раздел относится к разработчикам устройств и сборщикам решений.*

Используйте расширение Интернета вещей Azure CLI, чтобы просмотреть сообщения, отправляемые устройствами IoT Central и наблюдать за изменениями в двойникаах устройств. Это средство можно использовать для отладки и наблюдения за подключением устройств, а также для диагностики проблем, связанных с сообщениями устройств, которые не находятся в облаке или устройствах, не отвечающих на изменения двойника.

[Дополнительные сведения см. в справочнике по расширениям Azure CLI.](/cli/azure/ext/azure-iot/iot/central)

## <a name="prerequisites"></a>Предварительные требования

+ Azure CLI установлен и имеет версию 2.7.0 или более позднюю. Проверьте версию Azure CLI, выполнив `az --version` . Узнайте, как устанавливать и обновлять [документы Azure CLI](/cli/azure/install-azure-cli)
+ Рабочая или учебная учетная запись в Azure, добавленная в качестве пользователя в IoT Centralое приложение.

## <a name="install-the-iot-central-extension"></a>Установка расширения IoT Central

Чтобы установить средство, в командной строке выполните следующую команду:

```azurecli
az extension add --name azure-iot
```

Проверьте версию расширения, выполнив команду:

```azurecli
az --version
```

Вы должны увидеть расширение Azure-IOT 0.9.9 или более поздней версии. Если это не так, выполните:

```azurecli
az extension update --name azure-iot
```

## <a name="using-the-extension"></a>Использование расширения

В следующих разделах описаны общие команды и параметры, которые можно использовать при запуске `az iot central` . Чтобы просмотреть полный набор команд и параметров, передайте `--help` `az iot central` его подкоманды или любую из ее подкоманд.

### <a name="login"></a>Имя входа

Начните с входа в Azure CLI. 

```azurecli
az login
```

### <a name="get-the-application-id-of-your-iot-central-app"></a>Получение идентификатора приложения IoT Centralного приложения
В окне **Администрирование/параметры приложения** скопируйте **идентификатор приложения**. Это значение используется в последующих шагах.

### <a name="monitor-messages"></a>Мониторинг сообщений
Отслеживайте сообщения, которые отправляются в приложение IoT Central с устройств. Выходные данные включают все заголовки и заметки.

```azurecli
az iot central diagnostics monitor-events --app-id <app-id> --properties all
```

### <a name="view-device-properties"></a>Просмотр свойств устройства
Просмотр текущих свойств чтения и чтения и записи для данного устройства.

```azurecli
az iot central device twin show --app-id <app-id> --device-id <device-id>
```

## <a name="next-steps"></a>Дальнейшие действия

Если вы являетесь разработчиком устройства, рекомендуем следующий шаг — прочитать сведения о [подключении устройства в Azure IOT Central](./concepts-get-connected.md).