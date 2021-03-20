---
author: baanders
description: Файл включения для Azure Digital Twins — настройка последней версии расширения Интернета вещей
ms.service: digital-twins
ms.topic: include
ms.date: 7/31/2020
ms.author: baanders
ms.openlocfilehash: 984739a728f6ac5e28eeb561e0d7b6ec0485ca13
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90606906"
---
Сначала выполните эту команду, чтобы просмотреть список всех установленных расширений.

```azurecli-interactive
az extension list
```

Данные выводятся в виде массива уже имеющихся у вас расширений. Имена расширений можно увидеть в поле `"name"` для каждой записи списка.

С помощью выходных данных определите, какую из следующих команд нужно выполнить для настройки расширения (команд может быть несколько).
* Если список содержит `azure-iot`, это расширение уже установлено. Выполните эту команду, чтобы убедиться, что у вас установлены самые последние обновления:

   ```azurecli-interactive
   az extension update --name azure-iot
   ```

* Если список **не** содержит `azure-iot`, расширение необходимо установить. Используйте эту команду:

    ```azurecli-interactive
    az extension add --name azure-iot
    ```

* Если список содержит `azure-iot-cli-ext`, это устаревшая версия расширения. Может быть установлена только одна версия расширения, поэтому удалите устаревшее расширение. Используйте эту команду:

   ```azurecli-interactive
   az extension remove --name azure-cli-iot-ext
   ```