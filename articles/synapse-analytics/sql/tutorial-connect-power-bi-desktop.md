---
title: Руководство по подключению бессерверного пула SQL к Power BI Desktop и созданию отчета
description: В этом руководстве объясняется, как подключить бессерверный пул SQL в Azure Synapse Analytics к Power BI Desktop и создать демонстрационный отчет на основе представления.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 8dd3edd25d21bfcd0fde1bc8b5f103877d968c8a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98119991"
---
# <a name="tutorial-use-serverless-sql-pool-with-power-bi-desktop--create-a-report"></a>Руководство по использованию бессерверного пула SQL с Power BI Desktop и созданию отчета

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
>
> - создание демонстрационной базы данных;
> - создание представления для отчета;
> - Подключение Power BI Desktop к бессерверному пулу SQL
> - создание отчета на основе представления.

## <a name="prerequisites"></a>Предварительные требования

Для работы с данным руководством вам потребуется:

- [Power BI Desktop](https://powerbi.microsoft.com/downloads/) — требуется для визуализации данных и создания отчета.
- [Рабочая область Azure Synapse](../get-started-create-workspace.md) — требуется для создания базы данных, внешнего источника данных и представления.

Необязательное действие:

- инструмент SQL-запросов, например [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) или [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).

Значения для следующих параметров:

| Параметр                                 | Описание                                                   |
| ----------------------------------------- | ------------------------------------------------------------- |
| Адрес конечной точки службы бессерверного пула SQL    | Используется в качестве имени сервера.                                   |
| Регион конечной точки службы бессерверного пула SQL     | Используется для определения хранилища, используемого в примерах. |
| Имя пользователя и пароль для доступа к конечной точке | Используется для доступа к конечной точке.                               |
| База данных, которая будет использоваться для создания представлений     | База данных, используемая в качестве начальной точки в примерах.       |

## <a name="1---create-database"></a>1\. Создание базы данных

В качестве демонстрационной среды создайте собственную демонстрационную базу данных. Эта база данных будет использоваться для просмотра метаданных, а не для хранения фактических данных.

Создайте демонстрационную базу данных (и при необходимости удалите существующую), выполнив следующий скрипт Transact-SQL (T-SQL):

```sql
-- Drop database if it exists
DROP DATABASE IF EXISTS Demo
GO

-- Create new database
CREATE DATABASE [Demo];
GO
```

## <a name="2---create-data-source"></a>2\. Создание источника данных

Для доступа бессерверного пула SQL к файлам в хранилище необходим источник данных. Создайте источник данных для учетной записи хранения, расположенной в том же регионе, что и конечная точка. Хотя бессерверный пул SQL может получать доступ к учетным записям хранения из разных регионов, хранилище и конечная точка, принадлежащие к одному региону, обеспечивают лучшую производительность.

Создайте источник данных, выполнив следующий скрипт T-SQL:

```sql
-- There is no credential in data surce. We are using public storage account which doesn't need a secret.
CREATE EXTERNAL DATA SOURCE AzureOpenData
WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/')
```

## <a name="3---prepare-view"></a>3\. Подготовка представления

Создайте представление на основе внешних демонстрационных данных для Power BI, выполнив следующий скрипт T-SQL:

Создайте представление `usPopulationView` в `Demo` базы данных с помощью следующего запроса:

```sql
DROP VIEW IF EXISTS usPopulationView;
GO

CREATE VIEW usPopulationView AS
SELECT
    *
FROM
    OPENROWSET(
        BULK 'censusdatacontainer/release/us_population_county/year=20*/*.parquet',
        DATA_SOURCE = 'AzureOpenData',
        FORMAT='PARQUET'
    ) AS uspv;
```

Демонстрационные данные содержат указанные ниже наборы данных:

Данные о населении США в каждом округе США с разбивкой по половому и расовому признаку, полученные в результате переписи населения, осуществляемой раз в десять лет (за 2000–2010 гг.), в формате Parquet.

| Путь к папке                                                  | Описание                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /release/                                                    | Родительская папка для данных в демонстрационной учетной записи хранения.               |
| /release/us_population_county/                               | Файлы данных о населении США в формате Parquet, секционированные по годам с использованием схемы секционирования Hive на платформе Hadoop. |

## <a name="4---create-power-bi-report"></a>4\. Создание отчета Power BI

Создайте отчет для Power BI Desktop, выполнив следующие действия:

1. Откройте приложение Power BI Desktop и выберите раздел **Получить данные**.

   ![Открытие приложения Power BI Desktop и выбор раздела "Получить данные"](./media/tutorial-connect-power-bi-desktop/step-0-open-powerbi.png)

2. Выберите элементы **Azure** > **База данных SQL Azure**. 

   ![Выбор источника данных](./media/tutorial-connect-power-bi-desktop/step-1-select-data-source.png)

3. Введите имя сервера, на котором расположена база данных, в поле **Сервер**, затем введите `Demo` в поле имени базы данных. Выберите параметр **Импортировать** и нажмите кнопку **ОК**. 

   ![Выбор базы данных в конечной точке](./media/tutorial-connect-power-bi-desktop/step-2-db.png)

4. Выберите предпочтительный метод проверки подлинности:

    - Пример для Azure Active Directory. 
  
        ![Нажатие кнопки "Вход"](./media/tutorial-connect-power-bi-desktop/step-2.1-select-aad-auth.png)

    - Пример для имени входа SQL. Введите имя пользователя и пароль.

        ![Использование имени SQL для входа.](./media/tutorial-connect-power-bi-desktop/step-2.2-select-sql-auth.png)


5. Выберите представление `usPopulationView`, а затем нажмите кнопку **Загрузить**. 

   ![Выбор представления для указанной базы данных](./media/tutorial-connect-power-bi-desktop/step-3-select-view.png)

6. Дождитесь завершения операции, после чего появится всплывающее окно с сообщением `There are pending changes in your queries that haven't been applied` (В запросах есть ожидающие изменения, которые не были применены.). Нажмите кнопку **Применить изменения**. 

   ![Кнопка "Применить изменения"](./media/tutorial-connect-power-bi-desktop/step-4-apply-changes.png)

7. Дождитесь закрытия диалогового окна **Применить изменения запросов**, что может занять несколько минут. 

   ![Ожидание завершения запроса](./media/tutorial-connect-power-bi-desktop/step-5-wait-for-query-to-finish.png)

8. После завершения загрузки выберите следующие столбцы в указанном порядке, чтобы создать отчет:
   - countyName;
   - заполнение
   - stateName.

   ![Выбор необходимых столбцов, чтобы создать отчет с картой](./media/tutorial-connect-power-bi-desktop/step-6-select-columns-of-interest.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Завершив работу с этим отчетом, удалите ресурсы, выполнив следующие действия:

1. Удалите учетные данные для учетной записи хранения:

   ```sql
   DROP EXTERNAL DATA SOURCE AzureOpenData
   ```

2. Удалите представление:

   ```sql
   DROP VIEW usPopulationView;
   ```

3. Удалите базу данных:

   ```sql
   DROP DATABASE Demo;
   ```

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к статье о [файлах хранилища запросов](develop-storage-files-overview.md), чтобы узнать, как выполнять запросы к файлам хранилища с помощью SQL Synapse.