---
title: включить файл
description: включить файл
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: 75650d7ff0ac647aeb6dace76c270680b1b89347
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98954971"
---
## <a name="enable-logging"></a>Включение ведения журналов

Для упрощения отладки и устранения ошибок проверки подлинности в библиотеке проверки подлинности Майкрософт предусмотрена встроенная поддержка ведения журнала. Ведение журнала — каждая библиотека рассматривается в следующих статьях:

:::row:::
    :::column:::
        - [Ведение журнала в MSAL.NET](../articles/active-directory/develop/msal-logging-dotnet.md)
        - [Ведение журналов в MSAL для Android](../articles/active-directory/develop/msal-logging-android.md)
        - [Ведение журнала в MSAL.js](../articles/active-directory/develop/msal-logging-js.md)
    :::column-end:::
    :::column:::
        - [Вход в MSAL для iOS или macOS](../articles/active-directory/develop/msal-logging-ios.md)
        - [Вход в MSAL для Java](../articles/active-directory/develop/msal-logging-java.md)
        - [Ведение журналов в MSAL для Python](../articles/active-directory/develop/msal-logging-python.md)
    :::column-end:::
:::row-end:::

Ниже приведены некоторые рекомендации по сбору данных.

- Пользователи могут запросить помощь в случае возникновения проблем. Рекомендуется записывать и временно сохранять журналы. Укажите расположение, в котором пользователи могут отправлять журналы. MSAL предоставляет расширения ведения журнала для записи подробных сведений о проверке подлинности.

- Если данные телеметрии доступны, включите ее через MSAL для сбора данных о том, как пользователи входят в приложение.


## <a name="validate-your-integration"></a>Проверка интеграции

Протестируйте интеграцию согласно [контрольному списку для интеграции платформы удостоверений Майкрософт](../articles/active-directory/develop/identity-platform-integration-checklist.md).
