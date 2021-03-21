---
title: Моделирование реляционных данных SQL для импорта и индексирования
titleSuffix: Azure Cognitive Search
description: Узнайте, как моделировать реляционные данные, денормализованные в неструктурированный результирующий набор для индексирования и полнотекстового поиска в Когнитивный поиск Azure.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 6c70b42e7d0f647a3b2b60d29b5098a791e4975f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88924526"
---
# <a name="how-to-model-relational-sql-data-for-import-and-indexing-in-azure-cognitive-search"></a>Моделирование реляционных данных SQL для импорта и индексирования в Azure Когнитивный поиск

Когнитивный поиск Azure принимает плоский набор строк в качестве входных данных для [конвейера индексирования](search-what-is-an-index.md). Если исходные данные исходят из соединяемых таблиц в SQL Server реляционной базе данных, в этой статье объясняется, как создать результирующий набор и как моделировать связь типа «родители-потомки» в индексе Azure Когнитивный поиск.

На рисунке мы будем называть гипотетическую базу данных гостиниц на основе [демонстрационных данных](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/hotels). Предположим, база данных состоит из гостиницы $ с 50 гостиниц, а также таблицы помещений $ с комнатами различных типов, ставок и удобствами для всего 750 помещений. Между таблицами существует связь «один ко многим». В нашем подходе представление предоставит запрос, возвращающий 50 строк, по одной строке на гостиницу со сведениями о комнате, внедренными в каждую строку.

   ![Таблицы и представления в базе данных гостиниц](media/index-sql-relational-data/hotels-database-tables-view.png "Таблицы и представления в базе данных гостиниц")


## <a name="the-problem-of-denormalized-data"></a>Проблема денормализованных данных

Одной из трудностей при работе со связями «один ко многим» является то, что стандартные запросы, основанные на соединяемых таблицах, будут возвращать денормализованные данные, что плохо работает в сценарии Когнитивный поиск Azure. Рассмотрим следующий пример, который присоединяет Гостиницы и комнаты.

```sql
SELECT * FROM Hotels$
INNER JOIN Rooms$
ON Rooms$.HotelID = Hotels$.HotelID
```
Результаты из этого запроса возвращают все поля отеля, за которыми следуют все поля комнаты, и предварительные сведения о номере отеля повторяются для каждого значения комнаты.

   ![Денормализованные данные, избыточные данные отеля при добавлении полей комнаты](media/index-sql-relational-data/denormalize-data-query.png "Денормализованные данные, избыточные данные отеля при добавлении полей комнаты")


Хотя этот запрос завершается успешно на поверхности (предоставляя все данные в плоском наборе строк), он завершается неудачей, обеспечивая правильную структуру документа для ожидаемого поиска. Во время индексирования Когнитивный поиск Azure создаст по одному документу поиска для каждой полученной строки. Если поисковые документы выглядят так, как показано выше, вы воссмотрели дубликаты — семь отдельных документов для двойника Доум только отеля. Запрос к "гостиницам в Флориде" возвратит семь результатов только для двойника Доум отеля, помещая другие важные Гостиницы в результаты поиска.

Чтобы получить ожидаемый опыт одного документа на гостиницу, необходимо предоставить набор строк с правильной гранулярностью, но с полной информацией. К счастью, вы можете легко сделать это, применив методы, описанные в этой статье.

## <a name="define-a-query-that-returns-embedded-json"></a>Определение запроса, возвращающего внедренный код JSON

Чтобы обеспечить ожидаемый Поиск, набор данных должен состоять из одной строки для каждого документа поиска в Когнитивный поиск Azure. В нашем примере мы хотим получить по одной строке для каждого отеля, но нам нужно, чтобы наши пользователи могли выполнять поиск по другим связанным с комнатам полям, таким как ночная ставка, размер и количество кроватях, или представление пляже, все из которых являются частью сведений о комнате.

Решение состоит в том, чтобы записать сведения о комнате как вложенные данные JSON, а затем вставить структуру JSON в поле представления, как показано на втором шаге. 

1. Предположим, что у вас есть две соединяемые таблицы, Гостиницы $ и комнаты $, которые содержат сведения о 50 гостиницах и 750 комнатах и соединены с полем Хотелид. По отдельности эти таблицы содержат 50 гостиниц и 750 связанных комнат.

    ```sql
    CREATE TABLE [dbo].[Hotels$](
      [HotelID] [nchar](10) NOT NULL,
      [HotelName] [nvarchar](255) NULL,
      [Description] [nvarchar](max) NULL,
      [Description_fr] [nvarchar](max) NULL,
      [Category] [nvarchar](255) NULL,
      [Tags] [nvarchar](255) NULL,
      [ParkingIncluded] [float] NULL,
      [SmokingAllowed] [float] NULL,
      [LastRenovationDate] [smalldatetime] NULL,
      [Rating] [float] NULL,
      [StreetAddress] [nvarchar](255) NULL,
      [City] [nvarchar](255) NULL,
      [State] [nvarchar](255) NULL,
      [ZipCode] [nvarchar](255) NULL,
      [GeoCoordinates] [nvarchar](255) NULL
    ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
    GO

    CREATE TABLE [dbo].[Rooms$](
      [HotelID] [nchar](10) NULL,
      [Description] [nvarchar](255) NULL,
      [Description_fr] [nvarchar](255) NULL,
      [Type] [nvarchar](255) NULL,
      [BaseRate] [float] NULL,
      [BedOptions] [nvarchar](255) NULL,
      [SleepsCount] [float] NULL,
      [SmokingAllowed] [float] NULL,
      [Tags] [nvarchar](255) NULL
    ) ON [PRIMARY]
    GO
    ```

