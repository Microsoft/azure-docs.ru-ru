---
title: Как обновить аналитику контейнера для метрик | Документация Майкрософт
description: В этой статье описывается, как обновить контейнерную аналитику, чтобы включить функцию настраиваемых метрик, которая поддерживает Просмотр сводных метрик и оповещений о них.
ms.topic: conceptual
ms.date: 10/09/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: edca7e4e8f6a9ea8dd9efdaafab8c906efd671b6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708277"
---
# <a name="how-to-update-container-insights-to-enable-metrics"></a>Обновление контейнера Insights для включения метрик

Служба "аналитика контейнеров" предоставляет поддержку для сбора метрик из Azure Kubernetes Services (AKS) и ARC в Azure с включенными узлами кластеров Kubernetes, а также их записи в хранилище метрик Azure Monitor. Это изменение предназначено для предоставления улучшенной временной шкалы при представлении статистических вычислений (AVG, Count, Max, min, сумм) в диаграммах производительности, поддержки закрепления диаграмм производительности на портал Azure информационных панелях и поддержки оповещений о метриках.

>[!NOTE]
>В настоящее время эта функция не поддерживает кластеры Azure Red Hat OpenShift.
>

В составе этой функции включены следующие метрики:

| Пространство имен метрик | Метрика | Описание |
|------------------|--------|-------------|
| Аналитика. контейнеры и узлы | Кпуусажемилликорес, Кпуусажеперцентаже, Меморирссбитес, Меморирссперцентаже, Мемориворкингсетбитес, memoryWorkingSetPercentage, nodesCount, diskUsedPercentage, | Как и метрики *узлов* , они включают *узел* в качестве измерения. Они также включают<br> имя узла в качестве значения для измерения *узла* . |
| Аналитика. контейнеры и модули | Подкаунт, Комплетеджобскаунт, Рестартингконтаинеркаунт, Умкилледконтаинеркаунт, Подреадиперцентаже | Как метрики *Pod* , они включают в себя следующие измерения: ControllerName, Kubernetes Namespace, Name и Phase. |
| Аналитика. контейнеры и контейнеры | Кпуексцеедедперцентаже, Меморирссексцеедедперцентаже, Мемориворкингсетексцеедедперцентаже | |
| Аналитика. контейнер/персистентволумес | пвусажеексцеедедперцентаже | |

Для поддержки этих новых возможностей в выпуске включена новая контейнерная агент, версия **Microsoft/OMS: ciprod05262020** для AKS и версия **Microsoft/OMS: ciprod09252020** для Arc с поддержкой кластеров Kubernetes. Новые развертывания AKS автоматически включают эти изменения конфигурации и возможности. Обновление кластера для поддержки этой функции можно выполнить из портал Azure, Azure PowerShell или с помощью Azure CLI. С Azure PowerShell и CLI. Вы можете включить этот параметр для каждого кластера или для всех кластеров в подписке.

