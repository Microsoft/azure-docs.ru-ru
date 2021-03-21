---
title: Схема определения облачных служб Azure (классическая модель) (файл CSDEF) | Документация Майкрософт
description: Файл определения службы (. CSDEF) определяет модель службы для приложения, содержащую доступные роли, конечные точки и значения конфигурации для службы.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: b98534b049698ea95c6738ce3404dd5ef8ff7a28
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102502270"
---
# <a name="azure-cloud-services-classic-definition-schema-csdef-file"></a>Схема определения облачных служб Azure (классическая модель) (файл CSDEF)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

Файл определения службы определяет модель службы для приложения. В этом файле содержатся определения ролей, доступных в облачной службе, указываются конечные точки службы и устанавливаются параметры конфигурации для службы. Значения параметров конфигурации задаются в файле конфигурации службы, который описан в [схеме конфигурации облачной службы (классических)](/previous-versions/azure/reference/ee758710(v=azure.100)).

По умолчанию файл схемы конфигурации системы диагностики Azure устанавливается в каталог `C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK\<version>\schemas`. Замените `<version>` установленной версией [пакета SDK для Azure](https://www.windowsazure.com/develop/downloads/).

По умолчанию определения службы хранятся в файле с расширением .csdef.

## <a name="basic-service-definition-schema"></a>Базовая схема определения службы
Файл определения службы должен содержать один элемент `ServiceDefinition`. Определение службы должно содержать как минимум один элемент роли (`WebRole` или `WorkerRole`). В одном определении могут содержаться до 25 ролей. Также вы можете смешивать типы ролей. Определение службы также содержит необязательный элемент `NetworkTrafficRules`, ограничивающий роли, которые могут взаимодействовать с указанными внутренними конечными точками. Определение службы также содержит необязательный элемент `LoadBalancerProbes`, содержащий определенные пользователем зонды работоспособности конечных точек.

Базовый формат файла определения службы приведен ниже.

```xml
<ServiceDefinition name="<service-name>" topologyChangeDiscovery="<change-type>" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" upgradeDomainCount="<number-of-upgrade-domains>" schemaVersion="<version>">
  
  <LoadBalancerProbes>
         …
  </LoadBalancerProbes>
  
  <WebRole …>
         …
  </WebRole>
  
  <WorkerRole …>
         …
  </WorkerRole>
  
  <NetworkTrafficRules>
         …
  </NetworkTrafficRules>

</ServiceDefinition>
```

## <a name="schema-definitions"></a>Определения схем
В следующих разделах описаны схемы:

- [Схема LoadBalancerProbe](schema-csdef-loadbalancerprobe.md)
- [Схема этой роли](schema-csdef-webrole.md)
- [Схема WorkerRole](schema-csdef-workerrole.md)
- [Схема NetworkTrafficRules](schema-csdef-networktrafficrules.md)

##  <a name="servicedefinition-element"></a><a name="ServiceDefinition"></a> Элемент ServiceDefinition
Элемент `ServiceDefinition` — это элемент верхнего уровня файла определения службы.

В таблице ниже описаны атрибуты элемента `ServiceDefinition`.

| Атрибут               | Описание |
| ----------------------- | ----------- |
| name                    |Обязательный. Имя службы. Имя должно быть уникальным в пределах учетной записи службы.|
| topologyChangeDiscovery | Необязательный параметр. Указывает тип уведомления об изменении топологии. Возможны следующие значения:<br /><br /> -   `Blast` — как можно быстрее отправляет обновление всем экземплярам роли. Чтобы вы могли использовать этот параметр, роль должна иметь возможность обработать обновление топологии без перезапуска.<br />-   `UpgradeDomainWalk` — отправляет обновление каждому экземпляру роли в последовательном режиме после того, как предыдущий экземпляр успешно принял обновление.|
| schemaVersion           | Необязательный параметр. Указывает версию схемы определения службы. Версия схемы позволяет Visual Studio выбрать правильные средства пакета SDK для использования при проверке схемы, если установлено одновременно несколько версий пакета SDK.|
| upgradeDomainCount      | Необязательный параметр. Указывает число доменов обновления, между которыми распределены роли в этой службе. Экземпляры ролей назначаются домену обновления при развертывании службы. Дополнительные сведения см. в статьях [Обновление роли или развертывания облачной службы](cloud-services-how-to-manage-portal.md#update-a-cloud-service-role-or-deployment), [Управление доступностью виртуальных машин](../virtual-machines/availability.md) и [модель облачной службы](./cloud-services-model-and-package.md).<br /><br /> Можно указать до 20 доменов обновления. Если число доменов обновления не указано, по умолчанию оно равно 5.|