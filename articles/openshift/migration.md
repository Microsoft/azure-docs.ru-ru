---
title: Миграция из Azure Red Hat OpenShift 3.11 в Azure Red Hat OpenShift 4
description: Миграция из Azure Red Hat OpenShift 3.11 в Azure Red Hat OpenShift 4
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 08/13/2020
keywords: миграция, АТО, openshift, Red Hat
ms.openlocfilehash: 371672de83a6d745d7b367f8327a64e11059923e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100653284"
---
# <a name="migrate-from-azure-red-hat-openshift-311-to-azure-red-hat-openshift-4"></a>Миграция из Azure Red Hat OpenShift 3,11 в Azure Red Hat OpenShift 4

Azure Red Hat OpenShift в OpenShift 4 включает Kubernetes 1,16 в ОС Red Hat Core, частные кластеры, предоставляет собственную поддержку виртуальной сети и полную роль администратора кластера. Кроме того, теперь доступны многие новые функции, такие как поддержка инфраструктуры операторов, концентратора оператора и сетки службы OpenShift.

Чтобы успешно перейти с Azure Red Hat OpenShift 3,11 на Azure Red Hat OpenShift 4, ознакомьтесь с [различиями в хранилище, работе в сети, ведении журналов, безопасности и мониторинге](https://docs.openshift.com/container-platform/4.4/migration/migrating_3_4/planning-migration-3-to-4.html).

В этой статье мы продемонстрируем, как выполнить миграцию из кластера Azure Red Hat OpenShift 3,11 в кластер Azure Red Hat 4.

> [!NOTE]
> Средства миграции Red Hat OpenShift, такие как средство поддержки миграции плоскости управления и средство переноса приложений кластера (камера), не могут использоваться с кластерами Azure Red Hat OpenShift 3,11.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас уже есть кластер Azure Red Hat OpenShift 3,11.

## <a name="create-a-target-azure-red-hat-openshift-4-cluster"></a>Создание целевого кластера Azure Red Hat OpenShift 4

Сначала [Создайте кластер Azure Red Hat OpenShift 4](tutorial-create-cluster.md) , который вы хотите использовать в качестве целевого кластера. Здесь мы будем использовать базовую конфигурацию. Если вы заинтересованы в различных параметрах, см. [руководство по созданию кластера Red Hat OpenShift 4 в Azure](tutorial-create-cluster.md).

Создайте виртуальную сеть с двумя пустыми подсетями для главного и рабочего узлов.

```azurecli-interactive
    az network vnet create \
    --resource-group $RESOURCEGROUP \
    --name aro-vnet \
    --address-prefixes 10.0.0.0/22

    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name master-subnet \
    --address-prefixes 10.0.0.0/23 \
    --service-endpoints Microsoft.ContainerRegistry

    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name worker-subnet \
    --address-prefixes 10.0.2.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
```

Затем используйте следующую команду, чтобы создать кластер.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  # --domain foo.example.com # [OPTIONAL] custom domain
  # --pull-secret @pull-secret.txt # [OPTIONAL]
```

## <a name="configure-the-target-openshift-4-cluster"></a>Настройка целевого кластера OpenShift 4

### <a name="authentication"></a>Аутентификация

Чтобы пользователи могли взаимодействовать с Azure Red Hat OpenShift, они должны сначала пройти проверку подлинности в кластере. Уровень проверки подлинности определяет пользователя, связанного с запросами к API-интерфейсу Azure Red Hat OpenShift. Затем уровень авторизации использует сведения о запрашивающем пользователе, чтобы определить, разрешен ли запрос.

При создании кластера Azure Red Hat OpenShift 4 создается временный пользователь с правами администратора. [Подключитесь к кластеру](tutorial-connect-cluster.md), добавьте пользователей и группы и [настройте соответствующие разрешения](https://docs.openshift.com/container-platform/4.6/authentication/understanding-authentication.html) для обоих служб.

### <a name="networking"></a>Сеть

Azure Red Hat OpenShift 4 использует несколько различных операторов для настройки сети в кластере: [оператор сети кластера](https://docs.openshift.com/container-platform/4.6/networking/cluster-network-operator.html#nw-cluster-network-operator_cluster-network-operator), [оператор DNS](https://docs.openshift.com/container-platform/4.6/networking/dns-operator.html)и [оператор](https://docs.openshift.com/container-platform/4.6/networking/ingress-operator.html)входящего трафика. Дополнительные сведения о настройке сети в кластере Azure Red Hat OpenShift 4 см. в статье [Схема сетей](concepts-networking.md) и [Общие сведения о](https://docs.openshift.com/container-platform/4.6/networking/understanding-networking.html)работе в сети.

### <a name="storage"></a>Служба хранилища
Azure Red Hat OpenShift 4 поддерживает следующие подключаемые модули Персистентволуме:

- Хранилище эластичных блоков AWS (EBS)
- Диск Azure
- Файл Azure
- GCE постоянный диск
- хостпас
- iSCSI
- Локальный том
- NFS
- Хранилище контейнера Red Hat OpenShift

Сведения о настройке этих типов хранения см. в разделе [Настройка постоянного хранилища](https://access.redhat.com/documentation/azure_red_hat_openshift/4/html/storage/configuring-persistent-storage).

### <a name="registry"></a>Реестр

Azure Red Hat OpenShift 4 может создавать образы из исходного кода, развертывать их и управлять жизненным циклом. Для этого Azure Red Hat OpenShift предоставляет 4 [внутренний встроенный реестр образов контейнеров](https://docs.openshift.com/container-platform/4.6/registry/registry-options.html) , который можно развернуть в среде Azure Red Hat OpenShift для локального управления образами.

Если вы используете такие внешние реестры, как [Реестр контейнеров Azure](../container-registry/index.yml), [Red Hat Quay](https://docs.openshift.com/container-platform/4.6/registry/registry-options.html#registry-quay-overview_registry-options)или [Реестр Red Hat с поддержкой проверки подлинности](https://docs.openshift.com/container-platform/4.6/registry/registry-options.html#registry-authentication-enabled-registry-overview_registry-options), выполните действия по предоставлению учетных данных в кластер, чтобы разрешить кластеру доступ к репозиториям.

### <a name="monitoring"></a>Мониторинг

Azure Red Hat OpenShift включает предварительно настроенный, предварительно установленный и самообновляемый стек мониторинга, основанный на проекте с открытым исходным кодом Prometheus и его более широкой системе. Он обеспечивает мониторинг компонентов кластера и включает набор предупреждений для немедленного уведомления администратора кластера о возникших проблемах и наборе панелей мониторинга Grafana. Стек мониторинга кластера поддерживается только для мониторинга кластеров Azure Red Hat OpenShift. Дополнительные сведения см. в статье [мониторинг кластера для Azure Red Hat OpenShift](https://docs.openshift.com/container-platform/4.6/monitoring/understanding-the-monitoring-stack.html).

Если вы используете [Azure Monitor для контейнеров для Azure Red Hat OpenShift 3,11](../azure-monitor/containers/container-insights-azure-redhat-setup.md), можно также включить Azure Monitor для контейнеров для [кластеров Azure Red Hat OpenShift 4](../azure-monitor/containers/container-insights-azure-redhat4-setup.md) и продолжить использовать ту же log Analytics рабочую область.

## <a name="move-your-dns-or-load-balancer-configuration-to-the-new-cluster"></a>Перемещение конфигурации DNS или подсистемы балансировки нагрузки в новый кластер

Если вы используете диспетчер трафика Azure, добавьте конечные точки для ссылки на целевой кластер и определите приоритеты для этих конечных точек.

## <a name="deploy-application-to-your-target-cluster"></a>Развертывание приложения в целевом кластере

После правильной настройки целевого кластера для рабочей нагрузки [подключитесь к кластеру](tutorial-connect-cluster.md) и создайте необходимые приложения, компоненты или службы для проектов. Azure Red Hat OpenShift позволяет создавать их из Git, образов контейнеров, из каталога разработчиков Red Hat, Dockerfile, определения YAML/JSON или путем выбора службы базы данных из каталога.

## <a name="delete-your-source-cluster"></a>Удаление исходного кластера
Убедившись, что кластер Azure Red Hat OpenShift 4 настроен правильно, удалите кластер Azure Red Hat OpenShift 3,11.

```
az openshift delete --name $CLUSTER_NAME
                    --resource-group $RESOURCE_GROUP
                    [--no-wait]
                    [--subscription]
                    [--yes]
```
## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с документацией по Red Hat OpenShift [здесь](https://docs.openshift.com/container-platform/4.6/welcome/index.html).
