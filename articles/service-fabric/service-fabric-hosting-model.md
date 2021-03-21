---
title: Модель размещения Azure Service Fabric
description: В этой статье описывается связь между репликами (или экземплярами) развернутой службы Service Fabric и процессом размещения службы.
author: harahma
ms.topic: conceptual
ms.date: 04/15/2017
ms.author: harahma
ms.custom: devx-track-csharp
ms.openlocfilehash: 9932c11332a616928d59c213d4f4806feb81cfe2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791651"
---
# <a name="azure-service-fabric-hosting-model"></a>Модель размещения Azure Service Fabric
В этой статье представлен обзор моделей размещения приложений, предоставленных Azure Service Fabric, и приведены различия между моделями **с общим** и **монопольным процессом**. Здесь также описывается, как развернутое приложение выглядит на узле Service Fabric, и рассматриваются связи между репликами (или экземплярами) службы и процессом размещения служб.

Прежде чем продолжить, убедитесь, что вы понимаете различные понятия и связи, описанные в статье [Моделирование приложения в структуре службы][a1]. 

> [!NOTE]
> Если явно не указано иное, в этой статье:
>
> - Слово *реплика* относится к реплике службы с отслеживанием состояния или экземпляру службы без отслеживания состояния.
> - *CodePackage* считается эквивалентом процесса *ServiceHost*, который регистрирует *ServiceType* и размещает реплики служб этого *ServiceType*.
>

Чтобы понять модель размещения, давайте подробно рассмотрим пример. Допустим, мы имеем *тип приложения* MyAppType с типом службы *MyServiceType*, предоставляемым *пакетом службы* MyServicePackage с *пакетом кода* MyCodePackage, регистрирующим при запуске тип службы *MyServiceType*.

Допустим, мы имеем кластер из трех узлов и создаем *приложение* **fabric:/App1** типа MyAppType. Внутри этого приложения **fabric:/App1** мы создаем службу **fabric:/App1/ServiceA** типа MyServiceType. Служба имеет 2 раздела (например, **P1** и **P2**) с 3 репликами на каждый. На следующей схеме показано представление этого приложения, развернутого на узле.


![Схема, показывающая представление этого приложения в том виде, в котором оно было развернуто на узле.][node-view-one]


Service Fabric активирует пакет MyServicePackage, который запустил пакет MyCodePackage, где размещаются реплики из обоих разделов. Все узлы в кластере будут иметь одинаковое представление, так как мы выбрали число реплик на раздел, равное числу узлов в кластере. Теперь создадим другую службу, **fabric:/App1/ServiceB**, в приложении **fabric:/App1**. Служба имеет один раздел (например, **P3**) с тремя репликами. На следующей схеме показано новое представление узла:


![Схема, на которой показано новое представление на узле.][node-view-two]


Service Fabric поместил новую реплику раздела **P3** службы **fabric:/App1/ServiceB** в существующую активацию MyServicePackage. Теперь создадим другое приложение **fabric:/App2** типа MyAppType. В приложении **fabric:/App2** создадим службу **fabric:/App2/ServiceA**. Служба имеет 2 раздела (**P4** и **P5**) с 3 репликами на каждый. На следующих схемах показано новое представление узла:


![Схема, на которой показано новое представление узла.][node-view-three]


Service Fabric активирует новую копию пакета MyServicePackage, которая запускает новую копию пакета MyCodePackage. Реплики из обоих разделов службы **fabric:/App2/ServiceA** (например, **P4** и **P5**) размещаются в этой новой копии MyCodePackage.

## <a name="shared-process-model"></a>Модель с общим процессом
В предыдущем разделе была описана модель размещения по умолчанию, предоставляемая службой Service Fabric, которая называется моделью с общим процессом. В этой модели для конкретного приложения активируется только одна копия заданного *ServicePackage* на узле (которая запускает все содержащиеся в ней пакеты *CodePackage*). Все реплики всех служб заданного *ServiceType* размещаются в *CodePackage*, который регистрирует *ServiceType*. Другими словами, все реплики всех служб на узле заданного *ServiceType* совместно используют один процесс.

