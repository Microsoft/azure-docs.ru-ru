---
title: Включить файл
description: включить файл
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 07/17/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: e493d1c4f5851ee510ea83e706afce5fbb6f487e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "70049044"
---
<!-- This contains intro text for the "Get an IoT hub connection string" section in the iot-hub-lang-lang-schedule-jobs.md files-->

В этой статье вы создадите серверную службу, которая планирует задание для вызова прямого метода на устройстве, планирует задание для обновления двойника устройства и отслеживает ход выполнения каждого задания. Для выполнения этих операций службе требуются разрешения **registry read** и **registry write**. По умолчанию каждый Центр Интернета вещей создается с помощью политики общего доступа, называемой **registryReadWrite**, которая предоставляет это разрешение.
