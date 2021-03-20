---
title: Отладка приложения в Visual Studio
description: Повысьте надежность и производительность служб, разрабатывая их в Visual Studio и локальном кластере разработки.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 0b7d08d610c883240abedc66c55abba64a74c8e3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96576321"
---
# <a name="debug-your-service-fabric-application-by-using-visual-studio"></a>Отладка приложения Service Fabric с помощью Visual Studio
> [!div class="op_single_selector"]
> * [Visual Studio и CSharp](service-fabric-debugging-your-application.md) 
> * [Eclipse и Java](service-fabric-debugging-your-application-java.md)
>


## <a name="debug-a-local-service-fabric-application"></a>Отладка локального приложения Service Fabric
Вы можете сэкономить время и деньги, развернув приложение Azure Service Fabric и выполнив его отладку в кластере для разработки, состоящем из локальных компьютеров. Visual Studio 2019 или 2015 может развернуть приложение в локальном кластере и автоматически подключить отладчик ко всем экземплярам приложения. Для подключения отладчика необходимо запустить Visual Studio от имени администратора.

1. Запустите кластер локальной разработки, выполнив действия, описанные в статье [Настройка среды разработки Service Fabric](service-fabric-get-started.md).
2. Нажмите клавишу **F5** или выберите в меню **Отладка** > **Начать отладку**.
   
    ![Снимок экрана, показывающий меню отладки.][startdebugging]
3. Задайте в коде точки останова и поэтапно выполните приложение, выбирая соответствующие команды в меню **Отладка** .
   
   > [!NOTE]
   > Visual Studio подключается ко всем экземплярам приложения. При пошаговом выполнении кода разные процессы могут одновременно достигать точек останова. Это приводит к возникновению параллельных сеансов. Чтобы этого не происходило, настройте отключение точек останова после их достижения, сделав их зависимыми от идентификатора потока, или используйте события диагностики.
   > 
   > 
4. Окно **События диагностики** открывается автоматически и позволяет просматривать данные о событиях диагностики в реальном времени.
   
    ![Просмотр событий диагностики в режиме реального времени][diagnosticevents]
5. Кроме того, вы можете открыть окно **События диагностики** вручную, используя обозреватель облака.  В разделе **Service Fabric** щелкните любой узел правой кнопкой мыши и выберите пункт **Показать потоковую передачу трассировок**.
   
    ![Окно событий диагностики][viewdiagnosticevents]
   
    Если вы хотите отфильтровать трассировки для определенной службы или приложения, включите потоковую трассировку для этой конкретной службы или приложения.
6. События диагностики можно просматривать в автоматически созданном файле **ServiceEventSource.cs** и вызывать из кода приложения.
   
    ```csharp
    ServiceEventSource.Current.ServiceMessage(this, "My ServiceMessage with a parameter {0}", result.Value.ToString());
    ```
7. В окне **События диагностики** поддерживаются фильтрация, приостановка и проверка событий в реальном времени.  Фильтр представляет собой простой поиск по строкам сообщений о событиях, включая их содержимое.
   
    ![Фильтрация, приостановка, возобновление и проверка событий в реальном времени][diagnosticeventsactions]
8. Отладка служб аналогична отладке приложений. Обычно точки останова устанавливаются с помощью Visual Studio для простоты отладки. Несмотря на то, что надежные коллекции реплицируются на нескольких узлах, они все равно реализуют интерфейс IEnumerable. Эта реализация означает, что вы можете использовать представление результатов в Visual Studio во время отладки, чтобы увидеть, что вы сохранили в. Для этого установите точку останова в любом месте кода.
   
    ![Запуск отладки приложения][breakpoint]


### <a name="running-a-script-as-part-of-debugging"></a>Запуск скрипта в процессе отладки
В некоторых сценариях может потребоваться запустить сценарий в процессе запуска сеанса отладки (например, если службы по умолчанию не используются).

В Visual Studio можно добавить файл с именем **Start-Service.ps1** в папку **Scripts** проекта приложения Service Fabric (. sfproj). Этот скрипт будет вызываться после создания приложения в локальном кластере.


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## <a name="debug-a-remote-service-fabric-application"></a>Отладка удаленного приложения Service Fabric
Если Service Fabric приложения выполняются в кластере Service Fabric в Azure, вы можете выполнять удаленную отладку этих приложений непосредственно из Visual Studio.

