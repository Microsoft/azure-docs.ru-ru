---
title: Настройка политики cookie для иммерсивного чтения
titleSuffix: Azure Cognitive Services
description: В этой статье показано, как задать политику cookie для иммерсивного модуля чтения.
services: cognitive-services
author: nitinme
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: conceptual
ms.date: 01/06/2020
ms.author: nitinme
ms.custom: devx-track-js
ms.openlocfilehash: 02901e6b4a75257b2cda8dee84dbe3438dc15c18
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91332378"
---
# <a name="how-to-set-the-cookie-policy-for-the-immersive-reader"></a>Настройка политики cookie для иммерсивное средство чтения

По умолчанию иммерсивное средство чтения отключит использование файлов cookie. При включении использования файлов cookie иммерсивное средство чтения может использовать файлы cookie для сохранения настроек пользователей и отслеживания использования компонентов. При включении использования файлов cookie в иммерсивное средство чтения следует учитывать требования политики соответствия файлов cookie в ЕС. Ведущее приложение отвечает за получение любого необходимого согласия пользователя в соответствии с политикой соответствия файлов cookie ЕС.

Политику файлов cookie можно задать с помощью [параметров](../reference.md#options)иммерсивное средство чтения.

## <a name="enable-cookie-usage"></a>Включить использование файлов cookie

```javascript
var options = {
    'cookiePolicy': ImmersiveReader.CookiePolicy.Enable
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

## <a name="disable-cookie-usage"></a>Отключить использование файлов cookie

```javascript
var options = {
    'cookiePolicy': ImmersiveReader.CookiePolicy.Disable
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь с [кратким руководством для разработчиков Node.js](../quickstarts/client-libraries.md?pivots=programming-language-nodejs), чтобы узнать другие возможности пакета SDK иммерсивного средства чтения при использовании Node.js
* Ознакомьтесь с [руководством для разработчиков Android](../tutorial-android.md), чтобы узнать о других возможностях пакета SDK иммерсивного средства чтения при использовании Java или Kotlin для Android.
* Ознакомьтесь с [руководством для разработчиков iOS](../tutorial-ios.md), чтобы узнать о других возможностях пакета SDK иммерсивного средства чтения при использовании Swift для iOS.
* Ознакомьтесь с [руководством для разработчиков Python](../tutorial-python.md), чтобы узнать другие возможности пакета SDK иммерсивного средства чтения при использовании Python.
* Ознакомьтесь с разделом о [пакете SDK для иммерсивного средства чтения](https://github.com/microsoft/immersive-reader-sdk) и [справочнике по этому пакету](../reference.md).