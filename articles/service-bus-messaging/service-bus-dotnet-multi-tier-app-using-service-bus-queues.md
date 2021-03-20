---
title: Многоуровневое приложение .NET, использующее служебную шину Azure | Документация Майкрософт
description: Руководство для .NET, посвященное разработке многоуровневого приложения в Azure, в котором для взаимодействия между уровнями используются очереди служебной шины.
ms.devlang: dotnet
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 40529df5195a29fbf2ff4887311932c2ffbf471d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96029901"
---
# <a name="net-multi-tier-application-using-azure-service-bus-queues"></a>Многоуровневое приложение .NET, использующее очереди служебной шины Azure

С Visual Studio и бесплатным пакетом Azure SDK для .NET разрабатывать решения для Microsoft Azure очень просто. В этом руководстве рассматриваются действия по созданию приложения, в котором используется несколько ресурсов Azure, работающих в локальной среде.

Вы узнаете следующее:

* Как подготовить свой компьютер к разработке приложений для Azure, загрузив и установив всего один пакет.
* Как разработать приложение для Azure с помощью Visual Studio.
* Как создать многоуровневое приложение в Azure с использованием рабочей роли и веб-роли.
* Обеспечение обмена данными между уровнями с помощью очередей служебной шины.

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

В этом учебнике вы создадите и запустите многоуровневое приложение в облачной службе Azure. Внешний интерфейс реализован с использованием веб-роли MVC ASP.NET, а серверная часть — с помощью рабочей роли, в которой используется очередь служебной шины. Вы также можете создать аналогичное многоуровневое приложение. Его внешний интерфейс будет реализован в виде веб-проекта, развернутого на веб-сайте Azure, а не в облачной службе. Кроме того, можно ознакомиться с руководством по [гибридным локальным и облачным приложениям .NET](../azure-relay/service-bus-dotnet-hybrid-app-using-service-bus-relay.md).

На следующем снимке экрана показано готовое приложение.

![Снимок экрана со страницей отправки приложения.][0]

## <a name="scenario-overview-inter-role-communication"></a>Обзор сценария: обмен данными между ролями
Чтобы отправить заказ на обработку, компоненту внешнего пользовательского интерфейса, выполняющемуся в веб-роли, необходимо взаимодействовать с логикой среднего уровня в рабочей роли. В этом примере в качестве посредника при обмене данными между уровнями выступает служебная шина.

Обмен сообщениями между веб-уровнем и средним уровнем через служебную шину позволяет разделить два компонента. В отличие от прямого обмена сообщениями (TCP или HTTP), веб-уровень не связан непосредственно со средним уровнем. Вместо этого он передает рабочие единицы в виде сообщений в служебную шину, где они хранятся до тех пор, пока не будут востребованы для обработки средним уровнем.

В служебной шине обмен сообщениями через брокер реализуется с использованием двух видов сущностей — очередей и разделов. При использовании очередей каждое сообщение, помещенное в очередь, потребляется одним получателем. При использовании разделов каждое опубликованное сообщение становится доступным в рамках подписки, зарегистрированной с помощью этого раздела. Для каждой подписки ведется собственная логическая очередь сообщений. Кроме того, для подписки можно настроить правила фильтрации, с помощью которых ограничивается набор сообщений, передаваемых в очередь подписки. В этом примере используются очереди служебной шины.

![Схема взаимодействия между веб-ролью, служебной шиной и рабочей ролью.][1]

По сравнению с прямым обменом сообщениями этот механизм взаимодействия имеет ряд преимуществ.

