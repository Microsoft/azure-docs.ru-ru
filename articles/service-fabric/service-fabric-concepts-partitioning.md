---
title: Разделение служб Service Fabric
description: Сведения о секционировании служб без отслеживания состояния Service Fabric и с отслеживанием состояния
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 199ae9d9844149c1931da638633110f717fe0517
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97915901"
---
# <a name="partition-service-fabric-reliable-services"></a>Секционирование служб Reliable Services в Service Fabric
В этой статье рассматриваются основные понятия, связанные с секционированием служб Reliable Services в инфраструктуре Azure Service Fabric. Секционирование позволяет хранить данные на локальных компьютерах, чтобы их можно было масштабировать вместе.

> [!TIP]
> [Полный пример](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/classic/Services/AlphabetPartitions) кода в этой статье доступен на сайте GitHub.

## <a name="partitioning"></a>Секционирование
Секционирование не является уникальной особенностью Service Fabric. По сути, это основной способ создания масштабируемых служб. В более широком смысле понятие секционирования означает разбивку состояния (данных) и вычисления на более мелкие единицы для повышения производительности и улучшения масштабируемости. Распространенный пример секционирования — это [секционирование данных][wikipartition], которое еще называют сегментированием.

### <a name="partition-service-fabric-stateless-services"></a>Секционирование служб без отслеживания состояния в Service Fabric
В случае со службами без отслеживания состояния секцию можно описать как логическую единицу, содержащую один или несколько экземпляров службы. На рис. 1 показана служба без отслеживания состояния с пятью экземплярами, распределенными в кластере и объединенными в одну секцию.

![Служба без отслеживания состояния](./media/service-fabric-concepts-partitioning/statelessinstances.png)

Есть два типа решений служб без отслеживания состояния. Первый — это служба, которая сохраняет свое состояние извне, например в базе данных в базе данных SQL Azure (например, на веб-сайте, где хранятся сведения о сеансе и данные). Второй — только вычислительные службы (например, калькулятор или создание эскизов для изображений), которые не управляют устойчивыми состояниями.

Секционирование служб без отслеживания состояния выполняется очень редко. поэтому секционирование таких служб уместно лишь тогда, когда вам нужно выполнять особые запросы на маршрутизацию.

Примером может быть случай, когда пользователи с идентификаторами из определенного диапазона должны обслуживаться только определенным экземпляром службы. Другим примером использования службы без отслеживания состояния является наличие действительно секционированной серверной части (например, Сегментированная база данных в базе данных SQL) и необходимость контролировать, какой экземпляр службы должен записывать в сегмент базы данных, или выполнять другие подготовительные действия в службе без отслеживания состояния, для которой требуются те же сведения о секционировании, что и в серверной части. Подобные сценарии также можно решить другими способами, не требующими секционирования службы.

Оставшаяся часть этого пошагового руководства посвящена службам с отслеживанием состояния.

### <a name="partition-service-fabric-stateful-services"></a>Секционирование служб с отслеживанием состояния в Service Fabric
Service Fabric упрощает процесс разработки масштабируемых служб с отслеживанием состояния, предоставляя лучший способ секционирования состояния (данных). По сути, секция службы с отслеживанием состояния является единицей масштабирования, высокую надежность которой обеспечивают [реплики](service-fabric-availability-services.md) , равномерно распределяющиеся между узлами кластера.

В контексте служб Service Fabric с отслеживанием состояния секционирование — это процесс определения секции службы, которая отвечает за определенную часть полного состояния службы. (Как уже говорилось ранее, секция представляет собой набор [реплик](service-fabric-availability-services.md).) Service Fabric размещает секции на разных узлах. Из-за этого размер секции может увеличиваться в пределах ресурса узла. По мере роста объема данных размер секций увеличивается, и Service Fabric перераспределяет секции между узлами. Таким образом, аппаратные ресурсы постоянно используются с максимальной эффективностью.

Рассмотрим это все на примере. Предположим, у нас есть кластер из пяти узлов и служба, которая должна иметь десять секций и три реплики. В этом случае Service Fabric распределит реплики в кластере так, что мы получим по две первичные [реплики](service-fabric-availability-services.md) на каждом узле.
Если кластер разрастется до десяти узлов, Service Fabric перераспределит первичные [реплики](service-fabric-availability-services.md) между десятью узлами. Если количество узлов в кластере снова уменьшится до пяти, все реплики опять будут перераспределены между пятью узлами.  

