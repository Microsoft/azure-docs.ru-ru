---
title: Запрос данных Azure Блокчейн Workbench с помощью SQL Server Management Studio
description: Сведения о подключении к базе данных SQL Azure Blockchain Workbench из SQL Server Management Studio.
ms.date: 11/20/2019
ms.topic: how-to
ms.service: azure-blockchain
ms.reviewer: mmercuri
ms.openlocfilehash: 16e7f9a6c36ea42e1d0a4144e680baebee5a6c21
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96009472"
---
# <a name="using-azure-blockchain-workbench-data-with-sql-server-management-studio"></a>Использование данных Azure Blockchain Workbench с SQL Server Management Studio

Microsoft SQL Server Management Studio позволяет быстро записывать и тестировать запросы к базе данных SQL Azure Blockchain Workbench. В этом разделе содержится пошаговое руководство по подключению к базе данных SQL Azure Блокчейн Workbench из SQL Server Management Studio.

## <a name="prerequisites"></a>Предварительные условия

* Скачайте [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017).

## <a name="connecting-sql-server-management-studio-to-data-in-azure-blockchain-workbench"></a>Подключение SQL Server Management Studio к данным в Azure Blockchain Workbench

1. Откройте SQL Server Management Studio и выберите **Подключение**.
2. Выберите **Ядро СУБД**.

    ![Ядро СУБД](./media/data-sql-management-studio/database-engine.png)

3. В диалоговом окне **Подключение к серверу** введите имя сервера и учетные данные базы данных.

    Если вы используете учетные данные, созданные в процессе развертывания Azure Blockchain Workbench, именем пользователя будет **dbadmin**, а в качестве пароля будет использован пароль, указанный во время развертывания.

    ![Введение учетных данных SQL](./media/data-sql-management-studio/sql-creds.png)

   1. В SQL Server Management Studio показан список баз данных, представлений баз данных и хранимые процедуры в базе данных Azure Blockchain Workbench.

      ![Список баз данных](./media/data-sql-management-studio/db-list.png)

5. Чтобы просмотреть данные, связанные с любым из представлений базы данных, можно автоматически создать инструкцию select, выполнив следующие действия.
6. Щелкните правой кнопкой мыши любое из представлений базы данных в обозревателе объектов.
7. Выберите **Script View as** (Представление скрипта как).
8. Выберите **SELECT to** (ВЫБРАТЬ для).
9. Выберите **New Query Editor Window** (Окно редактирования нового запроса).
10. Запрос можно создать, выбрав **Новый запрос**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Database views in Azure Blockchain Workbench](database-views.md) (Представления базы данных в Azure Blockchain Workbench)
