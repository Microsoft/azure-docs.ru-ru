---
title: Подключение к выделенному пулу SQL (ранее — хранилище данных SQL) с помощью VSTS
description: Запросите выделенный пул SQL (ранее — хранилище данных SQL) в Azure синапсе Analytics с помощью Visual Studio.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 08/15/2019
ms.author: kevin
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 2b81ddedbcb254a840e85d41cf9d69c78b149bbd
ms.sourcegitcommit: aacbf77e4e40266e497b6073679642d97d110cda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/12/2021
ms.locfileid: "98121402"
---
# <a name="connect-to-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics-with-visual-studio-and-ssdt"></a>Подключение к выделенному пулу SQL (ранее — хранилище данных SQL) в Azure синапсе Analytics с помощью Visual Studio и SSDT

> [!div class="op_single_selector"]
> * [Azure Data Studio](../sql/get-started-azure-data-studio.md)
> * [Power BI](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](../sql/get-started-connect-sqlcmd.md) 
> * [SSMS](sql-data-warehouse-query-ssms.md)
> 
> 

Используйте Visual Studio для запроса выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе всего за несколько минут. В этом методе используется расширение SQL Server Data Tools (SSDT) в Visual Studio 2019. 

## <a name="prerequisites"></a>Предварительные требования
Для работы с этим руководством необходимы указанные ниже компоненты.

* Существующий выделенный пул SQL (ранее — хранилище данных SQL). Чтобы создать его, см. раздел [Создание выделенного пула SQL (ранее — хранилище данных SQL)](create-data-warehouse-portal.md).
* Расширение SSDT для Visual Studio. Если у вас есть Visual Studio, возможно, у вас уже есть SSDT для Visual Studio. Инструкции по установке и доступные варианты установки приведены в статье [Начало работы с Visual Studio 2019](sql-data-warehouse-install-visual-studio.md).
* Полное имя сервера SQL Server. Чтобы найти эти сведения, см. раздел [Подключение к выделенному пулу SQL (ранее — хранилище данных SQL)](sql-data-warehouse-connect-overview.md).

## <a name="1-connect-to-your-dedicated-sql-pool-formerly-sql-dw"></a>1. подключение к выделенному пулу SQL (ранее — хранилище данных SQL)
1. Откройте Visual Studio 2019.
2. Откройте обозреватель объектов SQL Server, выбрав **представление**  >  **Обозреватель объектов SQL Server**.
   
    ![Обозреватель объектов SQL Server](./media/sql-data-warehouse-query-visual-studio/open-ssdt.png)
3. Щелкните значок **Добавить SQL Server** .
   
    ![Добавить SQL Server](./media/sql-data-warehouse-query-visual-studio/add-server.png)
4. Заполните поля в окне «Подключение к серверу».
   
    ![Подключение к серверу](./media/sql-data-warehouse-query-visual-studio/connection-dialog.png)
   
   * **Имя сервера**. Введите найденное **имя сервера** .
   * **Проверка подлинности**. Выберите **Проверка подлинности SQL Server** или **Встроенная проверка подлинности Active Directory**.
   * **Имя пользователя** и **пароль**. Если вы выбрали проверку подлинности SQL Server, введите имя пользователя и пароль.
   * Нажмите кнопку **Соединить**.
5. Чтобы исследовать данные, разверните сервер Azure SQL Server. Вы можете просмотреть базы данных, связанные с сервером. Разверните AdventureWorksDW, чтобы просмотреть таблицы в образце базы данных.
   
    ![Обзор AdventureWorksDW](./media/sql-data-warehouse-query-visual-studio/explore-sample.png)

## <a name="2-run-a-sample-query"></a>2. Запуск пробного запроса
Теперь, когда мы подключились к базе данных, давайте напишем запрос.

1. Щелкните правой кнопкой мыши базу данных в обозревателе объектов SQL Server.
2. Выберите **Создать запрос**. Откроется новое окно запроса.
   
    ![Создать запрос](./media/sql-data-warehouse-query-visual-studio/new-query2.png)
3. Скопируйте следующий запрос T-SQL в окно запроса.
   
    ```sql
    SELECT COUNT(*) FROM dbo.FactInternetSales;
    ```
4. Запустите запрос, щелкнув зеленую стрелку или нажав клавиши `CTRL`+`SHIFT`+`E`.
   
    ![Выполнение запроса](./media/sql-data-warehouse-query-visual-studio/run-query.png)
5. Просмотрите результаты запроса. В этом примере таблица FactInternetSales содержит 60 398 строк.
   
    ![Результаты запроса](./media/sql-data-warehouse-query-visual-studio/query-results.png)

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы можете подключаться к базе данных и отправлять запросы, попробуйте [визуализировать данные с помощью Power BI](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect).

Сведения о настройке среды для проверки подлинности Azure Active Directory см. в разделе [Аутентификация в выделенном пуле SQL (ранее — хранилище данных SQL)](sql-data-warehouse-authentication.md).