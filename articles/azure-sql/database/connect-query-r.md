---
title: Использование языка R с помощью Служб машинного обучения в Базе данных SQL Azure (предварительная версия) для выполнения запросов к базе данных
titleSuffix: Azure SQL Database Machine Learning Services (preview)
description: В этой статье показано, как с помощью сценария R в Службах машинного обучения в Базе данных SQL Azure подключиться к базе данных службы "База данных SQL Azure" и отправлять к ней запросы, используя инструкции Transact-SQL.
services: sql-database
ms.service: sql-database
ms.subservice: machine-learning
ms.custom: sqldbrb=2
ms.devlang: python
ms.topic: quickstart
author: garyericson
ms.author: garye
ms.reviewer: davidph, sstein
manager: cgronlun
ms.date: 05/29/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 2e32a4abeae78aa7105f21ecffbb18c2eae841a4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96185629"
---
# <a name="quickstart-use-r-with-azure-sql-database-machine-learning-services-preview-to-query-a-database"></a>Краткое руководство. Использование языка R с помощью Служб машинного обучения в Базе данных SQL Azure (предварительная версия) для выполнения запросов к базе данных 

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этом кратком руководстве вы используете R и Службы машинного обучения в Базе данных SQL Azure, чтобы подключиться к базе данных службы "База данных SQL Azure", а затем с помощью инструкций T-SQL выполнить запрос данных.

[!INCLUDE[ml-preview-note](../../../includes/sql-database-ml-preview-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- [База данных SQL Azure](single-database-create-quickstart.md).
- [Службы машинного обучения](../managed-instance/machine-learning-services-overview.md) (с поддержкой R) включены.
- [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) (SSMS).

> [!IMPORTANT]
> Скрипты в этой статье предназначены для использования базы данных **Adventure Works**.

Службы машинного обучения с поддержкой R — это функция Базы данных SQL Azure, которая используется для выполнения скриптов R в базе данных. Дополнительные сведения см. [The R Project for Statistical Computing](https://www.r-project.org/) (Проект R для статических вычислений)

## <a name="get-the-sql-server-connection-information"></a>Получение сведений о подключении к SQL Server

Получите сведения, необходимые для подключения к базе данных в службе "База данных SQL Azure". Для дальнейших действий вам понадобится полное имя сервера или имя узла, имя базы данных и данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Перейдите на страницу **Базы данных SQL** или **Управляемые экземпляры SQL**.

3. На странице **Обзор** просмотрите полное имя сервера рядом с полем **Имя сервера** для базы данных в службе "База данных SQL Azure" или полное имя сервера рядом с полем **Узел** для управляемого экземпляра в Управляемом экземпляре SQL Azure. Чтобы скопировать имя сервера или имя узла, наведите на него указатель мыши и щелкните значок **копирования**.

## <a name="create-code-to-query-your-database"></a>Создание кода для запроса к базе данных

1. Откройте **SQL Server Management Studio** и подключитесь к базе данных.

   Если вам нужна помощь с подключением, ознакомьтесь с разделом [Краткое руководство. Подключение к базе данных в службе "База данных SQL Azure" и создание запросов к ней с помощью SQL Server Management Studio](connect-query-ssms.md).

1. Передайте полный сценарий R в хранимую процедуру [sp_execute_external_script](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql).

   Сценарий передается с помощью аргумента `@script`. Все, что находится внутри аргумента `@script`, должно быть допустимым кодом R.
   
   >[!IMPORTANT]
   >Данный пример кода использует данные AdventureWorksLT, которые можно выбрать в качестве источника при создании базы данных. Если в вашей базе данных содержатся другие данные, используйте таблицы из собственной базы данных в запросе SELECT. 

    ```sql
    EXECUTE sp_execute_external_script
    @language = N'R'
    , @script = N'OutputDataSet <- InputDataSet;'
    , @input_data_1 = N'SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName FROM [SalesLT].[ProductCategory] pc JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid'
    ```

   > [!NOTE]
   > Если вы получаете сообщения о каких-либо ошибках, это может быть связано с тем, что общедоступная предварительная версия Служб машинного обучения (с использованием R) не включена для базы данных. См. [предварительные требования](#prerequisites) выше.

## <a name="run-the-code"></a>Выполнение кода

1. Выполните хранимую процедуру [sp_execute_external_script](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql).

1. В окне **Сообщения**  убедитесь, что возвращены первые 20 строк с продуктами из категории.

## <a name="next-steps"></a>Дальнейшие действия

- [Учебник. Проектирование реляционной базы данных в службе "База данных SQL Azure" с помощью SSMS](design-first-database-tutorial.md)
- [Machine Learning Services (with R) in Azure SQL Database (preview)](../managed-instance/machine-learning-services-overview.md) (Службы машинного обучения (с использованием R) в Базе данных SQL Azure (предварительная версия))
- [Создание и выполнение простых сценариев R в Службах машинного обучения в Базе данных SQL Azure (предварительная версия)](/sql/machine-learning/tutorials/quickstart-r-create-script?context=%2fazure%2fazure-sql%2fmanaged-instance%2fcontext%2fml-context)