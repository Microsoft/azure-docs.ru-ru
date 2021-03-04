---
title: Создание FCI с общей папкой Premium
description: Используйте файловый ресурс Premium (PFS) для создания экземпляра отказоустойчивого кластера (FCI) с SQL Server на виртуальных машинах Azure.
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
ms.openlocfilehash: 80fe9f03f2c57eab8527e553153f3e65315a54bf
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102034852"
---
# <a name="create-an-fci-with-a-premium-file-share-sql-server-on-azure-vms"></a>Создание FCI с общей папкой Premium (SQL Server на виртуальных машинах Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье объясняется, как создать экземпляр отказоустойчивого кластера (FCI) с SQL Server на виртуальных машинах Azure с помощью [файлового ресурса](../../../storage/files/storage-how-to-create-file-share.md)уровня "Премиум".

Общие файловые ресурсы уровня "Премиум" — это Локальные дисковые пространстваные (SSD) согласованные файловые ресурсы с низкой задержкой, которые полностью поддерживаются для использования с экземплярами отказоустойчивого кластера для SQL Server 2012 или более поздней версии на Windows Server 2012 или более поздней версии. Общие папки уровня "Премиум" отличаются повышенной гибкостью: вы может менять их размеры и масштабировать их без простоя.

Дополнительные сведения см. в обзоре [FCI с SQL Server на виртуальных машинах Azure и в](failover-cluster-instance-overview.md) разделе рекомендации по работе с [кластерами](hadr-cluster-best-practices.md). 

## <a name="prerequisites"></a>Предварительные требования

Перед выполнением инструкций, приведенных в этой статье, у вас уже должны быть:

