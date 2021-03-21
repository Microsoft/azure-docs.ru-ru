---
title: Типы узлов и масштабируемые наборы виртуальных машин
description: Сведения о том, как типы узлов Service Fabric Azure связаны с масштабируемыми наборами виртуальных машин и удаленным подключением к экземпляру масштабируемого набора или узлу кластера.
ms.topic: conceptual
ms.date: 03/23/2018
ms.author: pepogors
ms.openlocfilehash: 9e30c02de54806006a1881448bcb9f788a57310c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97095259"
---
# <a name="azure-service-fabric-node-types-and-virtual-machine-scale-sets"></a>Типы узлов Azure Service Fabric и масштабируемые наборы виртуальных машин

[Масштабируемые наборы виртуальных машин](../virtual-machine-scale-sets/index.yml) являются вычислительными ресурсами Azure. Их можно использовать для развертывания коллекций виртуальных машин и управления ими в качестве набора. Каждый тип узла, определенный в кластере Azure Service Fabric, устанавливает ровно один масштабируемый набор: несколько типов узлов не могут поддерживаться одним и тем же масштабируемым набором, и один тип узла не должен поддерживаться несколькими масштабируемыми наборами.

Среда выполнения Service Fabric устанавливается на каждой виртуальной машине в масштабируемом наборе с помощью расширения виртуальной машины *Microsoft. Azure. ServiceFabric* . Все типы узлов можно масштабировать независимо друг от друга, изменять номер SKU операционной системы, работающей на узле кластера, открывать разные наборы портов и использовать различные метрики производительности.

На следующем рисунке показан кластер с двумя типами узлов, именуемыми *интерфейсным* и *внутренним*. Каждый тип узла имеет пять узлов.

![Кластер с двумя типами узлов][NodeTypes]

## <a name="map-virtual-machine-scale-set-instances-to-nodes"></a>Сопоставление экземпляров масштабируемых наборов виртуальных машин с узлами

Как показано на предыдущем рисунке, экземпляры масштабируемого набора начинаются с экземпляра 0, а затем увеличиваются на 1. Нумерация узлов отражается в именах. Например, узел BackEnd_0 является нулевым экземпляром масштабируемого набора BackEnd. Этот конкретный масштабируемый набор имеет пять экземпляров с именами BackEnd_0, BackEnd_1, BackEnd_2, BackEnd_3 и BackEnd_4.

При масштабировании масштабируемого набора создается новый экземпляр. Имя нового экземпляра масштабируемого набора, как правило, состоит из имени масштабируемого набора и номера следующего экземпляра. В нашем примере это BackEnd_5.

## <a name="map-scale-set-load-balancers-to-node-types-and-scale-sets"></a>Сопоставление балансировщиков нагрузки масштабируемых наборов с типами узлов и масштабируемыми наборами

При развертывании кластера на портале Azure или использовании примера шаблона Azure Resource Manager перечисляются все ресурсы в группе ресурсов. Отображаются балансировщики нагрузки для каждого масштабируемого набора или типа узла. Имя балансировщика нагрузки имеет следующий формат: **LB-&lt;имя типа узла&gt;**, например LB-sfcluster4doc-0, как показано на следующем рисунке:

![На снимке экрана показана группа ресурсов с двумя выделенными подсистемами балансировки нагрузки.][Resources]

## <a name="service-fabric-virtual-machine-extension"></a>Расширение виртуальной машины Service Fabric

Service Fabric расширение виртуальной машины используется для начальной загрузки Service Fabric на виртуальные машины Azure и настройки безопасности узла.

Ниже приведен фрагмент кода расширения виртуальной машины Service Fabric.

```json
"extensions": [
  {
    "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
    "properties": {
      "type": "ServiceFabricLinuxNode",
      "autoUpgradeMinorVersion": true,
      "protectedSettings": {
        "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
       },
       "publisher": "Microsoft.Azure.ServiceFabric",
       "settings": {
         "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
         "nodeTypeRef": "[variables('vmNodeType0Name')]",
         "durabilityLevel": "Silver",
         "enableParallelJobs": true,
         "nicPrefixOverride": "[variables('subnet0Prefix')]",
         "dataPath": "D:\\\\SvcFab",
         "certificate": {
           "commonNames": [
             "[parameters('certificateCommonName')]"
           ],
           "x509StoreName": "[parameters('certificateStoreValue')]"
         }
       },
       "typeHandlerVersion": "1.1"
     }
   },
```

Ниже приведены описания свойств.

| **имя**; | **Допустимые значения** | **Рекомендация или краткое описание** |
| --- | --- | --- | --- |
| name | строка | Уникальное имя для расширения |
| тип | "Сервицефабриклинуксноде" или "Сервицефабриквиндовсноде" | Идентифицирует Service Fabric ОС для начальной загрузки |
| autoUpgradeMinorVersion | true или false | Включить автоматическое обновление вспомогательных версий среды выполнения SF |
| publisher | Microsoft. Azure. ServiceFabric | Имя издателя расширения Service Fabric |
| клустерендпонт | строка | URI: порт для конечной точки управления |
| nodeTypeRef | строка | Имя nodeType |
| дурабилитилевел | Бронзовая, серебристая, золотость, Platinum | Время, отведенное на приостановку работы неизменяемой инфраструктуры Azure |
| енаблепараллелжобс | true или false | Включите COMPUTE Параллелжобс, например Remove VM и перезагрузите виртуальную машину в том же масштабируемом наборе параллельно |
| никпрефиксоверриде | строка | Префикс подсети, например "10.0.0.0/24" |
| коммоннамес | string[] | Распространенные имена установленных сертификатов кластера |
| Заданного параметром x509storename | строка | Имя магазина, в котором находится установленный сертификат кластера |
| typeHandlerVersion | 1,1 | Версия расширения. 1,0. рекомендуется использовать классическую версию расширения для обновления до 1,1 |
| Пути к данным | строка | Путь к диску, используемому для сохранения состояния Service Fabric системных служб и данных приложений.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о возможности развертывания в любом месте и сравнении с кластерами под управлением Azure см. в статье [Создание кластеров Service Fabric в Windows Server или Linux](service-fabric-deploy-anywhere.md).
* Сведения о [безопасности кластера](service-fabric-cluster-security.md).
* [Удаленное подключение](service-fabric-cluster-remote-connect-to-azure-cluster-node.md) к определенному экземпляру масштабируемого набора.
* [Обновление значений диапазона портов RDP](./scripts/service-fabric-powershell-change-rdp-port-range.md) в кластере виртуальных машин после развертывания
* [Обновление имени пользователя и пароля администратора](./scripts/service-fabric-powershell-change-rdp-user-and-pw.md) для виртуальных машин кластера

<!--Image references-->
[NodeTypes]: ./media/service-fabric-cluster-nodetypes/NodeTypes.png
[Resources]: ./media/service-fabric-cluster-nodetypes/Resources.png
[InboundNatPools]: ./media/service-fabric-cluster-nodetypes/InboundNatPools.png
