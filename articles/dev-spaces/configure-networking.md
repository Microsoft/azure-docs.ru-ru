---
title: Настройка сети для Azure Dev Spaces в различных сетевых топологиях
services: azure-dev-spaces
ms.date: 03/17/2020
ms.topic: conceptual
description: Описание требований к сети для запуска Azure Dev Spaces в Azure Kubernetes Services
keywords: Azure Dev Spaces, пространства разработки, Docker, Kubernetes, Azure, AKS, служба Kubernetes Azure, контейнеры, CNI, кубенет, SDN, сеть
ms.openlocfilehash: 09114ab13555cbf9ef42b37c86ffb76a8fe3ab3f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91970344"
---
# <a name="configure-networking-for-azure-dev-spaces-in-different-network-topologies"></a>Настройка сети для Azure Dev Spaces в различных сетевых топологиях

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces выполняется в кластерах Azure Kubernetes Service (AKS) с конфигурацией сети по умолчанию. Если вы хотите изменить конфигурацию сети кластера AKS, например размещение кластера за брандмауэром, использование групп безопасности сети или политик сети, необходимо включить дополнительные рекомендации для запуска Azure Dev Spaces.

![Конфигурация виртуальной сети](media/configure-networking/virtual-network-clusters.svg)

## <a name="virtual-network-or-subnet-configurations"></a>Конфигурация виртуальной сети или подсети

Кластер AKS может иметь другую конфигурацию виртуальной сети или подсети для ограничения входящего или исходящего трафика для кластера AKS. Например, кластер может находиться за брандмауэром, например брандмауэром Azure, или же можно использовать группы безопасности сети или пользовательские роли, чтобы ограничивать сетевой трафик. Пример конфигурации сети можно найти в [Azure dev Spaces примере репозитория на сайте GitHub][sample-repo].

Azure Dev Spaces имеет определенные требования для входящего *и* исходящего сетевого трафика, а также входящего *трафика.* Если вы используете Azure Dev Spaces в кластере AKS с конфигурацией виртуальной сети или подсети, которая ограничивают трафик для кластера AKS, для правильной работы Azure Dev Spaces необходимо следовать следующим требованиям к входящему и входящему трафику, а также порядку исходящего трафика.

### <a name="ingress-and-egress-network-traffic-requirements"></a>Требования к сетевому трафику входящих и исходящих данных

Azure Dev Spaces требуется входящий и исходящий трафик для следующих полных доменных имен:

| Полное доменное имя.                       | Порт       | Использование      |
|----------------------------|------------|----------|
| cloudflare.docker.com      | HTTPS: 443 | Извлечение образов DOCKER для Azure Dev Spaces |
| gcr.io                     | HTTPS: 443 | Извлечение Helm изображений для Azure Dev Spaces |
| storage.googleapis.com     | HTTPS: 443 | Извлечение Helm изображений для Azure Dev Spaces |

Обновите брандмауэр или конфигурацию безопасности, чтобы разрешить сетевой трафик для всех указанных выше доменных имен и [служб Azure dev Spaces инфраструктуры][service-tags]. Например, если для защиты сети используется брандмауэр, указанные выше полные доменные имена должны быть добавлены в правило приложения брандмауэра, а тег службы Azure Dev Spaces также должен быть [добавлен в брандмауэр][firewall-service-tags]. Оба эти обновления брандмауэра должны разрешать трафик из этих доменов и из них.

### <a name="ingress-only-network-traffic-requirements"></a>Только входящие требования к сетевому трафику

Azure Dev Spaces обеспечивает маршрутизацию на уровне пространства имен Kubernetes, а также открытый доступ к службам с помощью собственного полного доменного имени. Чтобы обе эти функции работали, обновите брандмауэр или конфигурацию сети, чтобы разрешить общий вход на внешний IP-адрес контроллера Azure Dev Spaces входящего трафика в кластере. Кроме того, можно создать [внутреннюю подсистему балансировки нагрузки][aks-internal-lb] и добавить правило NAT в брандмауэр, чтобы перевести общедоступный IP-адрес брандмауэра на IP-адрес внутренней подсистемы балансировки нагрузки. Можно также использовать [траефик][traefik-ingress] или [nginx][nginx-ingress] для создания пользовательского контроллера входящего трафика.

