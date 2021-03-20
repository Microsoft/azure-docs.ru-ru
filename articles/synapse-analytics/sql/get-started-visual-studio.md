---
title: Подключение и запрос синапсе SQL с помощью Visual Studio и SSDT
description: Используйте Visual Studio для запроса выделенного пула SQL с помощью Azure синапсе Analytics.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: ef8e2a3d1a6b78e8f2b6b9a900ed2485c1a4a5d7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96451593"
---
# <a name="connect-to-synapse-sql-with-visual-studio-and-ssdt"></a>Подключение к Synapse SQL с помощью Visual Studio и SSDT

> [!div class="op_single_selector"]
> * [Azure Data Studio](get-started-azure-data-studio.md)
> * [Power BI](get-started-power-bi-professional.md)
> * [Visual Studio](get-started-visual-studio.md)
> * [sqlcmd](get-started-connect-sqlcmd.md) 
> * [SSMS](get-started-ssms.md)
> 
> 

Используйте Visual Studio для запроса выделенного пула SQL с помощью Azure синапсе Analytics. В этом методе используется расширение SQL Server Data Tools (SSDT) в Visual Studio 2019. 

> [!NOTE]
> Несерверный пул SQL не поддерживается SSDT.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником необходимы следующие компоненты.

* Существующий выделенный пул SQL. Если у вас ее нет, см. раздел [Создание выделенного пула SQL](../sql-data-warehouse/create-data-warehouse-portal.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) для выполнения этого предварительного требования.
* Расширение SSDT для Visual Studio. Скорее всего, этот компонент у вас уже есть, если на вашем компьютере установлено приложение Visual Studio. Инструкции по установке и доступные варианты установки приведены в статье [Начало работы с Visual Studio 2019](../sql-data-warehouse/sql-data-warehouse-install-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
* Полное имя сервера SQL Server. Чтобы найти это имя сервера, см. раздел [Подключение к выделенному пулу SQL](connect-overview.md).

## <a name="1-connect-to-a-dedicated-sql-pool"></a>1. подключение к выделенному пулу SQL
1. Откройте Visual Studio 2019.
2. Откройте обозреватель объектов SQL Server, выбрав **Вид** > **Обозреватель объектов SQL Server**.
   
    ![Обозреватель объектов SQL Server](./media/get-started-visual-studio/open-ssdt.png)
3. Щелкните значок **Добавить SQL Server** .
   
    ![Добавить SQL Server](./media/get-started-visual-studio/add-server.png)
4. Заполните поля в окне «Подключение к серверу».
   
    ![Подключение к серверу](./media/get-started-visual-studio/connection-dialog.png)
   
   * **Имя сервера**: Введите найденное **имя сервера** .
   * **Аутентификация**. Выберите **Проверка подлинности SQL Server** или **Встроенная проверка подлинности Active Directory**.
   * **Имя пользователя** и **пароль**. Если вы выбрали проверку подлинности SQL Server, введите имя пользователя и пароль.
   * Нажмите кнопку **Соединить**.
5. Чтобы исследовать данные, разверните сервер Azure SQL Server. Вы можете просмотреть базы данных, связанные с сервером. Разверните AdventureWorksDW, чтобы просмотреть таблицы в образце базы данных.
   
    ![Обзор AdventureWorksDW](./media/get-started-visual-studio/explore-sample.png)

## <a name="2-run-a-sample-query"></a>2. Запуск пробного запроса
Теперь, когда вы подключились к базе данных, следует написать запрос.

1. Щелкните правой кнопкой мыши базу данных в обозревателе объектов SQL Server.
2. Выберите **Создать запрос**. Откроется новое окно запроса.
   
    ![Создать запрос](./media/get-started-visual-studio/new-query2.png)
3. Скопируйте следующий запрос T-SQL в окно запроса.
   
    ```sql
    SELECT COUNT(*) FROM dbo.FactInternetSales;
    ```
4. Запустите запрос, щелкнув зеленую стрелку или нажав клавиши `CTRL`+`SHIFT`+`E`.
   
    ![Выполнение запроса](./media/get-started-visual-studio/run-query.png)
5. Просмотрите результаты запроса. В этом примере таблица FactInternetSales содержит 60 398 строк.
   
    ![Результаты запроса](./media/get-started-visual-studio/query-results.png)

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы можете подключаться к базе данных и отправлять запросы, попробуйте [визуализировать данные с помощью Power BI](get-started-power-bi-professional.md).
Сведения о настройке среды для проверки подлинности Azure Active Directory см. [в разделе аутентификация в выделенном пуле SQL](../sql-data-warehouse/sql-data-warehouse-authentication.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
 