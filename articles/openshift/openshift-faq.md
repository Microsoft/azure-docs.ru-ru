---
title: Часто задаваемые вопросы о Azure Red Hat OpenShift
description: Ниже приведены ответы на часто задаваемые вопросы о Microsoft Azure Red Hat OpenShift.
author: jimzim
ms.author: jzim
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 07/31/2020
ms.openlocfilehash: a3721083e48774963cd761178abdb552c93b15c7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100634351"
---
# <a name="azure-red-hat-openshift-faq"></a>Вопросы и ответы об Azure Red Hat OpenShift

В этой статье содержатся ответы на часто задаваемые вопросы о Microsoft Azure Red Hat OpenShift.

## <a name="installation-and-upgrade"></a>Установка и обновление

### <a name="which-azure-regions-are-supported"></a>Какие регионы Azure поддерживаются?

Список поддерживаемых регионов для Azure Red Hat OpenShift 4. x см. в разделе [Доступные регионы](https://azure.microsoft.com/global-infrastructure/services/?products=openshift&regions=all).

Список поддерживаемых регионов для Azure Red Hat OpenShift 3,11 см. в разделе [продукты, доступные по регионам](supported-resources.md#azure-regions).

### <a name="what-virtual-machine-sizes-can-i-use"></a>Какие размеры виртуальных машин можно использовать?

Список поддерживаемых размеров виртуальных машин для Azure Red Hat OpenShift 4 см. в статье [Поддерживаемые ресурсы для Azure Red Hat OpenShift 4](support-policies-v4.md).

Список поддерживаемых размеров виртуальных машин для Azure Red Hat OpenShift 3,11 см. в статье [Поддерживаемые ресурсы для Azure Red Hat OpenShift 3,11](supported-resources.md).

### <a name="what-is-the-maximum-number-of-pods-in-an-azure-red-hat-openshift-cluster--what-is-the-maximum-number-of-pods-per-node-in-azure-red-hat-openshift"></a>Каково максимальное число модулей Pod в кластере Azure Red Hat OpenShift?  Каково максимальное число модулей Pod на узел в Azure Red Hat OpenShift?

Фактическое число поддерживаемых модулей модулями данных зависит от требований к памяти, ЦП и хранилищу приложения.

В Azure Red Hat OpenShift 4. x предусмотрено ограничение в 250 Pod на узел и ограничение на количество вычислений для узлов в 100. Это максимальное число модулей Pod, поддерживаемое в кластере, равно 250 &times; 100 = 25 000.

Azure Red Hat OpenShift 3,11 имеет ограничение на число модулей Pod 50 на один узел и 20 ГБ. Это максимальное число модулей Pod, поддерживаемое в кластере, равно 50 &times; 20 = 1 000.

### <a name="can-a-cluster-have-compute-nodes-across-multiple-azure-regions"></a>Может ли кластер вычислять узлы в нескольких регионах Azure?

Нет. Все узлы в кластере Azure Red Hat OpenShift должны быть созданы в одном регионе Azure.

### <a name="can-a-cluster-be-deployed-across-multiple-availability-zones"></a>Можно ли развернуть кластер в нескольких зонах доступности?

Да. Это происходит автоматически, если кластер развертывается в регионе Azure, поддерживающем зоны доступности. Дополнительные сведения см. в статье [Регионы и зоны доступности в Azure](../availability-zones/az-overview.md#availability-zones).

### <a name="are-control-plane-nodes-abstracted-away-as-they-are-with-azure-kubernetes-service-aks"></a>Являются ли узлы плоскости управления абстрактными, так как они находятся в службе Azure Kubernetes Service (AKS)?

Нет. Все ресурсы, включая главные узлы кластера, выполняются в клиентской подписке. Эти типы ресурсов помещаются в группу ресурсов только для чтения.

### <a name="does-the-cluster-reside-in-a-customer-subscription"></a>Находится ли кластер в клиентской подписке? 

Управляемое приложение Azure находится в заблокированной группе ресурсов с подпиской клиента. Клиенты могут просматривать объекты в этой группе ресурсов, но не изменять их.

### <a name="is-there-any-element-in-azure-red-hat-openshift-shared-with-other-customers-or-is-everything-independent"></a>Есть ли какой-либо элемент в Azure Red Hat OpenShift, совместно используемый с другими клиентами? Или является ли все независимым?

Каждый кластер Azure Red Hat OpenShift предназначен для конкретного клиента и находится в подписке клиента. 

### <a name="are-infrastructure-nodes-available"></a>Доступны ли узлы инфраструктуры?

В кластерах Azure Red Hat OpenShift 4. x в настоящее время узлы инфраструктуры недоступны.

В кластерах Azure Red Hat OpenShift 3,11 узлы инфраструктуры включены по умолчанию.

## <a name="how-do-i-handle-cluster-upgrades"></a>Разделы справки обработаны обновления кластера?

Сведения об обновлениях, обслуживании и поддерживаемых версиях см. в статье о [жизненном цикле поддержки](support-lifecycle.md).

### <a name="how-will-the-host-operating-system-and-openshift-software-be-updated"></a>Как будет обновляться операционная система узла и программное обеспечение OpenShift?

Операционные системы узла и программное обеспечение OpenShift обновляются как Azure Red Hat OpenShift использует дополнительные версии выпуска и исправления из вышестоящей платформы контейнеров OpenShift.

### <a name="whats-the-process-to-reboot-the-updated-node"></a>В чем заключается процедура перезагрузки обновленного узла?

Узлы перезапускаются в процессе обновления.

## <a name="cluster-operations"></a>Операции с кластером

### <a name="can-i-use-prometheus-to-monitor-my-applications"></a>Можно ли использовать Prometheus для мониторинга приложений?

Prometheus предварительно установлен и настроен для кластеров Azure Red Hat OpenShift 4. x. Узнайте больше о [мониторинге кластера](https://docs.openshift.com/container-platform/4.6/operators/operator_sdk/osdk-monitoring-prometheus.html).

Для кластеров Azure Red Hat OpenShift 3,11 можно развернуть Prometheus в пространстве имен и отслеживать приложения в пространстве имен. Дополнительные сведения см. [в статье развертывание экземпляра Prometheus в кластере Azure Red Hat OpenShift](howto-deploy-prometheus.md).

### <a name="can-i-use-prometheus-to-monitor-metrics-related-to-cluster-health-and-capacity"></a>Можно ли использовать Prometheus для мониторинга метрик, относящихся к работоспособности и емкости кластера?

В Azure Red Hat OpenShift 4. x: Да.

В Azure Red Hat OpenShift 3,11: нет.

### <a name="can-logs-of-underlying-vms-be-streamed-out-to-a-customer-log-analysis-system"></a>Могут ли журналы базовых виртуальных машин передаваться в потоковую систему анализа журнала клиента?

Журналы из базовых виртуальных машин обрабатываются управляемой службой и не предоставляются клиентам.

### <a name="how-can-a-customer-get-access-to-metrics-like-cpumemory-at-the-node-level-to-take-action-to-scale-debug-issues-etc-i-cannot-seem-to-run-kubectl-top-on-an-azure-red-hat-openshift-cluster"></a>Как клиент может получить доступ к метрикам, таким как ЦП/память на уровне узла, чтобы предпринять действия по масштабированию, отладке и т. д.? Я не могу запустить kubectl Top в кластере Azure Red Hat OpenShift.

Для кластеров Azure Red Hat OpenShift 4. x веб-консоль OpenShift содержит все метрики на уровне узла. Дополнительные сведения см. в документации по Red Hat по [просмотру сведений о кластере](https://docs.openshift.com/container-platform/4.6/web_console/using-dashboard-to-get-cluster-information.html).

Для кластеров Azure Red Hat OpenShift 3,11 Клиенты могут получать доступ к метрикам ЦП и памяти на уровне узла с помощью команды `oc adm top nodes` или `kubectl top nodes` роли кластера "клиент-Администратор". Клиенты также могут получить доступ к метрикам ЦП и памяти `pods` с помощью команды `oc adm top pods` или `kubectl top pods` .

### <a name="if-we-scale-up-the-deployment-how-do-azure-fault-domains-map-into-pod-placement-to-ensure-all-pods-for-a-service-do-not-get-knocked-out-by-a-failure-in-a-single-fault-domain"></a>Если развернуть развертывание, то как домены сбоя Azure сопоставляются с размещением Pod, чтобы не замаскировать все модули для службы в случае сбоя в одном домене сбоя?

При использовании масштабируемых наборов виртуальных машин в Azure по умолчанию используются пять доменов сбоя. Каждый экземпляр виртуальной машины в масштабируемом наборе будет помещен в один из этих доменов сбоя. Это гарантирует, что приложения, развернутые на вычисленных узлах в кластере, будут размещены в отдельных доменах сбоя.

Дополнительные сведения см. [в разделе Выбор правильного числа доменов сбоя для масштабируемого набора виртуальных машин](../virtual-machine-scale-sets/virtual-machine-scale-sets-manage-fault-domains.md).

### <a name="is-there-a-way-to-manage-pod-placement"></a>Существует ли способ управления размещением Pod?

Клиенты могут получать узлы и просматривать метки в качестве клиента-администратора. Это обеспечит возможность выбрать любую виртуальную машину в масштабируемом наборе.

При использовании специальных меток необходимо использовать осторожность.

- Имя узла не должно использоваться. Имя узла часто поворачивается с обновлением и обновлениями и гарантированно изменяется.
- Если у клиента есть запрос на определенные метки или стратегию развертывания, это можно сделать, но в настоящее время потребуется выполнить проектирование и не поддерживаться сегодня.

Дополнительные сведения см. в разделе [Управление размещением Pod](https://docs.openshift.com/container-platform/4.6/nodes/scheduling/nodes-scheduler-about.html).

### <a name="is-the-image-registry-available-externally-so-i-can-use-tools-such-as-jenkins"></a>Доступен ли реестр изображений извне, поэтому я могу использовать такие средства, как Jenkins?

Для кластеров 4. x необходимо предоставить безопасный реестр и настроить проверку подлинности. Дополнительные сведения см. в следующей документации по Red Hat:

- [Предоставление доступа к реестру](https://docs.openshift.com/container-platform/4.6/registry/securing-exposing-registry.html)
- [Доступ к реестру](https://docs.openshift.com/container-platform/4.6/registry/accessing-the-registry.html)

Для кластеров 3,11 доступен реестр образов DOCKER. Реестр DOCKER доступен в `https://docker-registry.apps.<clustername>.<region>.azmosa.io/` . Вы также можете использовать реестр контейнеров Azure.

## <a name="networking"></a>Сеть

### <a name="can-i-deploy-a-cluster-into-an-existing-virtual-network"></a>Можно ли развернуть кластер в существующей виртуальной сети?

В кластерах 4. x можно развернуть кластер в существующей виртуальной сети.

В кластерах 3,11 нельзя развернуть кластер в существующей виртуальной сети. Кластер Azure Red Hat OpenShift 3,11 можно подключить к существующей виртуальной сети через пиринг.

### <a name="is-cross-namespace-networking-supported"></a>Поддерживается ли поддержка сетей с несколькими пространствами имен?

Администраторы клиентов и индивидуальных проектов могут настраивать сеть между пространствами имен (включая отклонения) для каждого проекта с помощью `NetworkPolicy` объектов.

### <a name="i-am-trying-to-peer-into-a-virtual-network-in-a-different-subscription-but-getting-failed-to-get-vnet-cidr-error"></a>Я пытаюсь установить пиринг в виртуальную сеть в другой подписке, но получить ошибку CIDR виртуальной сети не удалось.

В подписке с виртуальной сетью обязательно выполните регистрацию `Microsoft.ContainerService` поставщика с помощью следующей команды: `az provider register -n Microsoft.ContainerService --wait`

### <a name="can-we-specify-ip-ranges-for-deployment-on-the-private-vnet-avoiding-clashes-with-other-corporate-vnets-once-peered"></a>Можно ли указать диапазоны IP-адресов для развертывания в частной виртуальной сети, избегая конфликта с другими корпоративными виртуальных сетей после пиринга?

В кластерах с 4. x можно указать собственные диапазоны IP-адресов.

В кластерах 3,11 Azure Red Hat OpenShift поддерживает пиринг виртуальных сетей и позволяет клиенту предоставлять виртуальную сеть для одноранговой сети и CIDR виртуальной сети, в которой будет действовать сеть OpenShift.

Виртуальная сеть, созданная Azure Red Hat OpenShift, будет защищена и не будет принимать изменения конфигурации. Виртуальная сеть, для которой установлено одноранговое подключение, управляется клиентом и находится в своей подписке.

### <a name="is-the-software-defined-network-module-configurable"></a>Является ли программно определяемый сетевой модуль настраиваемым?

Программно определенная сеть недоступна для `openshift-ovs-networkpolicy` настройки.

### <a name="what-azure-load-balancer-is-used-by-azure-red-hat-openshift--is-it-standard-or-basic-and-is-it-configurable"></a>Какую подсистему балансировки нагрузки Azure использует Azure Red Hat OpenShift?  Является ли он стандартным или базовым и является ли он настраиваемым?

Azure Red Hat OpenShift использует стандартные Azure Load Balancer и не настраивается.

## <a name="permissions"></a>Разрешения

### <a name="can-an-admin-manage-users-and-quotas"></a>Может ли администратор управлять пользователями и квотами?

Да. Администратор Red Hat OpenShift Azure может управлять пользователями и квотами в дополнение к доступу ко всем созданным пользователем проектам.

### <a name="can-i-restrict-a-cluster-to-only-certain-azure-ad-users"></a>Можно ли ограничить кластер только определенными пользователями Azure AD?

Да. Вы можете ограничить, какие пользователи Azure AD могут входить в кластер, настроив приложение Azure AD. Дополнительные сведения см. в разделе [практические руководства. ограничение приложения определенным набором пользователей](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md).

### <a name="can-i-restrict-users-from-creating-projects"></a>Можно ли запретить пользователям создавать проекты?

Да. Войдите в кластер от имени администратора и выполните следующую команду:

```
oc adm policy \
    remove-cluster-role-from-group self-provisioner \
    system:authenticated:oauth
```

Дополнительные сведения см. в документации OpenShift по отключению самостоятельной подготовки для версии кластера.

- [Отключение самостоятельной подготовки в кластерах 4,6](https://docs.openshift.com/container-platform/4.6/applications/projects/configuring-project-creation.html#disabling-project-self-provisioning_configuring-project-creation)
- [Отключение самостоятельной подготовки в кластерах 3,11](https://docs.openshift.com/container-platform/3.11/admin_guide/managing_projects.html#disabling-self-provisioning)

### <a name="which-unix-rights-in-iaas-are-available-for-mastersinfraapp-nodes"></a>Какие права UNIX (в IaaS) доступны для образцов, а так и для узлов с инфракрасными или приложениями?

Для кластеров 4. x доступ к узлу можно получить с помощью роли администратора кластера. Дополнительные сведения см. в разделе [Общие сведения о KUBERNETES RBAC](https://docs.openshift.com/container-platform/4.6/authentication/using-rbac.html).

Для кластеров 3,11 доступ к узлу запрещен.

### <a name="which-ocp-rights-do-we-have-cluster-admin-project-admin"></a>Какие права OCP у нас есть? Кластер — администратор? Проект — администратор?

Для кластеров 4. x доступна роль администратора кластера. Дополнительные сведения см. в разделе [Общие сведения о KUBERNETES RBAC](https://docs.openshift.com/container-platform/4.6/authentication/using-rbac.html).

Дополнительные сведения о кластерах 3,11 см. в разделе [Общие сведения об администрировании кластера](https://docs.openshift.com/aro/admin_guide/index.html) .

### <a name="which-identity-providers-are-available"></a>Какие поставщики удостоверений доступны?

Для кластеров 4. x вы настраиваете собственный поставщик удостоверений. Дополнительные сведения см. в документации по Red Hat по [настройке поставщиков удостоверений](https://docs.openshift.com/container-platform/4.6/authentication/identity_providers/configuring-ldap-identity-provider.html).

Для кластеров 3,11 можно использовать интеграцию Azure AD. 

## <a name="storage"></a>Служба хранилища

### <a name="is-data-on-my-cluster-encrypted"></a>Зашифрованы ли данные в кластере?

По умолчанию данные шифруются неактивных данных. Платформа службы хранилища Azure автоматически шифрует данные перед сохранением и расшифровывает их перед извлечением. Дополнительные сведения см. [в статье шифрование службы хранилища Azure для неактивных данных](../storage/common/storage-service-encryption.md).

### <a name="is-data-stored-in-etcd-encrypted-on-azure-red-hat-openshift"></a>Хранятся ли данные в etcd, зашифрованных в Azure Red Hat OpenShift?

Для кластеров Azure Red Hat OpenShift 4 данные по умолчанию не шифруются, но у вас есть возможность включить шифрование. Дополнительные сведения см. в инструкции по [шифрованию etcd](https://docs.openshift.com/container-platform/4.6/security/encrypting-etcd.html).

Для кластеров 3,11 данные не шифруются на уровне etcd. Параметр для включения шифрования в настоящее время не поддерживается. OpenShift поддерживает эту функцию, но требуется инженерная деятельность, чтобы сделать ее на схеме дороги. Данные шифруются на уровне диска. Дополнительные сведения см. в разделе [Шифрование данных на уровне хранилища](https://docs.openshift.com/container-platform/3.11/admin_guide/encrypting_data.html) .

### <a name="can-we-choose-any-persistent-storage-solution-like-ocs"></a>Можно ли выбрать любое решение постоянного хранения, например OCS? 

Для кластеров 4. x диск Azure (Premium_LRS) настраивается как класс хранения по умолчанию. Дополнительные поставщики хранилища и сведения о конфигурации (включая файл Azure) см. в документации по Red Hat в [постоянном хранилище](https://docs.openshift.com/container-platform/4.6/storage/understanding-persistent-storage.html).

Для кластеров 3,11 по умолчанию предоставляются два класса хранения: один для диска Azure (Premium_LRS) и один для файла Azure.

## <a name="does-aro-store-any-customer-data-outside-of-the-clusters-region"></a>Сохраняет ли АТО данные клиентов за пределами региона кластера?

Нет. Все данные, созданные в кластере АТО, сохраняются в регионе кластера.
