---
title: Логика повторного выполнения операций в пакете SDK служб мультимедиа для .NET | Документация Майкрософт
description: В этом разделе содержится обзор логики повторных попыток в пакете SDK служб мультимедиа для .NET.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 527b61a6-c862-4bd8-bcbc-b9aea1ffdee3
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: inhenkel
ms.openlocfilehash: fe37c9f7727535a292b029c342aa0e85ce0a86ec
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105644110"
---
# <a name="retry-logic-in-the-media-services-sdk-for-net"></a>Логика повторного выполнения операций в пакете SDK служб мультимедиа для .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

При работе со службами Microsoft Azure могут возникать временные сбои. В большинстве случаев после нескольких повторных попыток операция завершается успешно. Пакет SDK служб мультимедиа для .NET реализует логику повторного выполнения операций для обработки временных сбоев, связанных с исключениями и ошибками. Причиной этих ошибок и исключений могут быть веб-запросы, выполнение запросов, сохранение изменений и операции с хранилищем.  По умолчанию пакет SDK служб мультимедиа для .NET пытается выполнить операцию четыре раза, а затем возвращает исключение в приложение. Затем это исключение должно быть правильно обработано кодом приложения.  

 Вот краткое описание политик веб-запросов, операций с хранилищем, выполнения запросов и сохранения изменений:  

* Политика операций с хранилищем используется для операций с хранилищем BLOB-объектов (отправка и скачивание файлов ресурсов).  
* Политика веб-запросов используется для универсальных веб-запросов (например, для получения маркера аутентификации и разрешения конечной точки пользовательского кластера).  
* Политика выполнения запросов используется для отправки запросов к сущностям из REST (например, mediaContext.Assets.Where(…)).  
* Политика сохранения изменений используется для выполнения любых операций, изменяющих данные внутри службы (например, создание или обновление сущности, вызов функции службы для выполнения какого-то действия).  
  
  В этом разделе перечислены типы исключений и коды ошибок, которые обрабатываются логикой повторного выполнения операций в пакете SDK служб мультимедиа для .NET.  

## <a name="exception-types"></a>Типы исключений
В таблице ниже описаны исключения, которые пакет SDK служб мультимедиа для .NET обрабатывает или не обрабатывает для некоторых операций, которые могут вызывать временные сбои.  

| Исключение | Веб-запрос | Память | Запрос | Сохранение изменений |
| --- | --- | --- | --- | --- |
| WebException<br/>Дополнительные сведения см. в разделе [Коды состояний WebException](media-services-retry-logic-in-dotnet-sdk.md#WebExceptionStatus). |Да |Да |Да |Да |
| DataServiceClientException<br/> Дополнительные сведения см. в разделе [Коды состояний ошибок HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |нет |Да |Да |Да |
| DataServiceQueryException<br/> Дополнительные сведения см. в разделе [Коды состояний ошибок HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |нет |Да |Да |Да |
| DataServiceRequestException<br/> Дополнительные сведения см. в разделе [Коды состояний ошибок HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |нет |Да |Да |Да |
| DataServiceTransportException |нет |нет |Да |Да |
| TimeoutException |Да |Да |Да |нет |
| SocketException |Да |Да |Да |Да |
| StorageException |нет |Да |нет |нет |
| IOException |нет |Да |нет |нет |

### <a name="webexception-status-codes"></a><a name="WebExceptionStatus"></a> Коды состояний WebException
В таблице ниже показано, для каких кодов ошибок WebException реализована логика повторного выполнения операций. Перечисление [WebExceptionStatus](/dotnet/api/system.net.webexceptionstatus) определяет коды состояний.  

| Состояние | Веб-запрос | Память | Запрос | Сохранение изменений |
| --- | --- | --- | --- | --- |
| ConnectFailure |Да |Да |Да |Да |
| NameResolutionFailure |Да |Да |Да |Да |
| ProxyNameResolutionFailure |Да |Да |Да |Да |
| SendFailure |Да |Да |Да |Да |
| PipelineFailure |Да |Да |Да |нет |
| ConnectionClosed |Да |Да |Да |нет |
| KeepAliveFailure |Да |Да |Да |нет |
| UnknownError |Да |Да |Да |нет |
| ReceiveFailure |Да |Да |Да |нет |
| RequestCanceled |Да |Да |Да |нет |
| Время ожидания |Да |Да |Да |нет |
| ProtocolError <br/>Повторное выполнение операции при состоянии ProtocolError определяется обработкой кодов состояния HTTP. Дополнительные сведения см. в разделе [Коды состояний ошибок HTTP](media-services-retry-logic-in-dotnet-sdk.md#HTTPStatusCode). |Да |Да |Да |Да |

### <a name="http-error-status-codes"></a><a name="HTTPStatusCode"></a> Коды состояний ошибок HTTP
Когда операции выполнения запросов и сохранения изменений выдают исключение DataServiceClientException, DataServiceQueryException или DataServiceQueryException, код состояния ошибки HTTP возвращается в свойстве StatusCode.  В таблице ниже показано, для каких кодов ошибок реализована логика повторного выполнения операций.  

| Состояние | Веб-запрос | Память | Запрос | Сохранение изменений |
| --- | --- | --- | --- | --- |
| 401 |нет |Да |нет |нет |
| 403 |нет |Да<br/>Обработка повторных попыток с увеличением времени ожидания. |нет |нет |
| 408 |Да |Да |Да |Да |
| 429 |Да |Да |Да |Да |
| 500 |Да |Да |Да |нет |
| 502 |Да |Да |Да |нет |
| 503 |Да |Да |Да |Да |
| 504 |Да |Да |Да |нет |

Фактическую реализацию логики повторного выполнения операций в пакете SDK служб мультимедиа для .NET см. [здесь](https://github.com/Azure/azure-sdk-for-media-services/tree/dev/src/net/Client/TransientFaultHandling).

## <a name="next-steps"></a>Дальнейшие действия
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
