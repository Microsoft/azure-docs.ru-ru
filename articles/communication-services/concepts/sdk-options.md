---
title: Клиентские библиотеки и интерфейсы API для служб связи Azure
titleSuffix: An Azure Communication Services concept document
description: Дополнительные сведения о пакетах SDK и интерфейсах API для служб связи Azure.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2021
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: effd7658bbfe7359e1f99f9452857824c2c45c2f
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105107896"
---
# <a name="client-libraries-and-rest-apis"></a>Клиентские библиотеки и интерфейсы REST API

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]


Возможности служб связи Azure концептуально организованы в шесть областей. Некоторые области имеют полные пакеты SDK с открытым кодом. Вызывающий пакет SDK использует частные сетевые интерфейсы и в настоящее время закрыто источником, а библиотека разговора включает в себя зависимость с закрытым исходным кодом. Примеры и дополнительные технические сведения для пакетов SDK публикуются в [репозитории GitHub служб связи Azure](https://github.com/Azure/communication).

## <a name="client-libraries"></a>Клиентские библиотеки

| Сборка               | Протоколы             |Открытие и закрытие источника| Пространства имен                          | Возможности                                                      |
| ---------------------- | --------------------- | ---|-------------------------- | --------------------------------------------------------------------------- |
| Azure Resource Manager | REST | Открыть            | Azure. ResourceManager. Обмен данными | Подготавливайте ресурсы служб связи и управляйте ими             |
| Распространенные                 | REST | Открыть               | Azure. Communication. Common          | Предоставляет базовые типы для других пакетов SDK |
| Идентификация         | REST | Открыть               | Azure. Communication. Identity  | Управление пользователями, маркеры доступа |
| номера телефонов;         | REST | Открыть               | Azure. Communication. Фоненумберс  | Управление номерами телефонов |
| Чат                   | ПРОЧее с собственными сигналами | Открыть с помощью сигнального пакета с закрытым источником    | Azure. Communication. чат            | Добавление в приложения текста, основанного на режиме реального времени  |
| SMS                    | REST | Открыть              | Azure. Communication. SMS             | Отправка и получение текстовых сообщений |
| Вызов                | Собственный транспорт | Закрытое |Azure. Communication. вызов         | Использование голоса, видео, совместного использования экрана и других возможностей обмена данными в реальном времени          |

Обратите внимание, что пакеты SDK для Azure Resource Manager, идентификации и SMS ориентированы на интеграцию служб, и во многих случаях проблемы безопасности возникают, если вы интегрируете эти функции в приложения для конечных пользователей. Общепринятые и сеансовые пакеты SDK подходят для служб и клиентских приложений. Вызывающий пакет SDK предназначен для клиентских приложений. Пакет SDK, нацеленный на сценарии службы, находится в разработке.

### <a name="languages-and-publishing-locations"></a>Языки и расположения для публикации

Сведения о расположении публикации для отдельных пакетов SDK приведены ниже.

| Область           | JavaScript | .NET | Python | Java SE | iOS | Android | Другое                          |
| -------------- | ---------- | ---- | ------ | ---- | -------------- | -------------- | ------------------------------ |
| Azure Resource Manager | -         | [NuGet](https://www.nuget.org/packages/Azure.ResourceManager.Communication)    |   [PyPi](https://pypi.org/project/azure-mgmt-communication/)    |  -  | -              | -  | [Через GitHub](https://github.com/Azure/azure-sdk-for-go/releases/tag/v46.3.0) |
| Распространенные         | [npm](https://www.npmjs.com/package/@azure/communication-common)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Common/)    | Недоступно      | [Maven](https://search.maven.org/search?q=a:azure-communication-common)   | [GitHub](https://github.com/Azure/azure-sdk-for-ios/releases)            | [Maven](https://search.maven.org/artifact/com.azure.android/azure-communication-common)             | -                              |
| Идентификация | [npm](https://www.npmjs.com/package/@azure/communication-identity)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Identity)    | [PyPi](https://pypi.org/project/azure-communication-identity/)      | [Maven](https://search.maven.org/search?q=a:azure-communication-identity)   | -              | -              | -                            |
| Номера телефонов | [npm](https://www.npmjs.com/package/@azure/communication-phone-numbers)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.PhoneNumbers)    | [PyPi](https://pypi.org/project/azure-communication-phonenumbers/)      | [Maven](https://search.maven.org/search?q=a:azure-communication-phonenumbers)   | -              | -              | -                            |
| Чат           | [npm](https://www.npmjs.com/package/@azure/communication-chat)        | [NuGet](https://www.nuget.org/packages/Azure.Communication.Chat)     | [PyPi](https://pypi.org/project/azure-communication-chat/)     | [Maven](https://search.maven.org/search?q=a:azure-communication-chat)   | [GitHub](https://github.com/Azure/azure-sdk-for-ios/releases)  | [Maven](https://search.maven.org/search?q=a:azure-communication-chat)   | -                              |
| SMS            | [npm](https://www.npmjs.com/package/@azure/communication-sms)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Sms)    | [PyPi](https://pypi.org/project/azure-communication-sms/)       | [Maven](https://search.maven.org/artifact/com.azure/azure-communication-sms)   | -              | -              | -                              |
| Вызов        | [npm](https://www.npmjs.com/package/@azure/communication-calling)         | -      | -      | -     | [GitHub](https://github.com/Azure/Communication/releases)     | [Maven](https://search.maven.org/artifact/com.azure.android/azure-communication-calling/)            | -                              |
| Справочная документация     | [docs](https://azure.github.io/azure-sdk-for-js/communication.html)         | [docs](https://azure.github.io/azure-sdk-for-net/communication.html)      | -      | [docs](http://azure.github.io/azure-sdk-for-java/communication.html)     | [docs](/objectivec/communication-services/calling/)      | [docs](/java/api/com.azure.communication.calling)            | -                              |

## <a name="rest-apis"></a>Интерфейсы REST API

API-интерфейсы служб связи задокументированы вместе с другими API-интерфейсами Azure для [docs.Microsoft.com](/rest/api/azure/). В этой документации рассказывается, как структурировать сообщения HTTP и предлагает руководство по использованию POST. Эта документация также предлагается в формате Swagger на сайте [GitHub](https://github.com/Azure/azure-rest-api-specs).

## <a name="additional-support-details"></a>Дополнительные сведения о поддержке

### <a name="ios-and-android-support-details"></a>сведения о поддержке iOS и Android

- Пакеты SDK iOS для служб связи предназначены для iOS версии 13 + и Xcode 11 +.
- Пакеты SDK Java для Android целевые API Android уровень 21 + и Android Studio 4.0 +

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
- Unity 2018.1.
- Mono 5.4
- Xamarin iOS 10,14
- Xamarin Mac 3,8

## <a name="calling-sdk-timeouts"></a>Время ожидания вызова пакета SDK

Следующие времена ожидания применяются к пакетам SDK, вызывающим коммуникационные службы:

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
> В этом разделе содержатся рекомендации по интерфейсам API и пакетам SDK, помеченным как **стабильные**. API-интерфейсы, помеченные как предварительные, предварительные или бета-версии, могут быть изменены или устарели **без предварительного уведомления**.

В будущем мы можем снять с учета версии пакетов SDK для служб связи, а также внести существенные изменения в наши интерфейсы API и выпущенные пакеты SDK. Службы связи Azure, *как правило* , используют две политики поддержки для снятия версий службы:

- Вы получите оповещение по крайней мере за три года, прежде чем потребуется изменить код из-за изменения интерфейса служб связи. Все документированные API-интерфейсы и API-интерфейсы RESTFUL обычно работают по крайней мере за три года до выдачи интерфейсов.
- Перед обновлением сборок пакета SDK до последней дополнительной версии вы получите оповещение по крайней мере за один год. Эти необходимые обновления не должны требовать изменений в коде, так как они находятся в одной основной версии. Это особенно справедливо для библиотек вызовов и чатов, которые имеют компоненты в реальном времени, для которых часто требуется обновление безопасности и производительности. Мы настоятельно рекомендуем вам обновлять пакеты SDK для служб связи.

### <a name="api-and-sdk-decommissioning-examples"></a>Примеры списания API и пакета SDK

**Вы интегрируете v24 версию SMS REST API в приложение. Выпуски связи Azure V25.**

Прежде чем эти API перестанут работать и принудительно будут обновлены до V25, вы получите предупреждение об ошибке 3 года. Это обновление может потребовать изменения в коде.

**Вы интегрируете в приложение версию 2.02 вызывающего пакета SDK. Выпуски Azure Communication v 2.05.**

Может потребоваться обновление до версии 2.05 пакета SDK в течение 12 месяцев после выпуска v 2.05. Это должна быть простая замена артефакта без изменения кода, так как v 2.05 находится в основной версии v2 и не имеет критических изменений.

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения см. в следующих обзорах SDK:

- [Общие сведения о вызове SDK](../concepts/voice-video-calling/calling-sdk-features.md)
- [Обзор пакета SDK для чата](../concepts/chat/sdk-features.md)
- [Обзор пакета SDK для SMS](../concepts/telephony-sms/sdk-features.md)

Чтобы приступить к работе со службами связи Azure, сделайте следующее:

- [Создание ресурсов связи Azure](../quickstarts/create-communication-resource.md)
- Создание [маркеров доступа пользователей](../quickstarts/access-tokens.md)
