---
title: Ведение журнала ошибок и исключений в MSAL для Java
titleSuffix: Microsoft identity platform
description: Узнайте, как вести журнал ошибок и исключений в MSAL для Java
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
ms.openlocfilehash: d3fa13a6751b2d8acf1fc99aecbf174f1823fb0b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98954924"
---
# <a name="logging-in-msal-for-java"></a>Вход в MSAL для Java

[!INCLUDE [MSAL logging introduction](../../../includes/active-directory-develop-error-logging-introduction.md)]

## <a name="msal-for-java-logging"></a>MSAL для ведения журнала Java

MSAL для Java позволяет использовать библиотеку ведения журналов, которая уже используется в приложении, при условии, что оно совместимо с SLF4J. MSAL для Java использует [Простое ведение журнала фасад для Java](http://www.slf4j.org/) (SLF4J) как простое фасадной или абстракцию для различных платформ ведения журналов, таких как [Java. util. Logging](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html), [Logback](http://logback.qos.ch/) и [log4j](https://logging.apache.org/log4j/2.x/). SLF4J позволяет пользователю подключаться к нужной инфраструктуре ведения журналов во время развертывания.

Например, чтобы использовать Logback в качестве платформы ведения журнала в приложении, добавьте зависимость Logback в файл Maven POM для вашего приложения:

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

Затем добавьте файл конфигурации Logback:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">

</configuration>
```

SLF4J автоматически привязывается к Logback во время развертывания. Журналы MSAL будут записаны в консоль.

Инструкции по привязке к другим платформам ведения журналов см. в [руководстве по SLF4J](http://www.slf4j.org/manual.html).

### <a name="personal-and-organization-information"></a>Персональные данные и сведения об Организации

По умолчанию ведение журнала MSAL не записывает и не регистрирует личные или корпоративные данные. В следующем примере ведение журнала личных или организационных данных по умолчанию отключено.

```java
    PublicClientApplication app2 = PublicClientApplication.builder(PUBLIC_CLIENT_ID)
            .authority(AUTHORITY)
            .build();
```

Включите ведение журнала личных данных и организации, задав `logPii()` в построителе клиентских приложений. Если включить ведение журнала личных данных или организации, приложение должно соблюдать ответственность за безопасную обработку данных с высоким уровнем конфиденциальности и соблюдение нормативных требований.

В следующем примере включается ведение журнала личных или организационных данных.

```java
PublicClientApplication app2 = PublicClientApplication.builder(PUBLIC_CLIENT_ID)
        .authority(AUTHORITY)
        .logPii(true)
        .build();
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода см. в статье [примеры кода платформы идентификации Майкрософт](sample-v2-code.md).