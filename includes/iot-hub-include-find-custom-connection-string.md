---
title: Включить файл
description: включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 11/02/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 6f43bbcd83861f7d39de2aa89bbe035c2ff5b809
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "70050407"
---
<!-- This tells how to create a custom shared access policy for your IoT hub and get the connection string for it-->

Чтобы создать политику общего доступа, которая предоставляет разрешения **service connect** и **registry read**, и получить строку подключения для этой политики, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com) выберите **Группы ресурсов**. Выберите группу ресурсов, в которой находится центр, а затем выберите центр из списка ресурсов.

1. В левой части центра выберите **Политики общего доступа**.

1. В верхнем меню над списком политик выберите **Добавить**.

1. В разделе **Добавить политику общего доступа** введите описательное имя политики, например *serviceAndRegistryRead*. В разделе **Разрешения** выберите **Registry read** и **Service connect** и нажмите **Создать**.

    ![Показывает, как добавить новую политику общего доступа](./media/iot-hub-include-find-custom-connection-string/iot-hub-add-custom-policy.png)

1. Выберите новую политику из списка политик.

1. В разделе **Общие ключи доступа** нажмите на значок копирования рядом с пунктом **Строка подключения — первичный ключ** и сохраните значение.

    ![Получение строки подключения](./media/iot-hub-include-find-custom-connection-string/iot-hub-get-connection-string.png)

Дополнительные сведения о политиках и разрешениях общего доступа Центра Интернета вещей см. в разделе [Управления доступом и разрешения](../articles/iot-hub/iot-hub-devguide-security.md#access-control-and-permissions).
