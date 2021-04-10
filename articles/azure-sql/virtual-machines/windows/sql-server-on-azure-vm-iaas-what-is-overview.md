---
title: Обзор SQL Server на виртуальных машинах Windows в Azure | Документация Майкрософт
description: Сведения о том, как запускать полные выпуски SQL Server в Виртуальных машинах Azure в облаке без необходимости управлять локальным оборудованием.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
tags: azure-service-management
ms.assetid: c505089e-6bbf-4d14-af0e-dd39a1872767
ms.service: virtual-machines-sql
ms.topic: overview
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/27/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: b4b1234b88eafedbd03b78241baaa40231a6cbd6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97356809"
---
# <a name="what-is-sql-server-on-azure-virtual-machines-windows"></a>Что такое SQL Server в Виртуальных машинах Azure (Windows)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
> * [Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)

[SQL Server в Виртуальных машинах Azure](https://azure.microsoft.com/services/virtual-machines/sql-server/) позволяет использовать полные версии SQL Server в облаке без необходимости управлять каким-либо локальным оборудованием. Применение виртуальных машин для SQL Server также оптимизирует затраты на лицензирование при оплате по мере использования.

Виртуальные машины Azure работают во многих [географических регионах](https://azure.microsoft.com/regions/) по всему миру. На выбор предоставляется множество [размеров машин](../../../virtual-machines/sizes.md). Коллекция образов позволяет создать виртуальную машину SQL Server нужной версии и выпуска, а также с требуемой операционной системой. Благодаря этому виртуальные машины — это отличный вариант для различных рабочих нагрузок SQL Server.

Если вы не работали с SQL Server в Виртуальных машинах Azure, ознакомьтесь с *обзором SQL Server в Виртуальных машинах Azure* в [серии видео, посвященных SQL Azure](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner):
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/SQL-Server-on-Azure-VM-Overview-4-of-61/player]

## <a name="automated-updates"></a>Автоматические обновления

SQL Server в Виртуальных машинах Azure поддерживает [автоматизированное исправление](automated-patching.md) — функцию, которая позволяет планировать график автоматической установки важных обновлений для Windows и SQL Server.

## <a name="automated-backups"></a>Автоматическое резервное копирование

SQL Server в Виртуальных машинах Azure поддерживает [автоматическое резервное копирование](automated-backup.md), которое регулярно создает резервные копии базы данных в Хранилище BLOB-объектов. Это также можно выполнять вручную. Дополнительные сведения см. в статье [Использование службы хранилища Azure для архивации и восстановления SQL Server](azure-storage-sql-server-backup-restore-use.md).

Azure также предоставляет возможности резервного копирования корпоративного класса для SQL Server на виртуальных машинах Azure. Это полностью управляемое решение резервного копирования поддерживает группы доступности Always On, долгосрочное хранение, восстановление на определенный момент времени, а также централизованное управление и мониторинг. Подробные сведения см. в статье [Сведения о резервном копировании SQL Server в Виртуальных машинах Azure](../../../backup/backup-azure-sql-database.md).
  

## <a name="high-availability"></a>Высокий уровень доступности

Если требуется высокий уровень доступности, вам помогут группы доступности SQL Server. Они подразумевают использование нескольких экземпляров SQL Server в Виртуальных машинах Azure в виртуальной сети. Вы можете настроить решение с высоким уровнем доступности вручную или использовать для автоматической настройки шаблоны на портале Azure. Полный обзор средств для обеспечения высокого уровня доступности см. в статье [Обеспечение непрерывности бизнес-процессов и HADR для SQL Server в Виртуальных машинах Azure](business-continuity-high-availability-disaster-recovery-hadr-overview.md).

## <a name="performance"></a>Производительность

Виртуальные машины Azure предлагают другие размеры компьютера в соответствии с различными требованиями рабочих нагрузок. Виртуальные машины SQL Server также предусматривают автоматическую настройку хранилища, оптимизированную для требований к производительности. Сведения о настройке хранилища см. в статье [Настройка хранилища для виртуальных машин SQL Server](storage-configuration.md). Сведения о настройке производительности с в статье [Рекомендации по оптимизации производительности SQL Server в Виртуальных машинах Azure](performance-guidelines-best-practices.md).

## <a name="get-started-with-sql-server-vms"></a>Начало работы с виртуальными машинами SQL Server

Чтобы приступить к работе, выберите образ виртуальной машины SQL Server требуемой версии, выпуска и с нужной операционной системой. В следующих разделах приведены прямые ссылки на портал Azure для получения образов виртуальной машины SQL Server из коллекции.

> [!TIP]
> Сведения о ценах на образы SQL Server см. в статье [Руководство по ценам на SQL Server в Виртуальных машинах Azure](pricing-guidance.md). 

### <a name="pay-as-you-go"></a><a id="payasyougo"></a> Оплата по мере использования
В следующей таблице представлена матрица для образов SQL Server с оплатой по мере использования.

| Версия | Операционная система | Выпуск |
| --- | --- | --- |
| **SQL Server 2019** | Windows Server 2019 | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019enterprise), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019standard), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019web), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019sqldev) | 
| **SQL Server 2017** |Windows Server 2016 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseWindowsServer2016), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonWindowsServer2016), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonWindowsServer2016), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonWindowsServer2016), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonWindowsServer2016) |
| **SQL Server 2016 SP2** |Windows Server 2016 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2016SP2EnterpriseWindowsServer2016), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2016SP2StandardWindowsServer2016), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2016SP2WebWindowsServer2016), [Express](https://portal.azure.com/#create/Microsoft.FreeLicenseSQLServer2016SP2ExpressWindowsServer2016), [Developer](https://portal.azure.com/#create/Microsoft.FreeLicenseSQLServer2016SP2DeveloperWindowsServer2016) |
| **SQL Server 2014 SP2** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2StandardWindowsServer2012R2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2WebWindowsServer2012R2), [Express](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2ExpressWindowsServer2012R2) |
| **SQL Server 2012 с пакетом обновления 4 (SP4)** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4StandardWindowsServer2012R2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4WebWindowsServer2012R2), [Express](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4ExpressWindowsServer2012R2) |
| **SQL Server 2008 R2 SP3** |Windows Server 2008 R2|[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3EnterpriseWindowsServer2008R2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3StandardWindowsServer2008R2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3WebWindowsServer2008R2), [Express](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3ExpressWindowsServer2008R2) |

