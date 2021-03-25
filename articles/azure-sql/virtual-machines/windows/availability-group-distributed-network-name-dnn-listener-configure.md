---
title: Настройка прослушивателя DNN для группы доступности
description: Узнайте, как настроить прослушиватель имени распределенной сети (DNN), чтобы заменить прослушиватель имени виртуальной сети (VNN) и направить трафик в группу доступности Always On в SQL Server на виртуальной машине Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: jroth
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/07/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 7c3950b1aeae2b4e90b1ae4acfb9439c2cb159ca
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105026008"
---
# <a name="configure-a-dnn-listener-for-an-availability-group"></a>Настройка прослушивателя DNN для группы доступности
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

С SQL Server на виртуальных машинах Azure имя распределенной сети (DNN) направляет трафик в соответствующий кластеризованный ресурс. Он предоставляет более простой способ подключения к Always On группе доступности (AG), чем прослушиватель имени виртуальной сети (VNN) без необходимости в Azure Load Balancer.

В этой статье описывается, как настроить прослушиватель DNN, чтобы заменить прослушиватель VNN и направить трафик в группу доступности с SQL Server на виртуальных машинах Azure для обеспечения высокой доступности и аварийного восстановления (HADR).

Функция прослушивателя DNN в настоящее время доступна только начиная с SQL Server 2019 CU8 на Windows Server 2016 и более поздних версиях.

Для альтернативного варианта подключения рассмотрим [VNN прослушиватель и Azure Load Balancer](availability-group-vnn-azure-load-balancer-configure.md) .

## <a name="overview"></a>Обзор

Прослушиватель распределенного сетевого имени (DNN) заменяет традиционный прослушиватель группы доступности имени виртуальной сети (VNN) при использовании с [группы доступности Always on на виртуальных машинах SQL Server](availability-group-overview.md). Это отрицательно потребует Azure Load Balancer для маршрутизации трафика, упрощения развертывания, обслуживания и повышения отработки отказа.

Разверните прослушиватель DNN вместо существующего прослушивателя VNN или используйте их параллельно, чтобы у групп доступности было две раздельные точки подключения, из которых одна использует имя (и порт, если он отличается от стандартного) прослушивателя VNN, а вторая — имя и порт прослушивателя DNN.

> [!CAUTION]
> Поведение маршрутизации при использовании DNN отличается при использовании VNN. Не используйте порт 1433. Дополнительные сведения см. в разделе "вопросы и [порты](#port-considerations) " Далее в этой статье.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этой статье, необходимо следующее:

