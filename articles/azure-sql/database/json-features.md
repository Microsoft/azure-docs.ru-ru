---
title: Работа с данными JSON
description: База данных SQL Azure и Azure SQL Управляемый экземпляр позволяют анализировать, запрашивать и форматировать данные в нотации нотация объектов JavaScript (JSON).
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: ''
ms.date: 04/19/2020
ms.openlocfilehash: 4258c60335a13a6c0f27588aac1c6cdb185fcf6a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92782807"
---
# <a name="getting-started-with-json-features-in-azure-sql-database-and-azure-sql-managed-instance"></a>Приступая к работе с функциями JSON в базе данных SQL Azure и Управляемый экземпляр Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

База данных SQL Azure и Управляемый экземпляр Azure SQL позволяют анализировать и запрашивать данные, представленные в формате нотация объектов JavaScript [(JSON)](https://www.json.org/) , и экспортировать реляционные данные в виде текста JSON. Доступны следующие сценарии JSON:

- [Преобразование реляционных данных в формат JSON](#formatting-relational-data-in-json-format) с помощью предложения `FOR JSON`
- [Работа с данными JSON](#working-with-json-data)
- [Запрос данных JSON](#querying-json-data) с помощью скалярных функций JSON
- [Преобразование данных JSON в табличный формат](#transforming-json-into-tabular-format) с помощью функции `OPENJSON`

## <a name="formatting-relational-data-in-json-format"></a>Преобразование реляционных данных в формат JSON

При наличии веб-службы, которая извлекает данные из слоя базы данных и представляет ответ в формате JSON, либо клиентских платформ или библиотек JavaScript, которые принимают данные в формате JSON, можно преобразовывать содержимое базы данных в формат JSON непосредственно в SQL-запросе. Вам больше не нужно писать код приложения, который форматирует результаты из базы данных SQL Azure или Управляемый экземпляр SQL Azure в формате JSON, или включить некоторые библиотеки сериализации JSON для преобразования результатов табличных запросов и последующей сериализации объектов в формат JSON. Вместо этого можно использовать предложение FOR JSON, чтобы отформатировать результаты запроса SQL в формате JSON и использовать его непосредственно в приложении.

В следующем примере строки из таблицы Sales.Customer преобразовываются в формат JSON с помощью предложения FOR JSON.

```sql
select CustomerName, PhoneNumber, FaxNumber
from Sales.Customers
FOR JSON PATH
```

Предложение FOR JSON PATH форматирует результаты запроса в виде текста JSON. Имена столбцов используются в качестве ключей, а значения ячеек создаются как значения JSON.

```json
[
{"CustomerName":"Eric Torres","PhoneNumber":"(307) 555-0100","FaxNumber":"(307) 555-0101"},
{"CustomerName":"Cosmina Vlad","PhoneNumber":"(505) 555-0100","FaxNumber":"(505) 555-0101"},
{"CustomerName":"Bala Dixit","PhoneNumber":"(209) 555-0100","FaxNumber":"(209) 555-0101"}
]
```

Результирующий набор форматируется как массив JSON, в котором каждая строка форматируется как отдельный объект JSON.

PATH указывает, что можно настроить выходной формат результатов JSON, используя точечную нотацию в псевдонимах столбцов. Приведенный ниже запрос изменяет имя ключа CustomerName в выходном формате JSON и помещает номера телефона и факса в подобъект Contact.

```sql
select CustomerName as Name, PhoneNumber as [Contact.Phone], FaxNumber as [Contact.Fax]
from Sales.Customers
where CustomerID = 931
FOR JSON PATH, WITHOUT_ARRAY_WRAPPER
```

Выходные данные этого запроса имеют следующий вид.

```json
{
    "Name":"Nada Jovanovic",
    "Contact":{
           "Phone":"(215) 555-0100",
           "Fax":"(215) 555-0101"
    }
}
```

В этом примере мы вернули отдельный объект JSON вместо массива, указав параметр [WITHOUT_ARRAY_WRAPPER](/sql/relational-databases/json/remove-square-brackets-from-json-without-array-wrapper-option). Этот параметр можно использовать, если вы знаете, что результатом запроса является отдельный объект.

Основная ценность предложения FOR JSON заключается в том, что оно дает возможность возвращать сложные иерархические данные из базы данных в виде вложенных объектов JSON или массивов JSON. В приведенном ниже примере показано, как добавить строки из таблицы `Orders`, принадлежащие `Customer`, в виде вложенного массива объектов `Orders`.

```sql
select CustomerName as Name, PhoneNumber as Phone, FaxNumber as Fax,
        Orders.OrderID, Orders.OrderDate, Orders.ExpectedDeliveryDate
from Sales.Customers Customer
    join Sales.Orders Orders
        on Customer.CustomerID = Orders.CustomerID
where Customer.CustomerID = 931
FOR JSON AUTO, WITHOUT_ARRAY_WRAPPER
```

Вместо того, чтобы отправлять отдельные запросы для получения данных клиента (Customer), а затем получать список соответствующих заказов (Orders), можно получить все необходимые данные с помощью одного запроса, как показано в приведенном ниже примере выходных данных.

```json
{
  "Name":"Nada Jovanovic",
  "Phone":"(215) 555-0100",
  "Fax":"(215) 555-0101",
  "Orders":[
    {"OrderID":382,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":395,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":1657,"OrderDate":"2013-01-31","ExpectedDeliveryDate":"2013-02-01"}
  ]
}
```

## <a name="working-with-json-data"></a>Работа с данными JSON

Если у вас нет строго структурированных данных, имеются сложные вложенные объекты, массивы или иерархические данные либо структуры данных со временем развиваются, то формат JSON позволит представить любую сложную структуру данных.

JSON — это текстовый формат, который можно использовать как любой другой строковый тип в базе данных SQL Azure и Управляемый экземпляр Azure SQL. Данные JSON можно отправлять или хранить как стандартные данные типа NVARCHAR.

```sql
CREATE TABLE Products (
  Id int identity primary key,
  Title nvarchar(200),
  Data nvarchar(max)
)
go
CREATE PROCEDURE InsertProduct(@title nvarchar(200), @json nvarchar(max))
AS BEGIN
    insert into Products(Title, Data)
    values(@title, @json)
END
```

Данные JSON, используемые в этом примере, представлены с помощью типа NVARCHAR(MAX). Данные JSON можно вставить в эту таблицу или указать в качестве аргумента хранимой процедуры, используя стандартный синтаксис Transact-SQL, как показано в следующем примере.

```sql
EXEC InsertProduct 'Toy car', '{"Price":50,"Color":"White","tags":["toy","children","games"]}'
```

Любой язык или библиотека на стороне клиента, которая работает с строковыми данными в базе данных SQL Azure и Azure SQL Управляемый экземпляр, также будет работать с данными JSON. Данные JSON могут храниться в таблице, которая поддерживает тип NVARCHAR. Например, это может быть оптимизированная для памяти таблица или таблица с системным управлением версиями. Формат JSON не накладывает каких-либо ограничений на код на стороне клиента или на уровне базы данных.

## <a name="querying-json-data"></a>Запрос данных JSON

Если данные в формате JSON хранятся в таблицах Azure SQL, то с помощью функций JSON эти данные можно использовать в SQL-запросе.

Функции JSON, доступные в базе данных SQL Azure и Azure SQL Управляемый экземпляр позволяют обрабатывать данные в формате JSON как любые другие типы данных SQL. Можно легко извлечь значения из текста JSON и использовать данные JSON в любом запросе.

```sql
select Id, Title, JSON_VALUE(Data, '$.Color'), JSON_QUERY(Data, '$.tags')
from Products
where JSON_VALUE(Data, '$.Color') = 'White'

update Products
set Data = JSON_MODIFY(Data, '$.Price', 60)
where Id = 1
```

Функция JSON_VALUE извлекает значение из текста JSON, хранящегося в столбце данных. Эта функция использует путь, как в JavaScript, для указания ссылки на извлекаемое значение в тексте JSON. Извлеченное значение может использоваться в любой части SQL-запроса.

Функция JSON_QUERY аналогична функции JSON_VALUE. В отличие от JSON_VALUE, эта функция извлекает сложный подобъект, например массивы или объекты, которые помещаются в текст JSON.

Функция JSON_MODIFY позволяет указать путь к значению в тексте JSON, которое следует обновить, а также новое значение, которым заменяется старое значение. Таким образом можно легко обновить текст JSON, не анализируя всю структуру.

Так как данные JSON хранятся в виде обычного текста, то нет никакой гарантии, что значения, хранящиеся в текстовых столбцах, отформатированы правильно. Вы можете проверить, правильно ли отформатирован текст, хранящийся в столбце JSON, с помощью стандартных проверочных ограничений Базы данных SQL Azure и функции ISJSON.

```sql
ALTER TABLE Products
    ADD CONSTRAINT [Data should be formatted as JSON]
        CHECK (ISJSON(Data) > 0)
```

Если входной текст имеет правильный формат JSON, то функция ISJSON возвращает значение 1. При каждой операции вставки или обновления столбца JSON это ограничение будет проверять, правилен ли формат JSON нового текстового значения.

## <a name="transforming-json-into-tabular-format"></a>Преобразование данных JSON в табличный формат

База данных SQL Azure и Azure SQL Управляемый экземпляр также позволяют преобразовывать коллекции JSON в табличный формат и загружать или запрашивать данные JSON.

OPENJSON — это функция с табличным значением, которая анализирует текст JSON, находит массив объектов JSON, выполняет итерацию по элементам массива и возвращает одну строку для каждого элемента массива в выходных данных.

![Табличное представление JSON](./media/json-features/image_2.png)

В приведенном выше примере можно указать, где искать массив JSON, который нужно открыть (в пути $.Orders), какие столбцы должны быть возвращены в качестве результата и где искать значения JSON, которые будут возвращены в виде ячеек.

В переменной @orders можно преобразовать массив JSON в набор строк, проанализировать этот результирующий набор или вставить строки в стандартную таблицу.

```sql
CREATE PROCEDURE InsertOrders(@orders nvarchar(max))
AS BEGIN

    insert into Orders(Number, Date, Customer, Quantity)
    select Number, Date, Customer, Quantity
    FROM OPENJSON (@orders)
     WITH (
            Number varchar(200),
            Date datetime,
            Customer varchar(200),
            Quantity int
     )
END
```

Набор заказов, отформатированный как массив JSON и указанный в качестве параметра хранимой процедуры, можно проанализировать и вставить в таблицу Orders.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как интегрировать JSON в приложение, ознакомьтесь с приведенным ниже материалами.

Чтобы узнать о различных сценариях интеграции JSON в приложение, ознакомьтесь с роликами [Channel 9](https://channel9.msdn.com/Events/DataDriven/SQLServer2016/JSON-as-a-bridge-betwen-NoSQL-and-relational-worlds) или найдите подходящий вам сценарий в [записях блога о JSON](/archive/blogs/sqlserverstorageengine/json-in-sql-server-use-cases).