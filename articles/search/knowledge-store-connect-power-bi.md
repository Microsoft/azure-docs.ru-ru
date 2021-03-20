---
title: Подключение к хранилищу знаний с помощью Power BI
titleSuffix: Azure Cognitive Search
description: Подключение к хранилищу знаний службы "Когнитивный поиск Azure" с помощью Power BI для анализа и исследования.
author: HeidiSteen
ms.author: heidist
manager: nitinme
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 91e75b60f5324288c9f1adac59e31b9c1a1b0e9e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89289177"
---
# <a name="connect-a-knowledge-store-with-power-bi"></a>Подключение хранилища знаний с помощью Power BI

В этой статье объясняется, как подключить и изучить хранилище знаний с помощью Power Query в приложении Power BI Desktop. Вы можете быстрее приступить к работе с помощью шаблонов или создать пользовательскую панель мониторинга с нуля. В этом кратком видео показано, как можно расширить возможности работы с данными с помощью Когнитивный поиск Azure в сочетании с Power BI.


> [!VIDEO https://www.youtube.com/embed/XWzLBP8iWqg?version=3&start=593&end=663]



+ Выполните инструкции по [созданию хранилища знаний на портале Azure](knowledge-store-create-portal.md) или [созданию хранилища знаний службы "Когнитивный поиск Azure" с помощью REST](knowledge-store-create-rest.md), чтобы создать пример хранилища знаний для работы с этим пошаговым руководством. Кроме того, вам потребуется имя учетной записи хранения Azure, которая использовалась для создания хранилища знаний, а также ключ доступа к нему с портала Azure.

+ [Установка Power BI Desktop](https://powerbi.microsoft.com/downloads/)

## <a name="sample-power-bi-template---azure-portal-only"></a>Пример шаблона Power BI — только портал Azure

При создании [хранилища знаний с помощью портала Azure](knowledge-store-create-portal.md) можно загрузить [шаблон Power BI](https://github.com/Azure-Samples/cognitive-search-templates) на второй странице мастера **импорта данных**. Этот шаблона включает в себя несколько визуализаций, например WordCloud и Network Navigator, для текстового содержимого. 

Щелкните **Получить шаблон Power BI** на странице **Add cognitive skills** (Добавление когнитивных навыков), чтобы получить и скачать шаблон из общедоступного расположения на сайте GitHub. Мастер изменяет шаблон в соответствии с формой данных в проекциях хранилища знаний, указанных в мастере. По этой причине загружаемый шаблон отличается при каждом запуске мастера при условии, что используются разные входные данные и наборы навыков.

![Пример шаблона Power BI для Когнитивного поиска Azure](media/knowledge-store-connect-power-bi/powerbi-sample-template-portal-only.png "Пример шаблона Power BI")

> [!NOTE]
> Хотя шаблон загружается, пока мастер находится в середине выполнения, перед его использованием необходимо подождать, пока хранилище знаний не будет создано в Хранилище таблиц Azure.

## <a name="connect-with-power-bi"></a>Подключение с помощью Power BI

1. Запустите Power BI Desktop и выберите **Получение данных**.

1. В окне **Получение данных** выберите **Azure**, а затем выберите **Хранилище таблиц Azure**.

1. Нажмите кнопку **Подключить**.

1. В поле **Имя или URL-адрес учетной записи** введите имя своей учетной записи хранилища Azure (для вас будет создан полный URL-адрес).

1. При появлении запроса введите ключ учетной записи хранения.

1. Выберите таблицы, содержащие отзывы о гостиницах, созданные в предыдущих пошаговых руководствах. 

   + В пошаговом руководстве по работе с порталом используются имена таблиц *hotelReviewsSsDocument*, *hotelReviewsSsEntities*, *hotelReviewsSsKeyPhrases* и *hotelReviewsSsPages*. 
   
   + В пошаговом руководстве по работе с REST используются имена таблиц *hotelReviewsDocument*, *hotelReviewsPages*, *hotelReviewsKeyPhrases* и *hotelReviewsSentiment*.

1. Нажмите кнопку **Загрузить**.

1. На верхней ленте нажмите **Изменить запросы**, чтобы открыть **Редактор Power Query**.

   ![Открытие Power Query](media/knowledge-store-connect-power-bi/powerbi-edit-queries.png "Открытие Power Query")

1. Выберите *hotelReviewsSsDocument*, а затем удалите столбцы *PartitionKey*, *RowKey* и *Метка времени*. 
   ![Изменение таблиц](media/knowledge-store-connect-power-bi/powerbi-edit-table.png "Изменение таблиц")

1. Щелкните значок с противоположными стрелками в верхней правой части таблицы, чтобы развернуть *Содержимое*. Когда отобразится список столбцов, выберите все столбцы, а затем отмените выбор столбцов, начинающихся с "метаданных". Щелкните **ОК**, чтобы отобразить выбранные столбцы.

   ![Развертывание содержимого](media/knowledge-store-connect-power-bi/powerbi-expand-content-table.png "Развертывание содержимого")

1. Измените тип данных для следующих столбцов, щелкнув значок ABC-123 в левом верхнем углу столбца.

   + Для *content.latitude* и *Content.longitude* выберите **Десятичное число**.
   + Для *Content.reviews_date* и *Content.reviews_dateAdded* выберите **Дата и время**.

   ![Изменение типов данных](media/knowledge-store-connect-power-bi/powerbi-change-type.png "Изменение типов данных")

1. Выберите *hotelReviewsSsPages*, а затем повторите шаги 9 и 10, чтобы удалить столбцы и развернуть *Содержимое*.
1. Измените тип данных для *Content. SentimentScore* на **Десятичное число**.
1. Выберите *hotelReviewsSsKeyPhrases* и повторите шаги 9 и 10, чтобы удалить столбцы и развернуть *Содержимое*. В этой таблице не нужно изменять тип данных.

1. На панели команд щелкните **Закрыть и применить**.

1. Щелкните плитку "Модель" на левой панели навигации и убедитесь, что Power BI отображает связи между всеми тремя таблицами.

   ![Проверка связей](media/knowledge-store-connect-power-bi/powerbi-relationships.png "Проверка связей")

1. Дважды щелкните каждую связь и убедитесь, что **Направление перекрестной фильтрации** установлено на **Обе**.  Это позволяет обновлять визуальные элементы при применении фильтра.

1. Щелкните плитку отчета в левой области навигации, чтобы просмотреть данные с помощью визуализаций. Текстовые поля, таблицы и карточки являются полезными визуализациями. Чтобы заполнить таблицу или карточку, можно выбрать поля из каждой из трех таблиц. 

<!-- ## Try with larger data sets

We purposely kept the data set small to avoid charges for a demo walkthrough. For a more realistic experience, you can create and then attach a billable Cognitive Services resource to enable a larger number of transactions against the sentiment analyzer, keyphrase extraction, and language detector skills.

Create new containers in Azure Blob storage and upload each CSV file to its own container. Specify one of these containers in the data source creation step in Import data wizard.

| Description | Link |
|-------------|------|
| Free tier   | [HotelReviews_Free.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Free.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D) |
| Small (500 Records) | [HotelReviews_Small.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Small.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D) |
| Medium (6000 Records)| [HotelReviews_Medium.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Medium.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D)
| Large (Full dataset 35000 Records) | [HotelReviews_Large.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Large.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D). Be aware that very large data sets are expensive to process. This one costs roughly $1000 U.S dollars.|

In the enrichment step of the wizard, attach a billable [Cognitive Services](../cognitive-services/cognitive-services-apis-create-account.md) resource, created at the *S0* tier, in the same region as Azure Cognitive Search to use larger data sets. 

  ![Create a Cognitive Services resource](media/knowledge-store-connect-power-bi/create-cognitive-service.png "Create a Cognitive Services resource") -->

## <a name="clean-up"></a>Очистка

Если вы работаете в собственной подписке, в конце проекта следует решить, нужны ли вам созданные ресурсы. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок **Все ресурсы** или **Группы ресурсов** на панели навигации слева.

При работе с бесплатной версией службы помните о том, что вам доступно максимум три индекса, индексатора и источника данных. Вы можете удалить отдельные элементы на портале, чтобы не превысить лимит.

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как исследовать это хранилище знаний с помощью Обозревателя службы хранилища, см. в следующей статье.

> [!div class="nextstepaction"]
> [Просмотр данных с помощью Обозревателя службы хранилища](knowledge-store-view-storage-explorer.md)