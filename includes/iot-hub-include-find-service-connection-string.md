---
title: Включить файл
description: включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 07/16/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 55cdd864d73ce084d994c64233e79d5a58b17def
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "69558457"
---
<!-- This tells how to get the connection string for the service shared access policy of your IoT hub -->

Чтобы получить строку подключения Центра Интернета вещей для политики **службы**, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com) выберите **Группы ресурсов**. Выберите группу ресурсов, в которой находится центр, а затем выберите центр из списка ресурсов.

1. В левой части центра Интернета вещей выберите **Политики общего доступа**.

1. В списке политик выберите политику **службы**.

1. В разделе **Общие ключи доступа** нажмите на значок копирования рядом с пунктом **Строка подключения — первичный ключ** и сохраните значение.

    ![Получение строки подключения](./media/iot-hub-include-find-service-connection-string/iot-hub-get-connection-string.png)

Дополнительные сведения о политиках и разрешениях общего доступа Центра Интернета вещей см. в разделе [Управления доступом и разрешения](../articles/iot-hub/iot-hub-devguide-security.md#access-control-and-permissions).
