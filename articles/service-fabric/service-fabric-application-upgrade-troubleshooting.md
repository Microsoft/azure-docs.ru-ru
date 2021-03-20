---
title: Устранение неполадок при обновлении приложений
description: В этой статье рассматриваются некоторые общие проблемы, связанные с обновлением приложений структуры служб, и способы их устранения.
ms.topic: conceptual
ms.date: 2/23/2018
ms.openlocfilehash: d462f2c2482e0fbb4d252967754a9675ed362674
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96009353"
---
# <a name="troubleshoot-application-upgrades"></a>Устранение неполадок при обновлениях приложений

В этой статье рассматриваются некоторые общие проблемы, связанные с обновлением приложений Azure Service Fabric, и способы их устранения.

## <a name="troubleshoot-a-failed-application-upgrade"></a>Устранение неполадок при неудачном обновлении приложения

При сбое обновления выходные данные команды **Get-ServiceFabricApplicationUpgrade** содержат дополнительные сведения для отладки сбоя.  В следующем списке указано, как можно использовать эти дополнительные сведения.

1. Определение типа ошибки.
2. Определение причины ошибки.
3. Изоляция одного или нескольких отказавших компонентов для дальнейшего исследования.

Эти сведения будут доступны сразу же после того, как Service Fabric обнаружит сбой, независимо от того, будет ли в качестве действия **FailureAction** настроен откат или приостановка обновления.

### <a name="identify-the-failure-type"></a>Определение типа ошибки

В выходных данных **Get-ServiceFabricApplicationUpgrade****FailureTimestampUtc** определяет временную метку (по всеобщему скоординированному времени), при которой сбой обновления был обнаружен Service Fabric и было запущено действие **FailureAction**. **FailureReason** указывает на одну из трех потенциальных высокоуровневых причин сбоя:

1. UpgradeDomainTimeout указывает, что для определенного домена обновление занимает слишком много времени, что привело к истечению срока **UpgradeDomainTimeout** .
2. OverallUpgradeTimeout указывает, что обновление в целом заняло слишком много времени, что привело к истечению срока **UpgradeTimeout** .
3. HealthCheck указывает, что после обновления домена обновления приложение осталось неработоспособным в соответствии с конкретными политиками работоспособности, что привело к истечению времени **HealthCheckRetryTimeout** .

Эти записи отображаются в выходных данных только при сбое обновления и выполнении отката. Дальнейшие сведения отображаются в зависимости от типа сбоя.

### <a name="investigate-upgrade-timeouts"></a>Исследование времени ожидания обновления

Сбои, связанные с истечением времени ожидания, чаще всего вызваны проблемами с доступностью служб. Выходные данные, приведенные ниже в этом абзаце, характерны для обновлений, в которых реплики или экземпляры служб не удается запустить из-за новой версии кода. Поле **UpgradeDomainProgressAtFailure** выполняет моментальный снимок любого ожидаемого обновления в момент сбоя.

```powershell
Get-ServiceFabricApplicationUpgrade fabric:/DemoApp
```

```Output
ApplicationName                : fabric:/DemoApp
ApplicationTypeName            : DemoAppType
TargetApplicationTypeVersion   : v2
ApplicationParameters          : {}
StartTimestampUtc              : 4/14/2015 9:26:38 PM
FailureTimestampUtc            : 4/14/2015 9:27:05 PM
FailureReason                  : UpgradeDomainTimeout
UpgradeDomainProgressAtFailure : MYUD1

                                 NodeName            : Node4
                                 UpgradePhase        : PostUpgradeSafetyCheck
                                 PendingSafetyChecks :
                                     WaitForPrimaryPlacement - PartitionId: 744c8d9f-1d26-417e-a60e-cd48f5c098f0

                                 NodeName            : Node1
                                 UpgradePhase        : PostUpgradeSafetyCheck
                                 PendingSafetyChecks :
                                     WaitForPrimaryPlacement - PartitionId: 4b43f4d8-b26b-424e-9307-7a7a62e79750
UpgradeState                   : RollingBackCompleted
UpgradeDuration                : 00:00:46
CurrentUpgradeDomainDuration   : 00:00:00
NextUpgradeDomain              :
UpgradeDomainsStatus           : { "MYUD1" = "Completed";
                                 "MYUD2" = "Completed";
                                 "MYUD3" = "Completed" }
UpgradeKind                    : Rolling
RollingUpgradeMode             : UnmonitoredAuto
ForceRestart                   : False
UpgradeReplicaSetCheckTimeout  : 00:00:00
```

