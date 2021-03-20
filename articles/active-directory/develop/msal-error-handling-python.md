---
title: Обработка ошибок и исключений в MSAL для Python
titleSuffix: Microsoft identity platform
description: Сведения об обработке ошибок и исключений, проблемах условного доступа и повторных попыток в MSAL для приложений Python.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/26/2020
ms.author: marsma
ms.reviewer: saeeda, rayluo
ms.custom: aaddev
ms.openlocfilehash: 3046787393d38ed60b54236f33acc065db90321d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98761086"
---
# <a name="handle-errors-and-exceptions-in-msal-for-python"></a>Обработка ошибок и исключений в MSAL для Python

[!INCLUDE [Active directory error handling introduction](../../../includes/active-directory-develop-error-handling-introduction.md)]

## <a name="error-handling-in-msal-for-python"></a>Обработка ошибок в MSAL для Python

В MSAL для Python большинство ошибок передаются как возвращаемое значение из вызова API. Эта ошибка представлена в виде словаря, содержащего ответ JSON от платформы Microsoft Identity.

* В случае успешного выполнения ответ будет содержать ключ `"access_token"`. Формат ответа определяется протоколом OAuth2. Дополнительные сведения см. в разделе 5.1 [Успешный ответ](https://tools.ietf.org/html/rfc6749#section-5.1).
* Ответ со сведениями об ошибке содержит `"error"` и обычно `"error_description"`. Формат ответа определяется протоколом OAuth2. Дополнительные сведения см. в разделе 5.2 [Ответ со сведениями об ошибке](https://tools.ietf.org/html/rfc6749#section-5.2).

При возвращении ошибки ключ `"error_description"` содержит удобное для чтения сообщение, которое, в свою очередь, обычно содержит код ошибки платформы удостоверений Майкрософт. Дополнительные сведения см. в статье [Коды ошибок аутентификация Azure AD и авторизации](./reference-aadsts-error-codes.md).

В MSAL для Python исключения редки, поскольку большинство ошибок обрабатывается путем возврата значения ошибки. `ValueError`Исключение возникает только в том случае, если возникла ошибка при попытке использования библиотеки, например, если параметры API имеют неправильный формат.

[!INCLUDE [Active directory error handling claims challenges](../../../includes/active-directory-develop-error-handling-claims-challenges.md)]

[!INCLUDE [Active directory error handling retries](../../../includes/active-directory-develop-error-handling-retries.md)]

## <a name="next-steps"></a>Дальнейшие действия

Рекомендуется включить [ведение журнала в MSAL для Python](msal-logging-python.md) , чтобы помочь в диагностике и отладке проблем.
