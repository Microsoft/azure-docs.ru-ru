---
title: Общие сведения о службе Azure Kubernetes
description: Узнайте о возможностях и преимуществах Службы Azure Kubernetes при развертывании и администрировании контейнерных приложений в Azure.
services: container-service
ms.topic: overview
ms.date: 02/24/2021
ms.custom: mvc
ms.openlocfilehash: 1cddd39d0b95e021478235fcdafbacd40eb4097c
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105261"
---
# <a name="azure-kubernetes-service"></a>Служба Azure Kubernetes

Служба Azure Kubernetes (AKS) упрощает развертывание управляемого кластера Kubernetes в Azure, выгружая операционные издержки в Azure. Размещенная в Azure служба Kubernetes отвечает за критические задачи, в частности за мониторинг работоспособности и техническое обслуживание. Так как главными узлами Kubernetes управляет платформа Azure, вы администрируете и обслуживаете только узлы агентов. Поэтому AKS предоставляется бесплатно, оплачиваются только узлы агента в кластерах, а не мастера.  

Кластер AKS можно создать с помощью следующих средств:
* [CLI Azure.](kubernetes-walkthrough.md)
* [Портал Azure](kubernetes-walkthrough-portal.md)
* [Azure PowerShell](kubernetes-walkthrough-powershell.md)
* С помощью вариантов развертывания на основе шаблонов, таких как [шаблоны Azure Resource Manager](kubernetes-walkthrough-rm-template.md) и Terraform. 

При развертывании кластера AKS мастер Kubernetes и все узлы развертываются и настраиваются под ваши потребности. В процессе развертывания можно настроить дополнительные сетевые функции, интеграцию Azure Active Directory (Azure AD), мониторинг и другие возможности. 

Дополнительные сведения о Kubernetes см. в статье о [ключевых концепциях Kubernetes для службы Azure Kubernetes (AKS)][concepts-clusters-workloads].

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]
> AKS также поддерживает контейнеры Windows Server.

## <a name="access-security-and-monitoring"></a>Доступ, безопасность и мониторинг

С целью повышения безопасности и оптимизации управления в AKS предусмотрена интеграция с Azure AD для использования следующих возможностей:
* использование управления доступом Kubernetes на основе ролей (Kubernetes RBAC); 
* мониторинг работоспособности кластера и ресурсов.

### <a name="identity-and-security-management"></a>Управление идентификаторами и безопасностью

#### <a name="kubernetes-rbac"></a>Kubernetes RBAC

Чтобы ограничить доступ к ресурсам кластера, служба AKS поддерживает [Kubernetes RBAC][kubernetes-rbac]. Kubernetes RBAC управляет доступом к ресурсам и пространствам имен Kubernetes, а также разрешениями для доступа к ним.  

#### <a name="azure-ad"></a>Azure AD

Вы можете настроить кластер AKS для интеграции с Azure AD. Благодаря интеграции с Azure AD вы можете настроить доступ к Kubernetes на основе существующих удостоверений и членства в группах. Существующим пользователям и группам Azure AD можно предоставить доступ к ресурсам AKS, а также интегрированный единый вход.  

Дополнительные сведения об идентификаторе см. в статье о [возможностях контроля доступа и идентификации в AKS][concepts-identity].

Сведения о том, как обеспечить безопасность кластера AKS, см. в статье об [интеграции Azure Active Directory со службой Azure Kubernetes][aks-aad].

### <a name="integrated-logging-and-monitoring"></a>Встроенное ведение журналов и мониторинг

Azure Monitor для работоспособности контейнеров собирает метрики производительности памяти и процессоров из контейнеров, узлов и контроллеров в кластере AKS и развернутых приложениях. Вы можете просматривать как журналы контейнеров, так и [журналы главных узлов Kubernetes][aks-master-logs], которые:
* хранятся в рабочей области Azure Log Analytics;
* доступны через портал Azure, Azure CLI или конечную точку REST.

Дополнительные сведения см. в статье о [мониторинге работоспособности службы Azure Kubernetes (AKS)][container-health].

## <a name="clusters-and-nodes"></a>Кластеры и узлы

Узлы AKS выполняются на виртуальных машинах Azure. Благодаря узлам AKS можно подключить хранилище к узлам и объектам pod, обновить компоненты кластера и использовать GPU. AKS поддерживает кластеры Kubernetes, которые используют несколько пулов узлов для поддержки смешанных операционных систем и контейнеров Windows Server.  

