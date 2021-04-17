---
title: Включить файл
description: включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 08/07/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 3386cb51a8a728576f6615002d6154d89ca662c1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95993317"
---
<!-- This tells how to get the connection string for the registryReadWrite shared access policy of your IoT hub -->

Чтобы получить строку подключения Центра Интернета вещей для политики **registryReadWrite**, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com) выберите **Группы ресурсов**. Выберите группу ресурсов, в которой находится центр, а затем выберите центр из списка ресурсов.

2. В левой части центра выберите **Политики общего доступа**.

3. В списке политик выберите политику **registryReadWrite**.

4. В разделе **Общие ключи доступа** нажмите на значок копирования рядом с пунктом **Строка подключения — первичный ключ** и сохраните значение.

    ![Получение строки подключения](./media/iot-hub-include-find-registryrw-connection-string/iot-hub-get-connection-vs2019.png)

Дополнительные сведения о политиках и разрешениях общего доступа Центра Интернета вещей см. в разделе [Управления доступом и разрешения](../articles/iot-hub/iot-hub-devguide-security.md#access-control-and-permissions).