В этом примере произошел сбой обновления в домене обновления *MYUD1*, и оно остановилось на двух секциях (*744c8d9f-1d26-417e-a60e-cd48f5c098f0* и *4b43f4d8-b26b-424e-9307-7a7a62e79750*). Это произошло, так как среде выполнения не удалось разместить первичные реплики (*WaitForPrimaryPlacement*) на целевых узлах *Node1* и *Node4*.

Команда **Get ServiceFabricNode** может использоваться для проверки нахождения этих двух узлов в домене обновления *MYUD1*. Параметр *UpgradePhase* имеет значение *PostUpgradeSafetyCheck*, что означает, что эти проверки безопасности выполняются после обновления всех узлов в домене обновления. Все эти сведения вместе указывают на потенциальную проблему в новой версии кода приложения. Наиболее обычные неполадки представляют собой явные ошибки службы или ссылки на части основного кода.

Если параметр *UpgradePhase* имеет значение *PreUpgradeSafetyCheck*, то это значит, что при подготовке домена обновления перед началом самого обновления были обнаружены неполадки. Наиболее распространенные неполадки в этом случае представляют собой явные ошибки службы или отклонение от путей основного кода.

Текущее состояние **UpgradeState** имеет значение *RollingBackCompleted*, поэтому исходное обновление могло быть выполнено с использованием действия **FailureAction**, соответствующего откату, когда при сбое выполняется автоматический откат. Если исходное обновление было выполнено с использованием осуществляемого вручную действия **FailureAction**, то обновление вместо этого будет находиться в отложенном состоянии для выполнения динамической отладки приложения.

В редких случаях поле **UpgradeDomainProgressAtFailure** может быть пустым, если общее время ожидания обновления истекает к моменту, когда система завершает всю работу в текущем домене обновления. В этом случае попробуйте увеличить значения параметров обновления **UpgradeTimeout** и **UpgradeDomainTimeout** и повторите попытку обновления.

### <a name="investigate-health-check-failures"></a>Исследование ошибок проверки работоспособности

Сбои проверки работоспособности могут быть вызваны различными ошибками, которые могут возникать после того, как обновление будет завершено во всех узлах в домене обновления при успешном выполнении всех проверок безопасности. Представленные ниже в этом абзаце выходные данные характерны для сбоев при обновлении из-за проверок безопасности, выявивших нарушения. Поле **UnhealthyEvaluations** записывает моментальный снимок проверок работоспособности, выявивших ошибки во время обновления, в соответствии с заданной [политикой работоспособности](service-fabric-health-introduction.md).

```powershell
Get-ServiceFabricApplicationUpgrade fabric:/DemoApp
```

```Output
ApplicationName                         : fabric:/DemoApp
ApplicationTypeName                     : DemoAppType
TargetApplicationTypeVersion            : v4
ApplicationParameters                   : {}
StartTimestampUtc                       : 4/24/2015 2:42:31 AM
UpgradeState                            : RollingForwardPending
UpgradeDuration                         : 00:00:27
CurrentUpgradeDomainDuration            : 00:00:27
NextUpgradeDomain                       : MYUD2
UpgradeDomainsStatus                    : { "MYUD1" = "Completed";
                                          "MYUD2" = "Pending";
                                          "MYUD3" = "Pending" }
UnhealthyEvaluations                    :
                                          Unhealthy services: 50% (2/4), ServiceType='PersistedServiceType', MaxPercentUnhealthyServices=0%.

                                          Unhealthy service: ServiceName='fabric:/DemoApp/Svc3', AggregatedHealthState='Error'.

                                              Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                              Unhealthy partition: PartitionId='3a9911f6-a2e5-452d-89a8-09271e7e49a8', AggregatedHealthState='Error'.

                                                  Error event: SourceId='Replica', Property='InjectedFault'.

                                          Unhealthy service: ServiceName='fabric:/DemoApp/Svc2', AggregatedHealthState='Error'.

                                              Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                              Unhealthy partition: PartitionId='744c8d9f-1d26-417e-a60e-cd48f5c098f0', AggregatedHealthState='Error'.

                                                  Error event: SourceId='Replica', Property='InjectedFault'.

UpgradeKind                             : Rolling
RollingUpgradeMode                      : Monitored
FailureAction                           : Manual
ForceRestart                            : False
UpgradeReplicaSetCheckTimeout           : 49710.06:28:15
HealthCheckWaitDuration                 : 00:00:00
HealthCheckStableDuration               : 00:00:10
HealthCheckRetryTimeout                 : 00:00:10
UpgradeDomainTimeout                    : 10675199.02:48:05.4775807
UpgradeTimeout                          : 10675199.02:48:05.4775807
ConsiderWarningAsError                  :
MaxPercentUnhealthyPartitionsPerService :
MaxPercentUnhealthyReplicasPerPartition :
MaxPercentUnhealthyServices             :
MaxPercentUnhealthyDeployedApplications :
ServiceTypeHealthPolicyMap              :
```