- Подписка Azure.
- учетная запись с разрешениями на создание объектов как на виртуальных машинах Azure, так и в Active Directory;
- [Две или более подготовленные виртуальные машины Windows Azure](failover-cluster-instance-prepare-vm.md) в [группе доступности](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set) или разных [зонах доступности](../../../virtual-machines/windows/create-portal-availability-zone.md#confirm-zone-for-managed-disk-and-ip-address).
- [общая папка уровня "Премиум"](../../../storage/files/storage-how-to-create-file-share.md), которая будет использоваться в качестве кластеризованного диска, с учетом квоты хранилища для вашей базы с файлами данных.
- Последняя версия [PowerShell](/powershell/azure/install-az-ps). 

## <a name="mount-premium-file-share"></a>Подключить файловый ресурс Premium

1. Войдите на [портал Azure](https://portal.azure.com). и перейдите к своей учетной записи хранения.
1. Перейдите в **файловые ресурсы** в разделе **Служба файлов**, а затем выберите общую папку уровня "Премиум", которую вы хотите использовать для хранилища SQL.
1. Выберите **Подключить**, чтобы открыть строку подключения общей папки.
1. В раскрывающемся списке выберите букву диска, которую нужно использовать, а затем скопируйте оба блока кода в Блокнот.

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/premium-file-storage-commands.png" alt-text="Скопируйте обе команды PowerShell с портала подключения общей папки":::

1. Используйте протокол удаленного рабочего стола (RDP) для подключения к виртуальной машине SQL Server с учетной записью, которую SQL Server FCI будет использовать для учетной записи службы.
1. Откройте административную консоль командной строки PowerShell.
1. Выполните команды, сохраненные ранее во время работы на портале.
1. Перейдите в файловый ресурс с помощью проводника или диалогового окна " **выполнить** " (выберите Windows + R). Укажите сетевой путь `\\storageaccountname.file.core.windows.net\filesharename`. Например `\\sqlvmstorageaccount.file.core.windows.net\sqlpremiumfileshare`.

1. Создайте по крайней мере одну папку во вновь подключенной общей папке, чтобы поместить файлы данных SQL в.
1. Повторите эти действия для каждой виртуальной машины SQL Server, которая будет входить в состав кластера.

  > [!IMPORTANT]
  > - Рассмотрите возможность использования отдельной общей папки для файлов резервных копий, чтобы сохранить количество операций ввода-вывода в секунду и емкость этого общего ресурса для файлов данных и журналов. Для файлов резервных копий можно использовать файловый ресурс уровня "Премиум" или "Стандартный".
  > - Если вы используете Windows 2012 R2 или более ранней версии, выполните те же действия, чтобы подключить общую папку, которую планируется использовать в качестве файлового ресурса-свидетеля. 
  > 


## <a name="add-windows-cluster-feature"></a>Добавить компонент кластера Windows

1. Подключитесь к первой виртуальной машине с помощью протокола удаленного рабочего стола, используя учетную запись домена, которая является участником локальной группы администраторов, а также имеет разрешения на создание объектов в Active Directory. Используйте эту учетную запись для остальной части конфигурации.

1. [Добавьте отказоустойчивую кластеризацию на каждую виртуальную машину](availability-group-manually-configure-prerequisites-tutorial.md#add-failover-clustering-features-to-both-sql-server-vms).

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

## <a name="validate-cluster"></a>Проверка кластера

Проверьте кластер в пользовательском интерфейсе или с помощью PowerShell.

Чтобы проверить кластер с помощью пользовательского интерфейса, выполните следующие действия на одной из виртуальных машин:

1. На панели **Диспетчер серверов** выберите **Инструменты** и нажмите **Диспетчер отказоустойчивости кластеров**.
1. В разделе **Диспетчер отказоустойчивости кластеров** выберите **Действие**, а затем выберите **Проверить конфигурацию**.
1. Выберите **Далее**.
1. На вкладке **Выбор серверов или кластера** введите имена обеих виртуальных машин.
1. На вкладке **Параметры тестирования** выберите **Выполнять только выбранные тесты**. 
1. Выберите **Далее**.
1. В разделе **Выбор тестов** выберите все тесты, кроме **Хранилище** и **Локальные дисковые пространства**, как показано ниже:

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/cluster-validation.png" alt-text="Выбор проверочных тестов кластера":::

1. Выберите **Далее**.
1. В разделе **Подтверждение** нажмите **Далее**.

Мастер **проверки конфигурации** выполняет проверочные тесты.

Чтобы проверить кластер с помощью PowerShell, запустите следующий скрипт из сеанса PowerShell администратора на одной из виртуальных машин:

   ```powershell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Inventory", "Network", "System Configuration"
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


## <a name="test-cluster-failover"></a>Тестовая отработка отказа кластера

Протестируйте отработку отказа кластера. В **Диспетчер отказоустойчивости кластеров** щелкните правой кнопкой мыши кластер, выберите **другие действия**  >  **переместить основной кластер ресурс**  >  **выберите узел**, а затем выберите другой узел кластера. Перенесите основной кластерный ресурс на каждый узел кластера, а затем верните его на основной узел. Если вам удалось успешно перенести ресурс на каждый узел, вы готовы к установке SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Тестирование отказоустойчивости кластеров путем переноса основного кластерного ресурса на другие узлы":::


## <a name="create-sql-server-fci"></a>Создание экземпляра отказоустойчивого кластера SQL Server

После настройки отказоустойчивого кластера можно создать экземпляр отказоустойчивого кластера SQL Server.

1. Подключитесь к первой виртуальной машине Azure с помощью протокола удаленного рабочего стола (RDP).

1. В **Диспетчер отказоустойчивости кластеров** убедитесь, что все основные ресурсы кластера находятся на первой виртуальной машине. При необходимости переместите все ресурсы на эту виртуальную машину.

1. Найдите установочный носитель. Если на виртуальной машине используется один из образов Azure Marketplace, носитель находится в папке `C:\SQLServer_<version number>_Full`. 

1. Выберите **Установить**.

1. В диалоговом окне **Центр установки SQL Server** выберите **Установка**.

1. Выберите **новый SQL Server установка отказоустойчивого кластера** и следуйте инструкциям мастера для установки SQL Server FCI.

   Каталоги данных экземпляра отказоустойчивого кластера должны находиться в общей папки уровня "Премиум". Введите полный путь к общей папке в следующем формате: `\\storageaccountname.file.core.windows.net\filesharename\foldername` . Появится предупреждение о том, что вы указали файловый сервер в качестве каталога данных. Это нормально. Убедитесь, что учетная запись пользователя, которая использовалась для доступа к виртуальной машине по протоколу RDP, если вы сохранили общую папку, — это та же учетная запись, которую использует служба SQL Server, чтобы избежать возможных сбоев.

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/use-file-share-as-data-directories.png" alt-text="Использование общей папки в качестве каталога для хранения данных SQL":::

1. После завершения процедуры в мастере программа установки установит экземпляр отказоустойчивого кластера SQL Server на первом узле.

1. После установки экземпляра отказоустойчивого кластера на первом узле подключитесь ко второму узлу с помощью протокола удаленного рабочего стола.

1. Откройте **центр установки SQL Server** и выберите пункт **Установка**.

1. Выберите **Добавление узла в отказоустойчивый кластер SQL Server**. Следуйте указаниям мастера, чтобы установить сервер SQL и добавить его в экземпляр отказоустойчивого кластера.

   >[!NOTE]
   >При использовании образа коллекции Azure Marketplace с SQL Server средства SQL Server включаются в образ. Если вы не использовали один из этих образов, установите средства SQL Server отдельно. Дополнительные сведения см. в разделе [Скачивание SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).

1. Повторите эти действия на всех других узлах, которые необходимо добавить в экземпляр отказоустойчивого кластера SQL Server. 

## <a name="register-with-the-sql-vm-rp"></a>Регистрация в RP виртуальной машины SQL

Чтобы управлять виртуальной машиной SQL Server на портале, зарегистрируйте ее с помощью расширения агента IaaS SQL (RP) в [режиме упрощенного управления](sql-agent-extension-manually-register-single-vm.md#lightweight-management-mode)(в настоящее время это единственный режим, поддерживаемый с FCI и SQL Server на виртуальных машинах Azure). 

Регистрация SQL Server виртуальной машины в упрощенном режиме с помощью PowerShell (-LicenseType может быть `PAYG` или `AHUB` ):

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
         
# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
   -LicenseType ???? -SqlManagementType LightWeight  
```

## <a name="configure-connectivity"></a>Настройка подключения 

Чтобы правильно направить трафик на текущий основной узел, настройте параметр подключения, подходящий для вашей среды. Вы можете создать [балансировщик нагрузки Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) или, если вы используете SQL Server 2019 CU2 (или более поздней версии) и Windows Server 2016 (или более поздней версии), вместо этого можно использовать функцию [имени распределенной сети](failover-cluster-instance-distributed-network-name-dnn-configure.md) . 

Дополнительные сведения о вариантах подключения кластеров см. в статье [Рекомендации по настройке кластера (SQL Server на Виртуальных машинах Azure)](hadr-cluster-best-practices.md#connectivity). 

## <a name="limitations"></a>Ограничения

- Microsoft координатор распределенных транзакций (MSDTC) не поддерживается в Windows Server 2016 и более ранних версиях. 
- Функция файлового потока для отказоустойчивых кластеров с общими папками уровня "Премиум" не поддерживается. Чтобы использовать FILESTREAM, разверните кластер с помощью [Локальные дисковые пространства](failover-cluster-instance-storage-spaces-direct-manually-configure.md) или [общих дисков Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) .
- Поддерживается только регистрация с расширением агента IaaS SQL в [режиме упрощенного управления](sql-server-iaas-agent-extension-automate-management.md#management-modes) . 

## <a name="next-steps"></a>Дальнейшие действия

Если вы еще не сделали этого, настройте подключение к FCI с [именем виртуальной сети и подсистемой балансировки нагрузки Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) или [именем распределенной сети (DNN)](failover-cluster-instance-distributed-network-name-dnn-configure.md). 


Если файловые ресурсы уровня "Премиум" не являются подходящим решением для хранения FCI, рекомендуется создать FCI с помощью [общих дисков Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) или [Локальные дисковые пространства](failover-cluster-instance-storage-spaces-direct-manually-configure.md) . 

Дополнительные сведения см. в обзоре [FCI с SQL Server на виртуальных машинах Azure и в](failover-cluster-instance-overview.md) разделе рекомендации по [конфигурации кластера](hadr-cluster-best-practices.md). 

Дополнительные сведения см. в разделе: 
- [технологии кластера под управлением Windows](/windows-server/failover-clustering/failover-clustering-overview);   
- [Экземпляры отказоустойчивого кластера SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)