На рис. 2 показано распределение десяти разделов до и после масштабирования в кластере.

![Служба с отслеживанием состояния](./media/service-fabric-concepts-partitioning/partitions.png)

В итоге мы получаем увеличение масштаба, при котором запросы от клиентов распределяются между компьютерами. Повышается общая производительность приложения, а конкуренция за доступ к блокам данных уменьшается.

## <a name="plan-for-partitioning"></a>Планирование секционирования
Перед реализацией службы следует всегда учитывать стратегию секционирования, необходимую для масштабирования. Существуют различные способы, но все они сосредоточены на том, что необходимо для этого приложения. В этой статье мы затронем некоторые наиболее важные аспекты.

Сначала следует подумать о структуре состояния, которое нужно секционировать.

Давайте рассмотрим простой пример. Если бы вы создавали службу для опроса на уровне района, вы можете создать секцию для каждого города в округе. Затем в секции вы сохраняете голоса каждого человека в городе, который относится к соответствующей секции. На рис. 3 приведен набор людей и городов, в которых они находятся.

![Простой раздел](./media/service-fabric-concepts-partitioning/cities.png)

Так как население городов значительно отличается, в итоге может оказаться, что некоторые секции содержат большое количество данных (например, Сиэтл), а другие — незначительное (например, Кирклэнд). Каковы последствия наличия разделов с неравномерным состоянием?

Нетрудно заметить, что секция, которая содержит голоса для Сиэтла, будет получать больше трафика, чем секция для Кирклэнда. По умолчанию Service Fabric следит за тем, чтобы на каждом узле было примерно одинаковое количество первичных и вторичных реплик. Поэтому у вас могут получиться узлы, реплики на которых обслуживают разный объем трафика. Желательно избегать таких чрезмерно активных и пассивных участков в кластере.

Вот две рекомендации, как избежать такой ситуации.

* Старайтесь секционировать состояние так, чтобы оно равномерно распределялось между всеми разделами.
* Загрузите отчеты из каждой реплики для службы. (Чтобы узнать, как это сделать, прочитайте эту статью о [метриках и загрузке](service-fabric-cluster-resource-manager-metrics.md).) В Service Fabric есть возможность сообщать о нагрузке, создаваемой службами, по таким показателям, как объем памяти или число записей. На основе полученных метрик Service Fabric определяет, какие секции загружены больше, чем другие. Затем платформа перераспределяет реплики в кластере, перемещая их на более подходящие узлы, чтобы ни один из узлов не был перегружен.

В некоторых случаях невозможно заранее определить, какой объем данных будет в той или иной секции. Поэтому обычно рекомендуется использовать оба подхода: разработать стратегию секционирования, согласно которой данные будут равномерно распределяться между секциями, и использовать сведения о нагрузке.  Первый способ предотвращает ситуации, описанные в примере с опросом, тогда как второй помогает сгладить временную разницу в доступе или загрузке.

Другим аспектом планирования секционирования является выбор правильного количества разделов.
В Service Fabric нет никаких ограничений относительно использования большего количества секций, чем требуется для тех или иных целей.
Задать максимальное количество — вполне допустимый подход.

Большее количество секций, чем выбрано изначально, может понадобиться в редких случаях. Так как изменять выбранное количество секций нельзя, вам придется прибегнуть к дополнительным методам секционирования, в частности создать новый экземпляр службы того же типа. Кроме того, вам нужно будет внедрить клиентскую логику, которая будет направлять запросы к правильному экземпляру службы на основе сведений, которыми располагает клиент (в клиентском коде должна быть реализована соответствующая поддержка).

При планировании секционирования вам также необходимо учитывать доступные ресурсы компьютеров. Так как состояние нужно не только хранить, но и получать к нему доступ, вам нужно учитывать следующие ограничения:

* пропускная способность сети;
* память системы;
* место на диске.

Что произойдет, если во время работы кластера возникнут проблемы ограничения ресурсов? Ответ прост: кластер можно расширить в соответствии с новыми требованиями.

[Руководство по планированию емкости](service-fabric-capacity-planning.md) поможет определить требуемое количество узлов в кластере.