Исследование ошибок проверки работоспособности требует понимания модели работоспособности Service Fabric. Но даже без такого глубокого понимания можно увидеть, что две службы не прошли проверку работоспособности, *fabric:/DemoApp/Svc3* и *fabric:/DemoApp/Svc2*, а также имеются ошибочные отчеты проверки работоспособности (в данном случае InjectedFault). В этом примере две из четырех служб неработоспособны, что выше целевого показателя по умолчанию в 0 % неработоспособных служб (*MaxPercentUnhealthyServices*).

Обновление было приостановлено после сбоя, так как при запуске обновления для **FailureAction** было задано значение manual. Этот режим позволяет исследовать активную систему в неисправном состоянии перед выполнением дальнейших действий.

### <a name="recover-from-a-suspended-upgrade"></a>Восстановление при приостановке обновления

Если действию **FailureAction** назначен откат, то восстановление не требуется, так как при сбое обновления откат будет выполнен автоматически. При установке действия **FailureAction**, выполняемого вручную, существует несколько вариантов восстановления.

1.  Запуск отката.
2. Выполнение дальнейших этапов обновления вручную
3. Возобновление отслеживаемых обновлений

Команда **Start-ServiceFabricApplicationRollback** может использоваться в любое время для запуска отката приложения. После того успешного возврата команды запрос на откат регистрируется в системе и вскоре выполняется.

Команду **Resume-ServiceFabricApplicationUpgrade** можно использовать для выполнения дальнейших этапов обновления вручную последовательно, домен за доменом. В этом режиме системой выполняются только проверки безопасности. Никаких дополнительных проверок работоспособности выполняться не будет. Эту команду можно использовать, только когда параметр *UpgradeState* имеет значение *RollingForwardPending*, то есть когда текущий домен обновления завершил обновление, но обновление второго домена еще не началось (ожидается).

Команда **Update-ServiceFabricApplicationUpgrade** может использоваться для возобновления отслеживаемого обновления с проверкой безопасности и работоспособности.

```powershell
Update-ServiceFabricApplicationUpgrade fabric:/DemoApp -UpgradeMode Monitored
```

```Output
UpgradeMode                             : Monitored
ForceRestart                            :
UpgradeReplicaSetCheckTimeout           :
FailureAction                           :
HealthCheckWaitDuration                 :
HealthCheckStableDuration               :
HealthCheckRetryTimeout                 :
UpgradeTimeout                          :
UpgradeDomainTimeout                    :
ConsiderWarningAsError                  :
MaxPercentUnhealthyPartitionsPerService :
MaxPercentUnhealthyReplicasPerPartition :
MaxPercentUnhealthyServices             :
MaxPercentUnhealthyDeployedApplications :
ServiceTypeHealthPolicyMap              :
```

Обновление будет продолжено с последнего домена обновления, на котором оно было приостановлено, с использованием тех же параметров обновления и политик проверки работоспособности, что и раньше. Если это необходимо, любые из параметров обновления и политик проверки работоспособности, отображенных в выходных данных выше, могут быть изменены в той же команде, что и при возобновлении обновления. В этом примере обновление было возобновлено в отслеживаемом режиме с использованием параметров и политик работоспособности, оставшихся без изменений.

## <a name="further-troubleshooting"></a>Дальнейшее устранение неполадок

### <a name="service-fabric-is-not-following-the-specified-health-policies"></a>Service Fabric не следует указанным политикам работоспособности

Возможная причина 1.

Service Fabric преобразовывает все процентные значения в фактические количества сущностей (например, реплик, секций и служб) для оценки работоспособности и всегда округляет их до числа целых сущностей. Например, если максимальное значение *MaxPercentUnhealthyReplicasPerPartition* равно 21 % и есть пять реплик, то Service Fabric допустит выделение до двух неработоспособных реплик (т. е. `Math.Ceiling (5*0.21)`). Поэтому политики работоспособности должны быть настроены с учетом этого.

Возможная причина 2.

