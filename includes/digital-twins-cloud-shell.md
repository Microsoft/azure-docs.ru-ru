---
author: baanders
description: файл включения для Azure Digital Twins — настройка Cloud Shell и расширения Интернета вещей
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: d4d9efd99a60c93dbfef2d6f45971781d71e83fb
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105075"
---
Чтобы начать работу с Azure Digital Twins, сначала выполните вход в открытом окне [Azure Cloud Shell](https://shell.azure.com) и задайте контекст оболочки для подписки для этого сеанса. Выполните в Cloud Shell следующие команды:

```azurecli-interactive
az login
az account set --subscription "<your-Azure-subscription-ID>"
```
> [!TIP]
> В команде выше вы также можете использовать имя подписки вместо идентификатора. 

Если вы впервые используете эту подписку с Azure Digital Twins, выполните эту команду, чтобы зарегистрироваться в пространстве имен Azure Digital Twins. (Если вы не уверены, то можете запустить команду снова, даже если вы уже запускали ее.)

```azurecli-interactive
az provider register --namespace 'Microsoft.DigitalTwins'
```

Затем добавьте в Cloud Shell расширение [**Microsoft Azure IoT для Azure CLI**](/cli/azure/service-page/azure%20iot), чтобы разрешить использовать команды для взаимодействия с Azure Digital Twins и другими службами Интернета вещей. Выполните следующую команду, чтобы убедиться, что используется последняя версия расширения:

```azurecli-interactive
az extension add --upgrade -n azure-iot
```

Теперь вы готовы к работе с Azure Digital Twins в Cloud Shell.

Это можно проверить в любой момент, выполнив команду `az dt -h`, чтобы вывести список доступных команд Azure Digital Twins верхнего уровня.