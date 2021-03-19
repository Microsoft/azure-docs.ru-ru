---
title: Параметризация связанных служб в Фабрике данных Azure
description: Узнайте, как параметризовать связанные службы в Фабрике данных Azure и как во время выполнения передавать динамические значения.
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/18/2021
author: dcstwh
ms.author: weetok
ms.openlocfilehash: df26b77f37100ae41b26c013c57cccbfa0d7e205
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104595589"
---
# <a name="parameterize-linked-services-in-azure-data-factory"></a>Параметризация связанных служб в Фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Теперь можно параметризовать связанную службу и передавать динамические значения во время выполнения. Например, если вы хотите подключиться к разным базам данных на одном логическом сервере SQL Server, теперь можно параметризовать имя базы данных в определении связанной службы. Это не позволит вам создавать связанную службу для каждой базы данных на логическом сервере SQL Server. Например, можно параметризовать другие свойства в определении связанной службы, например *Имя пользователя*.

Пользовательский интерфейс фабрики данных можно использовать в портал Azure или в программном интерфейсе для параметризации связанных служб.

> [!TIP]
> Мы рекомендуем не параметризировать пароли или секреты. Вместо этого храните все строки подключения в Azure Key Vault и параметризируйте *Имя секрета*.

> [!Note]
> Есть открытая ошибка для использования "-" в именах параметров. рекомендуется использовать имена без "-", пока ошибка не будет устранена.

Уделите 7 минут своего времени, чтобы просмотреть следующее видео с кратким обзором и демонстрацией этой функции.

> [!VIDEO https://channel9.msdn.com/shows/azure-friday/Parameterize-connections-to-your-data-stores-in-Azure-Data-Factory/player]

## <a name="supported-linked-service-types"></a>Поддерживаемые типы связанных служб

Можно параметризовать любой тип связанной службы.
При создании связанной службы на пользовательском интерфейсе фабрика данных предоставляет встроенные возможности параметризации для следующих типов связанных служб. В колонке создание и изменение связанной службы можно найти параметры для новых параметров и добавить динамическое содержимое.

- Amazon Redshift
- Amazon S3
- хранилище BLOB-объектов Azure
- Azure Cosmos DB (SQL API)
- Azure Data Lake Storage 2-го поколения
- База данных Azure для MySQL
- Azure Databricks
- Azure Key Vault
- База данных SQL Azure
- Управляемый экземпляр SQL Azure
- Azure Synapse Analytics 
- Хранилище таблиц Azure
- Базовый протокол HTTP
- Базовый протокол REST
- MySQL
- Oracle;
- SQL Server

Для других типов связанных служб, которых нет в приведенном выше списке, можно параметризовать связанную службу, отредактировав JSON в пользовательском интерфейсе:

- В колонке создания или изменения связанной службы разверните внизу узел "Дополнительно", установите флажок "Указать динамическое содержимое в формате JSON" и укажите полезные данные JSON для связанной службы. 
- Кроме того, после создания связанной службы без параметризации в [концентраторе управления](author-visually.md#management-hub) — > связанные службы — > найти конкретную связанную службу > нажмите кнопку "код" (кнопка " {} "), чтобы изменить JSON. 

См. [пример JSON](#json) , чтобы добавить ` parameters` раздел для определения параметров и ссылки на параметр с помощью ` @{linkedService().paraName} ` .

## <a name="data-factory-ui"></a>Пользовательский интерфейс Фабрики данных

![Добавление динамического содержимого к определению связанной службы](media/parameterize-linked-services/parameterize-linked-services-image1.png)

![Создание параметра](media/parameterize-linked-services/parameterize-linked-services-image2.png)

## <a name="json"></a>JSON

```json
{
    "name": "AzureSqlDatabase",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:myserver.database.windows.net,1433;Database=@{linkedService().DBName};User ID=user;Password=fake;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        },
        "connectVia": null,
        "parameters": {
            "DBName": {
                "type": "String"
            }
        }
    }
}
```