Дополнительные сведения о возможностях кластеров, узлов и пулов узлов Kubernetes см. в статье [Ключевые понятия Kubernetes для AKS][concepts-clusters-workloads].

### <a name="cluster-node-and-pod-scaling"></a>Масштабирование узла и pod кластера

При изменении нагрузки на ресурсы также автоматически увеличивается или уменьшается число узлов кластера или объектов pod, на которых выполняются ваши службы. Вы можете настроить средство горизонтального автомасштабирования объектов pod или средство автомасштабирования кластера, чтобы адаптироваться к требованиям и задействовать только нужный объем ресурсов.

Дополнительные сведения см. в статье о [масштабировании кластера службы Azure Kubernetes][aks-scale].

### <a name="cluster-node-upgrades"></a>Обновления узла кластера

В AKS доступно несколько версий Kubernetes. При выпуске новых версий в AKS вы можете обновлять кластер с помощью портала Azure или Azure CLI. Чтобы свести к минимуму время простоя работающих приложений, в процессе обновления узлы тщательно блокируются и останавливаются.  

Дополнительные сведения о версиях жизненного цикла см. в статье о [поддерживаемых версиях Kubernetes в AKS][aks-supported versions]. Сведения об обновлении см. в статье [Обновление кластера службы Azure Kubernetes (AKS)][aks-upgrade].

### <a name="gpu-enabled-nodes"></a>Узлы с GPU

Служба AKS поддерживает создание пулов узлов с GPU. Сейчас Azure предоставляет одну или несколько виртуальных машин с поддержкой GPU. Виртуальные машины с поддержкой GPU предназначены для рабочих нагрузок с большим объемом вычислений, графической обработки и визуализаций.

Дополнительные сведения см. в статье об [использовании GPU в AKS][aks-gpu].

### <a name="confidential-computing-nodes-public-preview"></a>Узлы конфиденциальных вычислений (общедоступная предварительная версия)

AKS поддерживает создание пулов узлов конфиденциальных вычислений на основе Intel SGX (виртуальные машины DCSv2). Узлы конфиденциальных вычислений позволяют контейнерам работать в доверенной среде выполнения (в анклавах) на основе оборудования. Изоляция между контейнерами в сочетании с обеспечением целостности кода путем аттестации может помочь при разработке стратегии углубленной защиты контейнеров. Узлы конфиденциальных вычислений могут работать как с конфиденциальными контейнерами (существующие приложения Docker), так и с контейнерами, поддерживающими анклавы.

Дополнительные сведения см. в статье об [узлах конфиденциальных вычислений в AKS][conf-com-node].

### <a name="storage-volume-support"></a>Поддержка тома хранилища

Для поддержки рабочих нагрузок приложений можно подключить статические или динамические тома хранилища для постоянных данных. В зависимости от предполагаемого числа подключенных объектов pod, которые будут совместно использовать тома хранения, вы можете использовать хранилище, обслуживаемое:
* Дисками Azure для доступа к одному объекту pod; 
* Файлами Azure для параллельного доступа к нескольким объектам pod.

Дополнительные сведения см. в статье о [возможности хранения данных приложений в AKS][concepts-storage].

Для начала работы с динамическими постоянными томами см. статьи о [дисках Azure в AKS][azure-disk] и о [файлах Azure в AKS][azure-files].

## <a name="virtual-networks-and-ingress"></a>Виртуальные сети и входные данные

Кластер AKS можно развернуть в существующей виртуальной сети. В такой конфигурации каждому объекту pod в кластере назначается IP-адрес в виртуальной сети. Они могут напрямую взаимодействовать с:
* другими объектами pod в кластере; 
* другими узлами в виртуальной сети. 

Кроме того, объекты pod могут подключаться к другим службам пиринговой виртуальной сети, а также к локальным сетям через подключения ExpressRoute и VPN типа "сеть — сеть" (S2S).  

Дополнительные сведения см. в статье об [основных сетевых понятиях для приложений в AKS][aks-networking].

### <a name="ingress-with-http-application-routing"></a>Входящий трафик маршрутизации приложений HTTP

Надстройка маршрутизации приложений HTTP упрощает доступ к приложениям, развернутым в кластере AKS. При установке этого параметра решение маршрутизации приложений HTTP настраивает контроллер входящего трафика в кластере AKS.  

