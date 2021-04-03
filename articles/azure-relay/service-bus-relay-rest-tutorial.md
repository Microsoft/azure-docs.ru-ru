---
title: Руководство по Работа с REST с использованием Azure Relay
description: Руководство по Создание ведущего приложения Azure Relay, которое предоставляет интерфейс на основе REST.
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 0620f55650d0e4da0cd7a616649df952f3017455
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88922333"
---
# <a name="tutorial-azure-wcf-relay-rest-tutorial"></a>Руководство по Руководство по REST для ретранслятора WCF Azure

В этом руководстве описано, как создать ведущее приложение Azure Relay, предоставляющее интерфейс на основе REST. REST позволяет веб-клиенту, например веб-браузеру, получить доступ к интерфейсам API служебной шины с помощью HTTP-запросов.

В этом руководстве для создания службы REST в ретрансляторе Azure используется модель программирования REST Windows Communication Foundation (WCF). Дополнительные сведения см. в статьях [WCF Web HTTP Programming Model](/dotnet/framework/wcf/feature-details/wcf-web-http-programming-model) (Модель программирования HTTP для веб-службы WCF) и [Designing and Implementing Services](/dotnet/framework/wcf/designing-and-implementing-services) (Разработка и реализация служб).

В этом учебнике вы выполните следующие задачи:

> [!div class="checklist"]
>
> * Установка обязательных компонентов для работы с этим учебником.
> * Создание пространства имен ретранслятора.
> * Определение контракта службы WCF на основе REST.
> * Реализация контракта службы WCF на основе REST.
> * Размещение и запуск службы WCF на основе REST.
> * Запуск и тестирование службы.

## <a name="prerequisites"></a>Предварительные требования

