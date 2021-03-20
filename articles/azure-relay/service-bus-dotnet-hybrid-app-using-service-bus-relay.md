---
title: Гибридное локальное или облачное приложение ретранслятора Azure Windows Communication Foundation (WCF) | Документация Майкрософт
description: Здесь вы узнаете, как предоставлять локальную службу WCF для веб-приложения в облаке с помощью Azure Relay
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 22b582634b623b39545eca225c8df0130606c2bb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90983882"
---
# <a name="expose-an-on-premises-wcf-service-to-a-web-application-in-the-cloud-by-using-azure-relay"></a>Предоставление локальной службы WCF для веб-приложения в облаке с помощью Azure Relay

В этой статье показано, как создать гибридное облачное приложение с помощью Microsoft Azure и Visual Studio. Вы создадите приложение, использующее несколько ресурсов Azure в облаке. Этот учебник поможет вам изучить следующие задачи:

* Как создать или адаптировать существующую веб-службу для использования веб-решением.
* Как использовать службу ретранслятора Azure Windows Communication Foundation (WCF) для обмена данными между приложением Azure и веб-службой, размещенной в других местах.

В этом учебнике вы выполните следующие задачи:

> [!div class="checklist"]
>
> * Установка обязательных компонентов для работы с этим учебником.
> * Просмотр сценария.
> * Создание пространства имен.
> * Создайте локальный сервер.
> * Создайте приложение ASP .NET.
> * Запустите приложение на локальном компьютере.
> * Разверните веб-приложение в Azure.
> * Запустите приложение в Azure.

## <a name="prerequisites"></a>Предварительные требования