После развертывания приложений общедоступные DNS-имена будут настроены автоматически. Маршрутизация HTTP-трафика приложений настраивает зону DNS и интегрирует ее с кластером AKS. Затем можно развернуть ресурсы входящего трафика Kubernetes в обычном режиме.  

Чтобы начать работу с входящим трафиком, см. статью о [маршрутизации приложений HTTP][aks-http-routing].

## <a name="development-tooling-integration"></a>Интеграция средств разработки

Платформа Kubernetes включает обширную экосистему средств разработки и управления, совместимых с AKS. Это такие средства, как Helm и расширение Kubernetes для Visual Studio Code.   

Azure предоставляет ряд средств, помогающих оптимизировать работу с Kubernetes, например Azure Dev Spaces и DevOps Starter.  

### <a name="azure-dev-spaces"></a>Azure Dev Spaces

Рабочие среды Azure Dev Spaces предоставляют быстрый итеративный интерфейс разработки Kubernetes для команд. Для того чтобы запускать и отлаживать контейнеры непосредственно в службе Azure Kubernetes (AKS), требуется минимальная конфигурация. Чтобы начать работу, см. статью об [Azure Dev Spaces][azure-dev-spaces].

### <a name="devops-starter"></a>DevOps Starter

Начальный интерфейс DevOps обеспечивает простое решение, которое позволяет использовать существующий код и репозитории Git в Azure. DevOps Starter автоматически выполняет следующие задачи:
* создает ресурсы Azure (например, AKS); 
* настраивает в Azure DevOps Services конвейер выпуска, который включает конвейер сборки для непрерывной интеграции; 
* настраивает конвейер выпуска для непрерывного развертывания; 
* создает ресурс Azure Application Insights для мониторинга. 

Дополнительные сведения см. в статье о [начальном интерфейсе DevOps][azure-devops].

## <a name="docker-image-support-and-private-container-registry"></a>Поддержка образов Docker и частный реестр контейнеров

Служба Azure Kubernetes поддерживает формат образов Docker. Для частного хранилища образов Docker можно интегрировать AKS с Реестром контейнеров Azure (ACR).

Сведения о том, как создать хранилище частных образов, см. в статье о [Реестре контейнеров Azure][acr-docs].

## <a name="kubernetes-certification"></a>Сертификация Kubernetes

Служба AKS сертифицирована CNCF как соответствующая требованиям для Kubernetes.

## <a name="regulatory-compliance"></a>Соблюдение нормативных требований

AKS соответствует нормативным требованиям SOC, ISO, PCI DSS и HIPAA. Дополнительные сведения см. в документе о [предложениях Azure для соответствия требованиям][compliance-doc].

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о развертывании и администрировании AKS с помощью Azure CLI из следующего краткого руководства.

> [!div class="nextstepaction"]
> [Развертывание кластера AKS с помощью Azure CLI][aks-cli]

<!-- LINKS - external -->
[aks-engine]: https://github.com/Azure/aks-engine
[kubectl-overview]: https://kubernetes.io/docs/user-guide/kubectl-overview/
[compliance-doc]: https://azure.microsoft.com/en-us/overview/trusted-cloud/compliance/

<!-- LINKS - internal -->
[acr-docs]: ../container-registry/container-registry-intro.md
[aks-aad]: ./azure-ad-integration-cli.md
[aks-cli]: ./kubernetes-walkthrough.md
[aks-gpu]: ./gpu-cluster.md
[aks-http-routing]: ./http-application-routing.md
[aks-networking]: ./concepts-network.md
[aks-portal]: ./kubernetes-walkthrough-portal.md
[aks-scale]: ./tutorial-kubernetes-scale.md
[aks-upgrade]: ./upgrade-cluster.md
[azure-dev-spaces]: ../dev-spaces/index.yml
[azure-devops]: ../devops-project/overview.md
[azure-disk]: ./azure-disks-dynamic-pv.md
[azure-files]: ./azure-files-dynamic-pv.md
[container-health]: ../azure-monitor/containers/container-insights-overview.md
[aks-master-logs]: ./view-control-plane-logs.md
[aks-supported versions]: supported-kubernetes-versions.md
[concepts-clusters-workloads]: concepts-clusters-workloads.md
[kubernetes-rbac]: concepts-identity.md#kubernetes-rbac
[concepts-identity]: concepts-identity.md
[concepts-storage]: concepts-storage.md
[conf-com-node]: ../confidential-computing/confidential-nodes-aks-overview.md
