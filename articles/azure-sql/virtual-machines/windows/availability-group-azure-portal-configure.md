---
title: Настройка группы доступности (портал Azure)
description: Используйте портал Azure для создания отказоустойчивого кластера Windows, прослушивателя группы доступности и внутренней подсистемы балансировки нагрузки на виртуальной машине SQL Server в Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/20/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019, devx-track-azurecli
ms.openlocfilehash: 14760b4244d42e57aaed7f7d96f487a66147a554
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359512"
---
# <a name="use-azure-portal-to-configure-an-availability-group-preview-for-sql-server-on-azure-vm"></a>Использование портал Azure для настройки группы доступности (Предварительная версия) для SQL Server на виртуальной машине Azure 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье описывается, как использовать [портал Azure](https://portal.azure.com) для настройки группы доступности для SQL Server на виртуальных машинах Azure. 

Используйте портал Azure для создания нового кластера или подключения существующего кластера, а затем создайте группу доступности, прослушиватель и внутреннюю подсистему балансировки нагрузки. 

Эта функция в настоящее время находится на стадии предварительной версии. 

Хотя в этой статье для настройки среды группы доступности используется портал Azure, это можно сделать с помощью [PowerShell или Azure CLI](availability-group-az-commandline-configure.md), шаблонов быстрого запуска [Azure](availability-group-quickstart-template-configure.md)или [вручную](availability-group-manually-configure-tutorial.md) . 


## <a name="prerequisites"></a>Предварительные условия

Чтобы настроить Always On группы доступности с помощью портал Azure, необходимо выполнить следующие предварительные требования. 