* **Временное разделение.** В асинхронной модели обмена отправители и получатели сообщений не обязательно должны находиться в сети одновременно. Служба Service Bus хранит сообщения до тех пор, пока принимающая сторона не будет готова принять их. Это позволяет разделять компоненты распределенного приложения как в плановых ситуациях (например, для обслуживания), так и в экстренных случаях, чтобы исключить влияние сбоя одного компонента на систему в целом. Более того, потребляющему приложению достаточно находиться в сети несколько раз в день в определенное время.
* **Выравнивание нагрузки.** Во многих приложениях уровень нагрузки на систему меняется с течением времени, тогда как время, затрачиваемое на обработку каждой рабочей единицы, как правило, постоянно. Благодаря обмену сообщениями через посредника с использованием очереди потребляющее приложение (рабочая роль) достаточно подготовить для обработки средней, а не пиковой нагрузки. При колебаниях входящей нагрузки просто изменяется глубина очереди. Это позволяет заметно сократить расходы на инфраструктуру, необходимую для обработки нагрузки приложения.
* **Балансировка нагрузки.** По мере возрастания нагрузки могут потребоваться дополнительные рабочие процессы для чтения из очереди. Каждое сообщение обрабатывается одним рабочим процессом. Кроме того, балансировка нагрузки по запросу обеспечивает оптимальное использование ресурсов рабочих машин с разной вычислительной мощностью, поскольку каждая машина может извлекать сообщения со своей максимальной скоростью. данный шаблон часто называется *конкурирующий потребитель*
  
  ![Схема, показывающая взаимодействие между веб-ролью, служебной шиной и двумя рабочими ролями.][2]

В следующих разделах описывается код, позволяющий реализовать подобную архитектуру.

## <a name="create-a-namespace"></a>Создание пространства имен

Сначала нужно создать *пространство имен* и получить [ключ подписанного URL-адреса (SAS)](service-bus-sas.md) для него. Пространство имен определяет границы каждого приложения, предоставляемого через служебную шину. Ключ SAS создается системой при создании пространства имен. Сочетание имени пространства имен и ключа SAS дает учетные данные, на основе которых служебная шина осуществляет проверку подлинности и предоставляет доступ к приложению.

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="create-a-web-role"></a>Создание веб-роли

В этом разделе описывается создание интерфейсной части вашего приложения. Сначала вы создадите страницы, которые будут отображаться в приложении,
а затем — добавите код для отправки элементов в очередь служебной шины и отображения сведений о ее состоянии.

### <a name="create-the-project"></a>Создание проекта

1. Запустите Visual Studio, используя привилегии администратора, щелкнув правой кнопкой мыши значок программы **Visual Studio** и выбрав пункт **Запустить от имени администратора**. В эмуляторе вычислений Azure, описанном далее в этой статье, требуется запуск Visual Studio с правами администратора.
   
   В меню **Файл** Visual Studio выберите **Создать**, а затем — **Проект**.