Политики работоспособности указываются в процентах от общего числа служб, а не в конкретных экземплярах служб. Например, предположим, что перед обновлением у приложения имеется четыре экземпляра службы, A, B, C и D, и служба D неработоспособна, но лишь незначительно влияет на работу приложения. Мы собираемся пропустить известную неработоспособную службу D во время обновления и задать для параметра *MaxPercentUnhealthyServices* значение 25 %, предполагая, что только службы A, B и C должны оставаться работоспособными.

Однако во время обновления служба D может стать работоспособной, а служба C — потерять работоспособность. Обновление все равно пройдет успешно, так как только 25 % служб неработоспособно. Однако возможны непредвиденные ошибки из-за того, что служба C неожиданно стала неработоспособной вместо службы D. В такой ситуации службу D необходимо смоделировать как службу типа, отличного от типа служб A, B и C. Так как для типов служб можно задавать отдельные политики работоспособности, это позволяет применить разные пороговые процентные значения для разных служб. 

### <a name="i-did-not-specify-a-health-policy-for-application-upgrade-but-the-upgrade-still-fails-for-some-time-outs-that-i-never-specified"></a>Политики работоспособности при обновлении приложения не указаны, однако обновление все равно завершается со сбоем из-за не заданных значений времени ожидания.

Если политики работоспособности не переданы в запросе на обновление, они берутся из файла *ApplicationManifest.xml* текущей версии приложения. Например, при обновлении приложения X версии 1.0 до версии 2.0 используются политики работоспособности приложения, указанные для приложения X версии 1.0. Если для обновления должна использоваться другая политика работоспособности, то политику будет необходимо задать как часть вызова API обновления приложения. Политики, заданные в вызове API, применяются только во время обновления. После завершения обновления будут использоваться политики, заданные в файле *ApplicationManifest.xml* .

### <a name="incorrect-time-outs-are-specified"></a>Указаны неверные значения времени ожидания

Полагаю, вам интересно, что произойдет, если задать несогласованные значения времени ожидания. Например, может быть задано значение *UpgradeTimeout* меньше, чем *UpgradeDomainTimeout*. В этом случае будет возвращена ошибка. Ошибки возникнут, если значение *UpgradeDomainTimeout* меньше суммы значений параметров *HealthCheckWaitDuration* и *HealthCheckRetryTimeout* или значение *UpgradeDomainTimeout* меньше суммы значений *HealthCheckWaitDuration* и *HealthCheckStableDuration*.

### <a name="my-upgrades-are-taking-too-long"></a>Мои обновления выполняются слишком долго

Время завершения обновления зависит от настроенных проверок работоспособности и значений времени ожидания. Проверки работоспособности и значения времени ожидания зависят от того, сколько времени требуется для копирования, развертывания и стабилизации приложения. Слишком жесткое назначение времени ожидания может привести к большему числу сбоев при обновлении, поэтому при назначении значений времени ожидания рекомендуется консервативный подход.

Напомним о том, как значения времени ожидания сказываются на времени применения обновлений.

Обновление домена обновления не может выполняться быстрее, чем *хеалсчеккваитдуратион*  +  *HealthCheckStableDuration*.

Сбой обновления не может происходить быстрее, чем *хеалсчеккваитдуратион*  +  *HealthCheckRetryTimeout*.

Время обновления для домена обновления ограничено значением *UpgradeDomainTimeout*.  Если значения *HealthCheckRetryTimeout* и *HealthCheckStableDuration* оба не равны нулю, а работоспособность приложения постоянно меняется, то время ожидания обновления обязательно завершится при достижении значения *UpgradeDomainTimeout*. *UpgradeDomainTimeout* начинает обратный отсчет с момента начала текущего домена обновления.

## <a name="next-steps"></a>Дальнейшие действия

[Руководство по обновлению приложений Service Fabric с помощью Visual Studio](service-fabric-application-upgrade-tutorial.md) поможет вам выполнить поэтапное обновление приложения с помощью Visual Studio.

[Обновление приложения с помощью PowerShell](service-fabric-application-upgrade-tutorial-powershell.md) поможет вам выполнить обновление приложения с помощью PowerShell.

Управление обновлениями приложения осуществляется с помощью [параметров обновления](service-fabric-application-upgrade-parameters.md).

Изучите, как использовать [сериализацию данных](service-fabric-application-upgrade-data-serialization.md), чтобы обеспечить совместимость обновлений приложений.

Узнайте, как использовать расширенные функциональные возможности при обновлении приложения, обратившись к [дополнительным разделам](service-fabric-application-upgrade-advanced.md).