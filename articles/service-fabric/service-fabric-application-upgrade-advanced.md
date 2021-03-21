---
title: Разделы, посвященные расширенному обновлению приложений
description: В этой статье рассматриваются некоторые дополнительные темы, относящиеся к обновлению приложения Service Fabric.
ms.topic: conceptual
ms.date: 03/11/2020
ms.openlocfilehash: 6604300328f2d243077ba341a9028221438dce9d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98792054"
---
# <a name="service-fabric-application-upgrade-advanced-topics"></a>Обновление приложения Service Fabric: дополнительные разделы

## <a name="add-or-remove-service-types-during-an-application-upgrade"></a>Добавление или удаление типов служб во время обновления приложения

Если в приложение, которое опубликовано как обновление, добавляется новый тип службы, то он добавляется в развернутое приложение. Подобное обновление не влияет на какие-либо экземпляры служб, которые уже были частью приложения. Однако чтобы активировать новый тип службы, необходимо создать экземпляр типа службы, который был добавлен (см. статью о командлете [New-ServiceFabricService](/powershell/module/servicefabric/new-servicefabricservice)).

Точно так же типы служб могут быть удалены из приложения. Но в этом случае перед продолжением обновления все текущие экземпляры удаляемой службы должны быть удалены (см. статью о командлете [Remove-ServiceFabricService](/powershell/module/servicefabric/remove-servicefabricservice)).

## <a name="avoid-connection-drops-during-stateless-service-planned-downtime"></a>Предотвращение неразрывов подключений во время запланированного простоя службы без отслеживания состояния

Для запланированных простоев экземпляров без отслеживания состояния, таких как обновление приложения или кластера или деактивация узла, соединения могут быть удалены, так как предоставленная конечная точка удаляется после отключения экземпляра, что приводит к принудительному замыканию соединения.

Чтобы избежать этого, настройте функцию *рекуестдраин* , добавив в конфигурации службы *Длительность задержки закрытия экземпляра* , чтобы позволить существующим запросам в кластере выполнять сток на предоставленных конечных точках. Это достигается из-за того, что конечная точка, объявленная экземпляром без отслеживания состояния, удаляется *до* начала задержки перед закрытием экземпляра. Эта задержка позволяет корректно завершать существующие запросы до того, как экземпляр будет действительно остановлен. Клиенты получают уведомления об изменении конечной точки с помощью функции обратного вызова во время запуска задержки, чтобы они могли повторно разрешить конечную точку и избежать отправки новых запросов к экземпляру, который не работает. Эти запросы могут исходить от клиентов, использующих [обратный прокси-сервер](./service-fabric-reverseproxy.md) , или через API разрешения конечных точек службы с моделью уведомлений ([сервиценотификатионфилтердескриптион](/dotnet/api/system.fabric.description.servicenotificationfilterdescription)) для обновления конечных точек.

### <a name="service-configuration"></a>Конфигурация службы

Существует несколько способов настройки задержки на стороне службы.

 * **При создании новой службы** укажите `-InstanceCloseDelayDuration` :

    ```powershell
    New-ServiceFabricService -Stateless [-ServiceName] <Uri> -InstanceCloseDelayDuration <TimeSpan>
    ```

 * **При определении службы в разделе по умолчанию манифеста приложения** назначьте `InstanceCloseDelayDurationSeconds` свойство:

    ```xml
          <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]" InstanceCloseDelayDurationSeconds="15">
              <SingletonPartition />
          </StatelessService>
    ```

 * **При обновлении существующей службы** укажите `-InstanceCloseDelayDuration` :

    ```powershell
    Update-ServiceFabricService [-Stateless] [-ServiceName] <Uri> [-InstanceCloseDelayDuration <TimeSpan>]`
    ```

 * **При создании или обновлении существующей службы с помощью шаблона ARM** укажите `InstanceCloseDelayDuration` значение (минимальная поддерживаемая версия API: 2019-11-01-Preview):

    ```ARM template to define InstanceCloseDelayDuration of 30seconds
    {
      "apiVersion": "2019-11-01-preview",
      "type": "Microsoft.ServiceFabric/clusters/applications/services",
      "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
      "location": "[variables('clusterLocation')]",
      "dependsOn": [
        "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applications/', parameters('applicationName'))]"
      ],
      "properties": {
        "provisioningState": "Default",
        "serviceKind": "Stateless",
        "serviceTypeName": "[parameters('serviceTypeName')]",
        "instanceCount": "-1",
        "partitionDescription": {
          "partitionScheme": "Singleton"
        },
        "serviceLoadMetrics": [],
        "servicePlacementPolicies": [],
        "defaultMoveCost": "",
        "instanceCloseDelayDuration": "00:00:30.0"
      }
    }
    ```

### <a name="client-configuration"></a>Настройка клиента

Чтобы получить уведомление об изменении конечной точки, клиенты должны зарегистрировать обратный вызов, см. [сервиценотификатионфилтердескриптион](/dotnet/api/system.fabric.description.servicenotificationfilterdescription).
Уведомление об изменении указывает на то, что конечные точки изменились, клиент должен повторно разрешить конечные точки, а не использовать конечные точки, которые больше не объявлены, так как они скоро выйдут.

### <a name="optional-upgrade-overrides"></a>Необязательные переопределения обновления

Помимо задания длительности задержки по умолчанию для каждой службы, можно также переопределить задержку при обновлении приложения или кластера с помощью того же `InstanceCloseDelayDurationSec` параметра ():

```powershell
Start-ServiceFabricApplicationUpgrade [-ApplicationName] <Uri> [-ApplicationTypeVersion] <String> [-InstanceCloseDelayDurationSec <UInt32>]