## <a name="exclusive-process-model"></a>Модель с монопольным процессом
Другой моделью размещения, представляемой Service Fabric, является модель с монопольным процессом. В этой модели на заданном узле каждая реплика находится в отдельном выделенном процессе. Service Fabric активирует новую копию пакета *ServicePackage* (которая запускает все содержащиеся в ней пакеты *CodePackage*). Реплика размещается в *CodePackage*, который регистрирует *ServiceType* службы, к которой принадлежит реплика. 

В Service Fabric, начиная с версии 5.6, модель с монопольным процессом можно выбрать во время создания службы (с помощью [PowerShell][p1], [REST][r1] или [FabricClient][c1]). Укажите для параметра **ServicePackageActivationMode** значение ExclusiveProcess.

```powershell
PS C:\>New-ServiceFabricService -ApplicationName "fabric:/App1" -ServiceName "fabric:/App1/ServiceA" -ServiceTypeName "MyServiceType" -Stateless -PartitionSchemeSingleton -InstanceCount -1 -ServicePackageActivationMode "ExclusiveProcess"
```

```csharp
var serviceDescription = new StatelessServiceDescription
{
    ApplicationName = new Uri("fabric:/App1"),
    ServiceName = new Uri("fabric:/App1/ServiceA"),
    ServiceTypeName = "MyServiceType",
    PartitionSchemeDescription = new SingletonPartitionSchemeDescription(),
    InstanceCount = -1,
    ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
};

var fabricClient = new FabricClient(clusterEndpoints);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Если в манифесте приложения указана служба по умолчанию, можно выбрать модель с монопольным процессом, указав атрибут **ServicePackageActivationMode**.

```xml
<DefaultServices>
  <Service Name="MyService" ServicePackageActivationMode="ExclusiveProcess">
    <StatelessService ServiceTypeName="MyServiceType" InstanceCount="1">
      <SingletonPartition/>
    </StatelessService>
  </Service>