## <a name="get-started-with-partitioning"></a>Начало секционирования
В этом разделе описывается, как секционировать службу.

В Service Fabric можно выбрать одну из трех возможных схем секционирования.

* Секционирование по диапазонам значений (также известное как UniformInt64Partition).
* Секционирование по именам. Как правило, данные приложений, которые используют эту модель, можно поместить в контейнер в рамках ограниченного набора данных. Вот некоторые наиболее распространенные примеры полей данных, которые используются в качестве ключей для секционирования по именам: регионы, почтовые индексы, группы клиентов и др.
* Одноэлементное секционирование. Одноэлементное секционирование обычно используется, если служба не требует дополнительной маршрутизации. Например, службы без отслеживания состояния используют эту схему секционирования по умолчанию.

Схемы секционирования по именам и одноэлементного секционирования являются особыми формами секционирования по диапазонам. По умолчанию в шаблонах Visual Studio для Service Fabric используется секционирование по диапазонам, так как оно является наиболее распространенным и практичным. В оставшейся части этой статьи рассматривается схема секционирования по диапазонам.

### <a name="ranged-partitioning-scheme"></a>Схема секционирования по диапазонам
Она используется для указания целочисленного диапазона (определенного низким и высоким ключами) и количества секций (n). Создается n разделов, каждый из которых отвечает за неперекрывающийся поддиапазон всего диапазона ключей раздела. Например, схема секционирования по диапазонам, в которой нижний ключ равен 0, верхний ключ равен 99, а количество секций равно 4, создаст четыре секции, как показано ниже.

![Секционирование по диапазону](./media/service-fabric-concepts-partitioning/range-partitioning.png)

Наиболее распространенным приемом является создание хэша на основе уникального ключа в наборе данных. Вот некоторые наиболее распространенные примеры ключей: код идентификации транспортного средства (VIN), идентификатор сотрудника, уникальная строка и пр. Используя этот уникальный ключ, вы создаете хэш-код, модуль диапазона ключей, который будет использоваться как ключ. Для допустимого диапазона ключей можно указать верхнюю и нижнюю границы.

### <a name="select-a-hash-algorithm"></a>Выбор хэш-алгоритма
Важной частью хэширования является выбор хэш-алгоритма. Вам необходимо решить следующее: следует ли группировать одинаковые ключи рядом друг с другом (хэширование с учетом местоположения) или распределить действия по всем секциям (хэширование с распределением). Последний алгоритм используется чаще.

Хороший хэш-алгоритм с распределением отличается простотой вычисления, небольшим количеством конфликтов и равномерным распределением ключей. Примером эффективного хэш-алгоритма является [FNV-1](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function) .

