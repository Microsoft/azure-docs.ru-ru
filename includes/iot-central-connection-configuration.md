---
title: включить файл
description: включить файл
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 11/03/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: e4f9fd537d7743a5bbb9d129b21c4bf0a529d32d
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491156"
---
При запуске примера приложения для устройства на более позднем шаге этого руководства вам потребуются следующие значения конфигурации:

* Область идентификатора: В приложении IoT Central перейдите к пункту **Администрирование > Подключение устройства**. Запишите значение **области идентификатора**.
* Первичный ключ группы: В приложении IoT Central перейдите к пункту **Администрирование > Подключение устройства > SAS-IoT-Devices**. Запишите значение **первичного ключа** подписанного URL-адреса.

Используйте Cloud Shell, чтобы создать ключ устройства на основе полученного первичного ключа группы:

```azurecli-interactive
az extension add --name azure-iot
az iot central device compute-device-key --device-id sample-device-01 --pk <the group primary key value>
```

Запишите созданный ключ устройства. Он понадобится на более позднем этапе работы с этим учебником.
