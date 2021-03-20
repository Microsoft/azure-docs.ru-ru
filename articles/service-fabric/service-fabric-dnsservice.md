---
title: Служба Azure Service Fabric DNS
description: Используйте службу DNS Service Fabric для обнаружения микрослужб в кластере.
ms.topic: conceptual
ms.date: 7/20/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: f7f06920820cdc73f8d3101ab24ee46625931ee4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91268049"
---
# <a name="dns-service-in-azure-service-fabric"></a>Служба DNS в Azure Service Fabric
Служба DNS является необязательной системной службой, которую можно включить в кластере для обнаружения других служб с помощью протокола DNS. 

Ко многим службам, особенно контейнерным, можно обращаться через существующие URL-адреса. Предпочтительно разрешать эти службы с помощью стандартного протокола DNS, а не протокола службы именования Service Fabric. Служба DNS позволяет сопоставить DNS-имена с именем службы и, следовательно, разрешить IP-адреса конечных точек. Такая функция поддерживает переносимость контейнерных служб между различными платформами и упрощает выполнение сценариев "lift-and-shift", позволяя использовать существующие URL-адреса служб вместо того, чтобы переписывать код для использования службы именования. 

Служба DNS сопоставляет DNS-имена с именами служб, которые, в свою очередь, разрешаются службой именования, чтобы вернуть конечную точку службы. DNS-имя службы предоставляется при создании. На следующей схеме показано, как служба доменных имен (DNS) работает для служб без отслеживания состояния.

![Схема, показывающая, как DNS-имена сопоставляются с именами служб DNS для служб без отслеживания состояния.](./media/service-fabric-dnsservice/stateless-dns.png)

Начиная с версии 6.3 Service Fabric, протокол DNS Service Fabric был расширен и теперь включает схему адресации секционированных служб с отслеживанием состояния. Эти расширения позволяют разрешать IP-адреса определенных секций, используя сочетание DNS-имени службы с отслеживанием состояния и имени секции. Поддерживаются все три схемы секционирования:

- секционирование по именам;
- секционирование по диапазонам;
- секционирование по отдельным базам данных.

На следующей схеме показано, как служба доменных имен (DNS) работает для служб с отслеживанием состояния.

![Схема, показывающая, как DNS-имена сопоставляются с именами служб DNS для секционированных служб без отслеживания состояния.](./media/service-fabric-dnsservice/stateful-dns.png)

Динамические порты не поддерживаются службой DNS. Чтобы разрешать службы, предоставляемые через динамические порты, используйте [службу обратного прокси-сервера](./service-fabric-reverseproxy.md).

## <a name="enabling-the-dns-service"></a>Включение службы DNS
> [!NOTE]
> Служба DNS для служб Service Fabric пока что не поддерживается на платформе Linux.

При создании кластера с помощью портала служба DNS включается по умолчанию с помощью флажка **Включить службу DNS** в меню **Конфигурация кластера**.

![Включение службы DNS на портале](./media/service-fabric-dnsservice/enable-dns-service.png)

Если вы не используете портал для создания кластера или обновляете существующий кластер, необходимо включить службу DNS в шаблоне:

- Чтобы развернуть кластер, вы можете использовать [примеры шаблонов](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype) или создать свой шаблон Resource Manager. 
- Чтобы обновить существующий кластер, можно перейти к его группе ресурсов на портале и щелкнуть **Сценарий автоматизации** для работы с шаблоном, который отражает текущее состояние кластера и других ресурсов в группе. Чтобы узнать больше, ознакомьтесь с разделом [Экспорт шаблона из группы ресурсов](../azure-resource-manager/templates/export-template-portal.md).

Получив шаблон, вы можете включить службу DNS следующим образом.

1. Убедитесь, что версия `apiversion` ресурса `Microsoft.ServiceFabric/clusters` имеет значение `2017-07-01-preview`. В противном случае обновите ее, как показано в примере ниже:

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. Теперь включите службу DNS одним из следующих способов:

   - Чтобы включить службу DNS с параметрами по умолчанию, добавьте ее в раздел `addonFeatures` в разделе `properties`, как показано в следующем примере:

        ```json
          "properties": {
            ...
            "addonFeatures": [
              "DnsService"
              ],
            ...
          }
        ```

   - Чтобы включить службу с параметрами, отличными от параметров по умолчанию, добавьте блок `DnsService` в раздел `fabricSettings` внутри раздела `properties`. В этом случае добавлять DnsService в раздел `addonFeatures` не требуется. Дополнительные сведения о свойствах, которые можно задать для службы DNS, см. в разделе [параметров службы DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).

       ```json
           "properties": {
             ...  
             "fabricSettings": [
               ...
               {
                 "name": "DnsService",
                 "parameters": [
                   {
                     "name": "IsEnabled",
                     "value": "true"
                   },
                   {
                     "name": "PartitionSuffix",
                     "value": "--"
                   },
                   {
                     "name": "PartitionPrefix",
                     "value": "--"
                   }
                 ]
               },
               ...
              ]
            }
       ```
