---
title: Часто задаваемые вопросы о Azure Dev Spaces
services: azure-dev-spaces
ms.date: 01/28/2020
ms.topic: conceptual
description: Найдите ответы на некоторые распространенные вопросы о Azure Dev Spaces
keywords: 'Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, containers, Helm, service mesh, service mesh routing, kubectl, k8s '
ms.openlocfilehash: e83bed86714e4b92c63f4e7b7eb55df7a2a7eaff
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96548840"
---
# <a name="frequently-asked-questions-about-azure-dev-spaces"></a>Часто задаваемые вопросы о Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Здесь рассматриваются часто задаваемые вопросы о Azure Dev Spaces.

## <a name="what-versions-of-kubernetes-are-supported-for-azure-dev-spaces"></a>Какие версии Kubernetes поддерживаются для Azure Dev Spaces?

Azure Dev Spaces поддерживает [сейчас поддержку общедоступных версий Kubernetes в AKS до 1,18][aks-supported-k8s]. Kubernetes 1,19 и выше в AKS использует контейнер в качестве среды выполнения контейнера, который не работает с Azure Dev Spaces.

## <a name="which-azure-regions-currently-provide-azure-dev-spaces"></a>Какие регионы Azure в настоящее время предоставляют Azure Dev Spaces?

Полный список доступных регионов см. в разделе [Поддерживаемые регионы][supported-regions] .

## <a name="can-i-migrate-my-aks-cluster-with-azure-dev-spaces-to-another-region"></a>Можно ли перенести кластер AKS с Azure Dev Spaces в другой регион?

Да, если вы хотите переместить кластер AKS с Azure Dev Spaces в другой [поддерживаемый регион][supported-regions], рекомендуется создать новый кластер в другом регионе, а затем установить и настроить Azure dev Spaces и развернуть ресурсы и приложения в новом кластере. Дополнительные сведения о миграции AKS см. [в статье миграция в службу Kubernetes Azure (AKS)][aks-migration].

## <a name="can-i-use-azure-dev-spaces-with-existing-dockerfiles-or-helm-charts"></a>Можно ли использовать Azure Dev Spaces с существующими диаграммами файлы dockerfile или Helm?

Да, если проект уже содержит диаграмму Dockerfile или Helm, вы можете использовать эти файлы с Azure Dev Spaces. При запуске `azds prep` Используйте `--chart` параметр и укажите расположение диаграммы. Azure Dev Spaces будет по-прежнему создавать файл *аздс. YAML* и *Dockerfile. Разработка* , но он не заменит или не изменит существующую диаграмму Dockerfile или Helm. Может потребоваться изменить *аздс. YAML* и *Dockerfile. Разработка* файлов, чтобы все работало правильно с существующим приложением при запуске `azds up` .

При использовании собственной диаграммы Dockerfile или Helm существуют следующие ограничения.
* Если используется только один Dockerfile, он должен включать все необходимое для реализации сценариев разработки, таких как языковой пакет SDK не только для среды выполнения. При использовании отдельного Dockerfile для Azure Dev Spaces, например Dockerfile. Development, все, что необходимо для включения сценариев разработки, должно быть включено в этот Dockerfile.
* Helm диаграмма должна поддерживать передачу части или всего тега Image в виде значения из Values *. YAML*.
* Если вы изменяете что-то с помощью входящего, можно также обновить диаграмму Helm, чтобы использовать входящее решение, предоставляемое Azure Dev Spaces.
* Если вы хотите использовать [возможности маршрутизации, предоставляемые Azure dev Spaces][dev-spaces-routing], все службы для отдельного проекта должны быть размещены в одном пространстве имен Kubernetes и должны быть развернуты с простым именем, например *Service-a*. В стандартных диаграммах Helm это обновление именования можно выполнить, указав значение для свойства *фуллнамеоверриде* .

## <a name="can-i-modify-the-files-generated-by-azure-dev-spaces"></a>Можно ли изменить файлы, созданные Azure Dev Spaces?

Да, можно изменить файл *аздс. YAML* , Dockerfile и диаграмму Helm, [созданную Azure dev Spaces при подготовке проекта][dev-spaces-prep]. Изменение этих файлов изменяет процесс сборки и запуска проекта.

