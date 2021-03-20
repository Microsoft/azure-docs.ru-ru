---
title: Управление виртуальными машинами SQL Server в Azure с помощью портал Azure | Документация Майкрософт
description: Узнайте, как получить доступ к ресурсу виртуальной машины SQL Server, размещенной в Azure, на портале Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/13/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 68c7805136a7361a64a6ff6dfbb9e7d910b2ea9b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97357948"
---
# <a name="manage-sql-server-vms-in-azure-by-using-the-azure-portal"></a>Управление виртуальными машинами SQL Server в Azure с помощью портала Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В [портал Azure](https://portal.azure.com)ресурс [**виртуальных машин SQL**](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) — это независимая служба управления для управления SQL Server на виртуальных машинах Azure. В ней можно одновременно просматривать все виртуальные машины SQL Server и изменять параметры для SQL Server: 

![Ресурс "Виртуальные машины SQL"](./media/manage-sql-vm-portal/sql-vm-manage.png)


## <a name="remarks"></a>Remarks

- Мы рекомендуем использовать ресурс " [**виртуальные машины SQL**](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) " для просмотра SQL Server виртуальных машин в Azure и управления ими. Однако в настоящее время ресурс **Виртуальные машины SQL** не применяется для управления виртуальными машинами SQL Server, [поддержка которых прекращена](sql-server-2008-extend-end-of-support.md). Настроить параметры таких виртуальных машин можно на устаревшей вкладке [Конфигурация SQL Server](#access-the-sql-server-configuration-tab). 
- Ресурс " **виртуальные машины SQL** " доступен только для SQL Server виртуальных машин, [зарегистрированных в расширении агента IaaS SQL](sql-agent-extension-manually-register-single-vm.md). 


## <a name="access-the-sql-virtual-machines-resource"></a>Доступ к ресурсу "Виртуальные машины SQL"
Вот как можно открыть ресурс **Виртуальные машины SQL**:

1. Откройте [портал Azure](https://portal.azure.com). 
1. Выбор пункта **Все службы**. 
1. В поле поиска введите **Виртуальные машины SQL**.
1. (Необязательно.) Щелкните звезду рядом с названием **Виртуальные машины SQL**, чтобы добавить этот ресурс в меню **Избранное**. 
1. Выберите **Виртуальные машины SQL**. 

   ![Поиск виртуальных машин SQL Server среди всех служб](./media/manage-sql-vm-portal/sql-vm-search.png)

1. На портале перечислены все виртуальные машины SQL Server, доступные в рамках подписки. Выберите нужную виртуальную машину, чтобы открыть ресурс **Виртуальные машины SQL**. Если виртуальная машина SQL Server не отображается, воспользуйтесь полем поиска. 

   ![Все доступные виртуальные машины SQL Server](./media/manage-sql-vm-portal/all-sql-vms.png)

   При выборе виртуальной машины SQL Server откроется ресурс **Виртуальные машины SQL**: 


   ![Просмотр ресурса виртуальных машин SQL](./media/manage-sql-vm-portal/sql-vm-resource.png)

> [!TIP]
> Ресурс **Виртуальные машины SQL** предназначен для настройки параметров SQL Server. Чтобы открыть параметры, относящиеся к самой виртуальной машине, а не к SQL Server, выберите ее имя в поле **Виртуальная машина**. 

## <a name="access-the-sql-server-configuration-tab"></a>Доступ к вкладке "Конфигурация SQL Server"
Вкладка **Конфигурация SQL Server** устарела. В настоящее время это единственный способ управления [окончанием поддержки](sql-server-2008-extend-end-of-support.md) SQL Server виртуальных машин и SQL Server виртуальных машин, которые не были [зарегистрированы в расширении агента IaaS SQL](sql-agent-extension-manually-register-single-vm.md).

Чтобы открыть устаревшую вкладку **Конфигурация SQL Server**, перейдите к ресурсу **Виртуальные машины**. Выполните указанные ниже действия.

1. Откройте [портал Azure](https://portal.azure.com). 
1. Выбор пункта **Все службы**. 
1. В поле поиска введите **Виртуальные машины**.
1. (Необязательно.) Щелкните звезду рядом с названием **Виртуальные машины**, чтобы добавить этот ресурс в меню **Избранное**. 
1. Выберите **Виртуальные машины**. 

   ![Поиск виртуальных машин](./media/manage-sql-vm-portal/vm-search.png)

1. На портале отображается список всех виртуальных машин в рамках подписки. Выберите нужную виртуальную машину, чтобы открыть ресурс **Виртуальные машины**. Если виртуальная машина SQL Server не отображается, воспользуйтесь полем поиска. 
1. В области **Параметры** выберите пункт **Конфигурация SQL Server**, чтобы перейти к настройке виртуальной машины SQL Server. 

   ![Конфигурация SQL Server](./media/manage-sql-vm-portal/sql-vm-configuration.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях: 

* [Обзор SQL Server на виртуальных машинах Windows](sql-server-on-azure-vm-iaas-what-is-overview.md).
* [Вопросы и ответы об SQL Server на виртуальных машинах Windows](frequently-asked-questions-faq.md)
* [Руководство по ценам для виртуальных машин SQL Server в Azure](pricing-guidance.md)
* [Заметки о выпуске SQL Server на виртуальных машинах Windows](doc-changes-updates-release-notes.md)