3. После обновления шаблона кластера примените изменения и дождитесь завершения обновления. По завершении обновления в кластере запустится системная служба DNS. Имя этой службы — `fabric:/System/DnsService`, и ее можно найти в разделе служб **Система** в обозревателе Service Fabric. 

> [!NOTE]
> При обновлении DNS с отключенного на включено Service Fabric Explorer может не отражать новое состояние. Чтобы устранить эту проблему, перезапустите узлы, изменив UpgradePolicy в шаблоне Azure Resource Manager. Дополнительные сведения см. в [справочнике по шаблону Service Fabric](/azure/templates/microsoft.servicefabric/2019-03-01/clusters/applications) .

> [!NOTE]
> Включение службы DNS при разработке на локальном компьютере приведет к переопределению некоторых параметров DNS. Если возникают проблемы с подключением к Интернету, проверьте параметры DNS.

## <a name="setting-the-dns-name-for-your-service"></a>Настройка DNS-имени для службы
DNS-имена для служб можно задать декларативно для служб по умолчанию в файле ApplicationManifest.xml или с помощью команд PowerShell.

DNS-имя вашей службы разрешается в кластере, поэтому очень важно обеспечить уникальность DNS-имени в пределах кластера. 

Настоятельно рекомендуется использовать схему именования `<ServiceDnsName>.<AppInstanceName>`. Пример: `service1.application1`. Если приложение развернуто с помощью Docker Compose, службам автоматически присваиваются DNS-имена с помощью данной схемы именования.

### <a name="setting-the-dns-name-for-a-default-service-in-the-applicationmanifestxml"></a>Настройка DNS-имени службы по умолчанию в файле ApplicationManifest.xml
Откройте проект в Visual Studio или другом текстовом редакторе и откройте файл ApplicationManifest.xml. Перейдите к разделу служб по умолчанию и для каждой службы добавьте атрибут `ServiceDnsName`. В следующем примере показано, как установить для службы DNS-имя `service1.application1`.

```xml
    <Service Name="Stateless1" ServiceDnsName="service1.application1">
      <StatelessService ServiceTypeName="Stateless1Type" InstanceCount="[Stateless1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
```
После развертывания приложения в экземпляре службы в обозревателе Service Fabric отобразится DNS-имя для данного экземпляра, как показано на следующем изображении: 

![конечные точки служб](./media/service-fabric-dnsservice/service-fabric-explorer-dns.png)