## <a name="aks-cluster-network-requirements"></a>Требования к сети кластера AKS

AKS позволяет использовать [Сетевые политики][aks-network-policies] для управления входящим и исходящим трафиком между модулями Pod в кластере, а также исходящим трафиком из Pod. Azure Dev Spaces имеет определенные требования для входящего *и* исходящего сетевого трафика, а также входящего *трафика.* Если вы используете Azure Dev Spaces в кластере AKS с политиками сети AKS, то для правильной работы Azure Dev Spaces необходимо следовать следующим требованиям к входящему и входящему трафику, а также порядку исходящего трафика.

### <a name="ingress-and-egress-network-traffic-requirements"></a>Требования к сетевому трафику входящих и исходящих данных

Azure Dev Spaces позволяет напрямую взаимодействовать с Pod в пространстве разработчика в кластере для отладки. Чтобы эта функция работала, добавьте сетевую политику, которая разрешает входящий и исходящий обмен данными с IP-адресами инфраструктуры Azure Dev Spaces, которая [зависит от региона][service-tags].

### <a name="ingress-only-network-traffic-requirements"></a>Только входящие требования к сетевому трафику

Azure Dev Spaces обеспечивает маршрутизацию между модулями Pod между пространствами имен. Например, пространства имен с включенным Azure Dev Spaces могут иметь связь типа «родители-потомки», которая позволяет маршрутизировать сетевой трафик между модулями Pod в родительском и дочернем пространствах имен. Azure Dev Spaces также предоставляет конечные точки службы, используя собственное полное доменное имя. Чтобы настроить различные способы предоставления служб и их влияние на маршрутизацию на уровне пространства имен, см. раздел [использование различных параметров конечной точки][endpoint-options].

## <a name="using-azure-cni"></a>Использование Azure CNI

По умолчанию кластеры AKS настроены для использования [кубенет][aks-kubenet] для работы в сети, которая работает с Azure dev Spaces. Вы также можете настроить кластер AKS для использования [сетевого интерфейса контейнера Azure (CNI)][aks-cni]. Чтобы использовать Azure Dev Spaces с Azure CNI в кластере AKS, разрешите виртуальной сети и адресным пространствам подсети до 10 частных IP-адресов для модулей Pod, развернутых Azure Dev Spaces. Дополнительные сведения о разрешении частных IP-адресов доступны в [документации по AKS Azure CNI][aks-cni-ip-planning].

## <a name="using-api-server-authorized-ip-ranges"></a>Использование IP-диапазонов полномочных серверов API

Кластеры AKS позволяют настроить дополнительную безопасность, которая ограничивает IP-адрес, который может взаимодействовать с кластерами, например с помощью пользовательских виртуальных сетей или путем [защиты доступа к серверу API с помощью разрешенных IP-диапазонов][aks-ip-auth-ranges]. Чтобы использовать Azure Dev Spaces при использовании этой дополнительной защиты при [создании][aks-ip-auth-range-create] кластера, необходимо [разрешить дополнительные диапазоны на основе вашего региона][service-tags]. Можно также [Обновить][aks-ip-auth-range-update] существующий кластер, чтобы разрешить эти дополнительные диапазоны. Также необходимо разрешить IP-адрес любых компьютеров разработки, подключающихся к кластеру AKS для отладки, для подключения к серверу API.

## <a name="using-aks-private-clusters"></a>Использование частных кластеров AKS

В настоящее время Azure Dev Spaces не поддерживается для [частных кластеров AKS][aks-private-clusters].

## <a name="using-different-endpoint-options"></a>Использование различных параметров конечной точки

