---
title: Подготовка мобильных приложений — вызов веб-API для рабочей среды | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать мобильное приложение, вызывающее веб-API. (Подготовка приложений для рабочей среды.)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 08243fd06de289941d8e6a9197ccb349614af056
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104675963"
---
# <a name="prepare-mobile-apps-for-production"></a>Подготовка мобильных приложений для рабочей среды

В этой статье содержатся сведения о том, как улучшить качество и надежность мобильного приложения перед его переносом в рабочую среду.

## <a name="handle-errors"></a>Обработка ошибок

При подготовке мобильного приложения для рабочей среды может возникнуть несколько причин возникновения ошибки. К основным случаям, которые вы будете справляться, относятся аварийные сбои и переходы на взаимодействие. В число других условий, которые следует учитывать, входят ситуации отсутствия сети, простои служб, требования для предоставления прав администратора и другие случаи, связанные с сценариями.

Для каждого типа библиотеки проверки подлинности Майкрософт (MSAL) можно найти образцы кода и вики-страницы, описывающие способы устранения ошибок.

- [Вики-сайт MSAL Android](https://github.com/AzureAD/microsoft-authentication-library-for-android)
- [Вики-сайт MSAL iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki)
- [Вики-сайт MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki)


[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>Дальнейшие действия

Чтобы испытать дополнительные примеры, см. раздел [общедоступные клиентские приложения для настольных систем и мобильных устройств](sample-v2-code.md#desktop-and-mobile-public-client-apps).