Хороший выбор общих хэш-алгоритмов представлен в [Википедии на странице, посвященной хэш-функциям](https://en.wikipedia.org/wiki/Hash_function).

## <a name="build-a-stateful-service-with-multiple-partitions"></a>Создание службы с отслеживанием состояния с несколькими секциями
Далее вы создадите свою первую службу Reliable Services с отслеживанием состояния с несколькими разделами. В приведенном ниже примере мы создадим очень простое приложение, в каждой секции которого будут храниться фамилии, начинающиеся с определенной буквы.

Прежде чем приступать к написанию кода, следует определиться с секциями и их ключами. Нам потребуется 26 секций — по одной для каждой буквы английского алфавита. Но как определить нижнюю и верхнюю границы ключей?
Нам нужно иметь по одной секции на каждую букву. Следовательно, мы можем использовать 0 в качестве нижнего ключа и 25 — в качестве верхнего, так как каждая буква выступает собственным ключом.

> [!NOTE]
> Это упрощенный сценарий, так как на самом деле распределение было бы неравномерным. Фамилии, начинающиеся с буквы S или M, встречаются чаще, чем те, которые начинаются с X или Y.
> 
> 

1. Откройте **Visual Studio**  >  **файл**  >  **Новый**  >  **проект**.
2. В диалоговом окне **Создание проекта** выберите приложение Service Fabric.
3. Назовите проект AlphabetPartitions.
4. В диалоговом окне **Создание службы** выберите службу **с отслеживанием состояния** и назовите ее Alphabet.Processing.
5. Укажите количество секций. Откройте файл Applicationmanifest.xml в папке ApplicationPackageRoot проекта AlphabetPartitions и задайте для параметра Processing_PartitionCount значение 26, как показано ниже.
   
    ```xml
    <Parameter Name="Processing_PartitionCount" DefaultValue="26" />
    ```
   
    Кроме того, в элементе StatefulService в ApplicationManifest.xml нужно изменить свойства LowKey и HighKey, как показано ниже.
   
    ```xml
    <Service Name="Processing">
      <StatefulService ServiceTypeName="ProcessingType" TargetReplicaSetSize="[Processing_TargetReplicaSetSize]" MinReplicaSetSize="[Processing_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[Processing_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    ```
6. Чтобы сделать службу доступной, откройте конечную точку на порте. Для этого добавьте для службы Alphabet.Processing элемент конечной точки из файла ServiceManifest.xml (расположен в папке PackageRoot), как показано ниже.
   
    ```xml
    <Endpoint Name="ProcessingServiceEndpoint" Port="8089" Protocol="http" Type="Internal" />
    ```
   
    Теперь служба настроена на прослушивание внутренней конечной точки в 26 разделах.
7. Далее нужно переопределить метод `CreateServiceReplicaListeners()` класса Processing.
   
   > [!NOTE]
   > В этом примере предполагается, что вы используете простой прослушиватель HttpCommunicationListener. Дополнительные сведения о модели взаимодействия Reliable Services см. в [этой статье](service-fabric-reliable-services-communication.md).
   > 
   > 
8. URL-адрес, который реплика использует для ожидания передачи данных, рекомендуется указывать в таком формате: `{scheme}://{nodeIp}:{port}/{partitionid}/{replicaid}/{guid}`.
    Таким образом, вам нужно настроить прослушиватель на правильные конечные точки, используя именно этот шаблон.
   
    На одном компьютере может быть размещено несколько реплик этой службы, поэтому адрес должен быть уникальным для каждой реплики. Именно по этой причине в URL-адресе указываются идентификаторы секции и реплики. HttpListener может прослушивать несколько адресов на одном порту при условии, что префикс URL-адреса является уникальным.
   
    Дополнительный GUID используется в сложных случаях, когда вторичные реплики также прослушивают запросы только для чтения. Если это так, следует убедиться, что новый уникальный адрес используется при переходе от первичной реплики к вторичной для принудительного разрешения адресов клиентами. "+" используется в качестве адреса, чтобы реплика прослушивает все доступные узлы (IP-адрес, полное доменное имя, localhost и т. д.). В приведенном ниже коде показан пример.
   
    ```csharp
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
         return new[] { new ServiceReplicaListener(context => this.CreateInternalListener(context))};
    }
    private ICommunicationListener CreateInternalListener(ServiceContext context)
    {
   
         EndpointResourceDescription internalEndpoint = context.CodePackageActivationContext.GetEndpoint("ProcessingServiceEndpoint");
         string uriPrefix = String.Format(
                "{0}://+:{1}/{2}/{3}-{4}/",
                internalEndpoint.Protocol,
                internalEndpoint.Port,
                context.PartitionId,
                context.ReplicaOrInstanceId,
                Guid.NewGuid());
   
         string nodeIP = FabricRuntime.GetNodeContext().IPAddressOrFQDN;
   
         string uriPublished = uriPrefix.Replace("+", nodeIP);
         return new HttpCommunicationListener(uriPrefix, uriPublished, this.ProcessInternalRequest);
    }
    ```
   
    Также следует отметить, что опубликованный URL-адрес немного отличается от префикса URL-адреса прослушивания.
    URL-адрес прослушивания назначается HttpListener. Опубликованный URL-адрес — это URL-адрес, опубликованный в службе именования Service Fabric, который используется для обнаружения службы. Клиенты будут запрашивать этот адрес с помощью службы обнаружения. Адрес, который получают клиенты, должен содержать фактический IP-адрес или полное доменное имя узла (в противном случае подключение будет невозможно). Поэтому вам нужно вместо символа «+» указать IP-адрес или полное доменное имя, как показано выше.
9. Последним шагом является добавление логики обработки к службе, как показано ниже.
   
    ```csharp
    private async Task ProcessInternalRequest(HttpListenerContext context, CancellationToken cancelRequest)
    {
        string output = null;
        string user = context.Request.QueryString["lastname"].ToString();
   
        try
        {
            output = await this.AddUserAsync(user);
        }
        catch (Exception ex)
        {
            output = ex.Message;
        }
   
        using (HttpListenerResponse response = context.Response)
        {
            if (output != null)
            {
                byte[] outBytes = Encoding.UTF8.GetBytes(output);
                response.OutputStream.Write(outBytes, 0, outBytes.Length);
            }
        }
    }
    private async Task<string> AddUserAsync(string user)
    {
        IReliableDictionary<String, String> dictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<String, String>>("dictionary");
   
        using (ITransaction tx = this.StateManager.CreateTransaction())
        {
            bool addResult = await dictionary.TryAddAsync(tx, user.ToUpperInvariant(), user);
   
            await tx.CommitAsync();
   
            return String.Format(
                "User {0} {1}",
                user,
                addResult ? "successfully added" : "already exists");
        }
    }
    ```
   
    `ProcessInternalRequest` считывает значения параметра строки запроса, который используется для вызова раздела, и вызывает метод `AddUserAsync` для добавления lastname в надежный словарь `dictionary`.
10. Добавим в проект службу без отслеживания состояния, чтобы посмотреть, как можно вызвать определенный раздел.
    
    Эта служба выступает в качестве простого веб-интерфейса, который принимает фамилию как параметр строки запроса, определяет ключ секции и отправляет его на обработку в службу Alphabet.Processing.
11. В диалоговом окне **Создание службы** выберите службу **без отслеживания состояния** и назовите ее Alphabet.Web, как показано ниже.
    
    ![Снимок экрана, на котором изображена служба без отслеживания состояния](./media/service-fabric-concepts-partitioning/createnewstateless.png).
12. Откройте порт, изменив сведения о конечной точке в файле ServiceManifest.xml службы Alphabet.WebApi, как показано ниже.
    
    ```xml
    <Endpoint Name="WebApiServiceEndpoint" Protocol="http" Port="8081"/>
    ```
13. Необходимо вернуть коллекцию ServiceInstanceListeners в класс Web. Опять же, вы можете реализовать простой прослушиватель HttpCommunicationListener.
    
    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[] {new ServiceInstanceListener(context => this.CreateInputListener(context))};
    }
    private ICommunicationListener CreateInputListener(ServiceContext context)
    {
        // Service instance's URL is the node's IP & desired port
        EndpointResourceDescription inputEndpoint = context.CodePackageActivationContext.GetEndpoint("WebApiServiceEndpoint")
        string uriPrefix = String.Format("{0}://+:{1}/alphabetpartitions/", inputEndpoint.Protocol, inputEndpoint.Port);
        var uriPublished = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
        return new HttpCommunicationListener(uriPrefix, uriPublished, this.ProcessInputRequest);
    }
    ```
14. Теперь нам необходимо реализовать логику обработки. После получения запроса прослушиватель HttpCommunicationListener вызывает `ProcessInputRequest`, поэтому давайте добавим приведенный ниже код.
    
    ```csharp
    private async Task ProcessInputRequest(HttpListenerContext context, CancellationToken cancelRequest)
    {
        String output = null;
        try
        {
            string lastname = context.Request.QueryString["lastname"];
            char firstLetterOfLastName = lastname.First();
            ServicePartitionKey partitionKey = new ServicePartitionKey(Char.ToUpper(firstLetterOfLastName) - 'A');
    
            ResolvedServicePartition partition = await this.servicePartitionResolver.ResolveAsync(alphabetServiceUri, partitionKey, cancelRequest);
            ResolvedServiceEndpoint ep = partition.GetEndpoint();
    
            JObject addresses = JObject.Parse(ep.Address);
            string primaryReplicaAddress = (string)addresses["Endpoints"].First();
    
            UriBuilder primaryReplicaUriBuilder = new UriBuilder(primaryReplicaAddress);
            primaryReplicaUriBuilder.Query = "lastname=" + lastname;
    
            string result = await this.httpClient.GetStringAsync(primaryReplicaUriBuilder.Uri);
    
            output = String.Format(
                    "Result: {0}. <p>Partition key: '{1}' generated from the first letter '{2}' of input value '{3}'. <br>Processing service partition ID: {4}. <br>Processing service replica address: {5}",
                    result,
                    partitionKey,
                    firstLetterOfLastName,
                    lastname,
                    partition.Info.Id,
                    primaryReplicaAddress);
        }
        catch (Exception ex) { output = ex.Message; }
    
        using (var response = context.Response)
        {
            if (output != null)
            {
                output = output + "added to Partition: " + primaryReplicaAddress;
                byte[] outBytes = Encoding.UTF8.GetBytes(output);
                response.OutputStream.Write(outBytes, 0, outBytes.Length);
            }
        }
    }
    ```
    
    Давайте разберемся в процессе, шаг за шагом. Код считывает первую букву из параметра строки запроса в `lastname` в тип char. Затем он определяет ключ раздела для этой буквы, вычитая шестнадцатеричное значение `A` из шестнадцатеричного значения первой буквы фамилии.
    
    ```csharp
    string lastname = context.Request.QueryString["lastname"];
    char firstLetterOfLastName = lastname.First();
    ServicePartitionKey partitionKey = new ServicePartitionKey(Char.ToUpper(firstLetterOfLastName) - 'A');
    ```
    
    Напоминаем, что в этом примере мы используем 26 секций с одним ключом на секцию.
    Далее для этого ключа мы получаем секцию службы `partition`. Для этого мы используем метод `ResolveAsync` в объекте `servicePartitionResolver`. `servicePartitionResolver` определяется так:
    
    ```csharp
    private readonly ServicePartitionResolver servicePartitionResolver = ServicePartitionResolver.GetDefault();
    ```
    
    В качестве параметров метод `ResolveAsync` принимает универсальный код ресурса (URI) службы, ключ секции и маркер отмены. URI службы обработки выглядит так: `fabric:/AlphabetPartitions/Processing`. Далее мы получаем конечную точку секции.
    
    ```csharp
    ResolvedServiceEndpoint ep = partition.GetEndpoint()
    ```
    
    В самом конце мы создаем URL-адрес конечной точки и строку запроса и обращаемся к службе обработки.
    
    ```csharp
    JObject addresses = JObject.Parse(ep.Address);
    string primaryReplicaAddress = (string)addresses["Endpoints"].First();
    
    UriBuilder primaryReplicaUriBuilder = new UriBuilder(primaryReplicaAddress);
    primaryReplicaUriBuilder.Query = "lastname=" + lastname;
    
    string result = await this.httpClient.GetStringAsync(primaryReplicaUriBuilder.Uri);
    ```
    
    После выполнения обработки пишем вывод.
15. Последним шагом будет тестирование службы. Visual Studio использует параметры приложения для локального и облачного развертываний. Чтобы локально протестировать службу с 26 секциями, измените файл `Local.xml` в папке ApplicationParameters проекта AlphabetPartitions, как показано ниже.
    
    ```xml
    <Parameters>
      <Parameter Name="Processing_PartitionCount" Value="26" />
      <Parameter Name="WebApi_InstanceCount" Value="1" />
    </Parameters>
    ```
16. Когда служба будет развернута, все ее секции можно будет проверить в обозревателе Service Fabric.
    
    ![Снимок экрана, на котором изображен обозреватель Service Fabric](./media/service-fabric-concepts-partitioning/sfxpartitions.png)
17. Чтобы проверить логику секционирования в браузере, введите в адресной строке `http://localhost:8081/?lastname=somename`. Вы увидите, что все фамилии, начинающиеся с одинаковой буквы, хранятся в одной секции.
    
    ![Снимок экрана, на котором изображен браузер](./media/service-fabric-concepts-partitioning/samplerunning.png)

Полное решение кода, используемого в этой статье, можно найти здесь: https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/classic/Services/AlphabetPartitions .

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о службах Service Fabric:

* [Подключение к службам в Service Fabric и взаимодействие с ними](service-fabric-connect-and-communicate-with-services.md)
* [Доступность служб структуры служб](service-fabric-availability-services.md)
* [Масштабируемость служб структуры служб](service-fabric-concepts-scalability.md)

[wikipartition]: https://en.wikipedia.org/wiki/Partition_(database)

[1]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog-2.png