> [!NOTE]
> Компоненту требуется [пакет SDK 2.0 для Service Fabric](https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015) и [пакет SDK Azure для .NET 2.9](https://azure.microsoft.com/downloads/).    

<!-- -->
> [!WARNING]
> Удаленная отладка предназначена для сценариев разработки и тестирования и не должна использоваться в производственной среде, поскольку влияет на запущенные приложения.

1. Перейдите к кластеру в **Cloud Explorer**. Щелкните правой кнопкой мыши и выберите **включить отладку** .
   
    ![Включение удаленной отладки][enableremotedebugging]
   
    Это действие запускает процесс включения расширения удаленной отладки на узлах кластера и требуемых конфигураций сети.
2. Щелкните узел кластера в **обозревателе облака** правой кнопкой мыши и выберите пункт **Подключить отладчик**.
   
    ![Подключить отладчик][attachdebugger]
3. В диалоговом окне **Подключение к процессу** выберите процесс, который нужно отладить, и нажмите кнопку **Подключить**.
   
    ![Выбор процесса][chooseprocess]
   
    Имя процесса, к которому нужно подключиться, совпадает с именем сборки для проекта службы.
   
    Отладчик подключится ко всем узлам, на которых выполняется процесс.
   
   * Если выполняется отладка службы без отслеживания состояния, все экземпляры службы на всех узлах являются частью сеанса отладки.
   * При отладке службы с отслеживанием состояния только первичная реплика любого раздела будет активна и, следовательно, перехвачена отладчиком. Если во время сеанса отладки первичная реплика переносится, обработка этой реплики по-прежнему входит в сеанс отладки.
   * Чтобы перехватить только соответствующие секции или экземпляры данной службы, можно использовать условные точки останова, чтобы только приостановить определенную секцию или экземпляр.
     
     ![Условная точка останова][conditionalbreakpoint]
     
     > [!NOTE]
     > Сейчас отладка кластера Service Fabric с несколькими экземплярами исполняемого имени одной и той же службы не поддерживается.
     > 
     > 
4. После того как отладка приложения будет завершена, расширение удаленной отладки можно отключить, щелкнув кластер в **обозревателе облака** правой кнопкой мыши и выбрав пункт **Отключить отладку**.
   
    ![Отключение удаленной отладки][disableremotedebugging]

## <a name="streaming-traces-from-a-remote-cluster-node"></a>Потоковая передача трассировок из удаленного узла кластера
Вы также можете выполнять потоковую передачу трассировок непосредственно с удаленного узла кластера в Visual Studio. Эта функция позволяет передавать события трассировки ETW, возникающие на узле кластера Service Fabric.

> [!NOTE]
> Компоненту требуется [пакет SDK 2.0 для Service Fabric](https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015) и [пакет SDK Azure для .NET 2.9](https://azure.microsoft.com/downloads/).
> Эта функция поддерживает только кластеры на платформе Azure.
> 
> 

<!-- -->
> [!WARNING]
> Потоковая передача трассировок предназначена для сценариев разработки и тестирования и не должна использоваться в производственной среде, поскольку влияет на запущенные приложения.
> В рабочем сценарии следует применять пересылку событий с помощью средств диагностики Azure.

1. Перейдите к кластеру в **Cloud Explorer**. Щелкните правой кнопкой мыши и выберите **включить трассировки потоковой передачи** .
   
    ![Включение трассировки удаленной потоковой передачи][enablestreamingtraces]
   
    Это действие запускает процесс включения расширения трассировки потоковой передачи на узлах кластера, а также требуемых конфигураций сети.
2. Разверните элемент **Узлы** в **обозревателе облака**, щелкните правой кнопкой мыши узел, потоковую передачу трассировок с которого вы хотите включить, и выберите пункт **Показать потоковую передачу трассировок**.
   
    ![Просмотр трассировки удаленной потоковой передачи][viewremotestreamingtraces]
   
    Повторите шаг 2 для всех узлов, трассировку которых вы хотите получать. Потоковая передача каждого узла будет отображаться в отдельном окне.
   
    Теперь вы можете также получать трассировки, которые выпускает Service Fabric и ваши собственные службы. Если вы хотите отфильтровать события по определенному приложению, просто введите название этого приложения в качестве фильтра.
   
    ![Просмотр трассировки потоковой передачи][viewingstreamingtraces]
3. После того как потоковая передача трассировок будет завершена, вы можете отключить удаленную потоковую передачу трассировок, щелкнув кластер в **обозревателе облака** правой кнопкой мыши и выбрав пункт **Отключить трассировки потоковой передачи**.
   
    ![Отключение трассировки удаленной потоковой передачи][disablestreamingtraces]

## <a name="next-steps"></a>Дальнейшие действия
* [Протестируйте службу Service Fabric](service-fabric-testability-overview.md).
* [Управление приложениями Service Fabric в Visual Studio](service-fabric-manage-application-in-visual-studio.md)

<!--Image references-->
[startdebugging]: ./media/service-fabric-debugging-your-application/startdebugging.png
[diagnosticevents]: ./media/service-fabric-debugging-your-application/diagnosticevents.png
[viewdiagnosticevents]: ./media/service-fabric-debugging-your-application/viewdiagnosticevents.png
[diagnosticeventsactions]: ./media/service-fabric-debugging-your-application/diagnosticeventsactions.png
[breakpoint]: ./media/service-fabric-debugging-your-application/breakpoint.png
[enableremotedebugging]: ./media/service-fabric-debugging-your-application/enableremotedebugging.png
[attachdebugger]: ./media/service-fabric-debugging-your-application/attachdebugger.png
[chooseprocess]: ./media/service-fabric-debugging-your-application/chooseprocess.png
[conditionalbreakpoint]: ./media/service-fabric-debugging-your-application/conditionalbreakpoint.png
[disableremotedebugging]: ./media/service-fabric-debugging-your-application/disableremotedebugging.png
[enablestreamingtraces]: ./media/service-fabric-debugging-your-application/enablestreamingtraces.png
[viewingstreamingtraces]: ./media/service-fabric-debugging-your-application/viewingstreamingtraces.png
[viewremotestreamingtraces]: ./media/service-fabric-debugging-your-application/viewremotestreamingtraces.png
[disablestreamingtraces]: ./media/service-fabric-debugging-your-application/disablestreamingtraces.png
