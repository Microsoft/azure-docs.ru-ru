---
title: Настройка обновления приложения Service Fabric
description: Узнайте, как настроить параметры обновления приложения Service Fabric с помощью Microsoft Visual Studio.
ms.topic: conceptual
ms.date: 06/29/2017
ms.openlocfilehash: 11df0387db07669ebacdba2e3d850e32903da133
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96573686"
---
# <a name="configure-the-upgrade-of-a-service-fabric-application-in-visual-studio"></a>Настройка обновления приложения Service Fabric в Visual Studio
Средства Visual Studio для Azure Service Fabric обеспечивают поддержку обновления для публикации на локальных и удаленных кластерах. Существуют три сценария, в которых следует обновить приложение до более новой версии, а не заменить его, во время тестирования и отладки:

* данные приложения не будут потеряны во время обновления;
* достигается высокий уровень доступности, так как не будет перерывов в доступности службы по время обновления, если по доменам обновления распределено достаточное количество экземпляров службы.
* Тесты для приложения могут запускаться во время его обновления.

## <a name="parameters-needed-to-upgrade"></a>Параметры, необходимые для обновления
Существует два типа развертывания: обычное или обновление. При обычном развертывании стираются все предыдущие сведения о развертывании и данные в кластере, а при обновлении они сохраняются. При обновлении приложения Service Fabric в Visual Studio необходимо указать параметры обновления приложения и политики проверки работоспособности. Параметры обновления приложения помогают управлять обновлением, а политики проверки работоспособности определяют, было обновление успешным или нет. Дополнительные сведения см. в статье [Параметры обновления приложений](service-fabric-application-upgrade-parameters.md).

Существует три режима обновления: *отслеживаемое*, *неотслеживаемое автоматическое* и *неотслеживаемое ручное*.

* При отслеживаемом обновлении процесс обновления и проверка работоспособности приложения автоматизируются.
* При неотслеживаемом автоматическом обновлении процесс обновления автоматизируется, но проверка работоспособности приложения пропускается.
* При выполнении неотслеживаемого ручного обновления необходимо вручную обновить каждый домен обновления.

Для каждого режима обновления необходим различный набор параметров. Дополнительные сведения о доступных параметрах обновления см. в статье [Параметры обновления приложений](service-fabric-application-upgrade-parameters.md).

## <a name="upgrade-a-service-fabric-application-in-visual-studio"></a>Обновление приложения Service Fabric в Visual Studio
При использовании средств Service Fabric Visual Studio для обновления приложения Service Fabric можно указать, что процесс публикации будет подразумевать обновление, а не обычное развертывание, установив флажок **Обновить приложение** .

### <a name="to-configure-the-upgrade-parameters"></a>Настройка параметров обновления
1. Нажмите кнопку **Параметры** рядом с флажком. Откроется диалоговое окно **Изменение параметров обновления** . Диалоговое окно **Изменение параметров обновления** поддерживает отслеживаемое, неотслеживаемое автоматическое и ручное обновления.
2. Выберите режим обновления, который хотите использовать, и заполните сетку параметров.

    Каждый параметр имеет значения по умолчанию. Необязательный параметр *DefaultServiceTypeHealthPolicy* принимает входную хэш-таблицу. Ниже приведен пример формата входной хэш-таблицы для *DefaultServiceTypeHealthPolicy*:

    ```
    @{ ConsiderWarningAsError = "false"; MaxPercentUnhealthyDeployedApplications = 0; MaxPercentUnhealthyServices = 0; MaxPercentUnhealthyPartitionsPerService = 0; MaxPercentUnhealthyReplicasPerPartition = 0 }
    ```

    *ServiceTypeHealthPolicyMap* — еще один необязательный параметр, который принимает входную хэш-таблицу в следующем формате:

    ```    
    @ {"ServiceTypeName" : "MaxPercentUnhealthyPartitionsPerService,MaxPercentUnhealthyReplicasPerPartition,MaxPercentUnhealthyServices"}
    ```

    Ниже приведен пример из реальной жизни:

    ```
    @{ "ServiceTypeName01" = "5,10,5"; "ServiceTypeName02" = "5,5,5" }
    ```
3. При выборе неотслеживаемого автоматического обновления необходимо будет вручную запустить консоль PowerShell для продолжения и завершения процесса обновления. Чтобы узнать, как работает ручное обновление, обратитесь к разделу [Обновление приложения Service Fabric: дополнительные разделы](service-fabric-application-upgrade-advanced.md) .

## <a name="upgrade-an-application-by-using-powershell"></a>Обновление приложения с помощью PowerShell
Обновить приложение Service Fabric можно с помощью командлетов PowerShell. Подробные сведения см. в [руководстве по обновлению приложений Service Fabric](service-fabric-application-upgrade-tutorial.md) и статье, посвященной командлету [Start-ServiceFabricApplicationUpgrade](/powershell/module/servicefabric/start-servicefabricapplicationupgrade).

## <a name="specify-a-health-check-policy-in-the-application-manifest-file"></a>Задание политики проверки работоспособности в файле манифеста приложения
Каждая служба в приложении Service Fabric может иметь собственные параметры политики работоспособности, которые переопределяют значения по умолчанию. Эти значения параметров можно указать в файле манифеста приложения.

В следующем примере показано, как применить уникальную политику проверки работоспособности для каждой службы в манифесте приложения.

```xml
<Policies>
    <HealthPolicy ConsiderWarningAsError="false" MaxPercentUnhealthyDeployedApplications="20">
        <DefaultServiceTypeHealthPolicy MaxPercentUnhealthyServices="20"               
                MaxPercentUnhealthyPartitionsPerService="20"
                MaxPercentUnhealthyReplicasPerPartition="20" />
        <ServiceTypeHealthPolicy ServiceTypeName="ServiceTypeName1"
                MaxPercentUnhealthyServices="20"
                MaxPercentUnhealthyPartitionsPerService="20"
                MaxPercentUnhealthyReplicasPerPartition="20" />      
    </HealthPolicy>
</Policies>
```
## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения см. в статье об [обновлении приложения с помощью Visual Studio](service-fabric-application-upgrade-tutorial.md).
