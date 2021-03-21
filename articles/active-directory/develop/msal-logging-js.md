---
title: Ведение журнала ошибок и исключений в MSAL.js
titleSuffix: Microsoft identity platform
description: Сведения об записи ошибок и исключений в MSAL.js
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 01/25/2021
ms.author: marsma
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ms.openlocfilehash: d7463e3376847a219750254548c25ca79f4cdfb6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98954823"
---
# <a name="logging-in-msaljs"></a>Ведение журнала в MSAL.js

[!INCLUDE [MSAL logging introduction](../../../includes/active-directory-develop-error-logging-introduction.md)]

## <a name="configure-logging-in-msaljs"></a>Настройка ведения журнала в MSAL.js

Включите ведение журнала в MSAL.js (JavaScript), передав объект средства ведения журнала во время настройки для создания `UserAgentApplication` экземпляра. Этот объект средства ведения журнала имеет следующие свойства.

- `localCallback`: экземпляр обратного вызова, который разработчик может предоставить для использования и публикации журналов особым образом. Реализуйте метод localCallback в зависимости от желаемого способа перенаправления журналов.
- `level` (необязательно): настраиваемый уровень ведения журнала. Поддерживаются следующие уровни ведения журнала: `Error` , `Warning` , `Info` и `Verbose` . Значение по умолчанию — `Info`.
- `piiLoggingEnabled` (необязательно.): Если задано значение true, записывает личные данные и сведения Организации. По умолчанию это значение false, чтобы приложение не заносить персональные данные. Журналы персональных данных никогда не записываются в выходные данные по умолчанию, например в Console, Logcat или NSLog.
- `correlationId` (необязательно): уникальный идентификатор, используемый для отображения запроса с ответом для отладки. По умолчанию используется GUID RFC4122 версии 4 (128 бит).

```javascript
function loggerCallback(logLevel, message, containsPii) {
   console.log(message);
}

var msalConfig = {
    auth: {
        clientId: "<Enter your client id>",
    },
    system: {
        logger: new Msal.Logger(
            loggerCallback , {
                level: Msal.LogLevel.Verbose,
                piiLoggingEnabled: false,
                correlationId: '1234'
            }
        )
    }
}

var UserAgentApplication = new Msal.UserAgentApplication(msalConfig);
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода см. в статье [примеры кода платформы идентификации Майкрософт](sample-v2-code.md).