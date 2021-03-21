---
title: Получение пар "ключ-значение" на момент времени
titleSuffix: Azure App Configuration
description: Получение старых пар "ключ-значение" с помощью моментальных снимков на момент времени в конфигурации приложения Azure, которая поддерживает запись изменений значений ключа.
services: azure-app-configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 08/05/2020
ms.openlocfilehash: fa2dbb11b3b8b9afd90c7f6fe3ffe77e2e57c4e6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96929903"
---
# <a name="point-in-time-snapshot"></a>Моментальный снимок на определенный момент времени

В конфигурации приложений Azure сохраняется запись об изменениях, внесенных в значения ключа. Эта запись представляет собой временную шкалу изменений "ключ — значение". Можно восстановить историю любого значения ключа и предоставить свое значение в любое время в течение периода истории ключей (7 дней для хранилищ на уровне Free или 30 дней для магазинов уровня "Стандартный"). С помощью этой функции можно «пройти по времени» назад и получить старое значение ключа. Например, можно восстановить параметры конфигурации, используемые до последнего развертывания, чтобы выполнить откат приложения к предыдущей конфигурации.

## <a name="key-value-retrieval"></a>Получение пары "ключ — значение"

Для извлечения прошлых значений ключа можно использовать портал Azure или CLI. В Azure CLI используйте `az appconfig revision list` , добавив соответствующие параметры для получения необходимых значений.  Укажите экземпляр конфигурации приложения Azure, указав имя хранилища ( `--name <app-config-store-name>` ) или с помощью строки подключения ( `--connection-string <your-connection-string>` ). Ограничьте вывод, указав конкретный момент времени ( `--datetime` ) и указав максимальное число возвращаемых элементов ( `--top` ).

Если Azure CLI не установлен локально, можно также использовать [Azure Cloud Shell](../cloud-shell/overview.md).

Извлеките все записанные изменения в значения ключей.

```azurecli
az appconfig revision list --name <your-app-config-store-name>.
```

Получение всех записанных изменений для ключа `environment` и меток `test` и `prod` .

```azurecli
az appconfig revision list --name <your-app-config-store-name> --key environment --label test,prod
```

Получение всех записанных изменений в иерархическом пространстве ключа `environment:prod` .

```azurecli
az appconfig revision list --name <your-app-config-store-name> --key environment:prod:* 
```

Получение всех записанных изменений для ключа `color` в заданный момент времени.

```azurecli
az appconfig revision list --connection-string <your-app-config-connection-string> --key color --datetime "2019-05-01T11:24:12Z" 
```

Извлеките последние 10 записанных изменений в значения ключа и возвратите только значения для `key` `label` метки времени, и `last_modified` .

```azurecli-interactive
az appconfig revision list --name <your-app-config-store-name> --top 10 --fields key label last_modified
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание веб-приложения ASP.NET Core](./quickstart-aspnet-core-app.md)