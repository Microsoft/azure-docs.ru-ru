---
title: Обязанности клиента, выполняющие Azure Веснного облака в виртуальной сети
description: В этой статье описываются обязанности клиента, запускающие Azure Веснного облака в виртуальной сети.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 12/02/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 0c73d0394486472c2c3c92450aab6a1a0d329cf7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101698218"
---
# <a name="customer-responsibilities-for-running-azure-spring-cloud-in-vnet"></a>Обязанности клиента по запуску Azure Веснного облака в виртуальной сети
В этом документе содержатся спецификации использования Azure Веснного облака в виртуальной сети.

При развертывании Azure Веснного облака в виртуальной сети у него есть Исходящие зависимости от служб, находящихся за пределами виртуальной сети. Для управления и эксплуатации в Azure Веснного облака необходимо получить доступ к определенным портам и полным доменным именам (FQDN). Эти конечные точки необходимы для связи с плоскостью управления облаком Azure весны, а также для загрузки и установки компонентов кластера Core Kubernetes и обновлений безопасности.

По умолчанию Azure Веснного облака имеет неограниченный исходящий доступ к Интернету. Этот уровень доступа к сети позволяет приложениям, которые вы запускаете для доступа к внешним ресурсам по мере необходимости. Если вы хотите ограничить трафик исходящего трафика, для задач обслуживания должно быть доступно ограниченное число портов и адресов. Самым простым решением для защиты исходящих адресов является использование устройства брандмауэра, которое может управлять исходящим трафиком на основе доменных имен. Например, брандмауэр Azure может ограничивать исходящий трафик HTTP и HTTPS на основе полного доменного имени назначения. Вы также можете настроить предпочитаемый брандмауэр и правила безопасности, чтобы разрешить эти порты и адреса.

## <a name="azure-spring-cloud-resource-requirements"></a>Требования к облачным ресурсам Azure весны 

Ниже приведен список требований к ресурсам для облачных служб Azure весны. Как правило, не следует изменять группы ресурсов, созданные с помощью Azure Веснного облака и базовых сетевых ресурсов.
- Не изменяйте группы ресурсов, созданные и принадлежащие облаку Azure Веснного облака.
  - По умолчанию эти группы ресурсов именуются как AP-SVC-rt_ [SERVICE-INSTANCE-NAME]_[регион] * и AP_[Service-instance-name] _ [регион] *.
- Не изменяйте подсети, используемые в Azure Веснного облака.
- Не создавайте более одного экземпляра облачной службы Azure весны в одной подсети.
- При использовании брандмауэра для управления трафиком не *блокируйте следующий* исходящий трафик в облачные компоненты Azure "Весна", которые работают, обслуживают и поддерживают экземпляр службы.

## <a name="azure-spring-cloud-network-requirements"></a>Требования к облачной сети Azure весны

  | Целевая конечная точка | Порт | Использование | Примечание |
  |------|------|------|
  | *: 1194 *или* [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) -AzureCloud: 1194 | UDP: 1194 | Базовое управление кластерами Kubernetes. | |
  | *: 443 *или* [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) -AzureCloud: 443 | TCP: 443 | Управление облачными службами Azure весны. | Сведения об экземпляре службы "Рекуиредтраффикс" могут быть известны в полезных данных ресурса в разделе "networkProfile". |
  | *: 9000 *или* [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) -AzureCloud: 9000 | TCP: 9000 | Базовое управление кластерами Kubernetes. |
  | *: 123 *или* NTP.Ubuntu.com:123 | UDP:123 | Синхронизация времени NTP на узлах Linux. | |
  | *. azure.io:443 *или* [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) -азуреконтаинеррегистри: 443 | TCP: 443 | Реестр контейнеров Azure. | Можно заменить, включив [конечную точку службы](../virtual-network/virtual-network-service-endpoints-overview.md) *реестра контейнеров Azure* в виртуальной сети. |
  | *. core.windows.net:443 и *. core.windows.net:445 *или* [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) -Storage: 443 и хранилище: 445 | TCP: 443, TCP: 445 | Хранилище файлов Azure | Можно заменить, включив [конечную точку службы](../virtual-network/virtual-network-service-endpoints-overview.md) *хранилища Azure* в виртуальной сети. |
  | *. servicebus.windows.net:443 *или* [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) -EventHub: 443 | TCP: 443 | Концентратор событий Azure. | Можно заменить, включив [конечную точку службы](../virtual-network/virtual-network-service-endpoints-overview.md) *концентраторов событий Azure* в виртуальной сети. |
  

## <a name="azure-spring-cloud-fqdn-requirements--application-rules"></a>Требования к полному доменному имени Azure весны в облаке/правила приложений

Брандмауэр Azure предоставляет полное доменное имя (FQDN) **азурекубернетессервице** для упрощения следующих конфигураций.

  | Полное доменное имя назначения | Порт | Использование |
  |------|------|------|
  | *. azmk8s.io | HTTPS:443 | Базовое управление кластерами Kubernetes. |
  | <i>mcr.microsoft.com</i> | HTTPS:443 | Реестр контейнеров (Майкрософт) (мкр). |
  | *.cdn.mscr.io | HTTPS:443 | Хранилище мкр, поддерживаемое Azure CDN. |
  | *.data.mcr.microsoft.com | HTTPS:443 | Хранилище мкр, поддерживаемое Azure CDN. |
  | <i>management.azure.com</i> | HTTPS:443 | Базовое управление кластерами Kubernetes. |
  | <i>login.microsoftonline.com</i> | HTTPS:443 | Azure Active Directory проверка подлинности. |
  |<i>packages.microsoft.com</i>    | HTTPS:443 | Репозиторий пакетов Майкрософт. |
  | <i>acs-mirror.azureedge.net</i> | HTTPS:443 | Для установки необходимых двоичных файлов, таких как кубенет и Azure CNI, требуется репозиторий. |
  | *mscrl.microsoft.com* | HTTPS: 80 | Требуются пути к цепочке сертификатов Майкрософт. |
  | *crl.microsoft.com* | HTTPS: 80 | Требуются пути к цепочке сертификатов Майкрософт. |
  | *crl3.digicert.com* | HTTPS: 80 | Пути сторонних цепочек сертификатов SSL. |

## <a name="see-also"></a>См. также раздел
* [Доступ к приложению в частной сети](spring-cloud-access-app-virtual-network.md)
* [Предоставление приложений с помощью шлюза приложений и брандмауэра Azure](spring-cloud-expose-apps-gateway-azure-firewall.md)