---
title: Миграция из библиотеки обработчика веб-канала изменений в пакет SDK для .NET v3 Azure Cosmos DB
description: Узнайте, как перенести приложение из использования библиотеки обработчика веб-канала изменений в Azure Cosmos DB SDK v3.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 09/17/2019
ms.author: maquaran
ms.custom: devx-track-dotnet
ms.openlocfilehash: ce2d4d3ad3ae349718f01584ec077b18e11e4f8d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93341269"
---
# <a name="migrate-from-the-change-feed-processor-library-to-the-azure-cosmos-db-net-v3-sdk"></a>Миграция из библиотеки обработчика веб-канала изменений в пакет SDK для .NET v3 Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описываются шаги, необходимые для переноса кода существующего приложения, использующего [библиотеку обработчика веб-канала изменений](https://github.com/Azure/azure-documentdb-changefeedprocessor-dotnet) , в функцию [веб-канала изменений](change-feed.md) в последней версии пакета SDK для .NET (также называется пакетом SDK для .NET v3).

## <a name="required-code-changes"></a>Необходимые изменения в коде

Пакет SDK для .NET v3 имеет несколько критических изменений, поэтому ниже приведены основные этапы миграции приложения.

1. Преобразуйте `DocumentCollectionInfo` экземпляры в `Container` ссылки для контейнеров отслеживаемых и арендных адресов.
1. Настройки, использующие, `WithProcessorOptions` должны обновляться для использования `WithLeaseConfiguration` и `WithPollInterval` для интервалов `WithStartTime` [времени начала](./change-feed-processor.md#starting-time), а также для `WithMaxItems` определения максимального числа элементов.
1. Установите значение ON в параметре `processorName` `GetChangeFeedProcessorBuilder` , совпадающее со значением, настроенным в `ChangeFeedProcessorOptions.LeasePrefix` , или используйте `string.Empty` в противном случае.
1. Изменения больше не доставляются в виде `IReadOnlyList<Document>` , а именно, где — это тип, который необходимо `IReadOnlyCollection<T>` `T` определить, но класс базового элемента больше не существует.
1. Чтобы обрабатывал изменения, больше не требуется реализация, вместо этого необходимо [определить делегат](change-feed-processor.md#implementing-the-change-feed-processor). Делегат может быть статической функцией или, если необходимо поддерживать состояние между выполнениями, можно создать собственный класс и передать метод экземпляра в качестве делегата.

Например, если исходный код для сборки обработчика веб-канала изменений выглядит следующим образом:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=ChangeFeedProcessorLibrary)]

Перенесенный код будет выглядеть следующим образом:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=ChangeFeedProcessorMigrated)]

Делегат может быть статическим методом:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=Delegate)]

## <a name="state-and-lease-container"></a>Контейнер состояния и аренды

Как и в случае с библиотекой обработчика канала изменений, функция веб-канала изменений в пакете SDK для .NET v3 использует [контейнер аренды](change-feed-processor.md#components-of-the-change-feed-processor) для хранения состояния. Однако схемы различаются.

Обработчик веб-канала изменений пакета SDK v3 обнаружит старое состояние библиотеки и автоматически перенесет его в новую схему при первом выполнении перенесенного кода приложения. 

Вы можете безопасно остановить приложение с помощью старого кода, перенести код в новую версию, запустить перенесенное приложение и все изменения, произошедшие во время остановки приложения, будут отобраны и обработаны новой версией.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Azure Cosmos DB](sql-api-sdk-dotnet.md)
* [Примеры использования в GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [Дополнительные примеры на GitHub](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)

## <a name="next-steps"></a>Дальнейшие действия

Вы можете продолжить знакомство с обработчиком канала изменений, перейдя к следующим статьям:

* [Обработчик канала изменений в Azure Cosmos DB](change-feed-processor.md)
* [Использование оценщика канала изменений](how-to-use-change-feed-estimator.md)
* [Настройка времени запуска обработчика канала изменений](./change-feed-processor.md#starting-time)