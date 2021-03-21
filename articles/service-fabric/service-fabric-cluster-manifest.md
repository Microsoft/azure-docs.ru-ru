---
title: Настройка автономного кластера Azure Service Fabric
description: Узнайте, как настраивать изолированный или локальный кластер Service Fabric.
ms.topic: conceptual
ms.date: 11/12/2018
ms.openlocfilehash: fd93263b38340ce080cca1aecb98f3a599ff1861
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91843164"
---
# <a name="configuration-settings-for-a-standalone-windows-cluster"></a>Параметры конфигурации для изолированного кластера Windows
В этой статье описаны параметры конфигурации изолированного кластера Azure Service Fabric, которые можно задать в файле *ClusterConfig.json*. Этот файл используется для указания сведений об узлах кластера, конфигурациях безопасности, а также топологии сети в плане доменов сбоя и обновления.  После изменения или добавления параметров конфигурации вы можете [создать изолированный кластер](service-fabric-cluster-creation-for-windows-server.md) или [обновить конфигурацию изолированного кластера](service-fabric-cluster-config-upgrade-windows-server.md).

В [загружаемый изолированный пакет Service Fabric](service-fabric-cluster-creation-for-windows-server.md#downloadpackage) также включены примеры ClusterConfig.json. Примеры с DevCluster в имени позволяют создать кластер с тремя узлами на одном и том же компьютере, используя логические узлы. По крайней мере один узел должен быть помечен как первичный. Этот тип кластера удобен для работы в среде разработки или тестирования. Но он не предназначен для использования в качестве рабочего кластера. Примеры с MultiMachine в имени позволяют создать кластер рабочего уровня, каждый узел которого размещен на отдельном компьютере. Число первичных узлов такого кластера основано на его [уровне надежности](#reliability). В выпуске 5.7 (версия API 05-2017) мы удалили свойство уровня надежности (reliability-level). Вместо этого код вычисляет наиболее оптимизированный уровень надежности для кластера. Не пытайтесь задать значение для этого свойства начиная с версии 5.7.

* В примерах ClusterConfig.Unsecure.DevCluster.json и ClusterConfig.Unsecure.MultiMachine.json показано, как создать незащищенный тестовый или рабочий кластер соответственно.

* ClusterConfig.Windows.DevCluster.jsв и ClusterConfig.Windows.MultiMachine.jsо том, как создавать тестовые или рабочие кластеры, защищенные с помощью [системы безопасности Windows](service-fabric-windows-cluster-windows-security.md).

* ClusterConfig.X509.DevCluster.jsв и ClusterConfig.X509.MultiMachine.jsПокажите, как создавать тестовые или рабочие кластеры, защищенные с помощью [безопасности на основе сертификата X509](service-fabric-windows-cluster-x509-security.md).

Теперь рассмотрим различные разделы файла ClusterConfig.json.

## <a name="general-cluster-configurations"></a>Общие параметры кластера
Общие параметры кластера охватывают обширные конфигурации конкретного кластера, как показано в следующем фрагменте кода JSON:

```json
    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "01-2017",
```

Для кластера Service Fabric можно задать любое понятное имя, присвоив его переменной name. clusterConfigurationVersion — это номер версии кластера. Его следует увеличивать при каждом обновлении кластера Service Fabric. Оставьте для apiVersion значение по умолчанию.

## <a name="nodes-on-the-cluster"></a>Узлы в кластере
Можно настроить узлы в кластере Service Fabric с помощью раздела nodes, как показано в следующем фрагменте кода:
```json
"nodes": [{
    "nodeName": "vm0",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD0"
}, {
    "nodeName": "vm1",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType1",
    "faultDomain": "fd:/dc1/r1",
    "upgradeDomain": "UD1"
}, {
    "nodeName": "vm2",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType2",
    "faultDomain": "fd:/dc1/r2",
    "upgradeDomain": "UD2"
}],
```

Кластер Service Fabric должен содержать по крайней мере три узла. Можно добавить дополнительные узлы в этот раздел в соответствии с вашей конфигурацией. В следующей таблице описаны параметры конфигурации для каждого узла:

| **конфигурация узла;** | **Описание** |
| --- | --- |
| nodeName |Узлу можно присвоить любое понятное имя. |
| iPAddress |Узнать IP-адрес узла можно, открыв командное окно и введя `ipconfig`. Запишите этот IPv4-адрес и назначьте его переменной iPAddress. |
| nodeTypeRef |Узлам можно назначить разные типы узла. [Типы узлов](#node-types) определены в следующем разделе. |
| faultDomain |Домены сбоя позволяют администраторам кластера определять физические узлы, на которых из-за наличия общих физических зависимостей могут одновременно возникать сбои. |
| upgradeDomain |Домены обновления описывают наборы узлов, которые почти одновременно завершают работу, чтобы выполнить обновления Service Fabric. Так как они не ограничены какими-либо физическими требованиями, вы можете выбрать узлы, которые следует назначить тем или иным доменам обновления. |

## <a name="cluster-properties"></a>Свойства кластера
Раздел свойств в файле ClusterConfig.json используется для настройки кластера указанным ниже образом.

### <a name="reliability"></a>надежность;
Термин reliabilityLevel определяет количество реплик или экземпляров системных служб Service Fabric, которые могут выполняться на первичных узлах кластера. От этого зависит надежность этих служб и, следовательно, кластера. Значение вычисляется системой во время создания и обновления кластера.

### <a name="diagnostics"></a>Диагностика
В разделе diagnosticsStore вы можете настроить параметры, чтобы включить диагностику и устранение неполадок узлов и кластера, как показано в следующем фрагменте кода: 

```json
"diagnosticsStore": {
    "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
    "dataDeletionAgeInDays": "7",
    "storeType": "FileShare",
    "IsEncrypted": "false",
    "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
}
```

Переменная metadata представляет собой описание диагностики кластера и может быть задана в соответствии с вашей конфигурацией. Эти переменные помогают собирать журналы трассировки событий Windows, аварийные дампы и данные счетчиков производительности. Дополнительные сведения о журналах трассировки событий Windows см. в статье [Tracelog](/windows-hardware/drivers/devtest/tracelog) и [Трассировка событий Windows](/dotnet/framework/wcf/samples/etw-tracing). Все журналы, в том числе [аварийные дампы](https://techcommunity.microsoft.com/t5/ask-the-performance-team/bg-p/AskPerf) и данные [счетчиков производительности](/windows/win32/perfctrs/performance-counters-portal), можно перенаправить в папку connectionString на вашем компьютере. Службу хранилища Azure можно также использовать для хранения данных диагностики. См. следующий образец фрагмента кода:

```json
"diagnosticsStore": {
    "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
    "dataDeletionAgeInDays": "7",
    "storeType": "AzureStorage",
    "IsEncrypted": "false",
    "connectionstring": "xstore:DefaultEndpointsProtocol=https;AccountName=[AzureAccountName];AccountKey=[AzureAccountKey]"
}
```

### <a name="security"></a>Безопасность
Раздел security необходим для защиты изолированного кластера Service Fabric. В следующем фрагменте кода показана часть этого раздела:

```json
"security": {
    "metadata": "This cluster is secured using X509 certificates.",
    "ClusterCredentialType": "X509",
    "ServerCredentialType": "X509",
    . . .
}
```

Переменная metadata представляет собой описание защищенного кластера и может быть задана в соответствии с вашей конфигурацией. Переменные ClusterCredentialType и ServerCredentialType определяют тип безопасности, реализуемой кластером и узлами. Для них можно задать значение *X509* для безопасности на основе сертификатов или *Windows* для обеспечения безопасности на основе Active Directory. Остальная часть раздела security зависит от выбранного типа безопасности. Сведения о том, как заполнить остальную часть раздела security, см. в статьях [Защита автономного кластера под управлением Windows с помощью сертификатов X.509](service-fabric-windows-cluster-x509-security.md) или [Защита изолированного кластера под управлением Windows с помощью системы безопасности Windows](service-fabric-windows-cluster-windows-security.md).

### <a name="node-types"></a>Типы узлов
В разделе nodeTypes описывается тип узлов в кластере. Для кластера нужно указать по крайней мере один тип узла, как показано в следующем фрагменте кода: 

```json
"nodeTypes": [{
    "name": "NodeType0",
    "clientConnectionEndpointPort": "19000",
    "clusterConnectionEndpointPort": "19001",
    "leaseDriverEndpointPort": "19002"
    "serviceConnectionEndpointPort": "19003",
    "httpGatewayEndpointPort": "19080",
    "reverseProxyEndpointPort": "19081",
    "applicationPorts": {
        "startPort": "20575",
        "endPort": "20605"
    },
    "ephemeralPorts": {
        "startPort": "20606",
        "endPort": "20861"
    },
    "isPrimary": true
}]
```

name — понятное имя этого конкретного типа узла. Чтобы создать узел данного типа, присвойте его понятное имя переменной nodeTypeRef для этого узла, как [упоминалось раньше](#nodes-on-the-cluster). Для каждого типа узла определите конечные точки подключения, которые будут использоваться. Для этих конечных точек подключения можно выбрать любой номер порта, если он не конфликтует с другими конечными точками в данном кластере. В кластере с несколькими узлами будет один или несколько первичных узлов (т. е. isPrimary имеет значение *true*), в зависимости от значения [reliabilityLevel](#reliability). Сведения о первичных и непервичных типах узлов, в частности о параметрах nodeTypes и reliabilityLevel, см. в статье [Рекомендации по планированию загрузки кластера Service Fabric](service-fabric-cluster-capacity.md). 

#### <a name="endpoints-used-to-configure-the-node-types"></a>Конечные точки, используемые для настройки типов узлов
* clientConnectionEndpointPort — порт, используемый клиентом для подключения к кластеру при использовании клиентских API. 
* clusterConnectionEndpointPort — порт, по которому узлы взаимодействуют друг с другом.
* leaseDriverEndpointPort — порт, используемый драйвером аренды кластера для определения того, активны ли еще узлы. 
* serviceConnectionEndpointPort — порт, используемый приложениями и службами, развернутыми на узле, для связи с клиентом Service Fabric на данном узле.
* httpGatewayEndpointPort — порт, используемый Service Fabric Explorer для подключения к кластеру.
* ephemeralPorts переопределяет [динамические порты, используемые операционной системой](https://support.microsoft.com/kb/929851). Service Fabric использует часть из них в качестве портов приложений, а остальные доступны для операционной системы. Кроме того, этот диапазон сопоставляется с существующим диапазоном в операционной системе, поэтому для любых целей вы можете использовать диапазоны, указанные в примерах файлов JSON. Убедитесь в том, что разница между номерами начального и конечного портов составляет не менее 255. Если это различие слишком мало, могут возникнуть конфликты, потому что этот диапазон также используется для ОС. Чтобы увидеть настроенный динамический диапазон портов, запустите `netsh int ipv4 show dynamicport tcp`.
* applicationPorts — порты, которые используются приложениями Service Fabric. Диапазон портов приложения должен быть достаточным для обеспечения потребностей приложений в конечных точках. Этот диапазон должен отличаться от диапазона динамических портов на компьютере, т. е. диапазона ephemeralPorts, указанного в конфигурации. Service Fabric использует их каждый раз, когда требуются новые порты, а также разрешает их использование в брандмауэре. 
* reverseProxyEndpointPort — это необязательная конечная точка обратного прокси-сервера. Дополнительные сведения см. в статье [Обратный прокси-сервер в Azure Service Fabric](service-fabric-reverseproxy.md). 

### <a name="log-settings"></a>Параметры журнала
В разделе fabricSettings вы можете задать корневые каталоги для данных и журналов Service Fabric. Эти каталоги можно настроить только во время создания исходного кластера. Ниже приведен фрагмент кода этого раздела.

```json
"fabricSettings": [{
    "name": "Setup",
    "parameters": [{
        "name": "FabricDataRoot",
        "value": "C:\\ProgramData\\SF"
    }, {
        "name": "FabricLogRoot",
        "value": "C:\\ProgramData\\SF\\Log"
}]
```

Рекомендуется использовать диск без ОС в качестве FabricDataRoot и FabricLogRoot. Он обеспечивает большую надежность и независимость от ситуаций, когда операционная система перестает отвечать на запросы. Если настроить только корневой каталог данных, то корневой каталог файлов журнала будет помещен на один уровень ниже корневого каталога данных.

### <a name="stateful-reliable-services-settings"></a>Параметры надежной службы с отслеживанием состояния
В разделе KtlLogger можно задать глобальные параметры конфигурации для Reliable Services. Дополнительные сведения об этих параметрах см. в статье [Настройка надежных служб с отслеживанием состояния](service-fabric-reliable-services-configuration.md). Следующий пример показывает, как изменить общий журнал транзакций, который создается для обслуживания Reliable Collections для служб с отслеживанием состояния:

```json
"fabricSettings": [{
    "name": "KtlLogger",
    "parameters": [{
        "name": "SharedLogSizeInMB",
        "value": "4096"
    }]
}]
```

### <a name="add-on-features"></a>Функции надстройки
Для настройки функций надстройки параметр apiVersion должен иметь значение 04-2017 или выше, а для параметра addonFeatures необходимо задать следующее:

```json
"apiVersion": "04-2017",
"properties": {
    "addOnFeatures": [
        "DnsService",
        "RepairManager"
    ]
}
```
Все доступные надстройки можно просмотреть в [справочнике по Service Fabric REST API](/rest/api/servicefabric/sfrp-model-addonfeatures).

### <a name="container-support"></a>Поддержка контейнеров
Чтобы включить в изолированных кластерах поддержку контейнеров для контейнеров Windows Server и Hyper-V, необходимо включить компонент надстройки DnsService.

## <a name="next-steps"></a>Дальнейшие действия
После создания полной *ClusterConfig.js* файла, настроенного в соответствии с настройками автономного кластера, можно развернуть кластер. Следуйте инструкциям в статье [Создание изолированного кластера под управлением Windows Server](service-fabric-cluster-creation-for-windows-server.md), 

Если у вас развернут изолированный кластер, вы также можете [обновить его конфигурацию](service-fabric-cluster-config-upgrade-windows-server.md). 

Узнайте, как [визуализировать кластер с помощью Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).
