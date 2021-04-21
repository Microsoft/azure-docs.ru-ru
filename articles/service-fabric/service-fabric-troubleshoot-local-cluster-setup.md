---
title: Устранение неполадок в настройке локального кластера Azure Service Fabric
description: В этой статье описываются рекомендации по устранению неполадок с кластером локальной разработки.
ms.topic: conceptual
ms.date: 02/23/2018
ms.openlocfilehash: 20948cd1626c02d73fb6e9ef096b552bbab627fb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96575913"
---
# <a name="troubleshoot-your-local-development-cluster-setup"></a>Устранение неполадок в работе кластера локальной разработки
Если у вас возникли проблемы при взаимодействии с кластером локальной разработки Azure Service Fabric, ознакомьтесь со следующими возможностями для решения.

## <a name="cluster-setup-failures"></a>Ошибки настройки кластера
### <a name="cannot-clean-up-service-fabric-logs"></a>Не удается очистить журналы Service Fabric
#### <a name="problem"></a>Проблема
При выполнении скрипта DevClusterSetup появляется следующее сообщение об ошибке:

```output
Cannot clean up C:\SfDevCluster\Log fully as references are likely being held to items in it. Please remove those and run this script again.
At line:1 char:1 + .\DevClusterSetup.ps1
+ ~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,DevClusterSetup.ps1
```

#### <a name="solution"></a>Решение
Закройте текущее и откройте новое окно PowerShell от имени администратора. Теперь можно успешно запустить скрипт.

## <a name="cluster-connection-failures"></a>Ошибки подключения к кластеру

### <a name="type-initialization-exception"></a>Исключение типа инициализации
#### <a name="problem"></a>Проблема
При подключении к кластеру в PowerShell отображается ошибка TypeInitializationException для System.Fabric.Common.AppTrace.

#### <a name="solution"></a>Решение
При установке была неправильно настроена переменная пути. Выйдите из Windows и снова выполните вход. Путь обновится.

### <a name="cluster-connection-fails-with-object-is-closed"></a>Сбой подключения к кластеру с сообщением об ошибке "Объект закрыт"
#### <a name="problem"></a>Проблема
Вызов Connect-ServiceFabricCluster завершается со следующей ошибкой:

```output
Connect-ServiceFabricCluster : The object is closed.
At line:1 char:1
+ Connect-ServiceFabricCluster
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : InvalidOperation: (:) [Connect-ServiceFabricCluster], FabricObjectClosedException
+ FullyQualifiedErrorId : CreateClusterConnectionErrorId,Microsoft.ServiceFabric.Powershell.ConnectCluster
```

#### <a name="solution"></a>Решение
Закройте текущее и откройте новое окно PowerShell от имени администратора.

### <a name="fabric-connection-denied-exception"></a>Исключение Fabric Connection Denied
#### <a name="problem"></a>Проблема
При отладке в Visual Studio отображается ошибка FabricConnectionDeniedException.

#### <a name="solution"></a>Решение
Эта ошибка обычно возникает при попытке вручную запустить хост-процесс службы.

Убедитесь, что в решении нет проектов служб, настроенных в качестве запускаемых проектов. В качестве запускаемых проектов можно настраивать только проекты приложений Service Fabric.

> [!TIP]
> Если после завершения программы установки у локального кластера наблюдается нетипичное поведение, такой кластер можно сбросить с помощью приложения панели задач диспетчера локального кластера. Будет удален существующий кластер и настроен новый. Учтите, что все развернутые приложения и связанные данные будут удалены.
> 
> 

## <a name="next-steps"></a>Дальнейшие действия
* [Обзор и диагностика кластера с помощью системных отчетов о работоспособности](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)
* [Визуализация кластера с помощью обозревателя Service Fabric](service-fabric-visualizing-your-cluster.md)

