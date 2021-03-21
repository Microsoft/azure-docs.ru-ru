---
title: Необходимые условия для Azure Database Migration Service
description: Ознакомьтесь с предварительными требованиями для использования службы Azure Database Migration Service для переноса баз данных.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: conceptual
ms.date: 02/25/2020
ms.openlocfilehash: 55d5594229bccb5fcb6a406e671ed104c1e12378
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101094048"
---
# <a name="overview-of-prerequisites-for-using-the-azure-database-migration-service"></a>Предварительные требования для использования службы Azure Database Migration Service

Для обеспечения бесперебойной работы Azure Database Migration Service при миграции базы данных необходимо выполнить несколько предварительных требований. Одни предварительные требования применяются во всех сценариях (с парами исходного и целевого объектов), поддерживаемых службой, другие уникальны и используются в определенных сценариях.

В следующих разделах приводятся предварительные требования, касающиеся использования службы Azure Database Migration Service.

## <a name="prerequisites-common-across-migration-scenarios"></a>Общие предварительные требования для сценариев миграции

Ниже приведены предварительные требования для использования службы Azure Database Migration Service, которые применяются во всех поддерживаемых сценариях миграции.

* Создайте виртуальную сеть Microsoft Azure для Azure Database Migration Service с помощью модели развертывания Azure Resource Manager, которая обеспечивает подключение "сеть — сеть" к локальным исходным серверам с помощью [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Убедитесь, что правила группы безопасности сети для виртуальной сети не блокируют исходящий порт 443 ServiceTag для Служебной шины, службы хранилища и Azure Monitor. См. дополнительные сведения о [фильтрации трафика, предназначенного для виртуальной сети, с помощью групп безопасности сети](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Если перед исходными базами данных развернуто устройство брандмауэра, вам может понадобиться добавить правила брандмауэра, чтобы позволить службе Azure Database Migration Service обращаться к исходным базам данных для выполнения миграции.
* Настройте [брандмауэр Windows для доступа к ядру СУБД](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* При установке SQL Server Express протокол TCP/IP отключен по умолчанию. Включите его, выполнив инструкции в статье [Включение или отключение сетевого протокола сервера](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).

    > [!IMPORTANT]
    > Для создания экземпляра Azure Database Migration Service требуется доступ к параметрам виртуальной сети, которые обычно не находятся в одной группе ресурсов. В результате пользователь, создающий экземпляр DMS, требует разрешения на уровне подписки. Чтобы создать необходимые роли, которые можно назначить при необходимости, выполните следующий скрипт:
    >
    > ```
    >
    > $readerActions = `
    > "Microsoft.Network/networkInterfaces/ipConfigurations/read", `
    > "Microsoft.DataMigration/*/read", `
    > "Microsoft.Resources/subscriptions/resourceGroups/read"
    >
    > $writerActions = `
    > "Microsoft.DataMigration/services/*/write", `
    > "Microsoft.DataMigration/services/*/delete", `
    > "Microsoft.DataMigration/services/*/action", `
    > "Microsoft.Network/virtualNetworks/subnets/join/action", `
    > "Microsoft.Network/virtualNetworks/write", `
    > "Microsoft.Network/virtualNetworks/read", `
    > "Microsoft.Resources/deployments/validate/action", `
    > "Microsoft.Resources/deployments/*/read", `
    > "Microsoft.Resources/deployments/*/write"
    >
    > $writerActions += $readerActions
    >
    > # TODO: replace with actual subscription IDs
    > $subScopes = ,"/subscriptions/00000000-0000-0000-0000-000000000000/","/subscriptions/11111111-1111-1111-1111-111111111111/"
    >
    > function New-DmsReaderRole() {
    > $aRole = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
    > $aRole.Name = "Azure Database Migration Reader"
    > $aRole.Description = "Lets you perform read only actions on DMS service/project/tasks."
    > $aRole.IsCustom = $true
    > $aRole.Actions = $readerActions
    > $aRole.NotActions = @()
    >
    > $aRole.AssignableScopes = $subScopes
    > #Create the role
    > New-AzRoleDefinition -Role $aRole
    > }
    >
    > function New-DmsContributorRole() {
    > $aRole = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
    > $aRole.Name = "Azure Database Migration Contributor"
    > $aRole.Description = "Lets you perform CRUD actions on DMS service/project/tasks."
    > $aRole.IsCustom = $true
    > $aRole.Actions = $writerActions
    > $aRole.NotActions = @()
    >
    >   $aRole.AssignableScopes = $subScopes
    > #Create the role
    > New-AzRoleDefinition -Role $aRole
    > }
    > 
    > function Update-DmsReaderRole() {
    > $aRole = Get-AzRoleDefinition "Azure Database Migration Reader"
    > $aRole.Actions = $readerActions
    > $aRole.NotActions = @()
    > Set-AzRoleDefinition -Role $aRole
    > }
    >
    > function Update-DmsConributorRole() {
    > $aRole = Get-AzRoleDefinition "Azure Database Migration Contributor"
    > $aRole.Actions = $writerActions
    > $aRole.NotActions = @()
    > Set-AzRoleDefinition -Role $aRole
    > }
    >
    > # Invoke above functions
    > New-DmsReaderRole
    > New-DmsContributorRole
    > Update-DmsReaderRole
    > Update-DmsConributorRole
    > ```

## <a name="prerequisites-for-migrating-sql-server-to-azure-sql-database"></a>Предварительные требования для переноса SQL Server в Базу данных SQL Azure

Помимо предварительных требований для службы Azure Database Migration Service, которые являются общими для всех сценариев миграции, существуют также требования, которые применяются только к определенному сценарию.

При использовании службы Azure Database Migration Service для миграции SQL Server в Базу данных SQL Azure, помимо предварительных требований, которые являются общими для всех сценариев миграции, обязательно выполните следующие дополнительные требования:

* Создайте экземпляр базы данных SQL Azure, как описано в статье [Создание базы данных в базе данных SQL Azure на портал Azure](../azure-sql/database/single-database-create-quickstart.md).
* Скачайте и установите [Помощник по миграции данных](https://www.microsoft.com/download/details.aspx?id=53595) версии 3.3 или более поздней.
* Откройте брандмауэр Windows, чтобы предоставить Azure Database Migration Service доступ к исходному серверу SQL Server. По умолчанию это TCP-порт 1433.
* Если вы запустили несколько именованных экземпляров SQL Server, использующих динамические порты, вы можете включить службу обозревателя SQL и разрешить доступ к UDP-порту 1434 через брандмауэры. Это позволит службе Azure Database Migration Service подключиться к именованному экземпляру на исходном сервере.
* Создайте [правило брандмауэра](../azure-sql/database/firewall-configure.md) на уровне сервера для базы данных SQL, чтобы разрешить Azure Database Migration Service доступ к целевым базам данных. Задайте диапазон подсети в виртуальной сети, которая используется для Azure Database Migration Service.
* Убедитесь, что учетные данные, используемые для подключения к исходному экземпляру SQL Server, имеют разрешения [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
* Убедитесь, что учетные данные, используемые для подключения к целевой базе данных, имеют разрешение CONTROL DATABASE на целевую базу данных.

   > [!NOTE]
   > Полный список предварительных требований, необходимых для использования службы Azure Database Migration Service для миграции из SQL Server в Базу данных SQL Azure, см. в руководстве [Миграция с SQL Server в базу данных SQL Azure](./tutorial-sql-server-to-azure-sql.md).
   >

## <a name="prerequisites-for-migrating-sql-server-to-azure-sql-managed-instance"></a>Необходимые условия для миграции SQL Server в Azure SQL Управляемый экземпляр

* Создайте Управляемый экземпляр SQL, следуя сведениям в статье [создание управляемый экземпляр SQL Azure в портал Azure](../azure-sql/managed-instance/instance-create-quickstart.md).
* В брандмауэрах разрешите передачу трафика SMB через порт 445 для IP-адреса или диапазона подсети Azure Database Migration Service.
* Откройте брандмауэр Windows, чтобы предоставить Azure Database Migration Service доступ к исходному серверу SQL Server. По умолчанию это TCP-порт 1433.
* Если вы запустили несколько именованных экземпляров SQL Server, использующих динамические порты, вы можете включить службу обозревателя SQL и разрешить доступ к UDP-порту 1434 через брандмауэры. Это позволит службе Azure Database Migration Service подключиться к именованному экземпляру на исходном сервере.
* Убедитесь, что для подключения к исходному серверу SQL Server и целевому управляемому экземпляру используются имена, входящие в серверную роль sysadmin.
* Создайте общую сетевую папку, которую Azure Database Migration Service сможет использовать для резервного копирования базы данных-источника.
* Предоставьте учетной записи службы, от имени которой выполняется исходный экземпляр SQL Server, права на запись в эту созданную сетевую папку, а учетной записи компьютера для исходного сервера — права на чтение и запись для этой папки.
* Запишите имя пользователя и пароль учетной записи Windows, которой предоставлены полные права доступа к этой сетевой папке. Azure Database Migration Service олицетворяет учетные данные пользователя, чтобы передать файлы резервных копий в контейнер службы хранилища Azure для операции восстановления.
* Создайте контейнер больших двоичных объектов и получите универсальный код ресурса (URI) SAS, следуя инструкциям в статье [Управление ресурсами хранилища BLOB-объектов Azure с помощью обозревателя хранилищ (предварительная версия)](../vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container). Обязательно выберите все разрешения (чтение, запись, удаление и вывод списка) в окне политики при создании универсального кода ресурса (URI) SAS.

   > [!NOTE]
   > Полный список предварительных требований, необходимых для использования Azure Database Migration Service для выполнения миграции с SQL Server на SQL Управляемый экземпляр, см. в руководстве [миграция SQL Server в sql управляемый экземпляр](./tutorial-sql-server-to-managed-instance.md).

## <a name="next-steps"></a>Дальнейшие действия

Общие сведения о службе Azure Database Migration Service и доступности по регионам см. в статье [Что такое Azure Database Migration Service](dms-overview.md).