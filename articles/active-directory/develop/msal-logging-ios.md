---
title: Ведение журнала ошибок и исключений в MSAL для iOS/macOS
titleSuffix: Microsoft identity platform
description: Узнайте, как вести журнал ошибок и исключений в MSAL для iOS/macOS
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
ms.openlocfilehash: ee3837b75d586238e7ca6ac85434cc56f592929d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98763488"
---
# <a name="logging-in-msal-for-iosmacos"></a>Вход в MSAL для iOS или macOS

[!INCLUDE [MSAL logging introduction](../../../includes/active-directory-develop-error-logging-introduction.md)]

## <a name="objective-c"></a>[Objective-C](#tab/objc)

## <a name="msal-for-ios-and-macos-logging-objc"></a>MSAL для iOS и macOS Logging-ObjC

Настройте обратный вызов для записи журнала MSAL и его включения в журнал своего приложения. Сигнатура для обратного вызова выглядит следующим образом:

```objc
/*!
    The LogCallback block for the MSAL logger
 
    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Пример:

```objc
[MSALGlobalConfig.loggerConfig setLogCallback:^(MSALLogLevel level, NSString *message, BOOL containsPII)
    {
        if (!containsPII)
        {
#if DEBUG
            // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
            NSLog(@"MSAL log: %@", message);
#endif
        }
    }];
```

### <a name="personal-data"></a>Персональные данные

По умолчанию MSAL не записывает персональные данные и не регистрирует их. Библиотека позволяет разработчикам приложений включать это через свойство в классе Мсаллогжер. При включении `pii.Enabled` приложение отвечает за безопасную обработку данных с высоким уровнем конфиденциальности и соблюдение нормативных требований.

```objc
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = YES;

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = NO;
```

### <a name="logging-levels"></a>Уровни ведения журнала

Чтобы задать уровень ведения журнала при использовании MSAL для iOS и macOS, используйте одно из следующих значений:

|Level  |Описание |
|---------|---------|
| `MSALLogLevelNothing`| Отключить ведение журнала |
| `MSALLogLevelError` | Уровень по умолчанию, выводит сведения только при возникновении ошибок |
| `MSALLogLevelWarning` | Предупреждения |
| `MSALLogLevelInfo` |  Точки входа библиотеки с параметрами и различными операциями цепочки ключей |
|`MSALLogLevelVerbose`     |  Трассировка API |

Пример:

```objc
MSALGlobalConfig.loggerConfig.logLevel = MSALLogLevelVerbose;
 ```

 ### <a name="log-message-format"></a>Формат сообщения журнала

Часть сообщения журнала MSAL имеет формат `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`

Пример:

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Предоставление идентификаторов корреляции и отметок времени полезно для отслеживания проблем. Сведения о метке времени и ИДЕНТИФИКАТОРе корреляции доступны в сообщении журнала. Единственное надежное место для извлечения — это MSAL сообщения ведения журнала.

## <a name="swift"></a>[Swift](#tab/swift)

## <a name="msal-for-ios-and-macos-logging-swift"></a>Журналы MSAL для iOS и macOS — SWIFT

Настройте обратный вызов для записи журнала MSAL и его включения в журнал своего приложения. Сигнатура (представленная в цели-C) для обратного вызова выглядит следующим образом:

```objc
/*!
    The LogCallback block for the MSAL logger
 
    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Пример:

```swift
MSALGlobalConfig.loggerConfig.setLogCallback { (level, message, containsPII) in
    if let message = message, !containsPII
    {
#if DEBUG
    // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
    print("MSAL log: \(message)")
#endif
    }
}
```

### <a name="personal-data"></a>Персональные данные

По умолчанию MSAL не записывает персональные данные и не регистрирует их. Библиотека позволяет разработчикам приложений включать это через свойство в классе Мсаллогжер. При включении `pii.Enabled` приложение отвечает за безопасную обработку данных с высоким уровнем конфиденциальности и соблюдение нормативных требований.

```swift
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = true

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = false
```

### <a name="logging-levels"></a>Уровни ведения журнала

Чтобы задать уровень ведения журнала при использовании MSAL для iOS и macOS, используйте одно из следующих значений:

|Level  |Описание |
|---------|---------|
| `MSALLogLevelNothing`| Отключить ведение журнала |
| `MSALLogLevelError` | Уровень по умолчанию, выводит сведения только при возникновении ошибок |
| `MSALLogLevelWarning` | Предупреждения |
| `MSALLogLevelInfo` |  Точки входа библиотеки с параметрами и различными операциями цепочки ключей |
|`MSALLogLevelVerbose`     |  Трассировка API |

Пример:

```swift
MSALGlobalConfig.loggerConfig.logLevel = .verbose
 ```

### <a name="log-message-format"></a>Формат сообщения журнала

Часть сообщения журнала MSAL имеет формат `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`

Пример:

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Предоставление идентификаторов корреляции и отметок времени полезно для отслеживания проблем. Сведения о метке времени и ИДЕНТИФИКАТОРе корреляции доступны в сообщении журнала. Единственное надежное место для извлечения — это MSAL сообщения ведения журнала.

---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода см. в статье [примеры кода платформы идентификации Майкрософт](sample-v2-code.md).