Для работы с данным руководством вам потребуется:

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начать работу.
* [Visual Studio 2015 или более поздней версии](https://www.visualstudio.com). В примерах для этого учебника используется Visual Studio 2019.
* Пакет Azure SDK для .NET. Установите его со [страницы загрузки SDK](https://azure.microsoft.com/downloads/).

## <a name="how-azure-relay-helps-with-hybrid-solutions"></a>Как ретранслятор Azure помогает работать с гибридными решениями

Бизнес-решения обычно состоят из сочетания пользовательского кода и существующих функциональных возможностей. Пользовательский код решает новые и уникальные бизнес-требования. Уже имеющиеся решения и системы предоставляют существующие функциональные возможности.

Архитекторы решений начинают использовать облако для упрощения реализации масштабирования и снижения эксплуатационных расходов. При этом они находят, что существующие ресурсы службы, которые они хотели бы использовать в качестве стандартных блоков для своих решений, находятся внутри корпоративного брандмауэра и не просто доступны облачному решению. Многие внутренние службы не строятся и не размещаются так, чтобы их можно было легко представить на границе корпоративной сети.

[Azure Relay](https://azure.microsoft.com/services/service-bus/) принимает существующие веб-службы WCF и делает эти службы безопасно доступными для решений, находящихся за пределами периметра организации, не требуя вмешательства в инфраструктуру корпоративной сети. Такие службы ретрансляции по-прежнему размещаются внутри существующей среды, однако они делегируют функции прослушивания входящих сеансов и запросов размещенной в облаке службе ретрансляции. Ретранслятор Azure также защищает эти службы от несанкционированного доступа, применяя аутентификацию на основе [подписанного URL-адреса](../service-bus-messaging/service-bus-sas.md) (SAS).

## <a name="review-the-scenario"></a>Просмотр сценария

В этом руководстве вы создадите веб-сайт ASP.NET, который позволит просматривать список продуктов на странице складских запасов.

![Сценарий][0]

В учебнике предполагается, что у вас есть сведения о продуктах в существующей локальной системе, для доступа к которой используется ретранслятор Azure. Такая ситуация имитируется веб-службой, запущенной в простом консольном приложении. Он содержит набор продуктов в памяти. Вы можете запустить это консольное приложение на своем компьютере и развернуть веб-роль в Azure. Это позволит увидеть, как веб-роль, выполняемая в центре обработки данных Azure, обращается к вашему компьютеру. Этот вызов происходит, даже если компьютер почти наверняка находится по крайней мере на одном брандмауэре и на уровне преобразования сетевых адресов (NAT).

## <a name="set-up-the-development-environment"></a>Настройка среды разработки

Прежде чем начать разработку приложения для Azure, скачайте нужные инструменты и настройте среду разработки.

1. Установите пакет SDK Azure для .NET, скачав его с [этой страницы](https://azure.microsoft.com/downloads/).
1. В столбце **.NET** выберите версию [Visual Studio](https://www.visualstudio.com) , которую вы используете. В этом руководстве используется Visual Studio 2019.
1. При появлении запроса на запуск или сохранение установщика выберите **Run (выполнить**).
1. В диалоговом окне **установщик веб-платформы** выберите **установить** и продолжить установку.

После завершения установки вы получите все необходимое для начала разработки приложения. В состав пакета SDK входят инструменты для эффективной разработки приложений Azure в Visual Studio.

## <a name="create-a-namespace"></a>Создание пространства имен

Первым шагом является создание пространства имен и получение ключа [подписанного URL-адрес (SAS)](../service-bus-messaging/service-bus-sas.md) . Пространство имен определяет границы каждого приложения, предоставляемого через службу ретрансляции. Ключ SAS автоматически создается системой при создании пространства имен службы. Сочетание пространства имен и ключа SAS образует учетные данные, на основе которых Azure осуществляет аутентификацию доступа к приложению.

[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="create-an-on-premises-server"></a>Создание локального сервера

Во-первых, вы создадите имитированную локальную систему каталогов продукции.  Этот проект запускается как консольное приложение Visual Studio, используя [пакет NuGet служебной шины Azure](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) для включения библиотек и параметров конфигурации служебной шины. <a name="create-the-project"></a>

1. Запустите Microsoft Visual Studio от имени администратора. Для этого щелкните правой кнопкой мыши значок программы Visual Studio и выберите **Запустить от имени администратора**.
1. В Visual Studio выберите **Создать проект**.
1. В окне **Создание нового проекта** выберите **консольное приложение (платформа .NET Framework)** для C# и нажмите кнопку **Далее**.
1. Присвойте проекту имя *ProductsServer* и выберите **создать**.

   ![Настройка нового проекта][11]

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект **ProductsServer** , а затем выберите **Управление пакетами NuGet**.
1. Нажмите **Обзор**, найдите и выберите **WindowsAzure.ServiceBus**. Нажмите **Установить** и примите условия использования.

   ![Установка пакета NuGet][13]

   На необходимые клиентские сборки теперь имеется ссылка.

1. Добавьте новый класс для контракта на свою продукцию. В **Обозреватель решений** щелкните правой кнопкой мыши проект **ProductsServer** и выберите пункт **добавить**  >  **класс**.
1. В поле **имя** введите имя *продуктсконтракт. CS* и нажмите кнопку **добавить**.

Внесите следующие изменения в код решения:

1. В *ProductsContract.cs* замените определение пространства имен на приведенный ниже код, определяющий контракт для службы.

    ```csharp
    namespace ProductsServer
    {
        using System.Collections.Generic;
        using System.Runtime.Serialization;
        using System.ServiceModel;

        // Define the data contract for the service
        [DataContract]
        // Declare the serializable properties.
        public class ProductData
        {
            [DataMember]
            public string Id { get; set; }
            [DataMember]
            public string Name { get; set; }
            [DataMember]
            public string Quantity { get; set; }
        }

        // Define the service contract.
        [ServiceContract]
        interface IProducts
        {
            [OperationContract]
            IList<ProductData> GetProducts();

        }

        interface IProductsChannel : IProducts, IClientChannel
        {
        }
    }
    ```

1. В *Program. CS* Замените определение пространства имен следующим кодом, который добавляет службу профиля и узел для нее.

    ```csharp
    namespace ProductsServer
    {
        using System;
        using System.Linq;
        using System.Collections.Generic;
        using System.ServiceModel;

        // Implement the IProducts interface.
        class ProductsService : IProducts
        {

            // Populate array of products for display on website
            ProductData[] products =
                new []
                    {
                        new ProductData{ Id = "1", Name = "Rock",
                                         Quantity = "1"},
                        new ProductData{ Id = "2", Name = "Paper",
                                         Quantity = "3"},
                        new ProductData{ Id = "3", Name = "Scissors",
                                         Quantity = "5"},
                        new ProductData{ Id = "4", Name = "Well",
                                         Quantity = "2500"},
                    };

            // Display a message in the service console application
            // when the list of products is retrieved.
            public IList<ProductData> GetProducts()
            {
                Console.WriteLine("GetProducts called.");
                return products;
            }

        }

        class Program
        {
            // Define the Main() function in the service application.
            static void Main(string[] args)
            {
                var sh = new ServiceHost(typeof(ProductsService));
                sh.Open();

                Console.WriteLine("Press ENTER to close");
                Console.ReadLine();

                sh.Close();
            }
        }
    }
    ```

1. В **обозревателе решений** дважды щелкните файл **App.config**, чтобы открыть его в редакторе Visual Studio. В нижней части `<system.ServiceModel>` элемента, но по-прежнему в `<system.ServiceModel>` добавьте следующий XML-код. Не забудьте заменить `yourServiceNamespace` именем вашего пространства имен и `yourKey` ключом SAS, полученным ранее с портала:

    ```xml
    <system.serviceModel>
    ...
      <services>
         <service name="ProductsServer.ProductsService">
           <endpoint address="sb://yourServiceNamespace.servicebus.windows.net/products" binding="netTcpRelayBinding" contract="ProductsServer.IProducts" behaviorConfiguration="products"/>
         </service>
      </services>
      <behaviors>
         <endpointBehaviors>
           <behavior name="products">
             <transportClientEndpointBehavior>
                <tokenProvider>
                   <sharedAccessSignature keyName="RootManageSharedAccessKey" key="yourKey" />
                </tokenProvider>
             </transportClientEndpointBehavior>
           </behavior>
         </endpointBehaviors>
      </behaviors>
    </system.serviceModel>
    ```

    > [!NOTE]
    > Ошибка, вызванная, `transportClientEndpointBehavior` — это просто предупреждение, а не блокирующая проблема в этом примере.

1. По-прежнему в *App.config* в `<appSettings>` элементе замените значение строки соединения строкой подключения, полученной ранее с портала.

    ```xml
    <appSettings>
       <!-- Service Bus specific app settings for messaging connections -->
       <add key="Microsoft.ServiceBus.ConnectionString"
           value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey"/>
    </appSettings>
    ```

1. Нажмите сочетание клавиш CTRL + SHIFT + B или выберите **Build**  >  **Build Solution** (сборка сборки), чтобы создать приложение и проверить точность работы на данный момент.

## <a name="create-an-aspnet-application"></a>Создание приложения ASP.NET

В этом разделе вы создадите простое приложение ASP.NET, которое будет отображать данные, полученные из службы продукции.

### <a name="create-the-project"></a>Создание проекта

1. Убедитесь, что Visual Studio работает от имени администратора.
1. В Visual Studio выберите **Создать проект**.
1. В окне **Создание нового проекта** выберите **ASP.NET веб-приложение (платформа .NET Framework)** для C# и нажмите кнопку **Далее**.
1. Присвойте проекту имя *ProductsPortal* и выберите **создать**.
1. В разделе **Создание нового веб-приложения ASP.NET** выберите **MVC** и щелкните **изменить** под **проверкой подлинности**.

   ![Выбор веб-приложения ASP.NET][16]

1. В окне **Изменение проверки подлинности** выберите **без проверки подлинности** и нажмите кнопку **ОК**. В этом руководстве вы развертываете приложение, для входа в котором не требуется пользователь.

    ![Указание аутентификации][18]

1. Вернитесь к странице **Создание нового веб-приложения ASP.NET** и выберите **создать** , чтобы создать приложение MVC.
1. Настройка ресурсов Azure для нового веб-приложения. Выполните действия, описанные в статье [Публикация веб-приложения](../app-service/quickstart-dotnet-framework.md#launch-the-publish-wizard). Затем вернитесь к этому учебнику и перейдите к следующему шагу.
1. В **Обозреватель решений** щелкните правой кнопкой мыши элемент **модели** и выберите команду **добавить**  >  **класс**.
1. Назовите класс *Product. CS*, а затем нажмите кнопку **Добавить**.

    ![Создание модели продукта][17]

### <a name="modify-the-web-application"></a>Изменение веб-приложения

1. В файле *Product. CS* в Visual Studio замените существующее определение пространства имен следующим кодом:

   ```csharp
    // Declare properties for the products inventory.
    namespace ProductsWeb.Models
    {
       public class Product
       {
           public string Id { get; set; }
           public string Name { get; set; }
           public string Quantity { get; set; }
       }
    }
    ```

1. В **Обозреватель решений** разверните узел **контроллеры**, а затем дважды щелкните **HomeController. CS** , чтобы открыть файл в Visual Studio.
1. В файле *HomeController.cs* замените имеющееся определение пространства имен следующим кодом:

    ```csharp
    namespace ProductsWeb.Controllers
    {
        using System.Collections.Generic;
        using System.Web.Mvc;
        using Models;

        public class HomeController : Controller
        {
            // Return a view of the products inventory.
            public ActionResult Index(string Identifier, string ProductName)
            {
                var products = new List<Product>
                    {new Product {Id = Identifier, Name = ProductName}};
                return View(products);
            }
         }
    }
    ```

1. В **Обозреватель решений** разверните Общие **представления**  >  , а затем дважды щелкните **_layout. cshtml** , чтобы открыть файл в редакторе Visual Studio.
1. Измените все вхождения `My ASP.NET Application` на *продукты Northwind Traders*.
1. Удалите `Home` ссылки, `About` и `Contact` . В следующем примере удалите выделенный код.

    ![Удаление созданных элементов списка][41]

1. В **Обозреватель решений** разверните узел **представления**  >  **Главная**, а затем дважды щелкните **index. cshtml** , чтобы открыть файл в редакторе Visual Studio. Замените все содержимое файла следующим кодом:

   ```html
   @model IEnumerable<ProductsWeb.Models.Product>

   @{
            ViewBag.Title = "Index";
   }

   <h2>Prod Inventory</h2>

   <table>
             <tr>
                 <th>
                     @Html.DisplayNameFor(model => model.Name)
                 </th>
                 <th></th>
                 <th>
                     @Html.DisplayNameFor(model => model.Quantity)
                 </th>
             </tr>

   @foreach (var item in Model) {
             <tr>
                 <td>
                     @Html.DisplayFor(modelItem => item.Name)
                 </td>
                 <td>
                     @Html.DisplayFor(modelItem => item.Quantity)
                 </td>
             </tr>
   }

   </table>
   ```

1. Чтобы проверить точность выполненной работы, можно нажать клавиши CTRL + SHIFT + B, чтобы построить проект.

### <a name="run-the-app-locally"></a>Локальный запуск приложения

Запустите приложение, чтобы убедиться, что оно работает.

1. **ProductsPortal** должен быть активным проектом. Щелкните правой кнопкой мыши имя проекта в **Обозреватель решений** и выберите **Назначить запускаемым проектом**.
1. В Visual Studio выберите F5.

Приложение должно начать выполняться в браузере.

![На снимке экрана показан пример приложения, выполняемого в браузере с выделенным URL-адресом.][21]

## <a name="put-the-pieces-together"></a>Объединение элементов

Следующим шагом является подключение локального сервера продуктов к приложению ASP.NET.

1. Если он еще не открыт, в Visual Studio откройте проект **ProductsPortal** , созданный в разделе [Создание приложения ASP.NET](#create-an-aspnet-application) .
1. Как и на шаге в разделе [Создание локального сервера](#create-an-on-premises-server) , добавьте пакет NuGet в ссылки проекта. В **Обозреватель решений** щелкните правой кнопкой мыши проект **ProductsPortal** , а затем выберите **Управление пакетами NuGet**.
1. Выполните поиск по фразе *WindowsAzure.ServiceBus* и выберите элемент **WindowsAzure.ServiceBus**. Затем завершите установку и закройте это диалоговое окно.
1. В **Обозреватель решений** щелкните правой кнопкой мыши проект **ProductsPortal** , а затем выберите **добавить**  >  **существующий элемент**.
1. Перейдите к файлу *ProductsContract.cs* в консольном проекте **ProductsServer**. Выделите *продуктсконтракт. CS*. Щелкните стрелку вниз рядом с пунктом **Добавить**, а затем выберите **Добавить как связь**.

   ![Добавление в виде ссылки][24]

1. Теперь откройте файл *HomeController.cs* в редакторе Visual Studio и замените существующее определение пространства имен приведенным ниже кодом. Не забудьте заменить `yourServiceNamespace` именем пространства имен службы и `yourKey` ключом SAS. Этот код позволяет клиенту вызывать локальную службу, возвращая результат вызова.

   ```csharp
   namespace ProductsWeb.Controllers
   {
       using System.Linq;
       using System.ServiceModel;
       using System.Web.Mvc;
       using Microsoft.ServiceBus;
       using Models;
       using ProductsServer;

       public class HomeController : Controller
       {
           // Declare the channel factory.
           static ChannelFactory<IProductsChannel> channelFactory;

           static HomeController()
           {
               // Create shared access signature token credentials for authentication.
               channelFactory = new ChannelFactory<IProductsChannel>(new NetTcpRelayBinding(),
                   "sb://yourServiceNamespace.servicebus.windows.net/products");
               channelFactory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior {
                   TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                       "RootManageSharedAccessKey", "yourKey") });
           }

           public ActionResult Index()
           {
               using (IProductsChannel channel = channelFactory.CreateChannel())
               {
                   // Return a view of the products inventory.
                   return this.View(from prod in channel.GetProducts()
                                    select
                                        new Product { Id = prod.Id, Name = prod.Name,
                                            Quantity = prod.Quantity });
               }
           }
       }
   }
   ```

1. В **Обозреватель решений** щелкните правой кнопкой мыши решение **ProductsPortal** . Не забудьте щелкнуть решение правой кнопкой мыши, а не проект. Выберите **Добавить** > **Существующий проект**.
1. Перейдите к проекту **ProductsServer**, а затем дважды щелкните файл решения *ProductsServer.csproj*, чтобы добавить его.
1. Чтобы отобразить данные в **ProductsPortal**, необходимо запустить **ProductsServer** . В **Обозреватель решений** щелкните правой кнопкой мыши решение **ProductsPortal** и выберите **Свойства** , чтобы отобразить **страницы свойств**.
1. Выберите пункт **Общие свойства**  >  **запускаемый проект** и выберите **Несколько запускаемых проектов**. Убедитесь, что **ProductsServer** и **ProductsPortal** отображаются в указанном порядке и что **действие** для обоих них **запущено**.

      ![Несколько запускаемых проектов][25]

1. Выберите **Общие свойства**  >  **зависимости проекта** в левой части.
1. Для **проектов** выберите **ProductsPortal**. Установите флажок **ProductsServer**.

    ![Зависимости проектов][26]

1. Для **проектов** выберите **ProductsServer**. Убедитесь, что **ProductsPortal** не выбран, а затем нажмите кнопку **ОК** , чтобы сохранить изменения.

## <a name="run-the-project-locally"></a>Локальный запуск проекта

Чтобы протестировать приложение локально, в Visual Studio выберите F5. Сначала должен начаться локальный сервер **ProductsServer**, после чего приложение **ProductsPortal** должно запуститься в окне браузера. На этот раз вы сможете проследить получение данных списка складских запасов продукции из локальной системы службы продукции.

![Веб-приложение][10]

На странице **ProductsPortal** выберите **Обновить** . При каждом обновлении страницы после вызова `GetProducts()` из **ProductsServer** в серверном приложении будет отображаться сообщение.

Закройте оба приложения, прежде чем перейти к следующему разделу.

## <a name="deploy-the-productsportal-project-to-an-azure-web-app"></a>Развертывание проекта ProductsPortal в веб-приложении Azure

Следующим шагом является повторная публикация внешнего интерфейса веб-приложения Azure **ProductsPortal** :

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект **ProductsPortal** и выберите Publish ( **опубликовать**). На странице **Публикация** нажмите кнопку **Опубликовать**.

   > [!NOTE]
   > Если веб-проект **ProductsPortal** запустится после развертывания автоматически, в окне браузера может появиться сообщение об ошибке. Это ожидаемое поведение, и означает, что приложение **ProductsServer** еще не запущено.
   >

1. Скопируйте URL развернутого веб-приложения. Этот URL-адрес потребуется позже. Этот URL-адрес можно также получить из окна **действия службы приложений Azure** в Visual Studio:

   ![URL-адрес развернутого приложения][9]

1. Закройте окно браузера, чтобы остановить выполняющееся приложение.

<a name="set-productsportal-as-web-app"></a>Перед запуском приложения в облаке необходимо убедиться, что **ProductsPortal** запускается из Visual Studio в качестве веб-приложения.

1. В Visual Studio щелкните правой кнопкой мыши проект **ProductsPortal** и выберите пункт **свойства**.
1. Выберите **Интернет**. В разделе **действие при запуске** выберите **начальный URL-адрес**. Введите URL-адрес ранее развернутого веб-приложения, в этом примере — `https://productsportal20190906122808.azurewebsites.net/` .

    ![Начальный URL-адрес][27]

1. Нажмите **Файл** > **Сохранить все**.
1. Выберите **Сборка**  >  **Перестроение решения**.

## <a name="run-the-application"></a>Выполнение приложения

Нажмите клавишу F5, чтобы создать и запустить приложение. Сначала должен начаться локальный сервер, который является консольным приложением **ProductsServer** , а затем приложение **ProductsPortal** должно запускаться в окне браузера, как показано ниже:

   ![Запуск веб-приложения в Azure][1]

В перечне продуктов перечисляются данные, полученные из локальной системы службы продуктов, и отображаются эти данные в веб-приложении. Проверьте URL-адрес. Приложение **ProductsPortal** должно запускаться в облаке как веб-приложение Azure.

   > [!IMPORTANT]
   > Консольное приложение **ProductsServer** должно запускаться с возможностью передачи данных в приложение **ProductsPortal**. Если в браузере отображается сообщение об ошибке, подождите несколько секунд для загрузки **ProductsServer** и отобразите следующее сообщение, а затем обновите браузер.
   >

В браузере обновите страницу **ProductsPortal** . При каждом обновлении страницы после вызова `GetProducts()` из **ProductsServer** в серверном приложении будет отображаться сообщение.

![Обновленные выходные данные][38]

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующему руководству:

> [!div class="nextstepaction"]
>[Предоставление локальной службы WCF для клиента WCF за пределами сети](service-bus-relay-tutorial.md)

[0]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hybrid.png
[1]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app.png
[NuGet]: https://nuget.org

[11]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/configure-productsserver.png
[13]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/install-nuget-service-bus-productsserver.png
[15]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-2.png
[16]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/choose-web-application-template.png
[17]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/add-class-productsportal.png
[18]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/change-authentication.png
[9]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/web-publish-activity.png
[10]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app-locally.png

[21]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/run-web-app-locally-no-content.png
[24]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/add-existing-item-link.png
[25]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-13.png
[26]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-14.png
[27]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/launch-app-as-web-app.png

[36]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
[37]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service1.png
[38]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service2.png
[41]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-40.png
[43]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-43.png
