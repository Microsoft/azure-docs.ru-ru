---
title: включить файл
description: включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 04/03/2020
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 1a0502568c1673bcd7f57d3e9bc9c95ed90bbefa
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80756903"
---
<!-- This tells how to create a custom shared access policy that has service connect and registry RW permissions for your IoT hub and get the connection string for it-->

Чтобы создать политику общего доступа, которая предоставляет разрешения для **подключения к службе** и **записи в реестр** , а затем получить строку подключения для этой политики, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com) выберите **Группы ресурсов**. Выберите группу ресурсов, в которой находится центр, а затем выберите центр из списка ресурсов.

1. В левой части центра выберите **Политики общего доступа**.

1. В верхнем меню над списком политик выберите **Добавить**.

1. В разделе **Добавление политики общего доступа** введите описательное имя политики, например *сервицеандрегистриреадврите*. В разделе **разрешения** выберите **запись реестра** и **Подключение к службе**, а затем щелкните **создать**. (Разрешение на **Чтение реестра** включается автоматически при выборе **параметра запись в реестр**.)

    ![Показывает, как добавить новую политику общего доступа](./media/iot-hub-include-find-service-regrw-connection-string/iot-hub-add-svc-regrw-policy.png)

1. Выберите новую политику из списка политик.

1. В разделе **Общие ключи доступа** нажмите на значок копирования рядом с пунктом **Строка подключения — первичный ключ** и сохраните значение.

    ![Получение строки подключения](./media/iot-hub-include-find-service-regrw-connection-string/iot-hub-get-connection-string.png)

Дополнительные сведения о политиках и разрешениях общего доступа Центра Интернета вещей см. в разделе [Управления доступом и разрешения](../articles/iot-hub/iot-hub-devguide-security.md#access-control-and-permissions).
