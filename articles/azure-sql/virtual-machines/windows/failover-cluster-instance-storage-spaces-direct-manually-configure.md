---
title: Создание FCI с Локальные дисковые пространства
description: Используйте Локальные дисковые пространства, чтобы создать экземпляр отказоустойчивого кластера (FCI) с SQL Server на виртуальных машинах Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.custom: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/18/2020
ms.author: mathoma
ms.openlocfilehash: aa19cf6b59b1efa4b14501fbf64e319da3e4c0b3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102048647"
---
# <a name="create-an-fci-with-storage-spaces-direct-sql-server-on-azure-vms"></a>Создание FCI с Локальные дисковые пространства (SQL Server на виртуальных машинах Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье объясняется, как создать экземпляр отказоустойчивого кластера (FCI) с помощью [Локальные дисковые пространства](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) с SQL Server на виртуальных машинах Azure. Локальные дисковые пространства выступает в качестве программной виртуальной сети хранения данных (VSAN), которая синхронизирует хранилище (диски с данными) между узлами (ВМ Azure) в кластере Windows. 

Дополнительные сведения см. в обзоре [FCI с SQL Server на виртуальных машинах Azure и в](failover-cluster-instance-overview.md) разделе рекомендации по работе с [кластерами](hadr-cluster-best-practices.md). 


## <a name="overview"></a>Обзор 

[Локальные дисковые пространства (S2D)](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) поддерживает два типа архитектур: схождение и гиперконвергентном. Инфраструктура гиперконвергентном размещает хранилище на тех же серверах, где размещается кластерное приложение, так что хранилище находится на каждом узле SQL Server FCI. 

На следующей схеме показано полное решение, которое использует гиперконвергентном Локальные дисковые пространства с SQL Server на виртуальных машинах Azure: 

![Схема полного решения с использованием гиперконвергентном Локальные дисковые пространства](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/00-sql-fci-s2d-complete-solution.png)

На предыдущей схеме показаны следующие ресурсы в одной группе ресурсов:

- Две виртуальные машины в отказоустойчивом кластере Windows Server. Когда виртуальная машина находится в отказоустойчивом кластере, она также называется *узлом кластера* или просто *узлом*.
- В каждой виртуальной машине есть два или больше дисков данных.
- Локальные дисковые пространства синхронизируют данные на диске данных и представляют синхронизированное хранилище в качестве пула носителей.
- Пул носителей представляет общий том кластера (CSV) для отказоустойчивого кластера.
- Роль экземпляра кластера SQL Server FCI использует CSV для дисков данных.
- Azure Load Balancer для хранения IP-адреса для экземпляра отказоустойчивого кластера SQL Server.
- В группе доступности Azure хранятся все ресурсы.

   > [!NOTE]
   > На основе шаблона в Azure можно создать полное решение. Пример шаблона доступен на странице шаблонов быстрого запуска [Azure](https://github.com/MSBrett/azure-quickstart-templates/tree/master/sql-server-2016-fci-existing-vnet-and-ad) для GitHub. Этот пример не предназначен или не протестирован для какой-либо конкретной рабочей нагрузки. Вы можете запустить шаблон, чтобы создать экземпляр отказоустойчивого кластера SQL Server с хранилищем Локальных дисковых пространств, подключенным к вашему домену. Шаблон можно оценить и изменить в соответствии со своими потребностями.


## <a name="prerequisites"></a>Предварительные требования

Перед выполнением инструкций, приведенных в этой статье, у вас уже должны быть:

- Подписка Azure. Начните работу [бесплатно](https://azure.microsoft.com/free/). 
- [Две или более подготовленные виртуальные машины Windows Azure](failover-cluster-instance-prepare-vm.md) в [группе доступности](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set).
- учетная запись с разрешениями на создание объектов как на виртуальных машинах Azure, так и в Active Directory;
- Последняя версия [PowerShell](/powershell/azure/install-az-ps). 


## <a name="add-the-windows-cluster-feature"></a>Добавление компонента кластера Windows

1. Подключитесь к первой виртуальной машине с помощью протокол удаленного рабочего стола (RDP) с учетной записью домена, которая является членом локальной группы администраторов и имеет разрешение на создание объектов в Active Directory. Используйте эту учетную запись для остальной части конфигурации.

1. Добавьте отказоустойчивую кластеризацию на каждую виртуальную машину.

   Чтобы установить отказоустойчивую кластеризацию из пользовательского интерфейса, выполните следующие действия на обеих виртуальных машинах:

   1. В **диспетчере сервера** выберите **Управление**, а затем — **Добавить роли и компоненты**.
   1. В мастере **добавления ролей и компонентов** выберите **Далее** , пока не появится возможность **выбрать компоненты**.
   1. В разделе **Выбор компонентов** выберите **Отказоустойчивая кластеризация**. Включите все необходимые компоненты и средства управления. 
   1. Выберите **Добавить компоненты**.
   1. Чтобы установить компоненты, нажмите кнопку **Далее**, а затем — **Готово**.

   Чтобы установить отказоустойчивую кластеризацию с помощью PowerShell, выполните следующий скрипт из сеанса PowerShell администратора на одной из виртуальных машин:

   ```powershell
   $nodes = ("<node1>","<node2>")
   Invoke-Command  $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools}
   ```

Дополнительные сведения о дальнейших действиях см. в подразделе "шаг 3. Настройка Локальные дисковые пространства" [решения гиперконвергентном с помощью Локальные дисковые пространства в Windows Server 2016](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-3-configure-storage-spaces-direct).


## <a name="validate-the-cluster"></a>Проверка кластера

Проверьте кластер в пользовательском интерфейсе или с помощью PowerShell.

Чтобы проверить кластер с помощью пользовательского интерфейса, выполните следующие действия на одной из виртуальных машин:

1. На панели **Диспетчер серверов** выберите **Инструменты** и нажмите **Диспетчер отказоустойчивости кластеров**.
1. В разделе **Диспетчер отказоустойчивости кластеров** выберите **Действие**, а затем выберите **Проверить конфигурацию**.
1. Выберите **Далее**.
1. На вкладке **Выбор серверов или кластера** введите имена обеих виртуальных машин.
1. На вкладке **Параметры тестирования** выберите **Выполнять только выбранные тесты**. 
1. Выберите **Далее**.
1. В разделе **Выбор тестов** выберите все тесты, кроме **Хранилище**, как показано ниже:

   ![Выбор проверочных тестов кластера](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/10-validate-cluster-test.png)

1. Выберите **Далее**.
1. В разделе **Подтверждение** нажмите **Далее**.

    Мастер **проверки конфигурации** выполняет проверочные тесты.

Чтобы проверить кластер с помощью PowerShell, запустите следующий скрипт из сеанса PowerShell администратора на одной из виртуальных машин:

   ```powershell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Storage Spaces Direct", "Inventory", "Network", "System Configuration"
   ```

После проверки кластера создайте отказоустойчивый кластер.


## <a name="create-failover-cluster"></a>Создание отказоустойчивого кластера

Для создания отказоустойчивого кластера требуется следующее.

- Имена виртуальных машин, которые становятся узлами кластера.
- Имя отказоустойчивого кластера.
- IP-адрес отказоустойчивого кластера. Вы можете использовать IP-адрес, который не используется в той же виртуальной сети и подсети Azure, где расположены узлы кластера.


# <a name="windows-server-2012---2016"></a>[Windows Server 2012–2016](#tab/windows2012)

Следующий скрипт PowerShell создает отказоустойчивый кластер для Windows Server версий с 2012 по 2016. Обновите скрипт, введя имена узлов (имена виртуальных машин) и доступный IP-адрес из виртуальной сети Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage
```   

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows2019)

Следующий скрипт PowerShell создает отказоустойчивый кластер для Windows Server 2019.  Обновите скрипт, введя имена узлов (имена виртуальных машин) и доступный IP-адрес из виртуальной сети Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage -ManagementPointNetworkType Singleton 
```

Дополнительные сведения см. в статье [Отказоустойчивый кластер: сетевой объект кластера](https://blogs.windows.com/windowsexperience/2018/08/14/announcing-windows-server-2019-insider-preview-build-17733/#W0YAxO8BfwBRbkzG.97).

---


## <a name="configure-quorum"></a>Настройка кворума

Настройте решение кворума, которое лучше подходит для ваших бизнес-потребностей. Можно настроить диск- [свидетель](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum), [облачный следящий сервер](/windows-server/failover-clustering/deploy-cloud-witness)или [файловый ресурс-свидетель](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum). Дополнительные сведения см. [в статье кворум с виртуальными машинами SQL Server](hadr-cluster-best-practices.md#quorum). 

## <a name="add-storage"></a>Добавление хранилища

Диски для Локальных дисковых пространств должны быть пустыми. Они не могут содержать разделы или другие данные. Чтобы очистить диски, следуйте инструкциям в разделе [развертывание Локальные дисковые пространства](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-31-clean-drives).

1. [Включите Локальные дисковые пространства](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-35-enable-storage-spaces-direct).

   Следующий скрипт PowerShell включает Локальные дисковые пространства:  

   ```powershell
   Enable-ClusterS2D
   ```

   В **диспетчере отказоустойчивости кластеров** теперь отображается пул носителей.

1. [Создайте том](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-36-create-volumes).

   Локальные дисковые пространства автоматически создают пул носителей при включении. Теперь можно создать том. Командлет PowerShell `New-Volume` автоматизирует процесс создания тома. Этот процесс включает форматирование, добавление тома в кластер и создание CSV-файла. В этом примере создается CSV-файл размером 800 гигабайт (ГБ):

   ```powershell
   New-Volume -StoragePoolFriendlyName S2D* -FriendlyName VDisk01 -FileSystem CSVFS_REFS -Size 800GB
   ```   

   После выполнения предыдущей команды том 800 ГБ монтируется в качестве ресурса кластера. Том находится в папке `C:\ClusterStorage\Volume1\`.

   На этом снимке экрана показан CSV-файл с Локальные дисковые пространства:

   ![Снимок экрана общий том кластера с Локальные дисковые пространства](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/15-cluster-shared-volume.png)



## <a name="test-cluster-failover"></a>Тестовая отработка отказа кластера

Протестируйте отработку отказа кластера. В **Диспетчер отказоустойчивости кластеров** щелкните правой кнопкой мыши кластер, выберите **другие действия**  >  **переместить основной кластер ресурс**  >  **выберите узел**, а затем выберите другой узел кластера. Перенесите основной кластерный ресурс на каждый узел кластера, а затем верните его на основной узел. Если вам удалось успешно перенести ресурс на каждый узел, вы готовы к установке SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Тестирование отказоустойчивости кластеров путем переноса основного кластерного ресурса на другие узлы":::

## <a name="create-sql-server-fci"></a>Создание экземпляра отказоустойчивого кластера SQL Server

После настройки отказоустойчивого кластера и всех компонентов кластера, включая хранилище, можно создать экземпляр отказоустойчивого кластера SQL Server.

1. Подключитесь к первой виртуальной машине Azure с помощью RDP.

1. В **Диспетчер отказоустойчивости кластеров** убедитесь, что все основные ресурсы кластера находятся на первой виртуальной машине. При необходимости переместите все ресурсы на эту виртуальную машину.

1. Найдите установочный носитель. Если на виртуальной машине используется один из образов Azure Marketplace, носитель находится в папке `C:\SQLServer_<version number>_Full`. Выберите **Установить**.

1. В диалоговом окне **Центр установки SQL Server** выберите **Установка**.

1. Щелкните **Новая установка отказоустойчивого кластера SQL Server**. Следуйте указаниям мастера, чтобы установить экземпляр отказоустойчивого кластера SQL Server.

   Каталоги данных экземпляра отказоустойчивого кластера должны находиться в кластеризованном хранилище. В Локальные дисковые пространства это не общий диск, а точка подключения к тому на каждом сервере. Локальные дисковые пространства синхронизируют том между обоими узлами. Том представлен кластеру в виде CSV-файла. Используйте точки подключения CSV для каталогов данных.

   ![Каталоги данных](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/20-data-dicrectories.png)

1. После завершения инструкций в мастере программа установки установит SQL Server FCI на первом узле.

1. После установки экземпляра отказоустойчивого кластера на первом узле подключитесь ко второму узлу с помощью протокола удаленного рабочего стола.

1. Откройте диалоговое окно **Центр установки SQL Server**. Нажмите **Установка**.

1. Выберите **Добавление узла в отказоустойчивый кластер SQL Server**. Следуйте указаниям мастера, чтобы установить сервер SQL и добавить его в экземпляр отказоустойчивого кластера.

   >[!NOTE]
   >При использовании образа коллекции Azure Marketplace, который содержит SQL Server, средства SQL Server включаются в образ. Если вы не использовали один из этих образов, установите средства SQL Server отдельно. Дополнительные сведения см. в разделе [Скачивание SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).
   >


## <a name="register-with-the-sql-vm-rp"></a>Регистрация в RP виртуальной машины SQL

Чтобы управлять виртуальной машиной SQL Server на портале, зарегистрируйте ее с помощью расширения агента IaaS SQL (RP) в [режиме упрощенного управления](sql-agent-extension-manually-register-single-vm.md#lightweight-management-mode)(в настоящее время это единственный режим, поддерживаемый с FCI и SQL Server на виртуальных машинах Azure). 


Регистрация SQL Server виртуальной машины в упрощенном режиме с помощью PowerShell:  

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
         
# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
   -LicenseType PAYG -SqlManagementType LightWeight  
```

## <a name="configure-connectivity"></a>Настройка подключения 

Чтобы правильно направить трафик на текущий основной узел, настройте параметр подключения, подходящий для вашей среды. Вы можете создать [балансировщик нагрузки Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) или, если вы используете SQL Server 2019 CU2 (или более поздней версии) и Windows Server 2016 (или более поздней версии), вместо этого можно использовать функцию [имени распределенной сети](failover-cluster-instance-distributed-network-name-dnn-configure.md) . 

Дополнительные сведения о вариантах подключения кластеров см. в статье [Рекомендации по настройке кластера (SQL Server на Виртуальных машинах Azure)](hadr-cluster-best-practices.md#connectivity). 

## <a name="limitations"></a>Ограничения

- Виртуальные машины Azure поддерживают Microsoft координатор распределенных транзакций (MSDTC) в Windows Server 2019 с хранилищем в CSV и [стандартной подсистемой балансировки нагрузки](../../../load-balancer/load-balancer-overview.md).
- Диски, подключенные как диски с файловой системой NTFS, можно использовать с Локальные дисковые пространства только в том случае, если параметр приемлемость диска не установлен или снят при добавлении хранилища в кластер. 
- Поддерживается только регистрация с расширением агента IaaS SQL в [режиме упрощенного управления](sql-server-iaas-agent-extension-automate-management.md#management-modes) .

## <a name="next-steps"></a>Следующие шаги

Если вы еще не сделали этого, настройте подключение к FCI с [именем виртуальной сети и подсистемой балансировки нагрузки Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) или [именем распределенной сети (DNN)](failover-cluster-instance-distributed-network-name-dnn-configure.md). 

Если Локальные дисковые пространства не является соответствующим решением для хранения FCI, подумайте о создании FCI с помощью [общих дисков Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) или [общих файловых ресурсов](failover-cluster-instance-premium-file-share-manually-configure.md) уровня "Премиум". 

Дополнительные сведения см. в обзоре [FCI с SQL Server на виртуальных машинах Azure и в](failover-cluster-instance-overview.md) разделе рекомендации по [конфигурации кластера](hadr-cluster-best-practices.md). 

Дополнительные сведения см. в разделе: 
- [технологии кластера под управлением Windows](/windows-server/failover-clustering/failover-clustering-overview);   
- [Экземпляры отказоустойчивого кластера SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)