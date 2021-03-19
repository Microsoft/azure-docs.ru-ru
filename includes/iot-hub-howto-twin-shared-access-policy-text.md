---
title: включить файл
description: включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 08/07/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: e81ce2743d2509ce52f3190218decfa4e518bdad
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "70050432"
---
<!-- This contains intro text for the "Get an IoT hub connection string" section in the iot-hub-lang-lang-twin-getstarted.md files-->

В этой статье вы создадите серверную службу, которая добавляет требуемые свойства в двойник устройства, а затем запрашивает реестр удостоверений, чтобы найти все устройства с передаваемыми свойствами, которые были обновлены соответствующим образом. Службе требуется разрешение **service connect** для изменения требуемых свойств двойника устройства и разрешение **registry read** для запроса реестра удостоверений. Политика общего доступа по умолчанию, которая содержит только эти два разрешения, не существует, поэтому необходимо создать ее.
