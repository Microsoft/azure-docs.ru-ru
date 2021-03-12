---
title: Клиентские библиотеки и интерфейсы API для служб связи Azure
titleSuffix: An Azure Communication Services concept document
description: Дополнительные сведения о клиентских библиотеках служб связи Azure и интерфейсах API для службы взаимодействия.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/18/2020
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: b5c75f62f730c1bbc09a66e81f70f0e71a246fde
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102613363"
---
# <a name="client-libraries-and-rest-apis"></a>Клиентские библиотеки и интерфейсы REST API

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]


Возможности служб связи Azure концептуально организованы в шесть областей. Некоторые области имеют полностью открытые клиентские библиотеки с открытым кодом. Вызывающая клиентская библиотека использует частные сетевые интерфейсы и в настоящее время закрыто источником, а библиотека разговора включает в себя зависимость с закрытым исходным кодом. Примеры и дополнительные технические сведения для клиентских библиотек публикуются в [репозитории GitHub служб связи Azure](https://github.com/Azure/communication).

## <a name="client-libraries"></a>Клиентские библиотеки

| Сборка               | Протоколы             |Открытие и закрытие источника| Пространства имен                          | Характеристики                                                      |
| ---------------------- | --------------------- | ---|-------------------------- | --------------------------------------------------------------------------- |
| Azure Resource Manager | REST | Открыть            | Azure. ResourceManager. Обмен данными | Подготавливайте ресурсы служб связи и управляйте ими             |
| Распространенные                 | REST | Открыть               | Azure. Communication. Common          | Предоставляет базовые типы для других клиентских библиотек |
| Идентификация         | REST | Открыть               | Azure. Communication. Identity  | Управление пользователями, маркеры доступа |
| Чат                   | ПРОЧее с собственными сигналами | Открыть с помощью сигнального пакета с закрытым источником    | Azure. Communication. чат            | Добавление в приложения текста, основанного на режиме реального времени  |
| SMS                    | REST | Открыть              | Azure. Communication. SMS             | Отправка и получение текстовых сообщений |
| Вызов                | Собственный транспорт | Закрытое |Azure. Communication. вызов         | Использование голоса, видео, совместного использования экрана и других возможностей обмена данными в реальном времени          |

Обратите внимание, что клиентские библиотеки Azure Resource Manager, удостоверений и SMS ориентированы на интеграцию служб, и во многих случаях проблемы безопасности возникают, если вы интегрируете эти функции в приложения для конечных пользователей. Клиентские библиотеки Common и Chat подходят для служб и клиентских приложений. Вызывающая клиентская библиотека разработана для клиентских приложений. Клиентская библиотека, нацеленная на сценарии службы, находится в разработке.

### <a name="languages-and-publishing-locations"></a>Языки и расположения для публикации

Сведения о расположении публикации для отдельных пакетов клиентских библиотек приведены ниже.

| Область           | JavaScript | .NET | Python | Java SE | iOS | Android | Другое                          |
| -------------- | ---------- | ---- | ------ | ---- | -------------- | -------------- | ------------------------------ |
| Azure Resource Manager | -         | [NuGet](https://www.nuget.org/packages/Azure.ResourceManager.Communication)    |   [PyPi](https://pypi.org/project/azure-mgmt-communication/)    |  -  | -              | -  | [Через GitHub](https://github.com/Azure/azure-sdk-for-go/releases/tag/v46.3.0) |
| Распространенные         | [npm](https://www.npmjs.com/package/@azure/communication-common)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Common/)    | Недоступно      | [Maven](https://search.maven.org/search?q=a:azure-communication-common)   | [GitHub](https://github.com/Azure/azure-sdk-for-ios/releases)            | [Maven](https://search.maven.org/artifact/com.azure.android/azure-communication-common)             | -                              |
| Идентификация | [npm](https://www.npmjs.com/package/@azure/communication-identity)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Identity)    | [PyPi](https://pypi.org/project/azure-communication-identity/)      | [Maven](https://search.maven.org/search?q=a:azure-communication-identity)   | -              | -              | -                            |
| Чат           | [npm](https://www.npmjs.com/package/@azure/communication-chat)        | [NuGet](https://www.nuget.org/packages/Azure.Communication.Chat)     | [PyPi](https://pypi.org/project/azure-communication-chat/)     | [Maven](https://search.maven.org/search?q=a:azure-communication-chat)   | [GitHub](https://github.com/Azure/azure-sdk-for-ios/releases)  | [Maven](https://search.maven.org/search?q=a:azure-communication-chat)   | -                              |
| SMS            | [npm](https://www.npmjs.com/package/@azure/communication-sms)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Sms)    | [PyPi](https://pypi.org/project/azure-communication-sms/)       | [Maven](https://search.maven.org/artifact/com.azure/azure-communication-sms)   | -              | -              | -                              |
| Вызов        | [npm](https://www.npmjs.com/package/@azure/communication-calling)         | -      | -      | -     | [GitHub](https://github.com/Azure/Communication/releases)     | [Maven](https://search.maven.org/artifact/com.azure.android/azure-communication-calling/)            | -                              |
| Справочная документация     | [docs](https://azure.github.io/azure-sdk-for-js/communication.html)         | [docs](https://azure.github.io/azure-sdk-for-net/communication.html)      | -      | [docs](http://azure.github.io/azure-sdk-for-java/communication.html)     | [docs](/objectivec/communication-services/calling/)      | [docs](/java/api/com.azure.communication.calling)            | -                              |

## <a name="rest-apis"></a>Интерфейсы REST API

API-интерфейсы служб связи задокументированы вместе с другими API-интерфейсами Azure для [docs.Microsoft.com](/rest/api/azure/). В этой документации рассказывается, как структурировать сообщения HTTP и предлагает руководство по использованию POST. Эта документация также предлагается в формате Swagger на сайте [GitHub](https://github.com/Azure/azure-rest-api-specs).

## <a name="additional-support-details"></a>Дополнительные сведения о поддержке

### <a name="ios-and-android-support-details"></a>сведения о поддержке iOS и Android

- Клиентские библиотеки iOS для служб Communication Services предназначены для iOS версии 13 + и Xcode 11 +.
- Клиентские библиотеки Android Java. целевые API Android уровень 21 + и Android Studio 4.0 +

### <a name="net-support-details"></a>Сведения о поддержке .NET

За исключением вызова, пакеты служб связи нацелены на .NET Standard 2,0, поддерживающие перечисленные ниже платформы.

Поддержка через платформа .NET Framework 4.6.1
- Windows 10, 8,1, 8 и 7
- Windows Server 2012 R2, 2012 и 2008 R2 SP1

Поддержка через .NET Core 2,0:
- Windows 10 (1607 +), 7 SP1 +, 8,1
- Windows Server 2008 R2 с пакетом обновления 1 (SP1) или более поздняя версия.
- Максимальная OS X 10.12 +
- Несколько версий и дистрибутивов Linux
- UWP 10.0.16299 (RS3), Сентябрь 2017
- Unity 2018,1
- Mono 5.4
- Xamarin iOS 10,14
- Xamarin Mac 3,8

## <a name="calling-client-library-timeouts"></a>Вызывающий тайм-аут клиентской библиотеки

Следующие таймауты применяются к службам связи, вызывающим клиентские библиотеки:

| Действие           | Время ожидания в секундах |
| -------------- | ---------- |
| Участник повторного подключения или удаления | 120 |
| Добавление или удаление новой модальности из вызова (запуск/завершение видео или общий доступ к экрану) | 40 |
| Время ожидания операции обмена вызовами | 60 |
| 1:1 время ожидания установки вызова | 85 |
| Время ожидания установки вызова группы | 85 |
| Время ожидания установки звонка PSTN | 115 |
| Повышение времени ожидания при вызове группы (1:1) | 115 |


## <a name="api-stability-expectations"></a>Ожидание стабильности API

> [!IMPORTANT]
> В этом разделе содержатся рекомендации по интерфейсам API и клиентским библиотекам, помеченным как **стабильные**. API-интерфейсы, помеченные как предварительные, предварительные или бета-версии, могут быть изменены или устарели **без предварительного уведомления**.

В будущем мы можем снять с учета версии клиентских библиотек служб Communication Services, а также внести существенные изменения в наши API-интерфейсы и выпущенные клиентские библиотеки. Службы связи Azure, *как правило* , используют две политики поддержки для снятия версий службы:

- Вы получите оповещение по крайней мере за три года, прежде чем потребуется изменить код из-за изменения интерфейса служб связи. Все документированные интерфейсы API и API-интерфейсы RESTFUL, как правило, работают по крайней мере за три года до выдачи интерфейсов.
- Перед обновлением сборок клиентской библиотеки до последней дополнительной версии вы получите оповещение по крайней мере за один год. Эти необходимые обновления не должны требовать изменений в коде, так как они находятся в одной основной версии. Это особенно справедливо для библиотек вызовов и чатов, которые имеют компоненты в реальном времени, для которых часто требуется обновление безопасности и производительности. Мы настоятельно рекомендуем обновлять клиентские библиотеки служб связи.

### <a name="api-and-client-library-decommissioning-examples"></a>Примеры списания API и клиентской библиотеки

**Вы интегрируете v24 версию SMS REST API в приложение. Выпуски связи Azure V25.**

Прежде чем эти API перестанут работать и принудительно будут обновлены до V25, вы получите предупреждение об ошибке 3 года. Это обновление может потребовать изменения в коде.

**Вы интегрируете в приложение версию 2.02 вызывающей клиентской библиотеки. Выпуски Azure Communication v 2.05.**

Возможно, потребуется выполнить обновление до версии 2.05 вызывающей клиентской библиотеки в течение 12 месяцев после выпуска v 2.05. Это должна быть простая замена артефакта без изменения кода, так как v 2.05 находится в основной версии v2 и не имеет критических изменений.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих обзорах клиентской библиотеки:

- [Обзор вызова клиентской библиотеки](../concepts/voice-video-calling/calling-sdk-features.md)
- [Обзор клиентской библиотеки чата](../concepts/chat/sdk-features.md)
- [Общие сведения о клиентской библиотеке SMS](../concepts/telephony-sms/sdk-features.md)

Чтобы приступить к работе со службами связи Azure, сделайте следующее:

- [Создание ресурсов связи Azure](../quickstarts/create-communication-resource.md)
- Создание [маркеров доступа пользователей](../quickstarts/access-tokens.md)
