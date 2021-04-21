---
title: Подключение к Synapse SQL с помощью Azure Data Studio
description: Использование Azure Data Studio в Azure Synapse Analytics для подключения и отправки запросов к Synapse SQL.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 1529e45bd7f799a727a29c8c2e26f7ed77c4e2a0
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565599"
---
# <a name="connect-to-synapse-sql-with-azure-data-studio"></a>Подключение к Synapse SQL с помощью Azure Data Studio

> [!div class="op_single_selector"]
>
> * [Azure Data Studio](get-started-azure-data-studio.md)
> * [Power BI](get-started-power-bi-professional.md)
> * [Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
> * [sqlcmd](get-started-connect-sqlcmd.md)
> * [SSMS](get-started-ssms.md)

Вы можете использовать [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio?view=azure-sqldw-latest&preserve-view=true) в Azure Synapse Analytics для подключения и отправки запросов к Synapse SQL. 

## <a name="connect"></a>Подключение

Чтобы подключиться к Synapse SQL, откройте Azure Data Studio и выберите **Новое подключение**.

![Откройте Azure Data Studio.](./media/get-started-azure-data-studio/1-start.png)

Выберите **Microsoft SQL Server** в качестве **типа подключения**.

Для подключения требуются следующие параметры.

* **Сервер:** сервер в формате `<Azure Synapse workspace name>`-ondemand.sql.azuresynapse.net
* **База данных:** Имя базы данных

> [!NOTE]
> Если вы хотите использовать **бессерверный пул SQL**, URL-адрес должен выглядеть следующим образом:
>
> - `<Azure Synapse workspace name>`-ondemand.sql.azuresynapse.net.
>
> Если же вы хотите использовать **выделенный пул SQL**, URL-адрес должен выглядеть так:
>
> - `<Azure Synapse workspace name>`.sql.azuresynapse.net

Выберите в качестве **типа проверки подлинности** **Проверка подлинности Windows**, **Azure Active Directory** или **Имя входа SQL**.

Чтобы использовать **Имя входа SQL** в качестве типа проверки подлинности, добавьте параметры имени пользователя или пароля:

* **Пользователь**: пользователь сервера в формате `<User>`
* **Пароль:** пароль, связанный с пользователем

Чтобы использовать Azure Active Directory, следует выбрать необходимый тип проверки подлинности.

![Проверка подлинности AAD](./media/get-started-azure-data-studio/3-aad-auth.png)

На следующем снимке экрана показаны **сведения о подключении** для **проверки подлинности Windows**:

![Проверка подлинности Windows](./media/get-started-azure-data-studio/3-windows-auth.png)

На следующем снимке экрана показаны **сведения о подключении** с использованием **имени для входа SQL**:

![Имя входа SQL](./media/get-started-azure-data-studio/2-database-details.png)

После успешного входа вы увидите следующую информационную панель: ![Панель мониторинга](./media/get-started-azure-data-studio/4-dashboard.png)

## <a name="query"></a>Запрос

После подключения Synapse SQL можно запросить с помощью поддерживаемых для экземпляра инструкций [Transact-SQL (T-SQL)](/sql/t-sql/language-reference?view=azure-sqldw-latest&preserve-view=true). Чтобы начать работу, выберите в представлении панели мониторинга **Новый запрос**.

![Новый запрос](./media/get-started-azure-data-studio/5-new-query.png)

Например, чтобы создать [запрос для файлов Parquet](query-parquet-files.md) с помощью бессерверного пула SQL можно использовать следующую инструкцию Transact-SQL:

```sql
SELECT COUNT(*)
FROM  
OPENROWSET(
    BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet',
    FORMAT='PARQUET'
)
```
## <a name="next-steps"></a>Дальнейшие действия 
Узнайте о других способах подключения к Synapse SQL: 

- [SSMS](get-started-ssms.md)
- [Power BI](get-started-power-bi-professional.md)
- [Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [sqlcmd](get-started-connect-sqlcmd.md)

Дополнительные сведения см. в руководстве [Использование Azure Data Studio для подключения и запроса данных с помощью выделенного пула SQL в Azure Synapse Analytics](/sql/azure-data-studio/quickstart-sql-dw).