В следующем примере для службы с отслеживанием состояния задается DNS-имя `statefulsvc.app`. Служба использует схему секционирования по именам. Обратите внимание, что имена секций имеют нижний регистр. Это обязательное требование для секций, которые будут применяться в DNS-запросах. Дополнительные сведения см. в разделе [Создание DNS-запросов в секции службы с отслеживанием состояния](#preview-making-dns-queries-on-a-stateful-service-partition).

```xml
    <Service Name="Stateful1" ServiceDnsName="statefulsvc.app" />
      <StatefulService ServiceTypeName="ProcessingType" TargetReplicaSetSize="2" MinReplicaSetSize="2">
        <NamedPartition>
         <Partition Name="partition1" />
         <Partition Name="partition2" />
        </NamedPartition>
      </StatefulService>
    </Service>
```

### <a name="setting-the-dns-name-for-a-service-using-powershell"></a>Задание DNS-имени службы с помощью PowerShell
DNS-имя службы можно задать при ее создании с помощью команды PowerShell `New-ServiceFabricService`. В следующем примере создается новая служба без отслеживания состояния с DNS-именем `service1.application1`.

```powershell
    New-ServiceFabricService `
    -Stateless `
    -PartitionSchemeSingleton `
    -ApplicationName `fabric:/application1 `
    -ServiceName fabric:/application1/service1 `
    -ServiceTypeName Service1Type `
    -InstanceCount 1 `
    -ServiceDnsName service1.application1
```

## <a name="preview-making-dns-queries-on-a-stateful-service-partition"></a>Создание запросов DNS в секции службы с отслеживанием состояния (предварительная версия)
Начиная с Service Fabric версии 6.3, служба DNS Service Fabric поддерживает запросы для секций службы.

Для секций, которые будут использоваться в запросах DNS, применяются следующие ограничения именования:

   - Имена секций должны быть совместимыми с DNS.
   - Имена секций с несколькими метками (которые содержат точку, ".", в имени) не следует использовать.
   - Имена секций должны использовать нижний регистр.

Запросы DNS, предназначенные для секции, имеют следующий формат:

```
    <First-Label-Of-Partitioned-Service-DNSName><PartitionPrefix><Target-Partition-Name>< PartitionSuffix>.<Remaining- Partitioned-Service-DNSName>
```
Где:

- *First-Label-Of-Partitioned-Service-DNSName* — это первая часть DNS-имени службы.
- *PartitionPrefix* — это значение, которое можно задать в разделе DnsService манифеста кластера или с помощью шаблона Resource Manager кластера. Значение по умолчанию — "--". Дополнительные сведения см. в разделе [параметров служб DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).
- *Target-Partition-Name* — это имя секции. 
- *PartitionSuffix* — это значение, которое можно задать в разделе DnsService манифеста кластера или с помощью шаблона Resource Manager кластера. Значение по умолчанию — пустая строка. Дополнительные сведения см. в разделе [параметров служб DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).
- *Remaining-Partitioned-Service-DNSName* — это оставшаяся часть DNS-имени службы.

В следующих примерах показаны DNS-запросы для секционированных служб, запущенных в кластере, где используются значения по умолчанию для `PartitionPrefix` и `PartitionSuffix`: 

- Чтобы разрешить секцию "0" службы с DNS-именем `backendrangedschemesvc.application` со схемой секционирования по диапазонам, используйте `backendrangedschemesvc-0.application`.
- Чтобы разрешить первую секцию службы с DNS-именем `backendnamedschemesvc.application` со схемой секционирования по имени, используйте `backendnamedschemesvc-first.application`.

Служба DNS возвращает IP-адрес первичной реплики секции. Если секция не указана, то служба возвращает IP-адрес первичной реплики случайно выбранной секции.

## <a name="using-dns-in-your-services"></a>Использование DNS в службах
При развертывании нескольких служб их конечные точки могут взаимодействовать с помощью DNS-имен. Служба DNS работает для служб без отслеживания состояния, а в Service Fabric версии 6.3 и более поздних версиях и для служб с отслеживанием состояния. Для служб с отслеживанием состояния, выполняющихся на Service Fabric версии ниже 6.3, можно использовать встроенную [службу обратного прокси-сервера](./service-fabric-reverseproxy.md) для HTTP-вызовов, чтобы вызывать определенную секцию службы. 

Динамические порты не поддерживаются службой DNS. Чтобы разрешать службы, использующие динамические порты, можно использовать службу обратного прокси-сервера.

В следующем коде показано, как вызвать службу без отслеживания состояния по DNS-имени. Это простой вызов HTTP, где в качестве части URL-адреса указывается DNS-имя, порт и любой дополнительный путь.

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

В следующем коде показан вызов определенной секции службы с отслеживанием состояния. В этом случае DNS-имя содержит имя секции (partition1). В вызове предполагается использования кластера со значениями по умолчанию для `PartitionPrefix` и `PartitionSuffix`.

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service2-partition1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

## <a name="known-issues"></a>Известные проблемы
* В Service Fabric 6.3 и более поздних версий возникает проблема с запросами DNS для служб, в именах которых содержится дефис в рамках имени DNS. Дополнительные сведения об этой проблеме см. [на сайте GitHub](https://github.com/Azure/service-fabric-issues/issues/1197). Эта проблема будет устранена в следующем обновлении 6.3. 

* Служба DNS для служб Service Fabric пока что не поддерживается на платформе Linux. Служба DNS поддерживается для контейнеров в Linux. В качестве альтернативы можете выполнить разрешение вручную с помощью клиента Fabric или ServicePartitionResolver.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о взаимодействии служб в кластере см. в статье [Подключение к службам в Service Fabric и взаимодействие с ними](service-fabric-connect-and-communicate-with-services.md).