- [Подписка Azure](https://azure.microsoft.com/free/).
- Группа ресурсов с контроллером домена. 
- Одна или несколько виртуальных машин, присоединенных к домену, [в Azure под управлением SQL Server 2016 (или более поздней версии) Enterprise Edition](./create-sql-vm-portal.md) в *одной* группе доступности или *разных* зонах доступности, которые были [зарегистрированы в расширении агента IaaS SQL в режиме полной управляемости](sql-agent-extension-manually-register-single-vm.md) , и используют одну учетную запись домена для службы SQL Server на каждой виртуальной машине.
- Два доступных (неиспользуемых сущностью) IP-адреса. Один предназначен для внутренней подсистемы балансировки нагрузки. Второй — для прослушивателя группы доступности в той же подсети, что и группа доступности. Если вы используете существующую подсистему балансировки нагрузки, для прослушивателя группы доступности требуется только один доступный IP-адрес. 

## <a name="permissions"></a>Разрешения

Для настройки группы доступности с помощью портал Azure необходимы следующие разрешения учетной записи: 

- Существующая учетная запись пользователя домена с разрешением на **создание объекта компьютера** в домене. Например, учетная запись администратора домена обычно имеет соответствующие разрешения (например: account@domain.com). _Эта учетная запись также должна входить в группу локальных администраторов на каждой виртуальной машине для создания кластера._
- Учетная запись пользователя домена, которая используется для управления SQL Server. Это должна быть одна и та же учетная запись для каждой SQL Server виртуальной машины, которую вы собираетесь добавить в группу доступности. 

## <a name="configure-cluster"></a>Настройка кластера

Настройте кластер с помощью портал Azure. Можно создать новый кластер или, если у вас уже есть кластер, можно подключить его к расширению агента IaaS SQL в целях управления порталом.


### <a name="create-a-new-cluster"></a>Создание нового кластера

Если у вас уже есть кластер, пропустите этот раздел и перейдите к подключению [существующего кластера](#onboard-existing-cluster) . 

Если у вас еще нет кластера, создайте его с помощью портал Azure, выполнив следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com). 
1. Перейдите к ресурсу [виртуальных машин SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) . 
1. Выберите **высокий уровень доступности** в разделе **Параметры**. 
1. Выберите **+ создать отказоустойчивый кластер Windows Server** , чтобы открыть страницу **Настройка отказоустойчивого кластера Windows** .  

   :::image type="content" source="media/availability-group-az-portal-configure/create-new-cluster.png" alt-text="Создайте новый кластер, выбрав + новый кластер на портале.":::

1. Присвойте имя кластеру и укажите учетную запись хранения, которая будет использоваться в качестве облачного следящего сервера. Используйте существующую учетную запись хранения или щелкните **создать** , чтобы создать новую учетную запись хранения. Имя учетной записи хранения должно содержать от 3 до 24 знаков и состоять только из цифр и букв нижнего регистра.

   :::image type="content" source="media/availability-group-az-portal-configure/configure-new-cluster-1.png" alt-text="Укажите имя, учетную запись хранения и учетные данные для кластера.":::

1. Разверните узел **учетные данные отказоустойчивого кластера Windows Server** , чтобы предоставить [учетные данные](/rest/api/sqlvm/sqlvirtualmachinegroups/createorupdate#wsfcdomainprofile) SQL Server учетной записи службы, а также оператор кластера и учетные записи начальной загрузки, если они отличаются от учетной записи, используемой для службы SQL Server. 

   :::image type="content" source="media/availability-group-az-portal-configure/configure-new-cluster-2.png" alt-text="Укажите учетные данные для учетной записи службы SQL, учетной записи оператора кластера и начальной загрузки кластера.":::

1. Выберите SQL Server виртуальные машины, которые необходимо добавить в кластер. Обратите внимание, требуется ли перезагрузка и будьте внимательны. Будут видны только виртуальные машины, зарегистрированные с расширением агента IaaS SQL в режиме полного управления и расположенные в одном и том же месте, домене и в той же виртуальной сети, что и первичная SQL Serverная виртуальная машина. 
1. Нажмите кнопку **Применить** , чтобы создать кластер. Состояние развертывания можно проверить в **журнале действий** , доступном по значку колокольчика на верхней панели навигации. 
1. Чтобы отказоустойчивый кластер поддерживался корпорацией Майкрософт, он должен пройти проверку кластера. Подключитесь к виртуальной машине с помощью предпочтительного метода (например, протокол удаленного рабочего стола (RDP)) и убедитесь, что кластер прошел проверку, прежде чем продолжить. Несоблюдение этого действия оставляет кластер в неподдерживаемом состоянии. Вы можете проверить кластер с помощью диспетчер отказоустойчивости кластеров (FCM) или следующей команды PowerShell:

    ```powershell
    Test-Cluster –Node ("<node1>","<node2>") –Include "Inventory", "Network", "System Configuration"
    ```
    


### <a name="onboard-existing-cluster"></a>Подключение существующего кластера

Если у вас уже есть кластер, настроенный в среде SQL Server ВМ, его можно подключить из портал Azure.

Для этого выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com). 
1. Перейдите к ресурсу [виртуальных машин SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) . 
1. Выберите **высокий уровень доступности** в разделе **Параметры**. 
1. Выберите подключить **существующий отказоустойчивый кластер Windows Server** , чтобы открыть страницу **отказоустойчивого кластера Windows Server** . 

   :::image type="content" source="media/availability-group-az-portal-configure/onboard-existing-cluster.png" alt-text="Подключение существующего кластера со страницы высокой доступности в ресурсе виртуальных машин SQL":::

1. Проверьте параметры кластера. 
1. Выберите **Применить** , чтобы подключить кластер, а затем выберите **Да** в командной строке для продолжения.

## <a name="create-availability-group"></a>Создание группы доступности

После того как кластер был создан или подключен, создайте группу доступности с помощью портал Azure. Для этого выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com). 
1. Перейдите к ресурсу [виртуальных машин SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) . 
1. Выберите **высокий уровень доступности** в разделе **Параметры**. 
1. Выберите **+ New Always on группа доступности** , чтобы открыть страницу **Создание группы доступности** .

   :::image type="content" source="media/availability-group-az-portal-configure/create-new-availability-group.png" alt-text="Выберите Создать группу доступности Always on, чтобы открыть страницу Создание группы доступности.":::

1. Введите имя группы доступности. 
1. Выберите **настроить прослушиватель** , чтобы открыть страницу **Настройка прослушивателя группы доступности** . 

   :::image type="content" source="media/availability-group-az-portal-configure/create-availability-group.png" alt-text="Укажите имя группы доступности и настройте прослушиватель.":::

1. Заполните значения и либо используйте существующую подсистему балансировки нагрузки, либо выберите **создать** , чтобы создать новую подсистему балансировки нагрузки.  Нажмите кнопку **Применить** , чтобы сохранить параметры и создать прослушиватель и подсистему балансировки нагрузки. 

   :::image type="content" source="media/availability-group-az-portal-configure/configure-new-listener.png" alt-text="Заполните значения в форме, чтобы создать новый прослушиватель и подсистему балансировки нагрузки.":::

1. Выберите **+ выбрать реплику** , чтобы открыть страницу **Настройка реплик группы доступности** .
1. Выберите виртуальные машины, которые необходимо добавить в группу доступности, и выберите параметры группы доступности, которые лучше всего соответствуют потребностям вашего бизнеса. Нажмите кнопку **Применить** , чтобы сохранить параметры. 

   :::image type="content" source="media/availability-group-az-portal-configure/add-replicas.png" alt-text="Выберите виртуальные машины для добавления в группу доступности и настройте параметры, подходящие для вашего бизнеса.":::

