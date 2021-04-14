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
ms.openlocfilehash: e81ce2743d2509ce52f3190218decfa4e518bdad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "70050432"
---
<!-- This contains intro text for the "Get an IoT hub connection string" section in the iot-hub-lang-lang-twin-getstarted.md files-->

В этой статье вы создадите серверную службу, которая добавляет требуемые свойства в двойник устройства, а затем запрашивает реестр удостоверений, чтобы найти все устройства с передаваемыми свойствами, которые были обновлены соответствующим образом. Службе требуется разрешение **service connect** для изменения требуемых свойств двойника устройства и разрешение **registry read** для запроса реестра удостоверений. Политика общего доступа по умолчанию, которая содержит только эти два разрешения, не существует, поэтому необходимо создать ее.
