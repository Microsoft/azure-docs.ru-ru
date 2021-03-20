---
title: Автоматизированная установка исправлений для виртуальных машин SQL Server (Resource Manager) | Документация Майкрософт
description: Эта статья содержит сведения о функции автоматической установки исправлений для виртуальных машин SQL Server в Azure для модели Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: 58232e92-318f-456b-8f0a-2201a541e08d
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/07/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 429fe39f84a54c22fa97178b85f417d76dc84a8e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359478"
---
# <a name="automated-patching-for-sql-server-on-azure-virtual-machines-resource-manager"></a>Автоматическая установка исправлений SQL Server на виртуальных машинах Azure (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

При автоматической установке исправлений на виртуальных машинах Azure с SQL Server задается период обслуживания. Установка автоматических обновлений возможна только в этот период обслуживания. Для SQL Server это ограничение гарантирует, что системные обновления и связанные с ними перезапуски системы будут происходить в наиболее удобное для базы данных время. 

> [!IMPORTANT]
> Устанавливаются только те обновления ОС Windows и SQL Server, которые помечены как **Важные** или **Критические**. Другие обновления SQL Server, такие как пакеты обновления и накопительные обновления, не помеченные как **Важные** или **Критические**, необходимо установить вручную. 

Автоматическая установка исправлений зависит от [расширения агента IaaS для SQL Server](sql-server-iaas-agent-extension-automate-management.md).

## <a name="prerequisites"></a>Предварительные требования
Для использования автоматической установки исправлений необходимо выполнить следующие предварительные требования.

**Операционная система**.

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016
* Windows Server 2019

**Версия SQL Server**

* SQL Server 2008 R2
* SQL Server 2012
* SQL Server 2014
* SQL Server 2016
* SQL Server 2017
* SQL Server 2019

**Azure PowerShell**

* [Установите последние команды Azure PowerShell](/powershell/azure/) , если планируете настраивать автоматические исправления с помощью PowerShell.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

> [!NOTE]
> Автоматическая установка исправлений зависит от расширения агента IaaS для SQL Server. В текущей коллекции образов виртуальных машин SQL это расширение присутствует по умолчанию. Дополнительные сведения см. в статье [SQL Server IaaS Agent Extension](sql-server-iaas-agent-extension-automate-management.md) (Расширение агента SQL Server IaaS).
> 
> 

## <a name="settings"></a>Настройки
В приведенной ниже таблице описаны параметры для настройки автоматической установки исправлений. Фактическая процедура настройки может варьироваться в зависимости от того, используете вы портал Azure или команды Azure Windows PowerShell.

| Параметр | Возможные значения | Описание |
| --- | --- | --- |
| **Автоматическое исправление** |Включено/отключено (отключено) |Включает или отключает автоматическую установку исправлений для виртуальной машины Azure. |
| **Расписание обслуживания** |Каждый день, понедельник, вторник, среда, четверг, пятница, суббота, воскресенье |Расписание для скачивания и установки обновлений Windows, SQL Server и обновлений Майкрософт для виртуальной машины. |
| **Время начала обслуживания** |0–24 |Локальное время начала обновления виртуальной машины. |
| **Длительность периода обслуживания** |30–180 |Допустимое количество минут для скачивания и установки обновлений. |
| **Категория исправления** |Важно! | Категория обновлений ОС Windows, которые будут скачаны и установлены.|

## <a name="configure-in-the-azure-portal"></a>Настройка на портале Azure
Для настройки автоматизированной установки исправлений во время подготовки виртуальных машин или для существующих виртуальных машин можно использовать портал Azure.

### <a name="new-vms"></a>Новые виртуальные машины
При создании новой виртуальной машины SQL Server с моделью развертывания с помощью Resource Manager настройте автоматизированную установку исправлений, используя портал Azure.

На вкладке **Настройки SQL Server** в разделе **Автоматизированное исправление** выберите **Изменить конфигурацию**. Колонка **Автоматическая установка исправлений SQL** показана на следующем снимке экрана портала Azure.

![Автоматизированная установка исправлений SQL на портале Azure](./media/automated-patching/azure-sql-arm-patching.png)

Дополнительные сведения см. в статье о [подготовке виртуальной машины SQL Server на портале Azure](create-sql-vm-portal.md).

### <a name="existing-vms"></a>Существующие виртуальные машины

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Для существующих виртуальных машин SQL Server перейдите к ресурсу [Виртуальные машины SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) и в разделе **Настройки** выберите **Исправление**. 

![Автоматизированная установка исправлений SQL для существующих виртуальных машин](./media/automated-patching/azure-sql-rm-patching-existing-vms.png)


По завершении нажмите кнопку **OK** в нижней части колонки **Конфигурация SQL Server**, чтобы сохранить изменения.

Если автоматизированная установка исправлений включается впервые, Azure настроит агент IaaS SQL Server в фоновом режиме. В течение этого времени портал Azure может не отображать информацию о том, что выполняется настройка автоматической установки исправлений. Установка и настройка агента занимают несколько минут. После этого новые параметры отобразятся на портале Azure.

## <a name="configure-with-powershell"></a>Настройка с помощью PowerShell
После подготовки виртуальной машины SQL используйте PowerShell для настройки автоматической установки исправлений.

В следующем примере для настройки автоматической установки исправлений на существующей виртуальной машине SQL Server используется PowerShell. Команда **New-AzVMSqlServerAutoPatchingConfig** настраивает новый период обслуживания для автоматической установки обновлений.

```azurepowershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"
$aps = New-AzVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"
s
Set-AzVMSqlServerExtension -AutoPatchingSettings $aps -VMName $vmname -ResourceGroupName $resourcegroupname
```

> [!IMPORTANT]
> Если расширение не установлено, при его установке перезапускается SQL Server.

В представленной ниже таблице показано фактическое воздействие на конечную виртуальную машину Azure на основе данного примера.

| Параметр | Действие |
| --- | --- |
| **DayOfWeek** |Исправления устанавливаются каждый четверг. |
| **MaintenanceWindowStartingHour** |Установка обновлений начинается в 11:00. |
| **MaintenanceWindowsDuration** |Обновления должны быть установлены в течение 120 минут. С учетом времени начала установка обновлений должна завершаться к 13:00. |
| **PatchCategory** |Единственное возможное значение для этого параметра — **Important**. При этом устанавливаются обновления ОС Windows с пометкой "Важно". Обновления SQL Server, не помеченные как важные, не устанавливаются. |

Установка и настройка агента SQL Server IaaS занимают несколько минут.

Чтобы отключить автоматическую установку обновлений, выполните тот же скрипт без параметра **-Enable** в команде **New-AzVMSqlServerAutoPatchingConfig**. Отсутствие параметра **-Enable** означает, что функцию нужно отключить.

## <a name="next-steps"></a>Дальнейшие действия
Сведения о других доступных задачах автоматизации см. в разделе [Расширение агента IaaS для SQL Server](sql-server-iaas-agent-extension-automate-management.md).

Дополнительные сведения о запуске SQL Server на виртуальных машинах Azure см. в [обзоре использования SQL Server на виртуальных машинах Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).