Start-ServiceFabricClusterUpgrade [-CodePackageVersion] <String> [-ClusterManifestVersion] <String> [-InstanceCloseDelayDurationSec <UInt32>]
```

Переопределенная длительность задержки применяется только к вызванному экземпляру обновления и не изменяет индивидуальные конфигурации задержки службы. Например, с его помощью можно задать задержку для `0` пропуска всех предварительно настроенных задержек обновления.

> [!NOTE]
> * Параметры для запросов на сток не позволят подсистеме балансировки нагрузки Azure отправлять новые запросы конечным точкам, которые находятся в процессе очистки.
> * Механизм разрешения на основе жалоб не приведет к корректному стоку запросов, так как он активирует разрешение службы после сбоя. Как было сказано ранее, это следует улучшить, чтобы подписываться на уведомления об изменениях конечной точки с помощью [сервиценотификатионфилтердескриптион](/dotnet/api/system.fabric.description.servicenotificationfilterdescription).
> * Параметры не учитываются, если обновление не оказывает влияния, т. е. когда реплики не будут выдаваться во время обновления.
>
>

> [!NOTE]
> Эту функцию можно настроить в существующих службах с помощью командлета Update-ServiceFabricService или шаблона ARM, как упоминалось выше, если используется версия кода кластера 7.1.XXX или выше.
>
>

## <a name="manual-upgrade-mode"></a>Ручной режим обновления

> [!NOTE]
> Для всех обновлений Service Fabric рекомендуется использовать *отслеживаемый* (Monitored) режим обновления.
> Режим *UnmonitoredManual* (неотслеживаемый, ручной) следует использовать только для сбойных или отложенных обновлений. 
>
>

В *отслеживаемом* режиме Service Fabric применяет политики работоспособности, чтобы гарантировать, что в процессе обновления приложение находится в работоспособном состоянии. Если политики работоспособности нарушены, обновление приостанавливается или автоматически откатывается. Это зависит от указанного атрибута *FailureAction*.

В режиме *UnmonitoredManual* администратор приложения полностью контролирует процесс обновления. Этот режим удобен, когда применяются настраиваемые политики оценки работоспособности или выполняется нестандартное обновление, чтобы полностью обойти проверки работоспособности (например, приложение уже потеряло некоторые данные). Обновление, выполняющееся в этом режиме, самостоятельно приостанавливается после применения обновлений к каждому домену обновления. Его необходимо возобновить с помощью командлета [Resume-ServiceFabricApplicationUpgrade](/powershell/module/servicefabric/resume-servicefabricapplicationupgrade). Когда обновление приостановлено и готово к возобновлению, его состояние изменится на *RollforwardPending* (Ожидание наката) (см. статью о команде [ApplicationUpgradeState Enum](/dotnet/api/system.fabric.applicationupgradestate)).

Наконец, режим *UnmonitoredAuto* (неотслеживаемый, автоматический) подходит для выполнения быстрых итераций обновления во время разработки или тестирования служб, так как пользовательский ввод не требуется, а политики работоспособности приложения не оцениваются.

## <a name="upgrade-with-a-diff-package"></a>Обновление при помощи пакета diff

Обновления могут выполняться путем подготовки пакетов разностных изменений (вместо полных пакетов приложений), содержащих только обновленные пакеты кода, конфигурации или данных, а также полные манифесты приложений и служб. Для первичной установки приложений в кластер необходимы полные пакеты приложений. При последующих обновлениях могут использоваться либо полные пакеты приложений, либо пакеты разностных изменений (diff).  

Любая ссылка в манифесте приложения или манифесте служб пакета diff, которую невозможно найти в пакете приложения, автоматически заменяется текущей подготовленной версией.

Использование пакета diff оптимально в следующих случаях:

* Когда используется объемный пакет приложения, который ссылается на несколько файлов манифеста службы или на несколько пакетов кода, пакетов конфигурации или пакетов данных.
* Когда система развертывания формирует структуру сборки напрямую в ходе создания сборки приложения. В этом случае, хотя в самом коде ничего не изменилось, новые сборки будут обладать другой контрольной суммой. Использование полного пакета приложения потребует обновления версии всех пакетов кода. При использовании пакета diff вы предоставляете только измененные файлы и файлы манифеста, версия которых изменилась.

При обновлении приложения с помощью Visual Studio пакет diff публикуется автоматически. Чтобы создать пакет diff вручную, необходимо обновить манифест приложения и манифесты служб, но только измененные пакеты должны быть включены в финальный пакет приложения.

К примеру, начнем со следующего приложения (номера версий приведены для простоты понимания):

```text
app1           1.0.0
  service1     1.0.0
    code       1.0.0
    config     1.0.0
  service2     1.0.0
    code       1.0.0
    config     1.0.0