Любой процесс назначает роль **издателя метрик мониторинга** субъекту-службе кластера или назначенному пользователем MSI для надстройки мониторинга, чтобы данные, собранные агентом, можно было опубликовать в ресурсе кластеров. Издатель метрик мониторинга имеет разрешение только на отправку метрик в ресурс, не может изменять состояние, обновлять ресурс или считывать данные. Дополнительные сведения о роли см. в разделе [мониторинг метрик роли издателя](../../role-based-access-control/built-in-roles.md#monitoring-metrics-publisher). Требование роли издателя метрики мониторинга неприменимо к кластерам Kubernetes с включенной службой ARC.

> [!IMPORTANT]
> Обновление не требуется для кластеров Kubernetes с поддержкой службы "Дуга Azure", так как они уже имеют минимальную требуемую версию агента.

## <a name="prerequisites"></a>Предварительные условия

Перед обновлением кластера подтвердите следующее.

* Пользовательские метрики доступны только в подмножестве регионов Azure. Список поддерживаемых регионов приведен [здесь](../essentials/metrics-custom-overview.md#supported-regions).

* Вы являетесь членом роли **[владельца](../../role-based-access-control/built-in-roles.md#owner)** в ресурсе кластера AKS, чтобы включить сбор настраиваемых метрик производительности node и Pod. Это требование не распространяется на кластеры Kubernetes с включенной службой ARC.

Если вы решили использовать Azure CLI, необходимо сначала установить интерфейс командной строки и использовать его локально. Необходимо запустить Azure CLI версии 2.0.59 или более поздней. Для определения версии выполните `az --version`. Если вам необходимо установить или обновить Azure CLI, ознакомьтесь со статьей [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="upgrade-a-cluster-from-the-azure-portal"></a>Обновление кластера с портал Azure

Для существующих кластеров AKS, отслеживаемых службой "аналитика контейнеров", **после выбора кластера** для просмотра его работоспособности из многокластерного представления в Azure Monitor или непосредственно из кластера. для этого в верхней части портала отобразится баннер.

![Обновление баннера кластера AKS в портал Azure](./media/container-insights-update-metrics/portal-banner-enable-01.png)

При нажатии кнопки **включить** будет инициирован процесс обновления кластера. Этот процесс может занять несколько секунд. Ход его выполнения можно просмотреть в разделе меню Уведомления.

## <a name="upgrade-all-clusters-using-bash-in-azure-command-shell"></a>Обновление всех кластеров с помощью Bash в командной оболочке Azure

Выполните следующие действия, чтобы обновить все кластеры в подписке с помощью Bash в командной оболочке Azure.

1. Выполните следующую команду с помощью Azure CLI.  Измените значение для **SubscriptionId** , используя значение на странице **обзора AKS** для кластера AKS.

    ```azurecli
    az login
    az account set --subscription "Subscription Name"
    curl -sL https://aka.ms/ci-md-onboard-atscale | bash -s subscriptionId   
    ```

    Для завершения изменения конфигурации может потребоваться несколько секунд. После завершения настройки конфигурации появится сообщение, похожее на приведенное ниже, с таким результатом:

    ```azurecli
    completed role assignments for all AKS clusters in subscription: <subscriptionId>
    ```

## <a name="upgrade-per-cluster-using-azure-cli"></a>Обновление на кластер с помощью Azure CLI

Выполните следующие действия, чтобы обновить конкретный кластер в подписке с помощью Azure CLI.

1. Выполните следующую команду с помощью Azure CLI. Измените значения для **SubscriptionId**, **resourceGroupName** и **имя_кластера** , используя значения на странице **Обзор AKS** для кластера AKS.  Чтобы получить значение **клиентидофспн**, оно возвращается при выполнении команды `az aks show` , как показано в примере ниже.

    ```azurecli
    az login
    az account set --subscription "<subscriptionName>"
    az aks show -g <resourceGroupName> -n <clusterName> 
    az role assignment create --assignee <clientIdOfSPN> --scope <clusterResourceId> --role "Monitoring Metrics Publisher" 
    ```

    Чтобы получить значение для **клиентидофспнормси**, можно выполнить команду, `az aks show` как показано в примере ниже. Если объект **сервицепринЦипалпрофиле** имеет допустимое значение *ClientID* , его можно использовать. В противном случае, если задано значение *MSI*, необходимо передать ClientID из `addonProfiles.omsagent.identity.clientId` .

    ```azurecli
    az login
    az account set --subscription "<subscriptionName>"
    az aks show -g <resourceGroupName> -n <clusterName> 
    az role assignment create --assignee <clientIdOfSPNOrMsi> --scope <clusterResourceId> --role "Monitoring Metrics Publisher"
    ```

## <a name="upgrade-all-clusters-using-azure-powershell"></a>Обновление всех кластеров с помощью Azure PowerShell

Чтобы обновить все кластеры в подписке с помощью Azure PowerShell, выполните следующие действия.

1. [Скачайте](https://github.com/microsoft/OMS-docker/blob/ci_feature_prod/docs/aks/mdmonboarding/mdm_onboarding_atscale.ps1) скрипт **mdm_onboarding_atscale.ps1** и сохраните его в локальной папке из репозитория GitHub.
2. Выполните следующую команду с помощью Azure PowerShell.  Измените значение для **SubscriptionId** , используя значение на странице **обзора AKS** для кластера AKS.

    ```powershell
    .\mdm_onboarding_atscale.ps1 subscriptionId
    ```
    Для завершения изменения конфигурации может потребоваться несколько секунд. После завершения настройки конфигурации появится сообщение, похожее на приведенное ниже, с таким результатом:

    ```powershell
    Completed adding role assignment for the aks clusters in subscriptionId :<subscriptionId>
    ```

## <a name="upgrade-per-cluster-using-azure-powershell"></a>Обновление на кластер с помощью Azure PowerShell

Выполните следующие действия, чтобы обновить конкретный кластер с помощью Azure PowerShell.

1. [Скачайте](https://github.com/microsoft/OMS-docker/blob/ci_feature_prod/docs/aks/mdmonboarding/mdm_onboarding.ps1) скрипт **mdm_onboarding.ps1** и сохраните его в локальной папке из репозитория GitHub.

2. Выполните следующую команду с помощью Azure PowerShell. Измените значения для **SubscriptionId**, **resourceGroupName** и **имя_кластера** , используя значения на странице **Обзор AKS** для кластера AKS.

    ```powershell
    .\mdm_onboarding.ps1 subscriptionId <subscriptionId> resourceGroupName <resourceGroupName> clusterName <clusterName>
    ```

    Для завершения изменения конфигурации может потребоваться несколько секунд. После завершения настройки конфигурации появится сообщение, похожее на приведенное ниже, с таким результатом:

    ```powershell
    Successfully added Monitoring Metrics Publisher role assignment to cluster : <clusterName>
    ```

## <a name="verify-update"></a>Проверка обновления

После запуска обновления с помощью одного из методов, описанных выше, можно использовать Azure Monitor обозреватель метрик и проверить из **пространства имен метрик** **, которое отображается** в списке. Если это так, можно приступить к настройке [оповещений метрик](../alerts/alerts-metric.md) или закрепления диаграмм на [панелях мониторинга](../../azure-portal/azure-portal-dashboards.md).  
