---
title: Включить файл
description: Включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 11/02/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 8d7ac457041474f4e774414b1d5e6f9ed09dc856
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185384"
---
<!-- this tells how to get the connection string for your hub -->
<!-- This assumes the user is looking at his hub in the portal. -->

После создания центра получите для него строку подключения. С ее помощью устройства и приложения будут подключаться к центру. 

1. Щелкните центр, чтобы открыть область "Центр Интернета вещей" с параметрами и другими сведениями. Щелкните **Политики общего доступа**.
   
2. В разделе **Политики общего доступа** выберите политику **iothubowner**. 

3. В разделе **Ключи общего доступа** скопируйте значение **Строка подключения — первичный ключ** для дальнейшего использования.

    ![Получение строки подключения](./media/iot-hub-include-find-connection-string/iot-hub-get-connection-string.png)

    Дополнительные сведения см. в разделе [Контроль доступа](../articles/iot-hub/iot-hub-devguide-security.md) в руководстве разработчика для Центра Интернета вещей Azure.