## <a name="can-i-use-azure-dev-spaces-without-a-public-ip-address"></a>Можно ли использовать Azure Dev Spaces без общедоступного IP-адреса?

Нет, вы не можете подготавливать Azure Dev Spaces в кластере AKS без общедоступного IP-адреса. [Azure dev Spaces для маршрутизации требуется][dev-spaces-routing]общедоступный IP-адрес.

## <a name="can-i-use-my-own-ingress-with-azure-dev-spaces"></a>Можно ли использовать собственный входной параметр с Azure Dev Spaces?

Да, можно настроить собственный входящий трафик на стороне, созданной Azure Dev Spaces. Например, можно использовать [траефик][ingress-traefik] или [nginx][ingress-nginx].

## <a name="can-i-use-https-with-azure-dev-spaces"></a>Можно ли использовать HTTPS с Azure Dev Spaces?

Да, вы можете настроить собственный входной параметр с помощью протокола HTTPS, используя [траефик][ingress-https-traefik] или [nginx][ingress-https-nginx].

## <a name="can-i-use-azure-dev-spaces-on-a-cluster-that-uses-cni-rather-than-kubenet"></a>Можно ли использовать Azure Dev Spaces в кластере, который использует CNI, а не кубенет? 

Да, можно использовать Azure Dev Spaces в кластере AKS, который использует CNI для работы в сети. Например, можно использовать Azure Dev Spaces в кластере AKS с [существующими контейнерами Windows][windows-containers], которые используют CNI для работы в сети. Дополнительные сведения об использовании CNI для работы в сети с Azure Dev Spaces см. [здесь](configure-networking.md#using-azure-cni).

## <a name="can-i-use-azure-dev-spaces-with-windows-containers"></a>Можно ли использовать Azure Dev Spaces с контейнерами Windows?

В настоящее время Azure Dev Spaces предназначен для работы только в Pod и узлах Linux, но вы можете запускать Azure Dev Spaces в кластере AKS с [существующими контейнерами Windows][windows-containers].

## <a name="can-i-use-azure-dev-spaces-on-aks-clusters-with-api-server-authorized-ip-address-ranges-enabled"></a>Можно ли использовать Azure Dev Spaces в кластерах AKS с разрешенными диапазонами IP-адресов для сервера API?

Да, можно использовать Azure Dev Spaces в кластерах AKS с разрешенными [диапазонами IP-адресов сервера API][aks-auth-range] . Дополнительные сведения об использовании кластеров AKS с разрешенными диапазонами IP-адресов сервера API с поддержкой Azure Dev Spaces доступны [здесь](configure-networking.md#using-api-server-authorized-ip-ranges).

## <a name="can-i-use-azure-dev-spaces-on-aks-clusters-with-restricted-egress-traffic-for-cluster-nodes"></a>Можно ли использовать Azure Dev Spaces в кластерах AKS с ограниченным трафиком исходящего трафика для узлов кластера?

Да, вы можете использовать Azure Dev Spaces в кластерах AKS с [ограниченным трафиком исходящего трафика для узлов кластера][aks-restrict-egress-traffic] , включенных после того, как правильные полные доменные имена разрешены. Дополнительные сведения об использовании кластеров AKS с ограниченным трафиком исходящего трафика для узлов кластера, включенных с Azure Dev Spaces, можно найти [здесь](configure-networking.md#ingress-and-egress-network-traffic-requirements).

## <a name="can-i-use-azure-dev-spaces-on-kubernetes-rbac-enabled-aks-clusters"></a>Можно ли использовать Azure Dev Spaces в кластерах AKS с поддержкой Kubernetes RBAC?

Да, вы можете использовать Azure Dev Spaces в кластерах AKS с включенным управлением доступом на основе ролей Kubernetes (Kubernetes RBAC) или без него.

## <a name="what-happens-when-i-enable-ingress-for-project-in-visual-studio"></a>Что происходит при включении входящих данных для проекта в Visual Studio?

При использовании Visual Studio для подготовки проекта можно включить входящий для службы. При включении входящих данных создается общедоступная конечная точка для доступа к службе при работе в кластере AKS, что является необязательным. Если вы не включаете входящий трафик, служба доступна только из кластера AKS.

## <a name="can-i-use-pod-managed-identities-with-azure-dev-spaces"></a>Можно ли использовать управляемые удостоверения Pod с Azure Dev Spaces?

Да, вы можете использовать [управляемые удостоверения Pod][aks-pod-managed-id] в кластерах AKS с включенным Azure dev Spaces, но после включения Azure dev Spaces в кластере с управляемыми идентификаторами Pod необходимо выполнить [Дополнительные действия по настройке][dev-spaces-pod-managed-id-steps] . Если у вас установлены управляемые удостоверения Pod и вы хотите удалить их, дополнительные сведения см. в [примечаниях об удалении][aks-pod-managed-id-uninstall].

## <a name="can-i-use-azure-dev-spaces-with-multiple-microservices-in-an-application"></a>Можно ли использовать Azure Dev Spaces с несколькими микрослужбами в приложении?

Да, можно использовать Azure Dev Spaces в приложении с несколькими микрослужбами, но необходимо подготовить и запустить отдельные микрослужбы в их корне. Azure Dev Spaces CLI, Azure Dev Spaces VS Code расширение и Рабочая нагрузка разработки Azure для Visual Studio предполагают, что файл *аздс. YAML* находится в корне микрослужбы для запуска и отладки. Пример использования нескольких микрослужб в одном приложении см. в примере [приложения для совместного использования велосипедов][bike-sharing] .

В Visual Studio Code можно [открыть отдельные проекты в одной рабочей области][vs-code-multi-root-workspaces] и отлаживать их отдельно с помощью Azure dev Spaces. Каждый проект должен быть самодостаточным и подготовленным для Azure Dev Spaces.

В Visual Studio можно настроить решения .NET Core для отладки с помощью Azure Dev Spaces.

## <a name="can-i-use-azure-dev-spaces-with-a-service-mesh"></a>Можно ли использовать Azure Dev Spaces с сеткой службы?

В настоящее время нельзя использовать Azure Dev Spaces с сетевыми сетками служб, такими как [Istio][istio] или [Linkerd][linkerd]. Azure Dev Spaces и сетевую сетку можно выполнять в одном кластере AKS, но в одном и том же пространстве имен нельзя включать и Azure Dev Spaces, и сетку услуг.

[aks-auth-range]: ../aks/api-server-authorized-ip-ranges.md
[aks-auth-range-create]: ../aks/api-server-authorized-ip-ranges.md#create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled
[aks-auth-range-update]: ../aks/api-server-authorized-ip-ranges.md#update-a-clusters-api-server-authorized-ip-ranges
[aks-migration]: ../aks/aks-migration.md
[aks-pod-managed-id]: ../aks/developer-best-practices-pod-security.md#use-pod-managed-identities
[aks-pod-managed-id-uninstall]: https://github.com/Azure/aad-pod-identity#uninstall-notes
[aks-restrict-egress-traffic]: ../aks/limit-egress-traffic.md
[aks-supported-k8s]: ../aks/supported-kubernetes-versions.md#azure-portal-and-cli-versions
[bike-sharing]: https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp
[dev-spaces-pod-managed-id-steps]: troubleshooting.md#error-no-azureassignedidentity-found-for-podazdsazds-webhook-deployment-id-in-assigned-state
[dev-spaces-prep]: how-dev-spaces-works-prep.md
[dev-spaces-routing]: how-dev-spaces-works-routing.md#how-routing-works
[ingress-nginx]: how-to/ingress-https-nginx.md#configure-a-custom-nginx-ingress-controller
[ingress-traefik]: how-to/ingress-https-traefik.md#configure-a-custom-traefik-ingress-controller
[ingress-https-nginx]: how-to/ingress-https-nginx.md#configure-the-nginx-ingress-controller-to-use-https
[ingress-https-traefik]: how-to/ingress-https-traefik.md#configure-the-traefik-ingress-controller-to-use-https
[istio]: https://istio.io/
[linkerd]: https://linkerd.io/
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[vs-code-multi-root-workspaces]: https://code.visualstudio.com/docs/editor/multi-root-workspaces
[windows-containers]: how-to/run-dev-spaces-windows-containers.md