2. В разделе **Установленные шаблоны** области **Visual C#** щелкните **Облако**, а затем — **Облачная служба Azure**. Присвойте проекту имя **MultiTierApp**. Нажмите кнопку **ОК**.
   
   ![Снимок экрана: диалоговое окно "новый проект" с выбранным облаком и Visual C# облачной службы Azure, выделенное для красного цвета.][9]
3. На панели **Роли** дважды щелкните **Веб-роль ASP.NET**.
   
   ![Снимок экрана: диалоговое окно нового Microsoft Azure облачной службы с выбранной и WebRole1 веб-ролью ASP.NET.][10]
4. Наведите указатель мыши на элемент **WebRole1** в разделе **Решение облачной службы Azure**, щелкните значок карандаша и переименуйте веб-роль на **FrontendWebRole**. Нажмите кнопку **ОК**. (Обратите внимание на написание слова «Frontend» со строчной буквой «e» — не «FrontEnd».)
   
   ![Снимок экрана: диалоговое окно "Создание Microsoft Azure облачной службы" с переименованным решением в Фронтендвеброле.][11]
5. В диалоговом окне **Новый проект ASP.NET** в списке **Выбор шаблона** выберите **MVC**.
   
   ![Скриншотоф диалоговое окно Новый проект ASP.NET с выделенной MVC и выделяется красным цветом и параметром изменить проверку подлинности, обозначенный красным.][12]
6. В диалоговом окне **Новый проект ASP.NET** нажмите кнопку **Изменить способ проверки подлинности**. В диалоговом окне **Изменить способ проверки подлинности** щелкните **Без проверки подлинности** и нажмите кнопку **ОК**. В этом учебнике выполняется развертывание приложения, для которого не требуется имя пользователя.
   
    ![Снимок экрана: диалоговое окно изменения проверки подлинности с выбранным параметром без проверки подлинности и выделено красным цветом.][16]
7. В диалоговом окне **Новый проект ASP.NET** нажмите кнопку **ОК**, чтобы создать проект.
8. В **обозревателе решений** в проекте **FrontendWebRole** щелкните правой кнопкой мыши **Ссылки** и выберите пункт **Управление пакетами NuGet**.
9. Щелкните вкладку **Обзор**, а затем найдите **WindowsAzure.ServiceBus**. Выберите пакет **WindowsAzure.ServiceBus**, щелкните **Установить** и примите условия использования.
   
   ![Снимок экрана: диалоговое окно "Управление пакетами NuGet" с выделенным WindowsAzure. ServiceBus и параметром установки, отмеченным красным цветом.][13]
   
   Заметьте, что добавлены ссылки на требуемые клиентские сборки, а также новые файлы кода.
10. В **обозревателе решений** щелкните правой кнопкой мыши **Модели**, а затем выберите **Добавить** и **Класс**. В поле **Имя** введите имя **OnlineOrder.cs**. Нажмите кнопку **Добавить**.

### <a name="write-the-code-for-your-web-role"></a>Написание кода веб-роли
В этом разделе вы создадите разные страницы, которые будут отображаться в приложении.

1. В файле OnlineOrder.cs в Visual Studio замените существующее определение пространства имен следующим кодом:
   
   ```csharp
   namespace FrontendWebRole.Models
   {
       public class OnlineOrder
       {
           public string Customer { get; set; }
           public string Product { get; set; }
       }
   }
   ```
2. В **обозревателе решений** дважды щелкните **Controllers\HomeController.cs**. Добавьте в начало файла оператор **using**, чтобы включить служебную шину и пространства имен для созданной модели.
   
   ```csharp
   using FrontendWebRole.Models;
   using Microsoft.ServiceBus.Messaging;
   using Microsoft.ServiceBus;
   ```
3. Кроме того, в файле HomeController.cs в Visual Studio замените существующее определение пространства имен следующим кодом. Этот код содержит методы, обрабатывающие операции отправки элементов в очередь.
   
   ```csharp
   namespace FrontendWebRole.Controllers
   {
       public class HomeController : Controller
       {
           public ActionResult Index()
           {
               // Simply redirect to Submit, since Submit will serve as the
               // front page of this application.
               return RedirectToAction("Submit");
           }
   
           public ActionResult About()
           {
               return View();
           }
   
           // GET: /Home/Submit.
           // Controller method for a view you will create for the submission
           // form.
           public ActionResult Submit()
           {
               // Will put code for displaying queue message count here.
   
               return View();
           }
   
           // POST: /Home/Submit.
           // Controller method for handling submissions from the submission
           // form.
           [HttpPost]
           // Attribute to help prevent cross-site scripting attacks and
           // cross-site request forgery.  
           [ValidateAntiForgeryToken]
           public ActionResult Submit(OnlineOrder order)
           {
               if (ModelState.IsValid)
               {
                   // Will put code for submitting to queue here.
   
                   return RedirectToAction("Submit");
               }
               else
               {
                   return View(order);
               }
           }
       }
   }
   ```
4. В меню **Сборка** выберите **Собрать решение**, чтобы проверить правильность кода.
5. Теперь создайте представление для ранее созданного метода `Submit()`. Щелкните правой кнопкой мыши метод `Submit()` (перегрузка `Submit()`, при которой не принимаются параметры) и выберите пункт **Добавить представление**.
   
   ![Снимок экрана кода с фокусом в методе Submit и раскрывающимся списком с выделенным параметром "добавить представление".][14]
6. Откроется диалоговое окно создания представления. В списке **Шаблон** выберите **Создать**. В списке **Класс модели** выберите класс **OnlineOrder**.
   
   ![Снимок экрана диалогового окна Добавление представления с раскрывающимися списками шаблон и класс модели, выделенными красным цветом.][15]
7. Нажмите кнопку **Добавить**.
8. Теперь измените отображаемое имя приложения. В **обозревателе решений** дважды щелкните файл **Views\Shared\\_Layout.cshtml**, чтобы открыть его в редакторе Visual Studio.
9. Замените все вхождения элемента **Мое приложение ASP.NET** текстом **Продукты Northwind Traders**.
10. Удалите ссылки **Главная**, **О программе** и **Контакт**. Удалите выделенный код.
    
    ![Снимок экрана кода с тремя строками кода ссылки действия H T M L.][28]
11. Наконец, добавьте на страницу отправки необходимые сведения об очереди. В **обозревателе решений** дважды щелкните файл **Views\Home\Submit.cshtml**, чтобы открыть его в редакторе Visual Studio. Добавьте следующую строку после `<h2>Submit</h2>`. Сейчас элемент `ViewBag.MessageCount` пустой. Вы заполните его позже.
    
    ```html
    <p>Current number of orders in queue waiting to be processed: @ViewBag.MessageCount</p>
    ```
12. Пользовательский интерфейс реализован. Нажмите клавишу **F5**, чтобы запустить приложение и подтвердить его работоспособность.
    
    ![Снимок экрана со страницей отправки приложения.][17]

### <a name="write-the-code-for-submitting-items-to-a-service-bus-queue"></a>Написание кода для отправки элементов в очередь Service Bus
Теперь добавьте код для отправки элементов в очередь. Сначала вам нужно создать класс, содержащий информацию о подключении к очереди служебной шины, а затем — инициализировать подключение из Global.aspx.cs. И наконец, измените код отправки, созданный ранее в файле HomeController.cs, чтобы элементы отправлялись в очередь служебной шины.

1. В **обозревателе решений** щелкните правой кнопкой мыши проект **FrontendWebRole** (проект, не роль). Щелкните **Добавить** и **Класс**.
2. Присвойте классу имя **QueueConnector.cs**. Нажмите кнопку **Добавить**, чтобы создать класс.
3. Теперь добавьте код, который инкапсулирует сведения о подключении и инициализирует подключение к очереди служебной шины. Замените все содержимое файла QueueConnector.cs приведенным ниже кодом. Введите значения для параметров `your Service Bus namespace` (имя пространства имен) и `yourKey` (**первичный ключ**, ранее полученный на портале Azure).
   
   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Web;
   using Microsoft.ServiceBus.Messaging;
   using Microsoft.ServiceBus;
   
   namespace FrontendWebRole
   {
       public static class QueueConnector
       {
           // Thread-safe. Recommended that you cache rather than recreating it
           // on every request.
           public static QueueClient OrdersQueueClient;
   
           // Obtain these values from the portal.
           public const string Namespace = "your Service Bus namespace";
   
           // The name of your queue.
           public const string QueueName = "OrdersQueue";
   
           public static NamespaceManager CreateNamespaceManager()
           {
               // Create the namespace manager which gives you access to
               // management operations.
               var uri = ServiceBusEnvironment.CreateServiceUri(
                   "sb", Namespace, String.Empty);
               var tP = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                   "RootManageSharedAccessKey", "yourKey");
               return new NamespaceManager(uri, tP);
           }
   
           public static void Initialize()
           {
               // Using Http to be friendly with outbound firewalls.
               ServiceBusEnvironment.SystemConnectivity.Mode =
                   ConnectivityMode.Http;
   
               // Create the namespace manager which gives you access to
               // management operations.
               var namespaceManager = CreateNamespaceManager();
   
               // Create the queue if it does not exist already.
               if (!namespaceManager.QueueExists(QueueName))
               {
                   namespaceManager.CreateQueue(QueueName);
               }
   
               // Get a client to the queue.
               var messagingFactory = MessagingFactory.Create(
                   namespaceManager.Address,
                   namespaceManager.Settings.TokenProvider);
               OrdersQueueClient = messagingFactory.CreateQueueClient(
                   "OrdersQueue");
           }
       }
   }
   ```
4. Теперь нужно обеспечить вызов метода **Initialize**. В **обозревателе решений** дважды щелкните **Global.asax\Global.asax.cs**.
5. В конце метода **Application_Start** добавьте следующую строку кода.
   
   ```csharp
   FrontendWebRole.QueueConnector.Initialize();
   ```
6. Наконец, обновите ранее созданный код веб-уровня, чтобы отправить элементы в очередь. В **обозревателе решений** дважды щелкните **Controllers\HomeController.cs**.
7. Обновите метод `Submit()` (перегрузка, при которой не принимаются параметры), как показано ниже, чтобы получить число сообщений в очереди.
   
   ```csharp
   public ActionResult Submit()
   {
       // Get a NamespaceManager which allows you to perform management and
       // diagnostic operations on your Service Bus queues.
       var namespaceManager = QueueConnector.CreateNamespaceManager();
   
       // Get the queue, and obtain the message count.
       var queue = namespaceManager.GetQueue(QueueConnector.QueueName);
       ViewBag.MessageCount = queue.MessageCount;
   
       return View();
   }
   ```
8. Обновите метод `Submit(OnlineOrder order)` (перегрузка, при которой не принимаются параметры), как показано ниже, чтобы отправить сведения о заказе в очередь.
   
   ```csharp
   public ActionResult Submit(OnlineOrder order)
   {
       if (ModelState.IsValid)
       {
           // Create a message from the order.
           var message = new BrokeredMessage(order);
   
           // Submit the order.
           QueueConnector.OrdersQueueClient.Send(message);
           return RedirectToAction("Submit");
       }
       else
       {
           return View(order); 
       }
   }
   ```
9. Теперь можно снова запустить приложение. Каждый раз при отправке заказа значение счетчика сообщений увеличивается.
   
   ![Снимок экрана страницы отправки приложения с числом сообщений, увеличивающимся до 1.][18]

## <a name="create-the-worker-role"></a>Создание рабочей роли
Сейчас вы создадите рабочую роль, которая будет обрабатывать отправку заказов. В этом примере используется шаблон проекта Visual Studio **Worker Role with Service Bus Queue** (Рабочая роль с очередью служебной шины). Необходимые учетные данные уже получены с портала.

1. Убедитесь, что среда Visual Studio подключена к вашей учетной записи Azure.
2. В **обозревателе решений** Visual Studio щелкните правой кнопкой мыши папку **Роли** в проекте **MultiTierApp**.
3. Нажмите кнопку **Добавить**, а затем щелкните **Создать проект рабочей роли**. Откроется диалоговое окно **Добавить новый проект роли**.
   
   ![Снимок экрана: панель обозревателя решение с выбранным параметром "создать проект рабочей роли" и "Добавить".][26]
4. В окне **Добавить новый проект роли** щелкните **Worker Role with Service Bus Queue** (Рабочая роль с очередью служебной шины).
   
   ![Снимок экрана: диалоговое окно "Создание проекта роли в Active Directory" с выделенным параметром "Рабочая роль с очередью служебной шины" и красным цветом.][23]
5. В поле **Имя** введите имя проекта **OrderProcessingRole**. Нажмите кнопку **Добавить**.
6. Скопируйте в буфер обмена строку подключения, полученную на шаге 9 в разделе "Создание пространства имен служебной шины".
7. В **обозревателе решений** щелкните правой кнопкой мыши элемент **OrderProcessingRole**, созданный на этапе 5 (это должен быть элемент **OrderProcessingRole** в разделе **Роли**, а не класс с аналогичным названием). Выберите **Свойства**.
8. В диалоговом окне **Свойства** на вкладке **Настройки** щелкните внутри поля **Значение** для элемента **Microsoft.ServiceBus.ConnectionString** и вставьте значение конечной точки, скопированное на этапе 6.
   
   ![Снимок экрана: диалоговое окно "Свойства" с выбранной вкладкой "Параметры" и строкой таблицы Microsoft. ServiceBus. ConnectionString, выделенной красным цветом.][25]
9. Создайте класс **OnlineOrder**, который будет представлять заказы из очереди по мере их обработки. При необходимости используйте ранее созданный класс. В **обозревателе решений** щелкните правой кнопкой мыши класс **OrderProcessingRole** (значок класса, а не роль). Нажмите кнопку **Добавить** и выберите **Существующий элемент**.
10. Перейдите во вложенную папку **FrontendWebRole\Models** и дважды щелкните файл **OnlineOrder.cs**, чтобы добавить его в проект.
11. В файле **WorkerRole.cs** измените значение переменной **QueueName** с `"ProcessingQueue"` на `"OrdersQueue"`, как показано ниже в коде.
    
    ```csharp
    // The name of your queue.
    const string QueueName = "OrdersQueue";
    ```
12. Добавьте следующую инструкцию using в начало файла WorkerRole.cs.
    
    ```csharp
    using FrontendWebRole.Models;
    ```
13. В функции `Run()` в вызове `OnMessage()` замените содержимое предложения `try` следующим кодом.
    
    ```csharp
    Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());
    // View the message as an OnlineOrder.
    OnlineOrder order = receivedMessage.GetBody<OnlineOrder>();
    Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");
    receivedMessage.Complete();
    ```
14. Приложение готово. Чтобы протестировать готовое приложение, нужно щелкнуть правой кнопкой мыши проект MultiTierApp в обозревателе решений, потом выбрать команду **Назначить запускаемым проектом**, а затем нажать клавишу F5. Обратите внимание, что значение счетчика сообщений не увеличивается, поскольку рабочая роль обрабатывает элементы из очереди и помечает их выполненными. Результаты трассировки рабочей роли вы можете просмотреть в пользовательском интерфейсе эмулятора среды выполнения приложений Azure. Для этого щелкните правой кнопкой мыши значок эмулятора в области уведомлений на панели задач и выберите **Show Compute Emulator UI** (Показать пользовательский интерфейс эмулятора вычислений).
    
    ![Снимок экрана, отображаемый при щелчке значка эмулятора. Отображать пользовательский интерфейс эмулятора вычислений — в списке параметров.][19]
    
    ![Снимок экрана: диалоговое окно "эмулятор вычислений Microsoft Azure (Express)".][20]

## <a name="next-steps"></a>Дальнейшие действия
Дополнительную информацию о Service Bus см. на следующих ресурсах.  

* [Начало работы с очередями служебной шины][sbacomqhowto]
* [Страница службы служебной шины][sbacom]  

Дополнительные сведения о многоуровневых сценариях см. здесь:  

* [Многоуровневое приложение .NET, использующее таблицы, очереди и большие двоичные объекты хранилища][mutitierstorage]  

[0]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png
[1]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png
[2]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png
[9]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.png
[10]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png
[11]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png
[12]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png
[13]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png
[14]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-33.png
[15]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png
[16]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-14.png
[17]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png
[18]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app2.png

[19]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png
[20]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png
[23]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png
[25]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRoleProperties.png
[26]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png
[28]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png

[sbacom]: https://azure.microsoft.com/services/service-bus/  
[sbacomqhowto]: service-bus-dotnet-get-started-with-queues.md  
[mutitierstorage]: https://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36