- SQL Server 2019 в CU8 или более поздней версии, на Windows Server 2016 и более поздних версиях
- Принято решение о том, что имя распределенной сети является подходящим [вариантом подключения для вашего решения HADR](hadr-cluster-best-practices.md#connectivity).
- Настроена [Группа доступности Always on](availability-group-overview.md). 
- Установлена последняя версия [PowerShell](/powershell/azure/install-az-ps). 
- Идентифицирует уникальный порт, который будет использоваться для прослушивателя DNN. Порт, используемый для прослушивателя DNN, должен быть уникальным для всех реплик группы доступности или экземпляра отказоустойчивого кластера.  Другие подключения не могут совместно использовать один и тот же порт.



## <a name="create-script"></a>Создать скрипт

С помощью PowerShell создайте ресурс с именем распределенной сети (DNN) и свяжите его с группой доступности.

Для этого выполните следующие действия.

1. Откройте текстовый редактор, например Блокнот.
1. Скопируйте и вставьте следующий скрипт.

   ```powershell
   param (
      [Parameter(Mandatory=$true)][string]$Ag,
      [Parameter(Mandatory=$true)][string]$Dns,
      [Parameter(Mandatory=$true)][string]$Port
   )
   
   Write-Host "Add a DNN listener for availability group $Ag with DNS name $Dns and port $Port"
   
   $ErrorActionPreference = "Stop"
   
   # create the DNN resource with the port as the resource name
   Add-ClusterResource -Name $Port -ResourceType "Distributed Network Name" -Group $Ag 
   
   # set the DNS name of the DNN resource
   Get-ClusterResource -Name $Port | Set-ClusterParameter -Name DnsName -Value $Dns 
   
   # start the DNN resource
   Start-ClusterResource -Name $Port
   
   
   $Dep = Get-ClusterResourceDependency -Resource $Ag
   if ( $Dep.DependencyExpression -match '\s*\((.*)\)\s*' )
   {
   $DepStr = "$($Matches.1) or [$Port]"
   }
   else
   {
   $DepStr = "[$Port]"
   }
   
   Write-Host "$DepStr"
   
   # add the Dependency from availability group resource to the DNN resource
   Set-ClusterResourceDependency -Resource $Ag -Dependency "$DepStr"
   
   
   #bounce the AG resource
   Stop-ClusterResource -Name $Ag
   Start-ClusterResource -Name $Ag
   ```

1. Сохраните скрипт в виде `.ps1` файла, например `add_dnn_listener.ps1` .

## <a name="execute-script"></a>Выполнить сценарий

Чтобы создать прослушиватель DNN, выполните скрипт, передав параметры для имени группы доступности, имени прослушивателя и порта.

Например, если имя группы доступности `ag1` , имя прослушивателя `dnnlsnr` и порт прослушивателя в качестве `6789` , выполните следующие действия.

1. Откройте средство интерфейса командной строки, например Командная строка или PowerShell.
1. Перейдите к месту сохранения `.ps1` скрипта, например к:\документс.
1. Выполните скрипт: ```add_dnn_listener.ps1 <ag name> <listener-name> <listener port>``` . Пример:

   ```console
   c:\Documents> add_dnn_listener.ps1 ag1 dnnlsnr 6789
   ```

## <a name="verify-listener"></a>Проверка прослушивателя

Чтобы убедиться, что прослушиватель DNN успешно создан, используйте либо SQL Server Management Studio, либо Transact-SQL.

### <a name="sql-server-management-studio"></a>SQL Server Management Studio.

Разверните узел **прослушиватели группы доступности** в [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) , чтобы просмотреть прослушиватель DNN:

:::image type="content" source="media/availability-group-distributed-network-name-dnn-listener-configure/dnn-listener-in-ssms.png" alt-text="Просмотр прослушивателя DNN в разделе Прослушиватели группы доступности в SQL Server Management Studio (SSMS)":::

### <a name="transact-sql"></a>Transact-SQL

Чтобы просмотреть состояние прослушивателя DNN, используйте Transact-SQL:

```sql
SELECT * FROM SYS.AVAILABILITY_GROUP_LISTENERS
```

Значение `1` для `is_distributed_network_name` указывает, что прослушиватель является прослушивателем имени распределенной сети (DNN):

:::image type="content" source="media/availability-group-distributed-network-name-dnn-listener-configure/dnn-listener-tsql.png" alt-text="Используйте sys.availability_group_listeners для обнаружения прослушивателей DNN, имеющих значение 1 в is_distributed_network_name":::

## <a name="update-connection-string"></a>Обновление строки подключения

Обновите строки подключения для приложений, чтобы они подключались к прослушивателю DNN. Чтобы обеспечить быстрое подключение при отработке отказа, добавьте `MultiSubnetFailover=True` в строку подключения, если клиент SQL его поддерживает.

## <a name="test-failover"></a>Тестовая отработка отказа

Выполните тестовую отработку отказа группы доступности, чтобы обеспечить функционирование.

Чтобы протестировать отработку отказа, выполните следующие действия.

1. Подключитесь к прослушивателю DNN или к одной из реплик с помощью [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).
1. Разверните **Always on группа доступности** в **обозревателе объектов**.
1. Щелкните правой кнопкой мыши группу доступности и выберите пункт **отработка отказа** , чтобы открыть **Мастер отработки отказа**.
1. Следуйте инструкциям на экране, чтобы выбрать цель отработки отказа и выполнить сбой группы доступности во вторичной реплике.
1. Убедитесь, что база данных находится в состоянии SYNCHRONIZED в новой первичной реплике.
1. Используемых Восстановить подключение к исходной основной или другой вторичной реплике.

## <a name="test-connectivity"></a>Проверка подключения

Проверьте подключение к прослушивателю DNN, выполнив следующие действия.

1. Откройте [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).
1. Подключитесь к прослушивателю DNN.
1. Откройте новое окно запроса и проверьте, к какой реплике вы подключены, выполнив `SELECT @@SERVERNAME` .
1. Сбой группы доступности в другой реплике.
1. По истечении разумного времени выполните команду, `SELECT @@SERVERNAME` чтобы подтвердить, что ваша группа доступности теперь размещена в другой реплике.

## <a name="limitations"></a>Ограничения

- В настоящее время прослушиватель DNN для группы доступности поддерживается только для SQL Server 2019 CU8 и более поздних версий на Windows Server 2016 и более поздних версиях. 
- Прослушиватели DNN **должны** быть настроены с помощью уникального порта.  Порт не может использоваться совместно с любым другим подключением к любой реплике.
- При работе с другими SQL Serverными функциями и группой доступности могут возникнуть дополнительные соображения, если вы используете DNN. Дополнительные сведения см. в статье группа [доступности с DNN взаимодействием](availability-group-dnn-interoperability.md). 

## <a name="port-considerations"></a>Рекомендации по портам

Прослушиватели DNN предназначены для прослушивания всех IP-адресов, но на конкретном уникальном порте. Запись DNS для имени прослушивателя должна разрешаться в адреса всех реплик в группе доступности. Это выполняется автоматически с помощью сценария PowerShell, приведенного в разделе [Создание скрипта](#create-script) . Так как прослушиватели DNN принимают подключения на всех IP-адресах, очень важно, чтобы порт прослушивателя был уникальным и не используется другими репликами в группе доступности. Поскольку SQL Server всегда прослушивает порт 1433, напрямую или через службу обозреватель SQL, порт 1433 не может использоваться ни для одного прослушивателя DNN.

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения о SQL Serverии функций HADR в Azure см. в разделе [группы доступности](availability-group-overview.md) и [экземпляр отказоустойчивого кластера](failover-cluster-instance-overview.md). Вы также [можете ознакомиться с рекомендациями](hadr-cluster-best-practices.md) по настройке среды для обеспечения высокого уровня доступности и аварийного восстановления. 