Azure Dev Spaces предоставляет возможность предоставлять конечные точки для служб, работающих на AKS. При включении Azure Dev Spaces в кластере доступны следующие варианты настройки типа конечной точки для кластера.

* *Общедоступная* конечная точка, используемая по умолчанию, развертывает входной контроллер с общедоступным IP-адресом. Общедоступный IP-адрес регистрируется в DNS-кластере, что обеспечивает общий доступ к службам с помощью URL-адреса. Этот URL-адрес можно просмотреть с помощью команды `azds list-uris` .
* *Частная* конечная точка развертывает входной контроллер с частным IP-адресом. При наличии частного IP-адреса подсистема балансировки нагрузки для кластера доступна только в виртуальной сети кластера. Частный IP-адрес подсистемы балансировки нагрузки регистрируется на DNS-сервере кластера, чтобы службы в виртуальной сети кластера могли получить доступ по URL-адресу. Этот URL-адрес можно просмотреть с помощью команды `azds list-uris` .
* Если параметр *None* установлен для конечной точки, то не будет развернут контроллер входящих данных. При отсутствии развернутого контроллера входящих данных [возможности маршрутизации Azure dev Spaces][dev-spaces-routing] не будут работать. При необходимости можно реализовать собственное решение контроллера входящих данных с помощью [траефик][traefik-ingress] или [nginx][nginx-ingress], которое позволит снова работать с возможностями маршрутизации.

Чтобы настроить параметр конечной точки, используйте *-e* или *--Endpoint* при включении Azure dev Spaces в кластере. Пример:

> [!NOTE]
> Для параметра конечной точки требуется, чтобы вы выполняли Azure CLI версии 2.2.0 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

```azurecli
az aks use-dev-spaces -g MyResourceGroup -n MyAKS -e private
```

## <a name="client-requirements"></a>Требования к клиенту

Azure Dev Spaces использует средства на стороне клиента, такие как расширение CLI Azure Dev Spaces, расширение Visual Studio Code и расширение Visual Studio, чтобы взаимодействовать с кластером AKS для отладки. Чтобы использовать средства Azure Dev Spaces на стороне клиента, разрешите трафик от компьютеров разработки к [инфраструктуре Azure dev Spaces][dev-spaces-allow-infrastructure]. При использовании [IP-диапазонов разрешенного сервера API][auth-range-section]необходимо также разрешить IP-адрес любых компьютеров разработки, подключающихся к кластеру AKS для отладки, для подключения к серверу API.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о принципах работы Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Принцип работы Azure Dev Spaces](how-dev-spaces-works.md)

[aks-cni]: ../aks/configure-azure-cni.md
[aks-cni-ip-planning]: ../aks/configure-azure-cni.md#plan-ip-addressing-for-your-cluster
[aks-kubenet]: ../aks/configure-kubenet.md
[aks-internal-lb]: ../aks/internal-lb.md
[aks-ip-auth-ranges]: ../aks/api-server-authorized-ip-ranges.md
[aks-ip-auth-range-create]: ../aks/api-server-authorized-ip-ranges.md#create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled
[aks-ip-auth-range-update]: ../aks/api-server-authorized-ip-ranges.md#update-a-clusters-api-server-authorized-ip-ranges
[aks-network-policies]: ../aks/use-network-policies.md
[aks-private-clusters]: ../aks/private-clusters.md
[auth-range-section]: #using-api-server-authorized-ip-ranges
[azure-cli-install]: /cli/azure/install-azure-cli
[dev-spaces-allow-infrastructure]: #virtual-network-or-subnet-configurations
[dev-spaces-routing]: how-dev-spaces-works-routing.md
[endpoint-options]: #using-different-endpoint-options
[firewall-service-tags]: ../firewall/service-tags.md
[traefik-ingress]: how-to/ingress-https-traefik.md
[nginx-ingress]: how-to/ingress-https-nginx.md
[sample-repo]: https://github.com/Azure/dev-spaces/tree/master/advanced%20networking
[service-tags]: ../virtual-network/service-tags-overview.md#available-service-tags