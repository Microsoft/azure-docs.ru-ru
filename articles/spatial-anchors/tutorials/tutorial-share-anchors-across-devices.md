---
title: Руководство по совместному использованию пространственных привязок между сеансами и устройствами
description: Из этого руководства вы узнаете, как совместно использовать идентификаторы пространственных привязок Azure между устройствами Android и iOS в Unity с внутренней службой.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: tutorial
ms.service: azure-spatial-anchors
ms.openlocfilehash: 3ed9b18144bb727af751fbcbbdd5cc6631afc40f
ms.sourcegitcommit: 91361cbe8fff7c866ddc4835251dcbbe2621c055
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "105727235"
---
# <a name="tutorial-share-spatial-anchors-across-sessions-and-devices"></a>Руководство по Совместное использование пространственных привязок между сеансами и устройствами

"Пространственные привязки Azure" — это кроссплатформенная служба разработчика, которая позволяет создавать среды смешанной реальности с применением объектов, не меняющих своего расположения на устройствах с течением времени. 

В этом руководстве описано, как использовать [Пространственные привязки Azure](../overview.md), чтобы создавать привязки во время одного сеанса, а затем обнаруживать их на том же или на другом устройстве. Те же самые привязки могут быть одновременно обнаружены несколькими устройствами в одном и том же расположении.

![Анимация, в которой показаны пространственные привязки, созданные с помощью мобильного устройства, и их использование на разных устройствах в разное время.](./media/persistence.gif)


Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Развертывание в Azure веб-приложения ASP.NET Core, которое можно применять для совместного использования привязок и сохранения привязок в памяти в течение указанного периода времени.
> * Настройка сцены AzureSpatialAnchorsLocalSharedDemo в примере Unity из наших кратких руководств для использования преимуществ веб-приложения для совместного использования пространственных привязок.
> * Развертывание и запуск привязок на одном или нескольких устройствах.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [Share Anchors Sample Prerequisites](../../../includes/spatial-anchors-share-sample-prereqs.md)]

> [!NOTE]
> В этом учебнике описано, как использовать Unity и веб-приложение ASP.NET Core, однако описанный здесь подход — это лишь пример того, как можно совместно использовать идентификаторы Пространственных привязок Azure на других устройствах. Вы можете использовать другие языки и серверные технологии в тех же целях.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="download-the-sample-project"></a>Загрузка примера проекта

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

## <a name="deploy-the-sharing-anchors-service"></a>Развертывание службы совместного использования привязок

## <a name="visual-studio"></a>[Visual Studio](#tab/VS)

Откройте Visual Studio, а затем откройте проект в папке *Sharing\SharingServiceSample*.

[!INCLUDE [Publish Azure](../../../includes/spatial-anchors-publish-azure.md)]

## <a name="visual-studio-code"></a>[Visual Studio Code](#tab/VSC)

Перед развертыванием службы в Visual Studio Code необходимо создать группу ресурсов и план службы приложений.

### <a name="sign-in-to-azure"></a>Вход в Azure

Перейдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>, а затем войдите в подписку Azure.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [resource group intro text](../../../includes/resource-group.md)]

Рядом с **группой ресурсов** выберите команду **Создать**.

Присвойте группе ресурсов имя **myResourceGroup** и щелкните **ОК**.

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

[!INCLUDE [app-service-plan](../../../includes/app-service-plan.md)]

Рядом с полем **План размещения** выберите **Создать**.

На панели **Настройка плана размещения** введите следующие параметры.

| Параметр | Рекомендуемое значение | Описание |
|-|-|-|
|План службы приложений| MySharingServicePlan | Имя плана службы приложений |
| Расположение | западная часть США | Центр обработки данных, в котором размещается веб-приложение |
| Размер | Free | [Ценовая категория](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), которая определяет возможности размещения |

Щелкните **ОК**.

Откройте Visual Studio Code, а затем откройте проект в папке *Sharing\SharingServiceSample*. 

Чтобы развернуть службу совместного использования с помощью Visual Studio Code, следуйте инструкциям в статье <a href="/aspnet/core/tutorials/publish-to-azure-webapp-using-vscode?view=aspnetcore-2.2&preserve-view=true#open-it-with-visual-studio-code" target="_blank">Публикация приложения ASP.NET Core в Azure с помощью Visual Studio Code</a>. Начните с раздела "Открытие проекта с помощью Visual Studio Code". Не создавайте еще один проект ASP.NET, как описано на предыдущем шаге, поскольку у вас уже есть проект, который нужно развернуть и опубликовать (SharingServiceSample).

---

## <a name="deploy-the-sample-app"></a>Развертывание примера приложения

[!INCLUDE [Run Share Anchors Sample](../../../includes/spatial-anchors-run-share-sample.md)]

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как развернуть веб-приложение ASP.NET Core в Azure, а затем настроить и развернуть приложение Unity. Вы знаете, как создать в приложении пространственные привязки и предоставить к ним доступ другим устройствам с помощью веб-приложения ASP.NET Core.

Вы можете улучшить веб-приложение ASP.NET Core, чтобы оно использовало Azure Cosmos DB для хранения хранилища общих идентификаторов пространственных привязок. Добавив поддержку Azure Cosmos DB, вы можете создать привязку в веб-приложении ASP.NET Core уже сегодня. Затем, благодаря идентификатору привязки, хранящемуся в веб-приложении, вы может вернуться к приложению через несколько дней и найти привязку снова.

> [!div class="nextstepaction"]
> [Совместное использование привязок между сеансами и устройствами с помощью Пространственных привязок Azure и серверной части службы Azure Cosmos DB](./tutorial-use-cosmos-db-to-store-anchors.md)