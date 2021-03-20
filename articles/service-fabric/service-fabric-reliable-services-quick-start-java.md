---
title: Создание первой надежной службы на Java
description: Общие сведения о создании Microsoft Azure Service Fabric приложения с помощью служб без отслеживания состояния и с отслеживанием состояния в Java.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-javai
ms.openlocfilehash: f67957d711958febdb01dfad0b3c44a92cb0bcfa
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91535242"
---
# <a name="get-started-with-reliable-services-in-java"></a>Приступая к работе с Reliable Services в Java
> [!div class="op_single_selector"]
> * [C# в Windows](service-fabric-reliable-services-quick-start.md)
> * [Java в Linux](service-fabric-reliable-services-quick-start-java.md)
>
>

В этой статье представлены общие сведения о Reliable Services в Azure Service Fabric, а также описывается процесс создания и развертывания простого приложения Reliable Services, написанного на языке Java. 

## <a name="installation-and-setup"></a>Установка и настройка
Перед началом установите и настройте на своем компьютере среду разработки Service Fabric.
Дополнительные сведения см. в статье [Настройка среды разработки для Mac OS X](service-fabric-get-started-mac.md) и [Подготовка среды разработки в Linux](service-fabric-get-started-linux.md).

## <a name="basic-concepts"></a>Основные понятия
Чтобы приступить к работе с Reliable Services, необходимо ознакомиться только с несколькими основными понятиями.

* **Тип службы**. Это ваша реализация службы. Ее определяет создаваемый вами класс, который расширяет `StatelessService`, и любой другой код или зависимости, используемые в нем, а также имя и номер версии.
* **Именованный экземпляр службы**. Для запуска службы создаются именованные экземпляры вашего типа службы, что во многом похоже на создание экземпляров объектов типа класса. Экземпляры службы фактически являются создаваемыми экземплярами объектов написанного вами класса службы.
* **Узел службы**. Создаваемые экземпляры службы должны выполняться на узле. Узел службы — это просто процесс, в котором можно запускать экземпляры вашей службы.
* **Регистрации службы**. Регистрация объединяет все элементы. Тип службы должен быть зарегистрирован в среде выполнения Service Fabric на узле службы, что позволит Service Fabric создавать экземпляры службы для запуска.  

## <a name="create-a-stateless-service"></a>Создание службы без отслеживания состояния
Для начала создайте приложение Service Fabric. В пакет SDK для Service Fabric для Linux входит генератор Yeoman, который позволяет формировать шаблоны для приложения Service Fabric с помощью службы без отслеживания состояния. Сначала выполните следующую команду Yeoman:

```bash
$ yo azuresfjava
```

Следуйте инструкциям по созданию **надежной службы без отслеживания состояния**. В этом руководстве в качестве имени приложения используется HelloWorldApplication, а в качестве имени службы — HelloWorld. Результат включает в себя каталоги для `HelloWorldApplication` и `HelloWorld`.

```bash
HelloWorldApplication/
├── build.gradle
├── HelloWorld
│   ├── build.gradle
│   └── src
│       └── statelessservice
│           ├── HelloWorldServiceHost.java
│           └── HelloWorldService.java
├── HelloWorldApplication
│   ├── ApplicationManifest.xml
│   └── HelloWorldPkg
│       ├── Code
│       │   ├── entryPoint.sh
│       │   └── _readme.txt
│       ├── Config
│       │   └── _readme.txt
│       ├── Data
│       │   └── _readme.txt
│       └── ServiceManifest.xml
├── install.sh
├── settings.gradle
└── uninstall.sh
```
### <a name="service-registration"></a>Регистрация службы
Типы службы должны быть зарегистрированы в среде выполнения Service Fabric. Тип службы определен в `ServiceManifest.xml` и в классе службы, реализующем `StatelessService`. Регистрация службы выполняется в главной точке входа в рамках процесса. В этом примере главная точка входа в рамках процесса — это `HelloWorldServiceHost.java`.

```java
public static void main(String[] args) throws Exception {
    try {
        ServiceRuntime.registerStatelessServiceAsync("HelloWorldType", (context) -> new HelloWorldService(), Duration.ofSeconds(10));
        logger.log(Level.INFO, "Registered stateless service type HelloWorldType.");
        Thread.sleep(Long.MAX_VALUE);
    }
    catch (Exception ex) {
        logger.log(Level.SEVERE, "Exception in registration:", ex);
        throw ex;
    }
}
```

## <a name="implement-the-service"></a>Реализация службы

Откройте файл **HelloWorldApplication/HelloWorld/src/statelessservice/HelloWorldService.java**. Этот класс определяет тип службы и может выполнить любой код. API службы предоставляет две точки входа для кода.

* Вызывается метод `runAsync()` с открытой точкой входа, в котором можно начать выполнение любой рабочей нагрузки, например длительных вычислений.

```java
@Override
protected CompletableFuture<?> runAsync(CancellationToken cancellationToken) {
    ...
}
```

* Точка входа связи, где можно подключить любой стек связи по выбору. С ее помощью вы можете получать запросы от пользователей и других служб.

```java
@Override
protected List<ServiceInstanceListener> createServiceInstanceListeners() {
    ...
}
```

В этом руководстве рассматривается метод точки входа `runAsync()`. Используя его, вы можете сразу же начать выполнение кода.

### <a name="runasync"></a>RunAsync
Платформа вызывает этот метод, когда экземпляр службы размещен и готов к выполнению. Для службы без отслеживания состояния это означает, что экземпляр службы открыт. Токен отмены предоставляется для координации закрытия экземпляра вашей службы. В Service Fabric этот цикл открытия и закрытия экземпляра службы может выполняться много раз за все время существования службы. Это может происходить по ряду причин.

* Система перемещает экземпляры служб для распределения ресурсов.
* В коде возникают ошибки.
* Выполняется обновление приложения или системы.
* Возникает аппаратный сбой.

Этой оркестрацией управляет Service Fabric для сохранения высокой доступности и правильного баланса службы.

Реализация `runAsync()` не должна использовать синхронную блокировку. Реализация runAsync должна вернуть CompletableFuture, чтобы среда выполнения могла продолжить операции. Если для вашей рабочей нагрузки необходимо реализовать длительную задачу, то она должна выполняться в пределах CompletableFuture.

#### <a name="cancellation"></a>Отмена
Отмена рабочей нагрузки является совместным действием, координированным предоставленным токеном отмены. Чтобы продолжать работу, система должна дождаться завершения задачи (успешного завершения, отмены или завершения ошибкой). Когда система запрашивает отмену задачи, важно как можно быстрее подтвердить токен отмены, завершить все выполняющиеся операции и выйти из `runAsync()` . Следующий пример демонстрирует, как обрабатывать событие отмены.

```java
@Override
protected CompletableFuture<?> runAsync(CancellationToken cancellationToken) {

    // TODO: Replace the following sample code with your own logic
    // or remove this runAsync override if it's not needed in your service.

    return CompletableFuture.runAsync(() -> {
        long iterations = 0;
        while(true)
        {
        cancellationToken.throwIfCancellationRequested();
        logger.log(Level.INFO, "Working-{0}", ++iterations);

        try {
            Thread.sleep(1000);
        } catch (InterruptedException ex){}
        }
    });
}
```

В этом примере службы без отслеживания состояния счетчик хранится в локальной переменной. Но так как это служба без отслеживания состояния, хранимое значение существует только для текущего жизненного цикла экземпляра службы. Если службу переместить или перезапустить, значение будет утеряно.

## <a name="create-a-stateful-service"></a>Создание службы с отслеживанием состояния
Service Fabric представляет новый вид службы с отслеживанием состояния. Служба с отслеживанием состояния может надежно поддерживать состояние в самой службе (совмещая с кодом, который его использует). Service Fabric делает состояние высокодоступным без необходимости сохранения состояния во внешнее хранилище.

Чтобы преобразовать значение счетчика без учета состояния в высокодоступное и постоянное (даже при перемещении или перезапуске службы), нужно использовать службу с отслеживанием состояния.

В том же каталоге приложения HelloWorld можно добавить новую службу, выполнив команду `yo azuresfjava:AddService`. Выберите параметр Reliable Stateful Service (Надежная служба с отслеживанием состояния) для платформы и назовите службу HelloWorldStateful. 

Теперь в вашем приложении должно быть две службы: служба без отслеживания состояния HelloWorld и служба с отслеживанием состояния HelloWorldStateful.

Служба с отслеживанием состояния имеет такие же точки входа, как и служба без отслеживания состояния. Основное различие заключается в доступности поставщика состояний, который надежно хранит состояние. Платформа Service Fabric поставляется с реализацией поставщика состояний Reliable Collections, которая позволяет создавать реплицированные структуры данных с помощью диспетчера надежных состояний. Служба Reliable Service с отслеживанием состояния использует этот поставщик состояний по умолчанию.

Откройте HelloWorldStateful.java в службе **HelloWorldStateful -> src**, содержащей следующий метод RunAsync.

```java
@Override
protected CompletableFuture<?> runAsync(CancellationToken cancellationToken) {
    Transaction tx = stateManager.createTransaction();
    return this.stateManager.<String, Long>getOrAddReliableHashMapAsync("myHashMap").thenCompose((map) -> {
        return map.computeAsync(tx, "counter", (k, v) -> {
            if (v == null)
                return 1L;
            else
                return ++v;
            }, Duration.ofSeconds(4), cancellationToken)
                .thenCompose((r) -> tx.commitAsync())
                .whenComplete((r, e) -> {
            try {
                tx.close();
            } catch (Exception e) {
                logger.log(Level.SEVERE, e.getMessage());
            }
        });
    });
}
```

### <a name="runasync"></a>RunAsync
`RunAsync()` работает одинаково в службах с отслеживанием и без отслеживания состояния. Однако в службе с отслеживанием состояния платформа выполняет дополнительные действия от вашего имени перед выполнением `RunAsync()`. Эти действия могут включать подготовку диспетчера надежных состояний и коллекций Reliable Collections к использованию.

### <a name="reliable-collections-and-the-reliable-state-manager"></a>Reliable Collections и диспетчер надежных состояний
```java
ReliableHashMap<String,Long> map = this.stateManager.<String, Long>getOrAddReliableHashMapAsync("myHashMap")
```

[ReliableHashMap](/java/api/microsoft.servicefabric.data.collections.reliablehashmap) — это реализация словаря, которая позволяет надежно хранить состояние службы. В Service Fabric с помощью Reliable HashMap можно хранить данные непосредственно в службе без использования внешнего постоянного хранилища. Reliable HashMap делает данные высокодоступными. Service Fabric выполняет эту задачу, автоматически создавая несколько *реплик* службы и управляя ими. Кроме того, платформа предоставляет API-интерфейс, который упрощает управление этими репликами и переходы между их состояниями.

В Reliable Collections можно хранить любые типы объектов Java, включая пользовательские типы. Необходимо только учитывать следующее.

* Service Fabric делает состояние высокодоступным, *реплицируя* его между узлами, а служба Reliable HashMap сохраняет ваши данные на локальном диске каждой реплики. Это означает, что все объекты, хранящиеся в Reliable HashMap, должны поддерживать *сериализацию*. 
* Когда вы зафиксируете транзакции в Reliable HashMap, объекты реплицируются и становятся высокодоступными. Объекты, сохраненные в Reliable HashMap, хранятся в локальной памяти вашей службы. Это означает, что у вас есть локальная ссылка на объект.
  
   Очень важно не изменять локальные экземпляры этих объектов без обновления коллекции в транзакции, так как эти изменения не реплицируются автоматически. Необходимо повторно вставить объект в словарь или использовать один из методов *update* в словаре.

Диспетчер надежных состояний автоматически управляет Reliable HashMap. Запросите у диспетчера надежных состояний имя надежной коллекции в любое время и в любом расположении службы и вы гарантированно получите ссылку. Мы не рекомендуем сохранять ссылки на экземпляры надежной коллекции в переменные или свойства членов класса. Необходимо внимательно следить, чтобы ссылка указывала на экземпляр на протяжении всего жизненного цикла службы. Диспетчер надежных состояний выполняет за вас эту работу. Кроме того, он оптимизирован для повторных визитов.


### <a name="transactional-and-asynchronous-operations"></a>Транзакционные и асинхронные операции
```java
return map.computeAsync(tx, "counter", (k, v) -> {
    if (v == null)
        return 1L;
    else
        return ++v;
    }, Duration.ofSeconds(4), cancellationToken)
        .thenCompose((r) -> tx.commitAsync())
        .whenComplete((r, e) -> {
    try {
        tx.close();
    } catch (Exception e) {
        logger.log(Level.SEVERE, e.getMessage());
    }
});
```

Операции в Reliable HashMap являются асинхронными. Это вызвано тем, что операции записи с Reliable Collections выполняют операции ввода-вывода для репликации и сохранения данных на диске.

Операции Reliable HashMap являются *транзакционными*, так что вы можете сохранять согласованное состояние между несколькими службами Reliable HashMap и операциями. Например, вы можете получить рабочий элемент из надежного словаря, выполнить с ним операцию и сохранить результат в другой службе Reliable HashMap, и это все в пределах одной транзакции. Такая операция называется атомарной. Предусматривается, что либо вся операция завершится успешно, либо она будет полностью отменена. Если ошибка возникнет после выведения элемента из очереди, но до сохранения результата, будет выполнен откат всей транзакции, а элемент останется в очереди для обработки.


## <a name="build-the-application"></a>Построение приложения

Формирование шаблонов Yeoman включает в себя сценарий Gradle, используемый для создания приложения, и сценарии bash, используемые для развертывания и удаления приложения. Чтобы запустить приложение, сначала его нужно создать с помощью следующего скрипта Gradle:

```bash
$ gradle
```

Этот сценарий создает пакет приложения Service Fabric, который можно развернуть с помощью интерфейса командной строки Service Fabric.

## <a name="deploy-the-application"></a>Развертывание приложения

Созданное приложение можно развернуть в локальном кластере.

1. Подключитесь к удаленному кластеру Service Fabric.

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```

2. С помощью скрипта установки (включен в шаблон) скопируйте пакет приложения в хранилище образов кластера, зарегистрируйте тип приложения и создайте экземпляр приложения.

    ```bash
    ./install.sh
    ```

Созданное приложение развертывается так же, как и любое другое приложение Service Fabric. Дополнительные сведения см. в документации по [управлению приложениями Service Fabric с помощью интерфейса командной строки Service Fabric](service-fabric-application-lifecycle-sfctl.md).

Параметры для этих команд можно найти в созданном манифесте в пакете приложения.

Когда приложение будет развернуто, откройте браузер и перейдите к [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) по адресу `http://localhost:19080/Explorer`. Затем разверните узел **Приложения**. Вы увидите одну запись для типа приложения и еще одну — для первого экземпляра этого типа.

> [!IMPORTANT]
> Чтобы развернуть приложение в защищенном кластере Linux в Azure, настройте сертификат для проверки приложения в среде выполнения Service Fabric. Таким образом, службы Reliable Services смогут взаимодействовать с API среды выполнения Service Fabric. Дополнительные сведения см. в разделе [Настройка приложения Reliable Services для запуска на кластерах Linux](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters).  
>

## <a name="next-steps"></a>Дальнейшие действия

* [Командная строка Azure Service Fabric](service-fabric-cli.md)