Для работы с данным руководством вам потребуется:

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начать работу.
* [Visual Studio 2015 или более поздней версии](https://www.visualstudio.com). В примерах для этого учебника используется Visual Studio 2019.
* Пакет Azure SDK для .NET. Установите его со [страницы загрузки SDK](https://azure.microsoft.com/downloads/).

## <a name="create-a-relay-namespace"></a>Создание пространства имен ретранслятора

Чтобы начать использовать функции ретранслятора Azure, необходимо сначала создать пространство имен службы. Пространство имен предоставляет контейнер для адресации ресурсов Azure в вашем приложении. Выполните [эти инструкции](relay-create-namespace-portal.md), чтобы создать пространство имен ретранслятора.

## <a name="define-a-rest-based-wcf-service-contract-to-use-with-azure-relay"></a>Определение контракта службы WCF на основе REST, используемого в Azure Relay

При создании службы WCF на основе REST необходимо определить контракт. Контракт определяет, какие операции поддерживает узел. Операция службы похожа на метод веб-службы. Контракты создаются путем определения интерфейса на C++, C# или Visual Basic. Каждый метод в интерфейсе соответствует определенной операции службы. Примените к каждому интерфейсу атрибут [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute), а к каждой операции — атрибут [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute). 

> [!TIP]
> Если у метода в интерфейсе с атрибутом [ServiceContractAttribute](/dotnet/api/system.servicemodel.servicecontractattribute) нет атрибута [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute), такой метод не предоставляется. Код для выполнения этих задач отображается в примере, приведенном после описания последовательности выполнения действий.

Основное различие между WCF и контрактом REST заключается в добавлении свойства к атрибуту [OperationContractAttribute](/dotnet/api/system.servicemodel.operationcontractattribute): [WebGetAttribute](/dotnet/api/system.servicemodel.web.webgetattribute). Это свойство позволяет сопоставить метод в интерфейсе с методом на другой стороне интерфейса. В этом примере для связывания метода с `HTTP GET`используется атрибут [WebGetAttribute](/dotnet/api/system.servicemodel.web.webgetattribute). Такой подход позволяет служебной шине точно извлекать и интерпретировать команды, отправляемые в интерфейс.

### <a name="to-create-a-contract-with-an-interface"></a>Создание контракта с помощью интерфейса

1. Запустите Microsoft Visual Studio от имени администратора. Для этого щелкните правой кнопкой мыши значок программы Visual Studio и выберите **Запустить от имени администратора**.
1. В Visual Studio выберите **Создать проект**.
1. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** для C# и нажмите кнопку **Далее**.
1. Присвойте проекту имя *ImageListener*. Выберите **Расположение** по умолчанию, а затем нажмите **Создать**.

   Visual Studio создает для проекта C# файл с именем *Program.cs*. Этот класс содержит пустой метод `Main()`, необходимый для правильной сборки проекта консольного приложения.

1. В **обозревателе решений** щелкните правой кнопкой мыши проект **ImageListener** и выберите пункт **Управление пакетами NuGet**.
1. Нажмите **Обзор**, найдите и выберите **WindowsAzure.ServiceBus**. Нажмите **Установить** и примите условия использования.

    На этом шаге добавляются ссылки на служебную шину и *System.ServiceModel.dll*. Этот пакет автоматически добавляет ссылки на библиотеки служебной шины и элемент WCF `System.ServiceModel`.

1. Явным образом добавьте в проект ссылки на `System.ServiceModel.Web.dll`. В **обозревателе решений** в папке проекта щелкните правой кнопкой мыши папку **Ссылки** и выберите **Добавить ссылку**.
1. В разделе **Добавление ссылки** выберите элемент **Платформа** и введите *System.ServiceModel.Web* в поле **Поиск**. Установите флажок **System.ServiceModel.Web**, а затем нажмите кнопку **ОК**.

Теперь внесите в проект следующие изменения.

1. Добавьте следующие инструкции `using` в начало файла *Program.cs*.

    ```csharp
    using System.ServiceModel;
    using System.ServiceModel.Channels;
    using System.ServiceModel.Web;
    using System.IO;
    ```

    * [System.ServiceModel](/dotnet/api/system.servicemodel) — это пространство имен, которое предоставляет программный доступ к основным функциям WCF. Ретранслятор WCF использует множество объектов и атрибутов WCF для определения контрактов службы. Это пространство имен будет использоваться в большинстве приложений ретранслятора.
    * [System.ServiceModel.Channels](/dotnet/api/system.servicemodel.channels) помогает определить канал, т. е. объект, посредством которого Azure Relay взаимодействует с веб-браузером клиента.
    * [System.ServiceModel.Web](/dotnet/api/system.servicemodel.web) содержит типы, позволяющие создавать веб-приложения.

1. Переименуйте пространство имен `ImageListener` в `Microsoft.ServiceBus.Samples`.

    ```csharp
    namespace Microsoft.ServiceBus.Samples
    {
        ...
    ```

1. Сразу после открывающей фигурной скобки в объявлении пространства имен определите новый интерфейс с именем `IImageContract` и примените к нему атрибут `ServiceContractAttribute` со значением `https://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1`. 

    ```csharp
    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1")]
    public interface IImageContract
    {
    }
    ```

    Значение пространства имен отличается от пространства имен, которое используется во всей области кода. Значение пространства имен является уникальным идентификатором контракта и должно содержать сведения о версии. Дополнительные сведения см. в статье [Управление версиями службы](/dotnet/framework/wcf/service-versioning). Явное указание пространства позволяет предотвратить добавление значение пространства имен по умолчанию к имени контракта.

1. В интерфейсе `IImageContract` объявите метод для отдельной операции, которую контракт `IImageContract` предоставляет в интерфейсе. Затем примените атрибут `OperationContract` к методу, который требуется предоставить как часть открытого контракта служебной шины.

    ```csharp
    public interface IImageContract
    {
        [OperationContract]
        Stream GetImage();
    }
    ```

1. Добавьте в атрибут `OperationContract` значение `WebGet`.

    ```csharp
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }
    ```

   Добавление значения `WebGet` позволит службе ретрансляции перенаправлять HTTP-запросы GET в `GetImage` и преобразовывать возвращаемые значения из `GetImage` в ответ `HTTP GETRESPONSE`. Далее в этом учебнике вы через веб-браузер обратитесь к этому методу и отобразите изображение.

1. Сразу после определения `IImageContract` объявите канал, наследующий от интерфейсов `IImageContract` и `IClientChannel`.

    ```csharp
    public interface IImageChannel : IImageContract, IClientChannel { }
    ```

   Канал представляет собой объект WCF, посредством которого служба и клиент обмениваются информацией. Канал в своем хост-приложении вы создадите позже. Azure Relay использует этот канал для передачи HTTP-запросов GET из браузера в созданную реализацию `GetImage`. Канал также требуется ретранслятору для получения возвращаемого значения `GetImage` и преобразования его в `HTTP GETRESPONSE` для браузера клиента.

1. Нажмите **Сборка** > **Собрать решение**, чтобы проверить правильность уже выполненных действий.

### <a name="example-that-defines-a-wcf-relay-contract"></a>Пример определения контракта для Ретранслятора WCF

Ниже приведен код базового интерфейса, определяющего контракт ретранслятора WCF.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "IImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## <a name="implement-the-rest-based-wcf-service-contract"></a>Реализация контракта службы WCF на основе REST

Перед созданием службы Ретранслятора WCF на основе REST создайте контракт с помощью интерфейса. Следующим шагом является реализация интерфейса. Эта процедура включает создание класса с именем `ImageService`, который реализует пользовательский интерфейс `IImageContract`. После реализации контракта необходимо настроить интерфейс с помощью файла *App.config*. Этот файл конфигурации содержит необходимые для приложения сведения. К ним относятся имя службы, имя контракта и тип протокола, используемого для взаимодействия со службой ретрансляции. Код для выполнения этих задач отображается в примере, приведенном после описания последовательности выполнения действий.

Как и на предыдущих шагах, разница между реализацией контракта REST и контракта Ретранслятора WCF невелика.

### <a name="to-implement-a-rest-style-service-bus-contract"></a>Реализация контракта REST служебной шины

1. Создайте новый класс с именем `ImageService` сразу после определения интерфейса `IImageContract`. Класс `ImageService` реализует интерфейс `IImageContract`.

    ```csharp
    class ImageService : IImageContract
    {
    }
    ```

    Как и в случае с другими реализациями интерфейсов, определение можно реализовать в другом файле. Но в этом руководстве реализация осуществляется в том же файле, что и определение интерфейса и метод `Main()`.

1. Примените атрибут [ServiceBehaviorAttribute](/dotnet/api/system.servicemodel.servicebehaviorattribute) к классу `IImageService`, чтобы указать, что класс является реализацией контракта WCF.

    ```csharp
    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
    }
    ```

    Как упоминалось ранее, это не совсем обычное пространство имен. Оно является частью архитектуры WCF, которая идентифицирует контракт. Дополнительные сведения см. в статье об [именах контракта данных](/dotnet/framework/wcf/feature-details/data-contract-names/).

1. Добавьте в проект изображение в формате *JPG*. Это рисунок, который служба отображает в браузере-получателе.

   1. Щелкните проект правой кнопкой мыши и выберите **Добавить**.
   1. Затем щелкните **Существующий элемент**.
   1. В окне **Добавление существующего элемента** найдите необходимый JPG-файл и нажмите кнопку **Добавить**. При добавлении файла выберите **Все файлы** в раскрывающемся списке рядом с полем **Имя файла**.

   Далее в этом учебнике предполагается, что изображение называется *image.jpg*. При использовании другого файла необходимо переименовать изображение или изменить код соответствующим образом.

1. Чтобы убедиться, что запущенная служба может найти файл изображения, в **обозревателе решений** щелкните файл изображения правой кнопкой мыши и выберите пункт **Свойства**. В области **Свойства** задайте для параметра **Копировать в выходной каталог** значение **Копировать, если новее**.

1. Используя процедуру, описанную в разделе [Создание контракта с помощью интерфейса](#to-create-a-contract-with-an-interface), добавьте в проект ссылку на сборку *System.Drawing.dll*.

1. Добавьте следующие связанные инструкции `using`.

    ```csharp
    using System.Drawing;
    using System.Drawing.Imaging;
    using Microsoft.ServiceBus;
    using Microsoft.ServiceBus.Web;
    ```

1. В класс `ImageService` добавьте следующий конструктор, который загружает точечный рисунок и подготавливает его для отправки в браузер клиента.

    ```csharp
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";
   
        Image bitmap;
   
        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }
    }
    ```

1. Сразу после предыдущего кода добавьте в класс `ImageService` следующий метод `GetImage`, который возвращает HTTP-сообщение с изображением.

    ```csharp
    public Stream GetImage()
    {
        MemoryStream stream = new MemoryStream();
        this.bitmap.Save(stream, ImageFormat.Jpeg);
   
        stream.Position = 0;
        WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";
   
        return stream;
    }
    ```

    Эта реализация использует класс `MemoryStream` для извлечения изображения и его подготовки к потоковой передаче в браузер. Он начинает поток с нулевой позиции, объявляет для содержимого потока тип *.jpg* и передает информацию.

1. Выберите **Сборка** > **Собрать решение**.

### <a name="to-define-the-configuration-for-running-the-web-service-on-service-bus"></a>Определение конфигурации для запуска веб-службы в служебной шине

1. В **обозревателе решений** дважды щелкните файл **App.config**, чтобы открыть его в редакторе Visual Studio.

    Файл *App.config* содержит имя службы, конечную точку и привязку. Конечной точкой называется расположение, которое Azure Relay предоставляет клиентам и узлам для взаимодействия друг с другом. Привязкой называется тип протокола, который используется для взаимодействия. Основное различие состоит в том, что настроенная конечная точка службы ссылается на привязку [WebHttpRelayBinding](/dotnet/api/microsoft.servicebus.webhttprelaybinding).

1. Элемент XML `<system.serviceModel>` представляет собой элемент WCF, определяющий одну или несколько служб. Здесь он используется для определения имени службы и конечной точки. В нижней части элемента `<system.serviceModel>` (но не выходя за рамки элемента `<system.serviceModel>`) добавьте элемент `<bindings>` с указанным ниже содержимым.

    ```xml
    <bindings>
        <!-- Application Binding -->
        <webHttpRelayBinding>
            <binding name="default">
                <security relayClientAuthenticationType="None" />
            </binding>
        </webHttpRelayBinding>
    </bindings>
    ```

    Это содержимое определяет привязки, используемые в приложении. Можно определить несколько привязок, но в этом учебнике определяется только одна.

    Предыдущий код определяет привязку [WebHttpRelayBinding](/dotnet/api/microsoft.servicebus.webhttprelaybinding) Ретранслятора WCF, в которой для свойства `relayClientAuthenticationType` задано значение `None`. Этот параметр указывает, что для конечной точки с этой привязкой не требуются учетные данные клиента.

1. После элемента `<bindings>` добавьте элемент `<services>`. Как и в случае с привязками, в одном файле конфигурации можно определить несколько служб. Но в этом учебнике определяется только одна.

    ```xml
    <services>
        <!-- Application Service -->
        <service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
            <endpoint name="RelayEndpoint"
                    contract="Microsoft.ServiceBus.Samples.IImageContract"
                    binding="webHttpRelayBinding"
                    bindingConfiguration="default"
                    behaviorConfiguration="sbTokenProvider"
                    address="" />
        </service>
    </services>
    ```

    Это содержимое настраивает службу, которая использует заданную ранее привязку по умолчанию `webHttpRelayBinding`. Она также использует значение по умолчанию `sbTokenProvider`, которое будет определено на следующем этапе.

1. После элемента `<services>` создайте элемент `<behaviors>` с указанным ниже содержимым, заменив `SAS_KEY` ключом подписанного URL-адреса (SAS). Чтобы получить ключ SAS на [портале Azure][Azure portal], воспользуйтесь статьей о [получении учетных данных для управления](service-bus-relay-tutorial.md#get-management-credentials).

    ```xml
    <behaviors>
        <endpointBehaviors>
            <behavior name="sbTokenProvider">
                <transportClientEndpointBehavior>
                    <tokenProvider>
                        <sharedAccessSignature keyName="RootManageSharedAccessKey" key="YOUR_SAS_KEY" />
                    </tokenProvider>
                </transportClientEndpointBehavior>
            </behavior>
            </endpointBehaviors>
            <serviceBehaviors>
                <behavior name="default">
                    <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
                </behavior>
            </serviceBehaviors>
    </behaviors>
    ```

1. В том же файле *App.config* замените в элементе `<appSettings>` все значение строки подключения тем значением, которое вы ранее получили на портале.

    ```xml
    <appSettings>
       <!-- Service Bus specific app settings for messaging connections -->
       <add key="Microsoft.ServiceBus.ConnectionString"
           value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_SAS_KEY"/>
    </appSettings>
    ```

1. Выберите **Сборка** > **Построить решение**, чтобы выполнить полную сборку решения.

### <a name="example-that-implements-the-rest-based-wcf-service-contract"></a>Пример с реализацией контракта службы WCF на основе REST

В следующем коде показана реализация контракта и службы для службы REST, запущенной в служебной шине с помощью привязки `WebHttpRelayBinding`.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{


    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

Ниже представлен пример файла *App.config*, связанного со службой.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2"/>
    </startup>
    <system.serviceModel>
        <extensions>
            <!-- In this extension section we are introducing all known service bus extensions. User can remove the ones they don't need. -->
            <behaviorExtensions>
                <add name="connectionStatusBehavior"
                    type="Microsoft.ServiceBus.Configuration.ConnectionStatusElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="transportClientEndpointBehavior"
                    type="Microsoft.ServiceBus.Configuration.TransportClientEndpointBehaviorElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="serviceRegistrySettings"
                    type="Microsoft.ServiceBus.Configuration.ServiceRegistrySettingsElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </behaviorExtensions>
            <bindingElementExtensions>
                <add name="netMessagingTransport"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingTransportExtensionElement, Microsoft.ServiceBus,  Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="tcpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.TcpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="httpsRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.HttpsRelayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="onewayRelayTransport"
                    type="Microsoft.ServiceBus.Configuration.RelayedOnewayTransportElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingElementExtensions>
            <bindingExtensions>
                <add name="basicHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.BasicHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="webHttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WebHttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="ws2007HttpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.WS2007HttpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netOnewayRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetOnewayRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netEventRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetEventRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
                <add name="netMessagingBinding"
                    type="Microsoft.ServiceBus.Messaging.Configuration.NetMessagingBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
            </bindingExtensions>
        </extensions>
      <bindings>
        <!-- Application Binding -->
        <webHttpRelayBinding>
          <binding name="default">
            <security relayClientAuthenticationType="None" />
          </binding>
        </webHttpRelayBinding>
      </bindings>
      <services>
        <!-- Application Service -->
        <service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
          <endpoint name="RelayEndpoint"
                  contract="Microsoft.ServiceBus.Samples.IImageContract"
                  binding="webHttpRelayBinding"
                  bindingConfiguration="default"
                  behaviorConfiguration="sbTokenProvider"
                  address="" />
        </service>
      </services>
      <behaviors>
        <endpointBehaviors>
          <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
              <tokenProvider>
                <sharedAccessSignature keyName="RootManageSharedAccessKey" key="YOUR_SAS_KEY" />
              </tokenProvider>
            </transportClientEndpointBehavior>
          </behavior>
        </endpointBehaviors>
        <serviceBehaviors>
          <behavior name="default">
            <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
          </behavior>
        </serviceBehaviors>
      </behaviors>
    </system.serviceModel>
    <appSettings>
        <!-- Service Bus specific app settings for messaging connections -->
        <add key="Microsoft.ServiceBus.ConnectionString"
            value="Endpoint=sb://yourNamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_SAS_KEY>"/>
    </appSettings>
</configuration>
```

## <a name="host-the-rest-based-wcf-service-to-use-azure-relay"></a>Размещение службы WCF на основе REST для использования Azure Relay

В этом разделе описан запуск веб-службы с помощью консольного приложения и Ретранслятора WCF. Полный пример готового кода, который создается в этом разделе, будет приведен сразу после описания процедуры.

### <a name="to-create-a-base-address-for-the-service"></a>Создание базового адреса для службы

1. В объявлении функции `Main()` создайте переменную для хранения пространства имен своего проекта. Не забудьте заменить `yourNamespace` именем созданного ранее пространства имен ретранслятора.

    ```csharp
    string serviceNamespace = "yourNamespace";
    ```

    Пространство имен используется в служебной шине для создания уникального URI.

1. Создайте экземпляр `Uri` для базового адреса службы на основе пространства имен.

    ```csharp
    Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");
    ```

### <a name="to-create-and-configure-the-web-service-host"></a>Создание и настройка узла веб-службы

Там же в `Main()` создайте узел веб-службы с помощью URI-адреса, созданного на предыдущем шаге.
  
```csharp
WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
```

Узел службы представляет собой объект WCF, который создает экземпляры хост-приложения. В этом примере ему передаются тип создаваемого узла, который имеет значение `ImageService` в нашем примере, и адрес, по которому предоставляется ведущее приложение.

### <a name="to-run-the-web-service-host"></a>Запуск узла веб-службы

1. Там же в `Main()` добавьте следующую строку, чтобы открыть службу.

    ```csharp
    host.Open();
    ```

    Теперь служба запущена.

1. Отобразите сообщение о том, что служба запущена, и о том, как остановить ее.

    ```csharp
    Console.WriteLine("Copy the following address into a browser to see the image: ");
    Console.WriteLine(address + "GetImage");
    Console.WriteLine();
    Console.WriteLine("Press [Enter] to exit");
    Console.ReadLine();
    ```

1. После завершения закройте узел службы.

    ```csharp
    host.Close();
    ```

### <a name="example-of-the-service-contract-and-implementation"></a>Пример контракта службы и его реализации

В следующем примере показаны контракт и реализация службы, созданные на предыдущих шагах, а также размещение службы в консольном приложении. Скомпилируйте следующий код в исполняемый файл с именем *ImageListener.exe*.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "ImageContract", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "https://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            string serviceNamespace = "InsertServiceNamespaceHere";
            Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");

            WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
            host.Open();

            Console.WriteLine("Copy the following address into a browser to see the image: ");
            Console.WriteLine(address + "GetImage");
            Console.WriteLine();
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            host.Close();
        }
    }
}
```

## <a name="run-and-test-the-service"></a>Запуск и тестирование службы

После построения решения выполните следующие действия, чтобы запустить приложение:

1. Нажмите клавишу F5 или перейдите к расположению исполняемого файла *ImageListener\bin\Debug\ImageListener.exe*, чтобы запустить службу. Оставьте приложение работать, так как оно потребуется для следующего шага.
1. Скопируйте из командной строки адрес и вставьте его в браузере, чтобы увидеть изображение.
1. Когда все будет готово, нажмите клавишу ВВОД в окне командной строки, чтобы закрыть приложение.

## <a name="next-steps"></a>Дальнейшие действия

Вы научились создавать приложение, которое использует службу ретранслятора Azure. Дополнительные сведения можно найти в следующих статьях:

* [Что такое ретранслятор Azure?](relay-what-is-it.md)
* [Предоставление локальной службы WCF на базе REST для внешнего клиента с помощью Ретранслятора WCF Azure](service-bus-relay-tutorial.md)

[Azure portal]: https://portal.azure.com
