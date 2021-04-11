---
title: Обзор SQL Server в Виртуальных машинах Azure для Linux | Документация Майкрософт
description: Сведения о том, как запускать полные выпуски SQL Server в Виртуальных машинах Azure для Linux. Прямые ссылки на все образы виртуальных машин SQL Server на базе Linux и соответствующие статьи.
services: virtual-machines-sql
documentationcenter: ''
author: MashaMSFT
tags: azure-service-management
ms.service: virtual-machines-sql
ms.topic: overview
ms.workload: iaas-sql-server
ms.date: 04/10/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 3613b8417c78d0a01582defb7396cc65521e2f4c
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106449488"
---
# <a name="overview-of-sql-server-on-azure-virtual-machines-linux"></a>Обзор SQL Server на виртуальных машинах Azure (Linux)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](../windows/sql-server-on-azure-vm-iaas-what-is-overview.md)
> * [Linux](sql-server-on-linux-vm-what-is-iaas-overview.md)

SQL Server в Виртуальных машинах Azure позволяет использовать полные версии SQL Server в облаке без необходимости управлять каким-либо локальным оборудованием. Кроме того, оплата по мере использования упрощает лицензирование для виртуальных машин SQL Server.

Виртуальные машины Azure работают во многих [географических регионах](https://azure.microsoft.com/regions/) по всему миру. На выбор предоставляется множество [размеров машин](../../../virtual-machines/sizes.md). Коллекция образов позволяет создать виртуальную машину SQL Server нужной версии и выпуска, а также с требуемой операционной системой. Благодаря этому виртуальные машины — отличный вариант для различных рабочих нагрузок SQL Server. 

Если вы не работали с Azure SQL, ознакомьтесь с *обзором SQL Server в Виртуальных машинах Azure* в [серии видео, посвященных SQL Azure](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner):
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/SQL-Server-on-Azure-VM-Overview-4-of-61/player]

## <a name="get-started-with-sql-server-vms"></a><a id="create"></a> Начало работы с виртуальными машинами SQL Server

Чтобы приступить к работе, выберите образ виртуальной машины SQL Server требуемой версии, выпуска и с нужной операционной системой. В следующих разделах приведены прямые ссылки на портал Azure для получения образов виртуальной машины SQL Server из коллекции.

> [!TIP]
> Сведения о ценах на образы SQL Server см. на странице [Цены на виртуальные машины Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

| Версия | Операционная система | Выпуск |
| --- | --- | --- |
| **SQL Server 2019** | Ubuntu 18.04 | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804enterprise-ARM), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804standard-ARM), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804web-ARM), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804sqldev-ARM) | 
| **SQL Server 2019** | Red Hat Enterprise Linux (RHEL) 8 | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8enterprise-ARM), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8standard-ARM), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8web-ARM), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8sqldev-ARM)|
| **SQL Server 2019** | SUSE Linux Enterprise Server (SLES) версия 12 с пакетом обновления 5 (SP5) | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5enterprise-ARM), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5standard-ARM), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5web-ARM), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5sqldev-ARM)|
| **SQL Server 2017** | Red Hat Enterprise Linux (RHEL) 7.4 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonRedHatEnterpriseLinux74), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonRedHatEnterpriseLinux74), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonRedHatEnterpriseLinux74), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonRedHatEnterpriseLinux74), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonRedHatEnterpriseLinux74) |
| **SQL Server 2017** | SUSE Linux Enterprise Server (SLES) версии 12 SP2 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonSLES12SP2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonSLES12SP2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonSLES12SP2), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonSLES12SP2), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonSLES12SP2) |
| **SQL Server 2017** | Ubuntu 16.04 LTS |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonUbuntuServer1604LTS), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonUbuntuServer1604LTS), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonUbuntuServer1604LTS), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonUbuntuServer1604LTS), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonUbuntuServer1604LTS) |

> [!NOTE]
> Сведения о доступных образах виртуальных машин SQL Server в Windows см. в статье [Обзор SQL Server в Виртуальных машинах Azure (Windows)](../windows/sql-server-on-azure-vm-iaas-what-is-overview.md).

## <a name="installed-packages"></a><a id="packages"></a> Установленные пакеты

При настройке SQL Server в Linux установите пакет ядра СУБД, а затем некоторые дополнительные пакеты с учетом ваших требований. Образы виртуальных машин Linux для SQL Server автоматически устанавливают большинство пакетов. В следующей таблице показаны установленные пакеты для каждого дистрибутива.

| Distribution | [Ядро СУБД](/sql/linux/sql-server-linux-setup) | [Инструменты](/sql/linux/sql-server-linux-setup-tools) | [Агент SQL Server](/sql/linux/sql-server-linux-setup-sql-agent) | [полнотекстовый поиск.](/sql/linux/sql-server-linux-setup-full-text-search) | [MSSQL Integration Services](/sql/linux/sql-server-linux-setup-ssis) | [Надстройка для обеспечения высокого уровня доступности](/sql/linux/sql-server-linux-business-continuity-dr) |
|---|---|---|---|---|---|---|
| RHEL | ![RHEL и ядро СУБД](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL и средства](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL и агент SQL Server](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL и полнотекстовый поиск](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL и SSIS](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL и надстройка HA](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) |
| SLES | ![SLES и ядро СУБД](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES и средства](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES и агент SQL Server](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES и полнотекстовый поиск](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES и SSIS](./media/sql-server-on-linux-vm-what-is-iaas-overview/no.png) | ![SLES и надстройка HA](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png)|
| Ubuntu | ![Ubuntu и ядро СУБД](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu и средства](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu и агент SQL Server](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu и полнотекстовый поиск](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu и SSIS](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu и надстройка HA](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) |

## <a name="related-products-and-services"></a>Связанные продукты и услуги

### <a name="linux-virtual-machines"></a>Виртуальные машины Linux

* [Обзор Виртуальных машин Azure](../../../virtual-machines/linux/overview.md)

### <a name="storage"></a>Память

* [Введение в службу хранилища Microsoft Azure](../../../storage/common/storage-introduction.md)

### <a name="networking"></a>Сеть

* [Обзор виртуальной сети](../../../virtual-network/virtual-networks-overview.md)
* [IP-адреса в Azure](../../../virtual-network/public-ip-addresses.md)
* [Создание полного доменного имени на портале Azure](../../../virtual-machines/create-fqdn.md)

### <a name="sql"></a>SQL

* [Документация по SQL Server в Linux](/sql/linux).
* [Сравнение со службой "База данных SQL"](../../azure-sql-iaas-vs-paas-what-is-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

Начало работы с SQL Server в Виртуальных машинах для Linux:

* [Создание виртуальной машины SQL Server на портале Azure](sql-vm-create-portal-quickstart.md).

Ответы на часто задаваемые вопросы о виртуальных машинах SQL Server для Linux:

* [Часто задаваемые вопросы об SQL Server на виртуальных машинах Azure](frequently-asked-questions-faq.md)