2. Создайте представление, состоящее из всех полей в родительской таблице ( `SELECT * from dbo.Hotels$` ), с добавлением нового поля *комнаты* , которое содержит выходные данные вложенного запроса. Предложение **for JSON Auto** для `SELECT * from dbo.Rooms$` структур выходных данных в виде JSON. 

     ```sql
   CREATE VIEW [dbo].[HotelRooms]
   AS
   SELECT *, (SELECT *
              FROM dbo.Rooms$
              WHERE dbo.Rooms$.HotelID = dbo.Hotels$.HotelID FOR JSON AUTO) AS Rooms
   FROM dbo.Hotels$
   GO
   ```

   На следующем снимке экрана показано итоговое представление с полем *комнаты* nvarchar внизу. Поле " *комнаты* " существует только в представлении хотелрумс.

   ![Хотелрумс, представление](media/index-sql-relational-data/hotelsrooms-view.png "Хотерумс, представление")

1. Выполните команду `SELECT * FROM dbo.HotelRooms` , чтобы получить набор строк. Этот запрос возвращает 50 строк, по одному на гостиницу, со связанной информацией о комнате в виде коллекции JSON. 

   ![Набор строк из представления Хотелрумс](media/index-sql-relational-data/hotelrooms-rowset.png "Набор строк из представления Хотелрумс")

Теперь этот набор строк готов к импорту в Когнитивный поиск Azure.

> [!NOTE]
> При таком подходе предполагается, что размер внедренного JSON ограничен [максимальным размером столбца SQL Server](/sql/sql-server/maximum-capacity-specifications-for-sql-server). 

 ## <a name="use-a-complex-collection-for-the-many-side-of-a-one-to-many-relationship"></a>Использование комплексной коллекции для связи «многие» отношения «один ко многим»

На стороне Azure Когнитивный поиск Создайте схему индекса, которая моделирует связь "один ко многим" с помощью вложенного JSON. Результирующий набор, созданный в предыдущем разделе, обычно соответствует схеме индекса, приведенной ниже (мы вырезаны некоторые поля для краткости).

Следующий пример аналогичен примеру в [модели моделирования сложных типов данных](search-howto-complex-data-types.md#creating-complex-fields). Структура *помещений* , которая была сосредоточена на этой статье, находится в коллекции полей индекса под названием *Гостиницы*. В этом примере также показан сложный тип для *Address*, который отличается от *комнат* в том, что он состоит из фиксированного набора элементов, в отличие от множества, разрешенного произвольного числа элементов в коллекции.

```json
{
  "name": "hotels",
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
    { "name": "Description_fr", "type": "Edm.String", "searchable": true, "analyzer": "fr.lucene" },
    { "name": "Category", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "facetable": true },
    { "name": "Address", "type": "Edm.ComplexType",
      "fields": [
        { "name": "StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true },
        { "name": "City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
        { "name": "StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true }
      ]
    },
    { "name": "Rooms", "type": "Collection(Edm.ComplexType)",
      "fields": [
        { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
        { "name": "Description_fr", "type": "Edm.String", "searchable": true, "analyzer": "fr.lucene" },
        { "name": "Type", "type": "Edm.String", "searchable": true },
        { "name": "BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true },
        { "name": "BedOptions", "type": "Edm.String", "searchable": true, "filterable": true, "facetable": true },
        { "name": "SleepsCount", "type": "Edm.Int32", "filterable": true, "facetable": true },
        { "name": "SmokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true },
        { "name": "Tags", "type": "Edm.Collection", "searchable": true }
      ]
    }
  ]
}
```

Учитывая предыдущий результирующий набор и схему индекса выше, у вас есть все необходимые компоненты для успешной операции индексирования. Преобразованный набор данных соответствует требованиям к индексированию, но не сохраняет подробные сведения. В индексе Azure Когнитивный поиск результаты поиска будут легко возвращаться в сущности на основе Гостиницы, сохраняя контекст отдельных комнат и их атрибуты.

## <a name="next-steps"></a>Дальнейшие действия

Используя собственный набор данных, можно создать и загрузить индекс с помощью [мастера импорта данных](search-import-data-portal.md) . Мастер обнаруживает встроенную коллекцию JSON, например содержащуюся в *комнатах*, и выводит схему индекса, включающую коллекцию со сложным типом. 

  ![Индекс, выводимый мастером импорта данных](media/index-sql-relational-data/search-index-rooms-complex-collection.png "Индекс, выводимый мастером импорта данных")

Чтобы ознакомиться с основными шагами мастера импорта данных, воспользуйтесь приведенным ниже кратким руководством.

> [!div class="nextstepaction"]
> [Краткое руководство. Создание индекса поиска с помощью портал Azure](search-get-started-portal.md)