1. Проверьте параметры группы доступности и нажмите кнопку " **Применить** ", чтобы создать группу доступности. 

Состояние развертывания можно проверить в **журнале действий** , доступном по значку колокольчика на верхней панели навигации. 

  > [!NOTE]
  > **Работоспособность синхронизации** на странице **высокой доступности** портал Azure будет отображаться как **неработоспособная** , пока вы не добавите базы данных в группу доступности. 


## <a name="add-database-to-availability-group"></a>Добавить базу данных в группу доступности

После завершения развертывания добавьте базы данных в группу доступности. В приведенных ниже шагах используется SQL Server Management Studio (SSMS), но можно также использовать [Transact-SQL или PowerShell](/sql/database-engine/availability-groups/windows/availability-group-add-a-database) . 

Чтобы добавить базы данных в группу доступности с помощью SQL Server Management Studio, выполните следующие действия.

1. Подключитесь к одной из SQL Server виртуальных машин с помощью предпочтительного метода, например подключение к удаленному рабочему столу (RDP). 
1. Откройте SQL Server Management Studio (SSMS).
1. Подключитесь к экземпляру SQL Server. 
1. Разверните **Always on высокий уровень доступности** в **обозревателе объектов**.
1. Разверните узел **группы доступности**, щелкните правой кнопкой мыши группу доступности и выберите **Добавить базу данных...**.

   :::image type="content" source="media/availability-group-az-portal-configure/add-database.png" alt-text="Щелкните правой кнопкой мыши группу доступности в обозревателе объектов и выберите Добавить базу данных.":::

1. Следуйте инструкциям на экране, чтобы выбрать базы данных, которые нужно добавить в группу доступности. 
1. Нажмите кнопку **ОК** , чтобы сохранить параметры и добавить базу данных в группу доступности. 
1. После добавления базы данных обновите **Обозреватель объектов** , чтобы подтвердить состояние базы данных, как `synchronized` . 

После добавления баз данных можно проверить состояние группы доступности в портал Azure: 

:::image type="content" source="media/availability-group-az-portal-configure/healthy-availability-group.png" alt-text="Проверка состояния группы доступности на странице высокой доступности из портал Azure после синхронизации баз данных":::

## <a name="add-more-vms"></a>Добавить дополнительные виртуальные машины

Чтобы добавить в кластер дополнительные SQL Server виртуальные машины, выполните следующие действия. 

