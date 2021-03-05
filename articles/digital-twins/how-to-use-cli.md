---
title: Использование CLI для Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Узнайте, как приступить к работе с и использовать интерфейс командной строки Azure Digital двойников.
author: baanders
ms.author: baanders
ms.date: 05/25/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: a24b8b18dd109f1d8ed5acaa7de55ce5a3cc1eb9
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102201113"
---
# <a name="use-the-azure-digital-twins-cli"></a>Использование CLI для Azure Digital Twins

Помимо управления экземпляром Azure Digital двойников в портал Azure, в Azure Digital двойников имеется **набор команд для [Azure CLI](/cli/azure/what-is-azure-cli)** , которые можно использовать для выполнения большинства основных действий со службой, включая:
* Управление экземпляром Digital двойников в Azure
* Управление моделями
* Управление цифровыми двойников
* Управление связями двойника
* Настройка конечных точек
* Управление [маршрутами](concepts-route-events.md)
* Настройка [безопасности](concepts-security.md) с помощью управления доступом на основе ролей Azure (Azure RBAC)

Набор команд называется **AZ DT** и является частью [расширения Интернета вещей Azure для Azure CLI](https://github.com/Azure/azure-iot-cli-extension). Полный список команд и их использование можно просмотреть в справочной документации по `az iot` набору команд: [ *AZ DT* Справочник по командам](/cli/azure/ext/azure-iot/dt).

## <a name="uses-deploy-and-validate"></a>Использование (развертывание и проверка)

В дополнение к обычному управлению экземпляром, интерфейс командной строки также является полезным инструментом для развертывания и проверки.
* Команды плоскости управления можно использовать, чтобы сделать развертывание нового экземпляра повторяющимся или автоматическим.
* Команды плоскости данных можно использовать для быстрой проверки значений в экземпляре и проверки того, что операции выполнены должным образом.

## <a name="get-the-command-set"></a>Получение набора команд

Команды Azure Digital двойников входят в [расширение Azure IOT для Azure CLI (Azure-IOT)](https://github.com/Azure/azure-iot-cli-extension), поэтому выполните следующие действия, чтобы убедиться, что у вас есть Последнее `azure-iot` расширение с командами **AZ DT** .

### <a name="cli-version-requirements"></a>Требования к версии CLI

Если вы используете Azure CLI с PowerShell, для пакета расширения требуется, чтобы в качестве версии Azure CLI использовалась версия **2.3.1** или выше.

Вы можете проверить версию Azure CLI с помощью команды CLI:
```azurecli
az --version
```

Инструкции по установке и обновлению Azure CLI до более новой версии см. в разделе [*установка Azure CLI*](/cli/azure/install-azure-cli).

### <a name="get-the-extension"></a>Получение расширения

Вы можете убедиться, что у вас установлена последняя версия `azure-iot` расширения, выполнив следующие действия. Эти команды можно выполнить в [Azure Cloud Shell](../cloud-shell/overview.md) или [локальном Azure CLI](/cli/azure/install-azure-cli).

[!INCLUDE [digital-twins-cloud-shell-extensions.md](../../includes/digital-twins-cloud-shell-extensions.md)]

## <a name="next-steps"></a>Дальнейшие действия

Изучите интерфейс командной строки и полный набор команд, используя справочные документы:
* [*AZ DT* Справочник по командам](/cli/azure/ext/azure-iot/dt)