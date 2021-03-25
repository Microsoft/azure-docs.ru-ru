---
title: Синхронизация данных из SQL Azure для пограничных вычислений с помощью Синхронизации данных SQL
description: Сведения о синхронизации данных из SQL Azure для пограничных вычислений с помощью Синхронизации данных SQL Azure
keywords: SQL Azure для пограничных вычислений, синхронизация данных из SQL Azure для пограничных вычислений, синхронизация данных SQL Azure для пограничных вычислений
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 5659ce5fa6f99463f58a33662563d768248fd8cb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93394906"
---
# <a name="tutorial-sync-data-from-sql-edge-to-azure-sql-database-by-using-sql-data-sync"></a>Руководство по синхронизации данных из SQL Azure для пограничных вычислений с Базой данных SQL Azure с помощью Синхронизации данных SQL

В этом учебнике вы узнаете, как с помощью *группы синхронизации* технологии "Синхронизация данных SQL Azure" выполнить добавочную синхронизацию данных из SQL Azure для пограничных вычислений с Базой данных SQL Azure. Синхронизация данных SQL — это служба, созданная на основе Базы данных SQL Azure, которая позволяет в двунаправленном режиме синхронизировать выбранные данные в нескольких базах данных SQL Azure и экземплярах SQL Server. Дополнительные сведения о Синхронизации данных SQL см. в [этой статье](../azure-sql/database/sql-data-sync-data-sql-server-sql-database.md).

Так как служба "SQL Azure для пограничных вычислений" основана на последней версии [ядра СУБД SQL Server](/sql/sql-server/sql-server-technical-documentation/), любой механизм синхронизации данных, применимый к экземпляру SQL Server, также можно использовать для синхронизации данных с экземпляром SQL для пограничных вычислений, выполняющимся на пограничном устройстве, или из него.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником требуется компьютер с ОС Windows, на котором настроен [Data Sync Agent для Синхронизации данных SQL Azure](../azure-sql/database/sql-data-sync-agent-overview.md).

## <a name="before-you-begin"></a>Перед началом

* Создайте базу данных в Базе данных SQL Azure. См. статью [Краткое руководство. Создание отдельной базы данных в Базе данных SQL Azure](../azure-sql/database/single-database-create-quickstart.md?tabs=azure-portal).

* Создайте таблицы и другие необходимые объекты в экземпляре Базы данных SQL Azure.

* Создайте необходимые таблицы и объекты в развертывании SQL Azure для пограничных вычислений. Дополнительные сведения см. в статье [Использование пакетов DAC Базы данных SQL с SQL Azure для пограничных вычислений](deploy-dacpac.md).

* Зарегистрируйте экземпляр SQL Azure для пограничных вычислений с помощью Data Sync Agent для Синхронизации данных SQL Azure. Дополнительные сведения см. в статье [Добавление базы данных SQL Server](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-on-prem).

## <a name="sync-data-between-a-database-in-azure-sql-database-and-sql-edge"></a>Синхронизация данных между базой данных в Базе данных SQL Azure и SQL для пограничных вычислений

Настройка синхронизации между базой данных в Базе данных SQL Azure и экземпляром SQL для пограничных вычислений с помощью Синхронизации данных SQL производится в три основных шага.  


1. Создайте группу синхронизации на портале Azure. Дополнительные сведения см. в разделе [о создании группы синхронизации](../azure-sql/database/sql-data-sync-sql-server-configure.md#create-sync-group). Вы можете использовать одну *центральную* базу данных для создания нескольких групп синхронизации, чтобы синхронизировать данные из разных экземпляров SQL Azure для пограничных вычислений с одной или несколькими базами данных в SQL Azure. 

2. Добавьте члены в группу синхронизации. Дополнительные сведения см. в разделе [Добавление членов синхронизации](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-sync-members).

3. Настройте группу синхронизации, выбрав таблицы, которые будут участвовать в синхронизации. Дополнительные сведения см. в разделе [Настройка группы синхронизации](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-sync-members).

После выполнения описанных выше действий вы получите группу синхронизации, которая включает в себя базу данных в Базе данных SQL Azure и экземпляр SQL для пограничных вычислений.

Дополнительные сведения о Синхронизации данных SQL см. в этих статьях:

* Сведения об [агенте синхронизации данных для синхронизации данных SQL Azure](../azure-sql/database/sql-data-sync-agent-overview.md).

* [Рекомендации по синхронизации данных SQL](../azure-sql/database/sql-data-sync-best-practices.md) и инструкции по [устранению неполадок с синхронизацией данных SQL](../azure-sql/database/sql-data-sync-troubleshoot.md).

* [Мониторинг синхронизации данных SQL с помощью журналов Azure Monitor](../azure-sql/database/monitor-tune-overview.md)

* Сведения об обновлении схемы синхронизации с помощью [Transact-SQL](../azure-sql/database/sql-data-sync-update-sync-schema.md) или [PowerShell](../azure-sql/database/scripts/update-sync-schema-in-sync-group.md).

## <a name="next-steps"></a>Дальнейшие действия


* [Использование PowerShell для синхронизации данных между Базой данных SQL и SQL Azure для пограничных вычислений](../azure-sql/database/scripts/sql-data-sync-sync-data-between-azure-onprem.md). Замените в этом учебнике сведения о базе данных `OnPremiseServer` сведениями о SQL Azure для пограничных вычислений.