```

Предположим, что необходимо обновить только код пакета service1, использующего пакет diff. Обновленное приложение включает следующие изменения версии:

```text
app1           2.0.0      <-- new version
  service1     2.0.0      <-- new version
    code       2.0.0      <-- new version
    config     1.0.0
  service2     1.0.0
    code       1.0.0
    config     1.0.0
```

В этом случае вы обновляете манифест приложения до версии 2.0.0 и манифест службы для service1 для отражения обновления пакета кода. Структура папок для пакета приложения будет выглядеть следующим образом.

```text
app1/
  service1/
    code/
```

Другими словами, создайте полный пакет приложения, а затем удалите все папки пакета кода, конфигурации или данных, для которых версия не изменилась.

## <a name="upgrade-application-parameters-independently-of-version"></a>Обновить параметры приложения независимо от версии

Иногда желательно изменить параметры Service Fabric приложения, не изменяя версию манифеста. Это можно сделать с помощью флага **-аппликатионпараметер** с командлетом **Start-ServiceFabricApplicationUpgrade** Azure Service Fabric PowerShell. Предположим, что Service Fabric приложение со следующими свойствами:

```PowerShell
PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/Application1

ApplicationName        : fabric:/Application1
ApplicationTypeName    : Application1Type
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Ok
ApplicationParameters  : { "ImportantParameter" = "1"; "NewParameter" = "testBefore" }
```

Теперь обновите приложение с помощью командлета **Start-ServiceFabricApplicationUpgrade** . В этом примере показано отслеживаемое обновление, но можно также использовать Неотслеживаемое обновление. Полное описание флагов, принятых этим командлетом, см. в [справочнике по модулю PowerShell для Service Fabric Azure](/powershell/module/servicefabric/start-servicefabricapplicationupgrade#parameters) .

```PowerShell
PS C:\> $appParams = @{ "ImportantParameter" = "2"; "NewParameter" = "testAfter"}

PS C:\> Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/Application1 -ApplicationTypeVers
ion 1.0.0 -ApplicationParameter $appParams -Monitored

```

После обновления убедитесь, что приложение имеет обновленные параметры и ту же версию:

```PowerShell
PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/Application1

ApplicationName        : fabric:/Application1
ApplicationTypeName    : Application1Type
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Ok
ApplicationParameters  : { "ImportantParameter" = "2"; "NewParameter" = "testAfter" }
```

## <a name="roll-back-application-upgrades"></a>Откат обновлений приложения

Хотя обновления можно выполнить в одном из трех режимов (*Monitored*, *UnmonitoredAuto* или *UnmonitoredManual*), откатить их можно только в режиме *UnmonitoredAuto* или *UnmonitoredManual*. Откат в режиме *UnmonitoredAuto* работает так же, как накат. Единственное исключение: другое значение по умолчанию *UpgradeReplicaSetCheckTimeout* (см. статью [Параметры обновления приложений](service-fabric-application-upgrade-parameters.md)). Откат в режиме *UnmonitoredManual* работает так же, как и накат. Он приостанавливается после завершения работы с каждым доменом обновления. Возобновить его можно с помощью командлета [ Resume-ServiceFabricApplicationUpgrade](/powershell/module/servicefabric/resume-servicefabricapplicationupgrade).

Откат может запускаться автоматически, если нарушены политики работоспособности обновления в режиме *Monitored* с указанным для *отката* атрибутом *FailureAction*, (см. статью [Параметры обновления приложения](service-fabric-application-upgrade-parameters.md)) или если используется командлет [Start-ServiceFabricApplicationRollback](/powershell/module/servicefabric/start-servicefabricapplicationrollback).

Во время отката все еще можно изменить значение *UpgradeReplicaSetCheckTimeout*. Режим также можно изменить в любое время с помощью командлета [Update-ServiceFabricApplicationUpgrade](/powershell/module/servicefabric/update-servicefabricapplicationupgrade).

## <a name="next-steps"></a>Дальнейшие действия
[Руководство по обновлению приложений Service Fabric с помощью Visual Studio](service-fabric-application-upgrade-tutorial.md) поможет вам выполнить поэтапное обновление приложения с помощью Visual Studio.

[Обновление приложения с помощью PowerShell](service-fabric-application-upgrade-tutorial-powershell.md) поможет вам выполнить обновление приложения с помощью PowerShell.

Управление обновлениями приложения осуществляется с помощью [параметров обновления](service-fabric-application-upgrade-parameters.md).

Изучите, как использовать [сериализацию данных](service-fabric-application-upgrade-data-serialization.md), чтобы обеспечить совместимость обновлений приложений.

Решения распространенных проблем при обновлении приложений см. в статье [Устранение неполадок при обновлении приложений](service-fabric-application-upgrade-troubleshooting.md).
