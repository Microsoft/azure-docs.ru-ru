---
title: Включить файл
description: Включить файл
services: functions
author: craigshoemaker
manager: gwallace
ms.service: azure-functions
ms.topic: include
ms.date: 01/28/2020
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: 6a37850eb6536c5399d63144e60ea210fbc194d8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "77198430"
---
#### <a name="azure-storage-sdk-version-in-functions-1x"></a>Версия пакета SDK для службы хранилища Azure в решении "Функции" 1.x

В решении "Функции" 1.x триггеры и привязки службы хранилища используют версию 7.2.1 пакета SDK для службы хранилища Azure (пакет NuGet [WindowsAzure.Storage](https://www.nuget.org/packages/WindowsAzure.Storage/7.2.1)). Если указать другую версию пакета SDK для службы хранилища и использовать привязку к типу пакета SDK для службы хранилища в сигнатуре функции, то среда выполнения "Функции" может сообщить, что выполнить привязку к этому типу невозможно. Чтобы избежать этого, убедитесь, в вашем проекте указан пакет [WindowsAzure.Storage 7.2.1](https://www.nuget.org/packages/WindowsAzure.Storage/7.2.1).