1. Войдите на [портал Azure](https://portal.azure.com). 
1. Перейдите к ресурсу [виртуальных машин SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) . 
1. Выберите **высокий уровень доступности** в разделе **Параметры**. 
1. Выберите **настроить отказоустойчивый кластер Windows Server** , чтобы открыть страницу **Настройка отказоустойчивого кластера Windows Server** . 

   :::image type="content" source="media/availability-group-az-portal-configure/configure-existing-cluster.png" alt-text="Выберите настроить отказоустойчивый кластер Windows Server, чтобы добавить виртуальные машины в кластер.":::

1. Разверните узел **учетные данные отказоустойчивого кластера Windows Server** и введите учетные записи, используемые для службы SQL Server, оператора кластера и начальной загрузки кластера. 
1. Выберите SQL Server виртуальные машины, которые необходимо добавить в кластер. 
1. Нажмите кнопку **Применить**. 

Состояние развертывания можно проверить в **журнале действий** , доступном по значку колокольчика на верхней панели навигации. 


## <a name="modify-availability-group"></a>Изменение группы доступности 


Можно **добавить дополнительные реплики** в группу доступности, **настроить прослушиватель** или **удалить прослушиватель** на странице **высокой доступности** в портал Azure, нажав кнопку с многоточием (...) рядом с группой доступности: 

:::image type="content" source="media/availability-group-az-portal-configure/configure-listener.png" alt-text="Щелкните многоточие рядом с группой доступности и выберите Добавить реплику, чтобы добавить дополнительные реплики в группу доступности.":::

## <a name="remove-cluster"></a>Удалить кластер

Удалите все SQL Server виртуальные машины из кластера, чтобы уничтожить их, а затем удалите метаданные кластера из расширения агента IaaS SQL. Это можно сделать с помощью последней версии [Azure CLI](/cli/azure/install-azure-cli) или PowerShell. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Сначала удалите все SQL Server виртуальные машины из кластера. Это приведет к физическому удалению узлов из кластера и уничтожении кластера:  

```azurecli-interactive
# Remove the VM from the cluster metadata
# example: az sql vm remove-from-group --name SQLVM2 --resource-group SQLVM-RG

az sql vm remove-from-group --name <VM1 name>  --resource-group <resource group name>
az sql vm remove-from-group --name <VM2 name>  --resource-group <resource group name>
```

Если это единственные виртуальные машины в кластере, кластер будет уничтожен. Если в кластере есть виртуальные машины, отличающиеся от SQL Server удаленных виртуальных машин, другие виртуальные машины не удаляются, и кластер не будет уничтожен. 

Затем удалите метаданные кластера из расширения агента IaaS SQL: 

```azurecli-interactive
# Remove the cluster from the SQL VM RP metadata
# example: az sql vm group delete --name Cluster --resource-group SQLVM-RG

az sql vm group delete --name <cluster name> --resource-group <resource group name>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Сначала удалите все SQL Server виртуальные машины из кластера. Это приведет к физическому удалению узлов из кластера и уничтожении кластера: 

```powershell-interactive
# Remove the SQL VM from the cluster
# example: $sqlvm = Get-AzSqlVM -Name SQLVM3 -ResourceGroupName SQLVM-RG
#  $sqlvm. SqlVirtualMachineGroup = ""
#  Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm

$sqlvm = Get-AzSqlVM -Name <VM Name> -ResourceGroupName <Resource Group Name>
   $sqlvm. SqlVirtualMachineGroup = ""
   
   Update-AzSqlVM -ResourceId $sqlvm -SqlVM $sqlvm
```

Если это единственные виртуальные машины в кластере, кластер будет уничтожен. Если в кластере есть виртуальные машины, отличающиеся от SQL Server удаленных виртуальных машин, другие виртуальные машины не удаляются, и кластер не будет уничтожен. 


Затем удалите метаданные кластера из расширения агента IaaS SQL: 

```powershell-interactive
# Remove the cluster metadata
# example: Remove-AzSqlVMGroup -ResourceGroupName "SQLVM-RG" -Name "Cluster"

Remove-AzSqlVMGroup -ResourceGroupName "<resource group name>" -Name "<cluster name>"
```

---

## <a name="troubleshooting"></a>Устранение неполадок

При возникновении проблем можно проверить историю развертывания и просмотреть распространенные ошибки, а также их разрешения. 

### <a name="check-deployment-history"></a>Проверка журнала развертывания

Изменения в кластере и группе доступности с помощью портала выполняются в развертываниях. Журнал развертывания может предоставить более подробные сведения, если возникли проблемы при создании или подключении кластера или при создании группы доступности.

Чтобы просмотреть журналы развертывания и проверить журнал развертывания, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите к группе ресурсов.
1. Выберите **Развертывания** в разделе **Параметры**.
1. Выберите нужное развертывание, чтобы узнать больше о развертывании. 


   :::image type="content" source="media/availability-group-az-portal-configure/failed-deployment.png" alt-text="Выберите развертывание, которое вы хотите изучить." :::

### <a name="common-errors"></a>Распространенные ошибки

Ознакомьтесь со следующими распространенными ошибками и их разрешениями. 

#### <a name="the-account-which-is-used-to-start-up-sql-service-is-not-a-domain-account"></a>Учетная запись, используемая для запуска службы SQL, не является учетной записью домена

Это означает, что поставщику ресурсов не удалось получить доступ к службе SQL Server с указанными учетными данными. Некоторые распространенные решения:
- Убедитесь, что контроллер домена работает.
- Убедитесь, что учетные данные, указанные на портале, соответствуют службам SQL Server. 


## <a name="next-steps"></a>Дальнейшие действия


Дополнительные сведения о группах доступности см. в следующих статьях:

- [Общие сведения о группах доступности](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)
* [Администрирование группы доступности](/sql/database-engine/availability-groups/windows/administration-of-an-availability-group-sql-server)   
* [Отслеживание групп доступности (SQL Server)](/sql/database-engine/availability-groups/windows/monitoring-of-availability-groups-sql-server)
* [Инструкции Transact-SQL группы доступности ](/sql/database-engine/availability-groups/windows/transact-sql-statements-for-always-on-availability-groups)   
* [Команды PowerShell для групп доступности](/sql/database-engine/availability-groups/windows/overview-of-powershell-cmdlets-for-always-on-availability-groups-sql-server)  

Дополнительные сведения о SQL Server виртуальных машинах см. в следующих статьях: 

* [Общие сведения о виртуальных машинах SQL Server](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Заметки о виртуальных машинах SQL Server](../../database/doc-changes-updates-release-notes.md)
* [Часто задаваемые вопросы о виртуальных машинах SQL Server](frequently-asked-questions-faq.md)