</DefaultServices>
```
Теперь создадим другую службу, **fabric:/App1/ServiceC**, в приложении **fabric:/App1**. Служба имеет 2 раздела (например, **P6** и **P7**) с 3 репликами на каждый. Укажите для параметра **ServicePackageActivationMode** значение ExclusiveProcess. На следующей схеме показано новое представление узла.


![Схема представления узла развернутого приложения][node-view-four]


Как видите, Service Fabric активировала две новые копии MyServicePackage (по одной на каждую реплику из раздела **P6** и **P7**). Service Fabric размещает каждую реплику в выделенную копию *CodePackage*. При использовании модели с монопольным процессом для заданного приложения на узле могут быть активны несколько копий заданного *ServicePackage*. В предыдущем примере мы видим, что для **fabric:/App1** активны три копии MyServicePackage. Каждая из этих активных копий MyServicePackage имеет связанный с ней **ServicePackageAtivationId**. Этот идентификатор определяет эту копию в приложении **fabric:/App1**.

Если для приложения используется только модель с общим процессом, на узле будет только одна активная копия *ServicePackage*. Значение **ServicePackageAtivationId** для этой активации *ServicePackage* будет соответствовать пустой строке. Так обстоит, к примеру, в случае с **fabric:/App2**.

> [!NOTE]
>- Модель размещения с общим процессом используется, когда для параметра **ServicePackageActivationMode** задано значение **SharedProcess**. Это модель размещения по умолчанию, поэтому во время создания службы необходимо указать значение **ServicePackageActivationMode**.
>
>- Модель размещения с монопольным процессом используется, когда для параметра **ServicePackageActivationMode** задано значение **ExclusiveProcess**. Чтобы использовать этот параметр, вы должны указать его явно во время создания службы. 
>
>- Модель размещения службы можно узнать, запросив [описание службы][p2], где нужно просмотреть значение **ServicePackageActivationMode**.
>
>

## <a name="work-with-a-deployed-service-package"></a>Работа с развернутым пакетом службы
Активная копия *ServicePackage* на узле называется [развернутым пакетом службы][p3]. Если при создании служб используется модель с монопольным процессом, для заданного приложения можно развернуть несколько пакетов служб одного пакета *ServicePackage*. Во время выполнения операций, относящихся к развернутому пакету службы, необходимо предоставить значение **ServicePackageActivationId**, позволяющее идентифицировать конкретный развернутый пакет службы. Например, для [отправки сведений о работоспособности развернутого пакета службы][p4] или при [перезапуске пакета кода развернутого пакета службы][p5].

**ServicePackageActivationId** развернутого пакета службы можно получить, запросив список [развернутых пакетов служб][p3] на узле. При запросе [типов развернутых служб][p6], [реплик][p7] и [пакетов кода][p8] на узле результат запроса также содержит значение **ServicePackageActivationId** родительского развернутого пакета службы.

> [!NOTE]
>- В модели размещения с общим процессом на заданном узле заданного приложения активируется только одна копия *ServicePackage*. При этом параметр **ServicePackageActivationId** соответствует *пустой строке*, и его не нужно указывать при выполнении операций, связанных с развернутым пакетом службы. 
>
> - В модели размещения с монопольным процессом на заданном узле заданного приложения активируется одна или несколько копий *ServicePackage*. Каждая активация имеет *непустое значение* **ServicePackageActivationId**, указанное при выполнении операций, связанных с развернутым пакетом службы. 
>
> - Если значение **ServicePackageActivationId** опущено, по умолчанию используется *пустая строка*. При наличии развернутого пакета службы, который был активирован с помощью модели с общим процессом, операция будет выполняться с ним. В противном случае операция завершится ошибкой.
>
> - Не выполняйте один запрос и не кэшируйте значение **ServicePackageActivationId**. Оно создается динамически и может изменяться по различным причинам. Перед выполнением операции, для которой требуется значение **ServicePackageActivationId**, сначала следует запросить список [развернутых пакетов служб][p3] на узле. Затем используйте значение **ServicePackageActivationId** из результата запроса для выполнения исходной операции.
>
>

## <a name="guest-executable-and-container-applications"></a>Гостевые исполняемые приложения и приложения-контейнеры
Service Fabric рассматривает [гостевые исполняемые приложения][a2] и [приложения-контейнеры][a3] как службы без отслеживания состояния, которые являются самодостаточными. Для них в процессе *ServiceHost* (в процессе или контейнере) нет среды выполнения Service Fabric. Так как эти службы являются самодостаточными, для этих служб число реплик на *ServiceHost* неприменимо. Самая распространенная конфигурация, которая используется с этими службами, — один раздел со значением [InstanceCount][c2] равным -1 (с одной копией кода службы на каждом узле кластера). 

**ServicePackageActivationMode** по умолчанию для этих служб — **SharedProcess**, в этом случае Service Fabric активирует только одну копию *ServicePackage* на узле для данного приложения.  Это означает, что на узле будет выполняться только одна копия кода службы. Если на узле требуется запустить несколько копий кода службы, во время создания службы для параметра **ServicePackageActivationMode** необходимо задать значение **ExclusiveProcess**. Например, при создании нескольких служб (от *Service1* до *ServiceN*) типа *ServiceType* (указанного в *ServiceManifest*) или если служба имеет несколько разделов. 

## <a name="change-the-hosting-model-of-an-existing-service"></a>Изменение модели размещения существующей службы
В настоящее время вы не можете изменить модель размещения существующей службы с модели с общим процессом на модель с монопольным процессом (или наоборот).

## <a name="choose-between-the-hosting-models"></a>Выбор между моделями размещения
Важно сравнить обе модели размещения, чтобы понять, какая из них лучше всего соответствует требованиям. Модель с общим процессом обеспечивает более эффективное использование ресурсов операционной системы, так как создается меньшее число процессов, и несколько реплик в одном процессе могут совместно использовать порты. Тем не менее, если в одной из реплик возникает ошибка, при которой ей придется отключить узел службы, это затронет все реплики в одном процессе.

 Модель с монопольным процессом обеспечивает лучшую изоляцию каждой реплики в своем собственном процессе. Некорректно работающая реплика не влияет на другие реплики. Это подходит для тех случаев, когда протокол связи не поддерживает совместное использование порта. Эта модель обеспечивает возможность управления ресурсами на уровне реплики. Однако монопольный процесс потребляет больше ресурсов ОС, так как для каждой реплики на узле создается один процесс.

## <a name="exclusive-process-model-and-application-model-considerations"></a>Рекомендации по модели с монопольным процессом и модели приложения
Для моделирования большинства приложений в Service Fabric рекомендуется иметь один *ServiceType* для каждого *ServicePackage*. 

Service Fabric предназначен для определенных вариантов использования и также позволяет использовать несколько значений *ServiceType* на значение *ServicePackage* (и одно значение *CodePackage* может зарегистрировать несколько значений *ServiceType*). Ниже приведены некоторые сценарии, в которых можно использовать эти конфигурации:

- Необходимо оптимизировать использование ресурсов за счет запуска меньшего числа процессов с более высокой плотностью реплик для каждого из них.
- Реплики из разных *типов serviceType* должны использовать одни общие данные с высокой инициализацией или стоимостью памяти.
- У вас есть предложение с бесплатной службой, и вы хотите установить ограничение на использование ресурсов путем размещения всех реплик службы в один процесс.

Модель размещения с монопольным процессом не согласуется с моделью приложения, которая имеет несколько типов *ServiceType* для каждого *ServicePackage*. Это связано с тем, что несколько *типов serviceType* на *ServicePackage* предназначены для достижения большего общего доступа к ресурсам между репликами и обеспечивают более высокую плотность реплик для каждого процесса. Модель с монопольным процессом предназначена для достижения других целей.

Рассмотрим случай использования нескольких типов *ServiceType* для каждого *ServicePackage*, при котором каждый *ServiceType* регистрируется с помощью разных *CodePackage*. Предположим, мы имеем пакет *ServicePackage* MultiTypeServicePackage, который содержит два пакета *CodePackage*:

- MyCodePackageA, который регистрирует *тип службы* MyServiceTypeA;
- "MyCodePackageB", который регистрирует *serviceType* "MyServiceTypeB".

Теперь создадим приложение **fabric:/SpecialApp**. Внутри **fabric:/SpecialApp** создадим две следующие службы с помощью модели с монопольным процессом:

- служба **fabric:/SpecialApp/ServiceA** типа MyServiceTypeA с двумя разделами (например, **P1** и **P2**) и тремя репликами на каждый раздел;
- служба **fabric:/SpecialApp/ServiceB** типа MyServiceTypeB с двумя разделами (например, **P3** и **P4**) и тремя репликами на каждый раздел.

На заданном узле обе службы имеют две реплики. Так как для создания служб мы использовали модель с монопольным процессом, для каждой реплики Service Fabric активирует новую копию MyServicePackge. Каждая активация MultiTypeServicePackage запускает копию MyCodePackageA и MyCodePackageB. Однако только в одной из копий (MyCodePackageA или MyCodePackageB) размещена реплика, для которой был активирован MultiTypeServicePackage. На следующей схеме показано представление узла:


![Схема, показывающая представление узла.][node-view-five]


В активации MultiTypeServicePackage для реплики раздела **P1** службы **fabric:/SpecialApp/ServiceA** в пакете MyCodePackageA размещается реплика. Пакет MyCodePackageB запущен и работает. Аналогично в активации MultiTypeServicePackage для реплики раздела **P3** службы **fabric:/SpecialApp/ServiceB** в пакете MyCodePackageB размещается реплика. Пакет MyCodePackageА запущен и работает. Таким образом, чем больше количество пакетов *CodePackage* (регистрирующих различные типы *ServiceType*) для каждого *ServicePackage*, тем выше избыточное использование ресурсов. 
 
 Однако, если мы создадим службы **fabric:/SpecialApp/ServiceA** и **fabric:/SpecialApp/ServiceB** с помощью модели с общим процессом, Service Fabric активирует только одну копию MultiTypeServicePackage для приложения **fabric:/SpecialApp**. В пакете MyCodePackageA разместятся все реплики службы **fabric:/SpecialApp/ServiceA**. В пакете MyCodePackageВ разместятся все реплики службы **fabric:/SpecialApp/ServiceВ**. На следующей схеме показано представление узла в такой конфигурации: 


![Схема представления узла развернутого приложения][node-view-six]


Из предыдущего примера можно сделать вывод, что если MyCodePackageA регистрирует оба типа MyServiceTypeA и MyServiceTypeB, но пакет MyCodePackageB отсутствует, избыточный *CodePackage* выполняться не будет. Хотя это верно, эта модель приложения не согласуется с моделью размещения с монопольным процессом. Если задача в том, чтобы разместить каждую реплику в свой собственный выделенный процесс, тогда регистрация обоих типов *ServiceType* из одного пакета *CodePackage* не требуется. Вместо этого проще поместить каждый *ServiceType* в свой собственный пакет *ServicePackage*.

### <a name="reliable-services-and-actor-forking-subprocesses"></a>Подпроцессы Reliable Services и "Разветвление субъекта"

Service Fabric не поддерживает подпроцессы надежных услуг и, следовательно, надежного разветвления субъекта. Примером того, почему они не поддерживаются, является [CodePackageActivationContext](/dotnet/api/system.fabric.codepackageactivationcontext), который нельзя использовать для регистрации неподдерживаемого подпроцесса, а токены отмены только отправляются на зарегистрированные процессы. В результате этого возникают всевозможные проблемы, такие как сбои обновлений, когда подпроцесс не закрывается после того, как родительский процесс получил токен отмены.

## <a name="next-steps"></a>Дальнейшие действия
[Создайте пакет приложения][a4] и подготовьте его к развертыванию.

[Развертывание и удаление приложений с помощью PowerShell][a5]. В статье представлены сведения об управлении экземплярами приложений с помощью PowerShell.

<!--Image references-->
[node-view-one]: ./media/service-fabric-hosting-model/node-view-one.png
[node-view-two]: ./media/service-fabric-hosting-model/node-view-two.png
[node-view-three]: ./media/service-fabric-hosting-model/node-view-three.png
[node-view-four]: ./media/service-fabric-hosting-model/node-view-four.png
[node-view-five]: ./media/service-fabric-hosting-model/node-view-five.png
[node-view-six]: ./media/service-fabric-hosting-model/node-view-six.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[a1]: service-fabric-application-model.md
[a2]: service-fabric-guest-executables-introduction.md
[a3]: service-fabric-containers-overview.md
[a4]: service-fabric-package-apps.md
[a5]: service-fabric-deploy-remove-applications.md

[r1]: /rest/api/servicefabric/sfclient-api-createservice

[c1]: /dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync
[c2]: /dotnet/api/system.fabric.description.statelessservicedescription.instancecount

[p1]: /powershell/module/servicefabric/new-servicefabricservice
[p2]: /powershell/module/servicefabric/get-servicefabricservicedescription
[p3]: /powershell/module/servicefabric/get-servicefabricdeployedservicepackage
[p4]: /powershell/module/servicefabric/send-servicefabricdeployedservicepackagehealthreport
[p5]: /powershell/module/servicefabric/restart-servicefabricdeployedcodepackage
[p6]: /powershell/module/servicefabric/get-servicefabricdeployedservicetype
[p7]: /powershell/module/servicefabric/get-servicefabricdeployedreplica
[p8]: /powershell/module/servicefabric/get-servicefabricdeployedcodepackage