Сведения о доступных образах виртуальных машин SQL Server в Linux см. в статье [Обзор SQL Server в Виртуальных машинах Azure (Linux)](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md).

> [!NOTE]
> Теперь вы можете изменить модель лицензирования с оплатой за использование для виртуальной машины SQL Server, чтобы использовать собственную лицензию. Дополнительные сведения см. в разделе [Изменение модели лицензирования для виртуальной машины SQL Server в Azure](licensing-model-azure-hybrid-benefit-ahb-change.md). 

### <a name="bring-your-own-license"></a><a id="BYOL"></a> Использование собственной лицензии
Вы также можете использовать собственную лицензию (BYOL). В этом сценарии вы платите только за виртуальную машину и не несете расходы на лицензирование SQL Server.  Использование технологии BYOL со временем позволяет снизить расходы на использование непрерывных производственных рабочих нагрузок. Требования для использования этой возможности см. в [руководстве по выбору ценовой категории для виртуальных машин Azure SQL Server](pricing-guidance.md#byol).

Чтобы использовать собственную лицензию, вы можете сменить для виртуальной машины SQL Server существующую модель лицензирования с оплатой по мере использования или развернуть образ с префиксом **{BYOL}** . Сведения о переходе с модели лицензирования с оплатой по мере использования на BYOL см. в статье [Изменение модели лицензирования для виртуальной машины SQL Server в Azure](licensing-model-azure-hybrid-benefit-ahb-change.md). 

| Версия | Операционная система | Выпуск |
| --- | --- | --- |
| **SQL Server 2019** | Windows Server 2019 | [Enterprise BYOL](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019-byolenterprise), [Standard BYOL](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019-byolstandard)| 
| **SQL Server 2017** |Windows Server 2016 |[Enterprise BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017EnterpriseWindowsServer2016), [Standard BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017StandardonWindowsServer2016) |
| **SQL Server 2016 SP2** |Windows Server 2016 |[Enterprise BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2016SP2EnterpriseWindowsServer2016), [Standard BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2016SP2StandardWindowsServer2016) |
| **SQL Server 2014 SP2** |Windows Server 2012 R2 |[Enterprise BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2014SP2EnterpriseWindowsServer2012R2), [Standard BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2014SP2StandardWindowsServer2012R2) |
| **SQL Server 2012 с пакетом обновления 4 (SP4)** |Windows Server 2012 R2 |[Enterprise BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2012SP4EnterpriseWindowsServer2012R2), [Standard BYOL](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2012SP4StandardWindowsServer2012R2) |

С помощью PowerShell вы можете развернуть старую версию образа SQL Server, недоступного на портале Azure. Чтобы просмотреть все доступные образы с помощью PowerShell, выполните следующую команду:

  ```powershell
  Get-AzVMImageOffer -Location $Location -Publisher 'MicrosoftSQLServer'
  ```

Дополнительные сведения о том, как развернуть виртуальные машины SQL Server с помощью PowerShell, см. в статье [Как подготовить виртуальные машины SQL Server с помощью Azure PowerShell](create-sql-vm-powershell.md).


### <a name="connect-to-the-vm"></a>Подключение к виртуальной машине
После создания виртуальной машины SQL Server установите подключение к ней из приложений или средств, таких как SQL Server Management Studio (SSMS). Инструкции см. в статье [Подключение к виртуальной машине SQL Server в Azure](ways-to-connect-to-sql.md).

### <a name="migrate-your-data"></a>Перенос данных
Если у вас есть база данных, вам нужно будет переместить ее в новую виртуальную машину SQL Server. Список вариантов и руководство по миграции баз данных см. в статье [Миграция базы данных SQL Server в экземпляр SQL Server на виртуальной машине Azure](migrate-to-vm-from-sql-server.md).

## <a name="create-and-manage-azure-sql-resources-with-the-azure-portal"></a>Создание ресурсов SQL Azure и управление ими с помощью портала Azure

Портал Azure предоставляет отдельную страницу, на которой вы можете управлять [всеми ресурсами SQL Azure](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Sql%2Fazuresql), а также виртуальными машинами SQL.

Чтобы открыть страницу **ресурсов SQL Azure**, щелкните **Azure SQL** в меню портала Azure или выполните поиск и выберите **Azure SQL** на любой странице.

![Поиск Azure SQL](./media/sql-server-on-azure-vm-iaas-what-is-overview/search-for-azure-sql.png)

> [!NOTE]
> Azure SQL обеспечивает быстрый и простой способ доступа ко всем базам данных SQL Azure, эластичным пулам, логическим серверам, управляемым экземплярам и виртуальным машинам. Azure SQ не является службой или ресурсом. 

Чтобы управлять существующими ресурсами, выберите нужный элемент в списке. Чтобы создать ресурсы SQL Azure, выберите **+ Добавить**. 

![Создание ресурса SQL Azure](./media/sql-server-on-azure-vm-iaas-what-is-overview/create-azure-sql-resource.png)

После выбора параметра **+ Добавить**, просмотрите дополнительные сведения о различных параметрах, щелкнув **Показать сведения** для любой плитки.

![сведения о плитке баз данных](./media/sql-server-on-azure-vm-iaas-what-is-overview/sql-vm-details.png)

Подробная информация доступна в следующих статьях:

- [Создание отдельной базы данных](../../database/single-database-create-quickstart.md)
- [Создание эластичного пула](../../database/elastic-pool-overview.md#creating-a-new-sql-database-elastic-pool-using-the-azure-portal)
- [Создание управляемого экземпляра](../../managed-instance/instance-create-quickstart.md)
- [Создание виртуальной машины SQL Server](sql-vm-create-portal-quickstart.md)

## <a name="sql-server-vm-image-refresh-policy"></a><a id="lifecycle"></a> Политика обновления образа виртуальной машины SQL Server
Azure поддерживает только один образ виртуальной машины для каждого сочетания поддерживаемой операционной системы, версии и выпуска. Это означает, что со временем образы обновляются, а старые образы удаляются. Дополнительные сведения см. в разделе **Образы** статьи [Часто задаваемые вопросы по SQL Server на виртуальных машинах Azure (Windows)](frequently-asked-questions-faq.md#images).

## <a name="customer-experience-improvement-program-ceip"></a>Программа улучшения качества программного обеспечения (CEIP)
Программа по улучшению качества программного обеспечения (CEIP) включена по умолчанию. Она периодически отправляет отчеты в Майкрософт, чтобы улучшить качество SQL Server. Если вы не хотите отключить CEIP после подготовки, то дополнительные задания управления не требуются. Вы можете настроить или отключить программу CEIP, подключившись к виртуальной машине с помощью удаленного рабочего стола. Затем запустите служебную программу **Служба отчетов об ошибках и использовании SQL Server** . Следуйте инструкциям в окнах этой программы, чтобы отключить создание отчетов. Дополнительные сведения о сборе данных см. в [заявлении о конфиденциальности SQL Server](/sql/sql-server/sql-server-privacy).

## <a name="related-products-and-services"></a>Связанные продукты и услуги
### <a name="windows-virtual-machines"></a>Виртуальные машины Windows
* [Обзор Виртуальных машин Azure](../../../virtual-machines/windows/overview.md)

### <a name="storage"></a>Память
* [Введение в службу хранилища Microsoft Azure](../../../storage/common/storage-introduction.md)

### <a name="networking"></a>Сеть
* [Обзор виртуальной сети](../../../virtual-network/virtual-networks-overview.md)
* [IP-адреса в Azure](../../../virtual-network/public-ip-addresses.md)
* [Создание полного доменного имени на портале Azure](../../../virtual-machines/create-fqdn.md)

### <a name="sql"></a>SQL
* [Документация по SQL Server](/sql/index)
* [Сравнение со службой "База данных SQL"](../../azure-sql-iaas-vs-paas-what-is-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

Начало работы с SQL Server на виртуальных машинах Azure:

* [Создание виртуальной машины SQL Server на портале Azure](sql-vm-create-portal-quickstart.md).

Ответы на часто задаваемые вопросы о виртуальных машинах SQL Server:

* [Часто задаваемые вопросы об SQL Server на виртуальных машинах Azure](frequently-asked-questions-faq.md)

Просмотрите эталонные образцы архитектуры для выполняющихся N-уровневых приложений на SQL Server в IaaS

* [Использование n-уровневого приложения с SQL Server в Azure](/azure/architecture/reference-architectures/n-tier/n-tier-sql-server)
* [Запуск n-уровневого приложения в нескольких регионах Azure для обеспечения высокой доступности](/azure/architecture/reference-architectures/n-tier/multi-region-sql-server)