---
title: Ограничения для исходящего трафика в Службе Azure Kubernetes (AKS)
description: Сведения о том, какие порты и адреса используются для управления исходящим трафиком в службе Azure Kubernetes (AKS)
services: container-service
ms.topic: article
ms.author: jpalma
ms.date: 11/09/2020
author: palma21
ms.openlocfilehash: 93c8d1392de8f502a829276287a4687476dd36de
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102505064"
---
# <a name="control-egress-traffic-for-cluster-nodes-in-azure-kubernetes-service-aks"></a>Управление исходящим трафиком для узлов кластера в службе Azure Kubernetes (AKS)

В этой статье содержатся необходимые сведения, позволяющие защитить исходящий трафик от службы Azure Kubernetes (AKS). Он содержит требования к кластеру для базового развертывания AKS, а также дополнительные требования к дополнительным надстройкам и функциям. В [конце приведен пример того, как настроить эти требования с помощью брандмауэра Azure](#restrict-egress-traffic-using-azure-firewall). Однако эти сведения можно применить к любому методу ограничения или устройству исходящего трафика.

## <a name="background"></a>Фон

Кластеры AKS развертываются в виртуальной сети. Эта сеть может быть управляемой (созданной AKS) или настраиваемой (предварительно настроенной пользователем). В любом случае кластер имеет Исходящие зависимости от служб, **находящихся** за пределами этой виртуальной сети (служба не имеет входящих зависимостей).

Для управления и эксплуатации узлам в кластере AKS нужен доступ к определенным портам и полным доменным именам (FQDN). Эти конечные точки необходимы для взаимодействия узлов с сервером API, а также для загрузки и установки компонентов кластера Core Kubernetes и обновлений безопасности узла. Например, кластеру необходимо извлечь базовые образы системных контейнеров из реестра контейнеров (мкр) Microsoft.

Исходящие зависимости AKS практически полностью определяются полным доменным Именом, у которых нет статических адресов. Отсутствие статических адресов означает, что группы безопасности сети нельзя использовать для блокировки исходящего трафика из кластера AKS. 

По умолчанию кластеры AKS имеют неограниченный исходящий доступ к Интернету. Такой уровень сетевого доступа позволяет работающим узлам и службам обращаться к внешним ресурсам по мере необходимости. Если вы хотите ограничить исходящий трафик, нужно сохранить доступ для ограниченного числа портов и адресов, чтобы обеспечить работоспособность для задач обслуживания кластера. Простейшее решение для защиты исходящих адресов заключается в использовании устройства брандмауэра, которое может управлять исходящим трафиком на основе доменных имен. Например, брандмауэр Azure может ограничивать исходящий трафик HTTP и HTTPS на основе полного доменного имени назначения. Вы также можете настроить предпочитаемый брандмауэр и правила безопасности, чтобы разрешить эти порты и адреса.

> [!IMPORTANT]
> В этом документе рассматривается только блокировка трафика, выходящего из подсети AKS. По умолчанию AKS не имеет требований к входящим значениям.  Блокировка **трафика внутренней подсети** с помощью групп безопасности сети (группы безопасности сети) и брандмауэров не поддерживается. Для управления и блокировки трафика в кластере используйте [**_политики сети_**][network-policy].

## <a name="required-outbound-network-rules-and-fqdns-for-aks-clusters"></a>Необходимые правила исходящей сети и FQDN для кластеров AKS

Для кластера AKS требуются следующие правила сети и полное доменное имя/параметры приложения. их можно использовать, если вы хотите настроить решение, отличное от брандмауэра Azure.

* Зависимости IP-адреса задаются для трафика, не использующего протокол HTTP/HTTPS (TCP и UDP-трафик)
* В устройство брандмауэра можно добавить FQDN конечных точек для протокола HTTP/HTTPS.
* Конечные точки HTTP и HTTPS с подстановочными знаками являются зависимостями, которые могут различаться в зависимости от количества квалификаторов в кластере AKS.
* AKS использует контроллер допуска для внедрения полного доменного имени в качестве переменной среды для всех развертываний в Kube-System и привратник-System, что гарантирует, что все системные соединения между узлами и сервером API используют полное доменное имя сервера API, а не IP-адрес сервера API. 
* Если у вас есть приложение или решение, которое должно взаимодействовать с сервером API, необходимо добавить **дополнительное** сетевое правило, разрешающее *TCP-соединение с портом 443 сервера API*.
* В редких случаях при выполнении операции обслуживания IP-адрес сервера API может измениться. Запланированные операции обслуживания, которые могут изменить IP-адрес сервера API, всегда передаются заранее.


### <a name="azure-global-required-network-rules"></a>Глобальные обязательные правила сети Azure

Необходимые правила сети и зависимости IP-адресов:

| Целевая конечная точка                                                             | Протокол | Порт    | Использование  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *Or* <br/> [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureCloud.<Region>:1194`** <br/> *Or* <br/> [Региональные Цидрс](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) - **`RegionCIDRs:1194`** <br/> *Or* <br/> **`APIServerPublicIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Для туннелирования безопасного взаимодействия между узлами и плоскостью управления. Это не требуется для [частных кластеров](private-clusters.md)|
| **`*:9000`** <br/> *Or* <br/> [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureCloud.<Region>:9000`** <br/> *Or* <br/> [Региональные Цидрс](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) - **`RegionCIDRs:9000`** <br/> *Or* <br/> **`APIServerPublicIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Для туннелирования безопасного взаимодействия между узлами и плоскостью управления. Это не требуется для [частных кластеров](private-clusters.md) |
| **`*:123`** или **`ntp.ubuntu.com:123`** (при использовании сетевых правил брандмауэра Azure)  | UDP      | 123     | Требуется для синхронизации времени по протоколу NTP на узлах Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Если вы используете пользовательские DNS-серверы, необходимо убедиться, что они доступны для узлов кластера. |
| **`APIServerPublicIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Требуется при запуске модулей Pod или развертываний, обращающихся к серверу API, эти модули и развертывания будут использовать IP-адрес API. Это не требуется для [частных кластеров](private-clusters.md)  |

### <a name="azure-global-required-fqdn--application-rules"></a>Глобальное требуемое полное доменное имя или правила приложения Azure 

Приведенные ниже FQDN и правила приложения являются обязательными.

| Полное доменное имя назначения                 | Порт            | Использование      |
|----------------------------------|-----------------|----------|
| **`*.hcp.<location>.azmk8s.io`** | **`HTTPS:443`** | Требуется для обмена данными с сервером < node->. Замените на регион, в *\<location\>* котором развернут кластер AKS. |
| **`mcr.microsoft.com`**          | **`HTTPS:443`** | Требуется для доступа к образам в реестре контейнеров Microsoft (мкр). Этот реестр содержит образы и диаграммы от сторонних производителей (например, Кореднс и т. д.). Эти образы необходимы для правильного создания и функционирования кластера, включая операции масштабирования и обновления.  |
| **`*.data.mcr.microsoft.com`**   | **`HTTPS:443`** | Требуется для хранилища мкр, поддерживаемого сетью доставки содержимого (CDN) Azure. |
| **`management.azure.com`**       | **`HTTPS:443`** | Требуется для операций Kubernetes с API Azure. |
| **`login.microsoftonline.com`**  | **`HTTPS:443`** | Требуется для Azure Active Directory проверки подлинности. |
| **`packages.microsoft.com`**     | **`HTTPS:443`** | Этот адрес содержит репозиторий пакетов Майкрософт, которые используются для кэширования операций *apt-get*.  Например, здесь хранятся пакеты для Moby, PowerShell и Azure CLI. |
| **`acs-mirror.azureedge.net`**   | **`HTTPS:443`** | Этот адрес предназначен для репозитория, необходимого для скачивания и установки необходимых двоичных файлов, таких как кубенет и Azure CNI. |

### <a name="azure-china-21vianet-required-network-rules"></a>Необходимые правила сети для Azure китайского Китая

Необходимые правила сети и зависимости IP-адресов:

| Целевая конечная точка                                                             | Протокол | Порт    | Использование  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *Or* <br/> [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureCloud.Region:1194`** <br/> *Or* <br/> [Региональные Цидрс](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) - **`RegionCIDRs:1194`** <br/> *Or* <br/> **`APIServerPublicIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Для туннелирования безопасного взаимодействия между узлами и плоскостью управления. |
| **`*:9000`** <br/> *Or* <br/> [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureCloud.<Region>:9000`** <br/> *Or* <br/> [Региональные Цидрс](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) - **`RegionCIDRs:9000`** <br/> *Or* <br/> **`APIServerPublicIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Для туннелирования безопасного взаимодействия между узлами и плоскостью управления. |
| **`*:22`** <br/> *Or* <br/> [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureCloud.<Region>:22`** <br/> *Or* <br/> [Региональные Цидрс](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) - **`RegionCIDRs:22`** <br/> *Or* <br/> **`APIServerPublicIP:22`** `(only known after cluster creation)`  | TCP           | 22      | Для туннелирования безопасного взаимодействия между узлами и плоскостью управления. |
| **`*:123`** или **`ntp.ubuntu.com:123`** (при использовании сетевых правил брандмауэра Azure)  | UDP      | 123     | Требуется для синхронизации времени по протоколу NTP на узлах Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Если вы используете пользовательские DNS-серверы, необходимо убедиться, что они доступны для узлов кластера. |
| **`APIServerPublicIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Требуется при запуске модулей Pod или развертываний, обращающихся к серверу API, эти модули и развертывания будут использовать IP-адрес API.  |

### <a name="azure-china-21vianet-required-fqdn--application-rules"></a>В Azure для Китая требуется полное доменное имя и правила приложения

Приведенные ниже FQDN и правила приложения являются обязательными.

| Полное доменное имя назначения                               | Порт            | Использование      |
|------------------------------------------------|-----------------|----------|
| **`*.hcp.<location>.cx.prod.service.azk8s.cn`**| **`HTTPS:443`** | Требуется для обмена данными с сервером < node->. Замените на регион, в *\<location\>* котором развернут кластер AKS. |
| **`*.tun.<location>.cx.prod.service.azk8s.cn`**| **`HTTPS:443`** | Требуется для обмена данными с сервером < node->. Замените на регион, в *\<location\>* котором развернут кластер AKS. |
| **`mcr.microsoft.com`**                        | **`HTTPS:443`** | Требуется для доступа к образам в реестре контейнеров Microsoft (мкр). Этот реестр содержит образы и диаграммы от сторонних производителей (например, Кореднс и т. д.). Эти образы необходимы для правильного создания и функционирования кластера, включая операции масштабирования и обновления. |
| **`.data.mcr.microsoft.com`**                  | **`HTTPS:443`** | Требуется для хранилища мкр, поддерживаемого сетью доставки содержимого (CDN) Azure. |
| **`management.chinacloudapi.cn`**              | **`HTTPS:443`** | Требуется для операций Kubernetes с API Azure. |
| **`login.chinacloudapi.cn`**                   | **`HTTPS:443`** | Требуется для Azure Active Directory проверки подлинности. |
| **`packages.microsoft.com`**                   | **`HTTPS:443`** | Этот адрес содержит репозиторий пакетов Майкрософт, которые используются для кэширования операций *apt-get*.  Например, здесь хранятся пакеты для Moby, PowerShell и Azure CLI. |
| **`*.azk8s.cn`**                               | **`HTTPS:443`** | Этот адрес предназначен для репозитория, необходимого для скачивания и установки необходимых двоичных файлов, таких как кубенет и Azure CNI. |

### <a name="azure-us-government-required-network-rules"></a>Правила сети, необходимые для государственных организаций США

Необходимые правила сети и зависимости IP-адресов:

| Целевая конечная точка                                                             | Протокол | Порт    | Использование  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *Or* <br/> [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureCloud.<Region>:1194`** <br/> *Or* <br/> [Региональные Цидрс](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) - **`RegionCIDRs:1194`** <br/> *Or* <br/> **`APIServerPublicIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Для туннелирования безопасного взаимодействия между узлами и плоскостью управления. |
| **`*:9000`** <br/> *Or* <br/> [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureCloud.<Region>:9000`** <br/> *Or* <br/> [Региональные Цидрс](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) - **`RegionCIDRs:9000`** <br/> *Or* <br/> **`APIServerPublicIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Для туннелирования безопасного взаимодействия между узлами и плоскостью управления. |
| **`*:123`** или **`ntp.ubuntu.com:123`** (при использовании сетевых правил брандмауэра Azure)  | UDP      | 123     | Требуется для синхронизации времени по протоколу NTP на узлах Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Если вы используете пользовательские DNS-серверы, необходимо убедиться, что они доступны для узлов кластера. |
| **`APIServerPublicIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Требуется при запуске модулей Pod или развертываний, обращающихся к серверу API, эти модули и развертывания будут использовать IP-адрес API.  |

### <a name="azure-us-government-required-fqdn--application-rules"></a>Для государственных организаций США требуются полные доменные имена и правила приложений 

Приведенные ниже FQDN и правила приложения являются обязательными.

| Полное доменное имя назначения                                        | Порт            | Использование      |
|---------------------------------------------------------|-----------------|----------|
| **`*.hcp.<location>.cx.aks.containerservice.azure.us`** | **`HTTPS:443`** | Требуется для обмена данными с сервером < node->. Замените на регион, в *\<location\>* котором развернут кластер AKS.|
| **`mcr.microsoft.com`**                                 | **`HTTPS:443`** | Требуется для доступа к образам в реестре контейнеров Microsoft (мкр). Этот реестр содержит образы и диаграммы от сторонних производителей (например, Кореднс и т. д.). Эти образы необходимы для правильного создания и функционирования кластера, включая операции масштабирования и обновления. |
| **`*.data.mcr.microsoft.com`**                          | **`HTTPS:443`** | Требуется для хранилища мкр, поддерживаемого сетью доставки содержимого (CDN) Azure. |
| **`management.usgovcloudapi.net`**                      | **`HTTPS:443`** | Требуется для операций Kubernetes с API Azure. |
| **`login.microsoftonline.us`**                          | **`HTTPS:443`** | Требуется для Azure Active Directory проверки подлинности. |
| **`packages.microsoft.com`**                            | **`HTTPS:443`** | Этот адрес содержит репозиторий пакетов Майкрософт, которые используются для кэширования операций *apt-get*.  Например, здесь хранятся пакеты для Moby, PowerShell и Azure CLI. |
| **`acs-mirror.azureedge.net`**                          | **`HTTPS:443`** | Это адрес для репозитория, необходимого для установки обязательных двоичных файлов, таких как kubenet и Azure CNI. |

## <a name="optional-recommended-fqdn--application-rules-for-aks-clusters"></a>Необязательное рекомендуемое полное доменное имя или правила приложения для кластеров AKS

Следующие правила полного доменного имени и приложения необязательны, но рекомендуются для кластеров AKS:

| Полное доменное имя назначения                                                               | Порт          | Использование      |
|--------------------------------------------------------------------------------|---------------|----------|
| **`security.ubuntu.com`, `azure.archive.ubuntu.com`, `changelogs.ubuntu.com`** | **`HTTP:80`** | Этот адрес позволяет узлам кластера Linux скачивать необходимые исправления и обновления для системы безопасности. |

Если вы решили заблокировать или запретить эти полные доменные имена, узлы получат обновления операционной системы только при [обновлении образа узла](node-image-upgrade.md) или [обновлении кластера](upgrade-cluster.md).

## <a name="gpu-enabled-aks-clusters"></a>Кластеры AKS с поддержкой GPU

### <a name="required-fqdn--application-rules"></a>Требуемое полное доменное имя/правила приложения

Для работы кластеров AKS с поддержкой GPU требуются следующие FQDN и правила приложения.

| Полное доменное имя назначения                        | Порт      | Использование      |
|-----------------------------------------|-----------|----------|
| **`nvidia.github.io`**                  | **`HTTPS:443`** | Этот адрес используется для правильной установки и работы драйвера на узлах на основе GPU. |
| **`us.download.nvidia.com`**            | **`HTTPS:443`** | Этот адрес используется для правильной установки и работы драйвера на узлах на основе GPU. |
| **`apt.dockerproject.org`**             | **`HTTPS:443`** | Этот адрес используется для правильной установки и работы драйвера на узлах на основе GPU. |

## <a name="windows-server-based-node-pools"></a>Пулы узлов на базе Windows Server 

### <a name="required-fqdn--application-rules"></a>Требуемое полное доменное имя/правила приложения

Для работы кластеров AKS с пулами узлов на основе Windows Server требуются следующие FQDN и правила приложения.

| Полное доменное имя назначения                                                           | Порт      | Использование      |
|----------------------------------------------------------------------------|-----------|----------|
| **`onegetcdn.azureedge.net, go.microsoft.com`**                            | **`HTTPS:443`** | Для установки двоичных файлов, связанных с Windows. |
| **`*.mp.microsoft.com, www.msftconnecttest.com, ctldl.windowsupdate.com`** | **`HTTP:80`**   | Для установки двоичных файлов, связанных с Windows. |

## <a name="aks-addons-and-integrations"></a>Надстройки и интеграции AKS

### <a name="azure-monitor-for-containers"></a>Azure Monitor для контейнеров

Существует два варианта предоставления доступа к Azure Monitor для контейнеров. Вы можете разрешить Azure Monitor [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) **или** предоставить доступ к необходимым правилам полного доменного имени или приложения.

#### <a name="required-network-rules"></a>Требуемые сетевые правила

Приведенные ниже FQDN и правила приложения являются обязательными.

| Целевая конечная точка                                                             | Протокол | Порт    | Использование  |
|----------------------------------------------------------------------------------|----------|---------|------|
| [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureMonitor:443`**  | TCP           | 443      | Эта конечная точка используется для отправки данных и журналов метрик в Azure Monitor и Log Analytics. |

#### <a name="required-fqdn--application-rules"></a>Требуемое полное доменное имя/правила приложения

Для работы кластеров AKS с поддержкой Azure Monitor для контейнеров требуются следующие FQDN и правила приложения.

| Полное доменное имя.                                    | Порт      | Использование      |
|-----------------------------------------|-----------|----------|
| dc.services.visualstudio.com | **`HTTPS:443`**    | Эта конечная точка используется для метрик и мониторинга телеметрии с помощью Azure Monitor. |
| *.ods.opinsights.azure.com    | **`HTTPS:443`**    | Эта конечная точка используется Azure Monitor для приема данных log Analytics. |
| *.oms.opinsights.azure.com | **`HTTPS:443`** | Эта конечная точка используется omsagent, которая используется для проверки подлинности службы log Analytics. |
| *.monitoring.azure.com | **`HTTPS:443`** | Эта конечная точка используется для отправки данных метрик в Azure Monitor. |

### <a name="azure-dev-spaces"></a>Azure Dev Spaces

Обновите брандмауэр или конфигурацию безопасности, чтобы разрешить сетевой трафик для всех указанных ниже доменных имен и [служб Azure dev Spaces инфраструктуры][dev-spaces-service-tags].

#### <a name="required-network-rules"></a>Требуемые сетевые правила

| Целевая конечная точка                                                             | Протокол | Порт    | Использование  |
|----------------------------------------------------------------------------------|----------|---------|------|
| [сервицетаг](../virtual-network/service-tags-overview.md#available-service-tags) - **`AzureDevSpaces`**  | TCP           | 443      | Эта конечная точка используется для отправки данных и журналов метрик в Azure Monitor и Log Analytics. |

#### <a name="required-fqdn--application-rules"></a>Требуемое полное доменное имя/правила приложения 

Для работы кластеров AKS с поддержкой Azure Dev Spaces требуются следующие FQDN и правила приложения.

| Полное доменное имя.                                    | Порт      | Использование      |
|-----------------------------------------|-----------|----------|
| `cloudflare.docker.com` | **`HTTPS:443`** | Этот адрес используется для извлечения образов Linux Alpine и других образов Azure Dev Spaces. |
| `gcr.io` | **`HTTPS:443`** | Этот адрес используется для извлечения образов Helm и Tiller. |
| `storage.googleapis.com` | **`HTTPS:443`** | Этот адрес используется для извлечения образов Helm и Tiller. |


### <a name="azure-policy"></a>Политика Azure

#### <a name="required-fqdn--application-rules"></a>Требуемое полное доменное имя/правила приложения 

Для работы кластеров AKS с поддержкой службы "Политика Azure" требуются следующие FQDN и правила приложения.

| Полное доменное имя.                                          | Порт      | Использование      |
|-----------------------------------------------|-----------|----------|
| **`data.policy.core.windows.net`** | **`HTTPS:443`** | Этот адрес используется для извлечения политик Kubernetes и для сообщения о состоянии соответствия кластера службе политики. |
| **`store.policy.core.windows.net`** | **`HTTPS:443`** | Этот адрес используется для извлечения артефактов встроенных политик. |
| **`gov-prod-policy-data.trafficmanager.net`** | **`HTTPS:443`** | Этот адрес используется для правильной работы службы "Политика Azure"  |
| **`raw.githubusercontent.com`**               | **`HTTPS:443`** | Этот адрес используется для извлечения встроенных политик из GitHub, чтобы обеспечить правильную работу службы "Политика Azure" |
| **`dc.services.visualstudio.com`**            | **`HTTPS:443`** | Надстройка службы "Политика Azure", которая отправляет данные телеметрии в конечную точку Application Insights. |

## <a name="restrict-egress-traffic-using-azure-firewall"></a>Ограничить трафик исходящего трафика с помощью брандмауэра Azure

Брандмауэр Azure предоставляет `AzureKubernetesService` тег полного доменного имени службы Azure Kubernetes (), чтобы упростить эту настройку. 

> [!NOTE]
> Тег FQDN содержит все указанные выше полные доменные имена и автоматически обновляется в актуальном состоянии.
>
> Мы рекомендуем использовать как минимум 20 интерфейсных IP-адресов в брандмауэре Azure для рабочих сценариев, чтобы избежать возникновения проблем нехватки портов SNAT.

Ниже приведен пример архитектуры развертывания.

![Заблокированная топология](media/limit-egress-traffic/aks-azure-firewall-egress.png)

* Общедоступные входящие принудительно проходят через фильтры брандмауэра.
  * Узлы агента AKS изолированы в выделенной подсети.
  * [Брандмауэр Azure](../firewall/overview.md) развернут в собственной подсети.
  * Правило ДНаТ преобразует общедоступный IP-адрес FW в интерфейсный IP-адрес балансировки нагрузки.
* Исходящие запросы запускаются с узлов агентов на внутренний IP-адрес брандмауэра Azure с помощью [определяемого пользователем маршрута](egress-outboundtype.md) .
  * Запросы от узлов агента AKS следуют по UDR, который был помещен в подсеть, где развернут кластер AKS
  * Брандмауэр Azure направляет исходящий трафик из виртуальной сети с клиентского компонента общедоступного IP-адреса
  * Доступ к общедоступному Интернету или другим службам Azure осуществляется через IP-адрес клиентского компонента брандмауэра
  * Кроме того, доступ к плоскости управления AKS обеспечивается с помощью [IP-диапазонов разрешенного сервера API](./api-server-authorized-ip-ranges.md), который включает общедоступный ИНТЕРФЕЙСНЫЙ IP-адрес брандмауэра.
* Внутренний трафик
  * Кроме того, вместо [Общедоступной Load Balancer](load-balancer-standard.md) можно использовать [внутренний Load Balancer](internal-lb.md) для внутреннего трафика, который можно изолировать также в собственной подсети.


Ниже приведены инструкции по использованию `AzureKubernetesService` тега FQDN в брандмауэре Azure для ограничения исходящего трафика из кластера AKS и предоставлен пример настройки общедоступного входящего трафика через брандмауэр.


### <a name="set-configuration-via-environment-variables"></a>Настройка конфигурации с помощью переменных среды

Определите набор переменных среды, которые будут использоваться при создании ресурсов.

```bash
PREFIX="aks-egress"
RG="${PREFIX}-rg"
LOC="eastus"
PLUGIN=azure
AKSNAME="${PREFIX}"
VNET_NAME="${PREFIX}-vnet"
AKSSUBNET_NAME="aks-subnet"
# DO NOT CHANGE FWSUBNET_NAME - This is currently a requirement for Azure Firewall.
FWSUBNET_NAME="AzureFirewallSubnet"
FWNAME="${PREFIX}-fw"
FWPUBLICIP_NAME="${PREFIX}-fwpublicip"
FWIPCONFIG_NAME="${PREFIX}-fwconfig"
FWROUTE_TABLE_NAME="${PREFIX}-fwrt"
FWROUTE_NAME="${PREFIX}-fwrn"
FWROUTE_NAME_INTERNET="${PREFIX}-fwinternet"
```

### <a name="create-a-virtual-network-with-multiple-subnets"></a>Создание виртуальной сети с несколькими подсетями

Подготавливает виртуальную сеть с двумя отдельными подсетями, одной для кластера, одной для брандмауэра. При необходимости можно также создать его для внутренних входных данных службы.

![Пустая топология сети](media/limit-egress-traffic/empty-network.png)

Создайте группу ресурсов для хранения всех ресурсов.

```azurecli
# Create Resource Group

az group create --name $RG --location $LOC
```

Создайте виртуальную сеть с двумя подсетями для размещения кластера AKS и брандмауэра Azure. В каждой будет собственная подсеть. Начнем с сети AKS.

```
# Dedicated virtual network with AKS subnet

az network vnet create \
    --resource-group $RG \
    --name $VNET_NAME \
    --location $LOC \
    --address-prefixes 10.42.0.0/16 \
    --subnet-name $AKSSUBNET_NAME \
    --subnet-prefix 10.42.1.0/24

# Dedicated subnet for Azure Firewall (Firewall name cannot be changed)

az network vnet subnet create \
    --resource-group $RG \
    --vnet-name $VNET_NAME \
    --name $FWSUBNET_NAME \
    --address-prefix 10.42.2.0/24
```

### <a name="create-and-set-up-an-azure-firewall-with-a-udr"></a>Создание и настройка брандмауэра Azure с помощью UDR

Необходимо настроить правила входящего и исходящего трафика брандмауэра Azure. Основная цель брандмауэра заключается в том, чтобы позволить организациям настраивать детализированные входящие и исходящие правила трафика в кластере AKS.

![Брандмауэр и UDR](media/limit-egress-traffic/firewall-udr.png)


> [!IMPORTANT]
> Если кластер или приложение создает большое количество исходящих подключений, направленных в одно или небольшое подмножество назначений, может потребоваться больше интерфейсных IP-адресов брандмауэра, чтобы избежать кардинально отличается портов на каждый интерфейсный IP.
> Дополнительные сведения о создании брандмауэра Azure с несколькими IP- [ **адресами** см. здесь.](../firewall/quick-create-multiple-ip-template.md)

Создайте ресурс общедоступного IP-адреса стандартного SKU, который будет использоваться в качестве интерфейса брандмауэра Azure.

```azurecli
az network public-ip create -g $RG -n $FWPUBLICIP_NAME -l $LOC --sku "Standard"
```

Зарегистрируйте предварительную версию расширения интерфейса командной строки, чтобы создать брандмауэр Azure.
```azurecli
# Install Azure Firewall preview CLI extension

az extension add --name azure-firewall

# Deploy Azure Firewall

az network firewall create -g $RG -n $FWNAME -l $LOC --enable-dns-proxy true
```
Созданный ранее IP-адрес теперь можно назначить клиентскому компоненту брандмауэра.

> [!NOTE]
> Настройка общедоступного IP-адреса для брандмауэра Azure может занять несколько минут.
> Чтобы использовать полное доменное имя в правилах сети, необходимо включить DNS-прокси, если брандмауэр прослушивает порт 53 и перенаправит запросы DNS на DNS-сервер, указанный выше. Это позволит брандмауэру автоматически перевести полное доменное имя.

```azurecli
# Configure Firewall IP Config

az network firewall ip-config create -g $RG -f $FWNAME -n $FWIPCONFIG_NAME --public-ip-address $FWPUBLICIP_NAME --vnet-name $VNET_NAME
```

После выполнения предыдущей команды сохраните IP-адрес клиентского компонента брандмауэра для настройки позже.

```bash
# Capture Firewall IP Address for Later Use

FWPUBLIC_IP=$(az network public-ip show -g $RG -n $FWPUBLICIP_NAME --query "ipAddress" -o tsv)
FWPRIVATE_IP=$(az network firewall show -g $RG -n $FWNAME --query "ipConfigurations[0].privateIpAddress" -o tsv)
```

> [!NOTE]
> Если вы используете безопасный доступ к серверу API AKS с [разрешениями диапазонов IP-адресов](./api-server-authorized-ip-ranges.md), необходимо добавить общедоступный IP-адрес брандмауэра в диапазон адресов, на которые он подлиннее.

### <a name="create-a-udr-with-a-hop-to-azure-firewall"></a>Создание UDR с прыжком к брандмауэру Azure

Azure автоматически направляет трафик между подсетями Azure, виртуальными и локальными сетями. Если нужно изменить какой-либо из стандартных маршрутов Azure, это можно сделать с помощью создания таблицы маршрутов.

Создайте пустую таблицу маршрутов, которая будет связана с заданной подсетью. В таблице маршрутов будет определен следующий прыжок к созданному выше брандмауэру Azure. С каждой подсетью может быть связана одна таблица маршрутов (или ноль).

```azurecli
# Create UDR and add a route for Azure Firewall

az network route-table create -g $RG -l $LOC --name $FWROUTE_TABLE_NAME
az network route-table route create -g $RG --name $FWROUTE_NAME --route-table-name $FWROUTE_TABLE_NAME --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address $FWPRIVATE_IP --subscription $SUBID
az network route-table route create -g $RG --name $FWROUTE_NAME_INTERNET --route-table-name $FWROUTE_TABLE_NAME --address-prefix $FWPUBLIC_IP/32 --next-hop-type Internet
```

Сведения о том, как переопределить системные маршруты Azure по умолчанию или добавить дополнительные маршруты в таблицу маршрутов подсети, см. в разделе [Документация по таблицам маршрутизации виртуальной сети](../virtual-network/virtual-networks-udr-overview.md#user-defined).

### <a name="adding-firewall-rules"></a>Добавление правил брандмауэра

Ниже приведены три сетевых правила, которые можно использовать для настройки в брандмауэре, может потребоваться адаптировать эти правила на основе развертывания. Первое правило разрешает доступ к порту 9000 через TCP. Второе правило разрешает доступ к портам 1194 и 123 по протоколу UDP (если вы развертываете в Azure для Китая в Китае, вам может потребоваться [больше](#azure-china-21vianet-required-network-rules)). Оба эти правила разрешают только трафик, предназначенный для CIDR региона Azure, который мы используем, в данном случае — Восточная часть США. Наконец, мы добавим третье сетевое правило, открывающее порт 123, в `ntp.ubuntu.com` полное доменное имя через UDP (Добавление полного доменного имени в качестве сетевого правила является одной из функций брандмауэра Azure, и вам потребуется адаптировать его при использовании собственных параметров).

После настройки правил сети мы также добавим правило приложения с помощью `AzureKubernetesService` , которое охватывает все необходимые полные доменные имена, доступные через TCP-порт 443 и порт 80.

```
# Add FW Network Rules

az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'apiudp' --protocols 'UDP' --source-addresses '*' --destination-addresses "AzureCloud.$LOC" --destination-ports 1194 --action allow --priority 100
az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'apitcp' --protocols 'TCP' --source-addresses '*' --destination-addresses "AzureCloud.$LOC" --destination-ports 9000
az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'time' --protocols 'UDP' --source-addresses '*' --destination-fqdns 'ntp.ubuntu.com' --destination-ports 123

# Add FW Application Rules

az network firewall application-rule create -g $RG -f $FWNAME --collection-name 'aksfwar' -n 'fqdn' --source-addresses '*' --protocols 'http=80' 'https=443' --fqdn-tags "AzureKubernetesService" --action allow --priority 100
```

Дополнительные сведения о службе брандмауэра Azure см. в разделе [Документация к брандмауэру Azure](../firewall/overview.md).

### <a name="associate-the-route-table-to-aks"></a>Привязка таблицы маршрутизации к AKS

Чтобы связать кластер с брандмауэром, выделенная выше подсеть для подсети кластера должна ссылаться на созданную выше таблицу маршрутизации. Привязку можно выполнить, создав команду для виртуальной сети, в которой находятся кластер и брандмауэр, чтобы обновить таблицу маршрутизации подсети кластера.

```azurecli
# Associate route table with next hop to Firewall to the AKS subnet

az network vnet subnet update -g $RG --vnet-name $VNET_NAME --name $AKSSUBNET_NAME --route-table $FWROUTE_TABLE_NAME
```

### <a name="deploy-aks-with-outbound-type-of-udr-to-the-existing-network"></a>Развертывание AKS с типом исходящего трафика UDR в существующую сеть

Теперь кластер AKS можно развернуть в существующей виртуальной сети. Мы также будем использовать [тип `userDefinedRouting` исходящего трафика ](egress-outboundtype.md). Эта функция гарантирует, что весь исходящий трафик будет принудительно проходить через брандмауэр, а другие пути исходящего трафика не будут существовать (по умолчанию может использоваться Load Balancer тип исходящих подключений).

![aks-deploy](media/limit-egress-traffic/aks-udr-fw.png)

### <a name="create-a-service-principal-with-access-to-provision-inside-the-existing-virtual-network"></a>Создание субъекта-службы с доступом для подготовки ресурсов в существующей виртуальной сети

Идентификатор кластера (управляемое удостоверение или субъект-служба) используется AKS для создания ресурсов кластера. Субъект-служба, который передается во время создания, используется для создания базовых ресурсов AKS, таких как ресурсы хранилища, IP-адреса и подсистемы балансировки нагрузки, используемые AKS (вы также можете использовать [управляемое удостоверение](use-managed-identity.md) ). Если вы не предоставили соответствующие разрешения ниже, вы не сможете подготавливать кластер AKS.

```azurecli
# Create SP and Assign Permission to Virtual Network

az ad sp create-for-rbac -n "${PREFIX}sp" --skip-assignment
```

Теперь замените `APPID` и `PASSWORD` ниже идентификатором appid субъекта-службы и паролем субъекта-службы, созданными автоматически в результате выполнения предыдущей команды. Мы будем ссылаться на идентификатор ресурса виртуальной сети, чтобы предоставить разрешения субъекту-службе, чтобы AKS мог развернуть в ней ресурсы.

```azurecli
APPID="<SERVICE_PRINCIPAL_APPID_GOES_HERE>"
PASSWORD="<SERVICEPRINCIPAL_PASSWORD_GOES_HERE>"
VNETID=$(az network vnet show -g $RG --name $VNET_NAME --query id -o tsv)

# Assign SP Permission to VNET

az role assignment create --assignee $APPID --scope $VNETID --role "Network Contributor"
```

Вы можете проверить подробные разрешения, которые необходимы [здесь](kubernetes-service-principal.md#delegate-access-to-other-azure-resources).

> [!NOTE]
> Если вы используете подключаемый модуль кубенет Network, вам нужно предоставить предварительно созданной таблице маршрутов субъекту-службе AKS или разрешения на управляемое удостоверение, так как кубенет требует таблицу маршрутов для добавления необходимых правил маршрутизации. 
> ```azurecli-interactive
> RTID=$(az network route-table show -g $RG -n $FWROUTE_TABLE_NAME --query id -o tsv)
> az role assignment create --assignee $APPID --scope $RTID --role "Network Contributor"
> ```

### <a name="deploy-aks"></a>Развертывание AKS

Наконец, кластер AKS можно развернуть в существующей подсети, выделенной для кластера. Целевая подсеть для развертывания определяется переменной среды `$SUBNETID`. Мы не определили переменную `$SUBNETID` на предыдущих шагах. Чтобы задать значение для идентификатора подсети, можно использовать следующую команду.

```azurecli
SUBNETID=$(az network vnet subnet show -g $RG --vnet-name $VNET_NAME --name $AKSSUBNET_NAME --query id -o tsv)
```

Вы определите тип исходящего трафика для использования UDR, который уже существует в подсети. Эта конфигурация позволит AKS пропустить установку и подготовку IP-адресов для балансировщика нагрузки.

> [!IMPORTANT]
> Дополнительные сведения о типе исходящего трафика UDR, включая ограничения, см. в разделе [**исходящий трафик с типом UDR**](egress-outboundtype.md#limitations).


> [!TIP]
> В развертывание кластера, например [**частный кластер**](private-clusters.md), можно добавить дополнительные функции. 
>
> Для ограничения доступа к серверу API только к общедоступной конечной точке брандмауэра можно добавить функцию AKS для [**IP-адресов разрешенного сервера API**](api-server-authorized-ip-ranges.md) . Функция "диапазоны разрешенных IP-адресов" отмечается на схеме как необязательная. Если вы включаете функцию разрешенных IP-адресов для ограничения доступа на сервер API, в инструментах разработчика следует использовать инсталляционный сервер из виртуальной сети брандмауэра или добавить все конечные точки разработчика в диапазон разрешенных IP-адресов.

```azurecli
az aks create -g $RG -n $AKSNAME -l $LOC \
  --node-count 3 --generate-ssh-keys \
  --network-plugin $PLUGIN \
  --outbound-type userDefinedRouting \
  --service-cidr 10.41.0.0/16 \
  --dns-service-ip 10.41.0.10 \
  --docker-bridge-address 172.17.0.1/16 \
  --vnet-subnet-id $SUBNETID \
  --service-principal $APPID \
  --client-secret $PASSWORD \
  --api-server-authorized-ip-ranges $FWPUBLIC_IP
```

### <a name="enable-developer-access-to-the-api-server"></a>Разрешение доступа разработчика к серверу API

Если вы использовали разрешенные IP-диапазоны для кластера на предыдущем шаге, необходимо добавить IP-адреса средств разработчика в список разрешенных IP-адресов в кластере AKS, чтобы получить доступ к серверу API. Кроме того, можно настроить инсталляционный сервер с необходимым инструментарием внутри отдельной подсети в виртуальной сети брандмауэра.

Добавьте еще один IP-адрес в утвержденный диапазон, выполнив следующую команду.

```bash
# Retrieve your IP address
CURRENT_IP=$(dig @resolver1.opendns.com ANY myip.opendns.com +short)

# Add to AKS approved list
az aks update -g $RG -n $AKSNAME --api-server-authorized-ip-ranges $CURRENT_IP/32

```

 Используйте команду [AZ AKS Get-credentials] [AZ-AKS-Get-Credential], чтобы настроить `kubectl` Подключение к созданному кластеру Kubernetes. 

 ```azurecli
 az aks get-credentials -g $RG -n $AKSNAME
 ```

### <a name="deploy-a-public-service"></a>Развертывание общедоступной службы
Теперь можно приступить к раскрытию служб и развертыванию приложений в этом кластере. В этом примере мы предоставляем общедоступную службу, но вы также можете предоставить доступ к внутренней службе через [внутреннюю подсистему балансировки нагрузки](internal-lb.md).

![Общедоступная служба ДНаТ](media/limit-egress-traffic/aks-create-svc.png)

Разверните приложение Azure для голосования, cкопировав YAML ниже в файл с именем `example.yaml`.

```yaml
# voting-storage-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-storage
  template:
    metadata:
      labels:
        app: voting-storage
    spec:
      containers:
      - name: voting-storage
        image: mcr.microsoft.com/aks/samples/voting/storage:2.0
        args: ["--ignore-db-dir=lost+found"]
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
---
# voting-storage-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: voting-storage-secret
type: Opaque
data:
  MYSQL_USER: ZGJ1c2Vy
  MYSQL_PASSWORD: UGFzc3dvcmQxMg==
  MYSQL_DATABASE: YXp1cmV2b3Rl
  MYSQL_ROOT_PASSWORD: UGFzc3dvcmQxMg==
---
# voting-storage-pv-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
# voting-storage-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-storage
  labels: 
    app: voting-storage
spec:
  ports:
  - port: 3306
    name: mysql
  selector:
    app: voting-storage
---
# voting-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-app
  template:
    metadata:
      labels:
        app: voting-app
    spec:
      containers:
      - name: voting-app
        image: mcr.microsoft.com/aks/samples/voting/app:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: MYSQL_HOST
          value: "voting-storage"
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
        - name: ANALYTICS_HOST
          value: "voting-analytics"
---
# voting-app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-app
  labels: 
    app: voting-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    name: http
  selector:
    app: voting-app
---
# voting-analytics-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-analytics
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-analytics
      version: "2.0"
  template:
    metadata:
      labels:
        app: voting-analytics
        version: "2.0"
    spec:
      containers:
      - name: voting-analytics
        image: mcr.microsoft.com/aks/samples/voting/analytics:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: MYSQL_HOST
          value: "voting-storage"
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
---
# voting-analytics-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-analytics
  labels: 
    app: voting-analytics
spec:
  ports:
  - port: 8080
    name: http
  selector:
    app: voting-analytics
```

Разверните службу, выполнив следующую команду.

```bash
kubectl apply -f example.yaml
```

### <a name="add-a-dnat-rule-to-azure-firewall"></a>Добавление правила DNAT в брандмауэр Azure

> [!IMPORTANT]
> Если для ограничения исходящего трафика вы примените Брандмауэр Azure и создадите определяемый пользователем маршрут для передачи всего исходящего трафика, обязательно создайте в брандмауэре соответствующее правило DNAT, чтобы правильно пропускать входящий трафик. Использование Брандмауэра Azure в сочетании с определяемыми пользователем маршрутами нарушает передачу входящего трафика из-за асимметричной маршрутизации. (Проблема возникает, если подсеть AKS имеет маршрут по умолчанию для передачи данных на частный IP-адрес брандмауэра, а параллельно с ним используется общедоступная подсистема балансировки нагрузки для входящего трафика или служба Kubernetes с типом LoadBalancer.) В этом случае входящий трафик подсистемы балансировки нагрузки поступает через общедоступный IP-адрес, а обратный путь проходит через частный IP-адрес брандмауэра. Так как брандмауэр отслеживает состояние и не имеет информации об активных сеансах, он отбрасывает возвращаемые пакеты. Сведения о том, как интегрировать Брандмауэр Azure с подсистемой балансировки нагрузки для входящего трафика или служб, см. в статье [Интеграция Брандмауэра Azure с Azure Load Balancer (цен. категория "Стандартный")](../firewall/integrate-lb.md).


Чтобы настроить возможность входящего подключения, необходимо записать в брандмауэр Azure правило DNAT. Для проверки подключения к кластеру определяется правило для интерфейса общедоступного IP-адреса брандмауэра, которое перенаправляется во внутренний IP-адрес, предоставляемый внутренней службой.

Адрес назначения можно настроить так же, как и порт в брандмауэре. Преобразованный адрес должен представлять собой IP-адрес внутреннего балансировщика нагрузки. Преобразованный порт должен представлять собой предоставленный порт для вашей службы Kubernetes.

Необходимо указать внутренний IP-адрес, назначенный подсистеме балансировки нагрузки, созданной службой Kubernetes. Получите адрес, выполнив команду:

```bash
kubectl get services
```

Необходимый IP-адрес будет указан в столбце «ВНЕШНИЕ IP-АДРЕСА» следующим образом.

```bash
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP      10.41.0.1       <none>        443/TCP        10h
voting-analytics   ClusterIP      10.41.88.129    <none>        8080/TCP       9m
voting-app         LoadBalancer   10.41.185.82    20.39.18.6    80:32718/TCP   9m
voting-storage     ClusterIP      10.41.221.201   <none>        3306/TCP       9m
```

Получите IP-адрес службы, выполнив:
```bash
SERVICE_IP=$(kubectl get svc voting-app -o jsonpath='{.status.loadBalancer.ingress[*].ip}')
```

Добавьте правило NAT, выполнив команду.
```azurecli
az network firewall nat-rule create --collection-name exampleset --destination-addresses $FWPUBLIC_IP --destination-ports 80 --firewall-name $FWNAME --name inboundrule --protocols Any --resource-group $RG --source-addresses '*' --translated-port 80 --action Dnat --priority 100 --translated-address $SERVICE_IP
```

### <a name="validate-connectivity"></a>Проверка подключения

Перейдите к IP-адресу клиентского компонента брандмауэра Azure в браузере, чтобы проверить подключение.

Вы должны увидеть приложение для голосования AKS. В этом примере общедоступный IP-адрес брандмауэра был `52.253.228.132` .


![На снимке экрана показано приложение для голосования с кнопками для кошки, собаки и сбросов, а также итоги.](media/limit-egress-traffic/aks-vote.png)


### <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы Azure, удалите группу ресурсов AKS.

```azurecli
az group delete -g $RG
```

## <a name="next-steps"></a>Следующие шаги

В этой статье вы узнали, какие порты и адреса разрешить, если требуется ограничить исходящий трафик для кластера. Вы также узнали, как защитить исходящий трафик с помощью брандмауэра Azure. 

При необходимости вы можете обобщить приведенные выше шаги, чтобы перенаправить трафик в предпочтительное решение для [исходящего трафика, следуя `userDefinedRoute` документации по типу исходящих](egress-outboundtype.md)сообщений.

Если необходимо ограничить способ взаимодействия между собой и East-West ограничения трафика в кластере, см. раздел [Защита трафика между модулями Pod с помощью сетевых политик в AKS][network-policy].

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policy]: use-network-policies.md
[azure-firewall]: ../firewall/overview.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[aks-upgrade]: upgrade-cluster.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[dev-spaces-service-tags]: ../dev-spaces/configure-networking.md#virtual-network-or-subnet-configurations
