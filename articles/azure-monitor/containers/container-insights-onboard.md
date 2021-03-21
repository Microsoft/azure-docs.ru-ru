---
title: Включить аналитику контейнера | Документация Майкрософт
description: В этой статье описывается, как включить и настроить аналитику контейнера, чтобы вы могли понять, как работает контейнер и какие проблемы с производительностью были обнаружены.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 58797221fa3380e4f7533a710e2f8dc658cb676c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708362"
---
# <a name="enable-container-insights"></a>Включить аналитику контейнера

В этой статье приводятся общие сведения о параметрах, доступных для настройки контейнеров для отслеживания производительности рабочих нагрузок, развернутых в средах Kubernetes и размещенных в.

- [Служба Azure Kubernetes (AKS)](../../aks/index.yml)  
- [Azure Red Hat OpenShift](../../openshift/intro-openshift.md) версии 3. x и 4. x  
- [Red Hat OpenShift](https://docs.openshift.com/container-platform/4.3/welcome/index.html) версии 4. x  
- [Кластер Kubernetes с поддержкой Arc](../../azure-arc/kubernetes/overview.md)

Кроме того, можно отслеживать производительность рабочих нагрузок, развернутых в самостоятельно управляемых кластерах Kubernetes, размещенных на:
- Azure с помощью [подсистемы AKS](https://github.com/Azure/aks-engine)
- [Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) или локально, с помощью подсистемы AKS.

Вы можете включить аналитику контейнера для нового развертывания или для одного или нескольких существующих развертываний Kubernetes с помощью любого из следующих поддерживаемых методов:

- Портал Azure
- Azure PowerShell
- Azure CLI
- [Terraform и AKS](/azure/developer/terraform/create-k8s-cluster-with-tf-and-aks)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, убедитесь, что выполнены следующие требования:

> [!IMPORTANT]
> Log Analytics контейнерный агент Linux (Pod) выполняет вызовы API во все узлы Windows на защищенном порту Kubelet (10250) в кластере для сбора метрик, относящихся к производительности узлов и контейнеров. Защищенный порт Kubelet (: 10250) должен быть открыт в виртуальной сети кластера как для входящей, так и для исходящей связи для работы коллекции метрик, связанной с производительностью узла и контейнера Windows.
>
> Если у вас есть кластер Kubernetes с узлами Windows, проверьте и настройте группу безопасности сети и политики сети, чтобы убедиться, что защищенный порт Kubelet (: 10250) открыт как для входящей, так и для исходящей в виртуальной сети кластера.


- У вас Log Analytics Рабочая область.

   Контейнерная аналитика поддерживает Log Analytics рабочей области в регионах, которые перечислены в списке [продукты, доступные по регионам](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor).

   Вы можете создать рабочую область, когда вы включите мониторинг для нового кластера AKS, или вы можете разрешить интегрированию создать рабочую область по умолчанию в группе ресурсов по умолчанию для подписки кластера AKS. 
   
   Если вы решили самостоятельно создать рабочую область, ее можно создать с помощью: 
   - [Azure Resource Manager](../logs/resource-manager-workspace.md)
   - [PowerShell](../logs/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json)
   - [Портал Azure](../logs/quick-create-workspace.md) 
   
   Список поддерживаемых пар сопоставлений, используемых для рабочей области по умолчанию, см. в разделе [сопоставление регионов для аналитики контейнера](container-insights-region-mapping.md).

- Вы являетесь членом группы *участников log Analytics* , чтобы включить мониторинг контейнеров. Дополнительные сведения о том, как управлять доступом к рабочей области Log Analytics, см. в статье [Управление рабочими областями](../logs/manage-access.md).

- Вы являетесь членом группы " [ *владелец*](../../role-based-access-control/built-in-roles.md#owner) " в ресурсе кластера AKS.

   [!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

- Для просмотра данных мониторинга необходимо иметь роль [*log Analytics читателя*](../logs/manage-access.md#manage-access-using-azure-permissions) в рабочей области log Analytics, настроенную с помощью аналитики контейнера.

- Метрики Prometheus не собираются по умолчанию. Перед [настройкой агента](container-insights-prometheus-integration.md) для сбора метрик необходимо ознакомиться с [документацией по Prometheus](https://prometheus.io/) , чтобы понять, какие данные могут быть оббракованы и какие методы поддерживаются.

## <a name="supported-configurations"></a>Поддерживаемые конфигурации

Аналитика в контейнере официально поддерживает следующие конфигурации:

- Среды: Azure Red Hat OpenShift, Kubernetes в локальной среде и подсистема AKS в Azure и Azure Stack. Дополнительные сведения см. [в разделе AKS Engine on Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview).
- Версии политики Kubernetes и поддержки аналогичны тем, которые [поддерживаются в службе Kubernetes Azure (AKS)](../../aks/supported-kubernetes-versions.md). 

## <a name="network-firewall-requirements"></a>Требования к брандмауэру в сети

В следующей таблице перечислены сведения о конфигурации прокси-сервера и брандмауэра, необходимые для обмена данными между контейнерными агентами и аналитикой. Весь сетевой трафик от агента исходит от Azure Monitor.

|Ресурс агента|Порт |
|--------------|------|
| `*.ods.opinsights.azure.com` | 443 |
| `*.oms.opinsights.azure.com` | 443 |
| `dc.services.visualstudio.com` | 443 |
| `*.monitoring.azure.com` | 443 |
| `login.microsoftonline.com` | 443 |

В следующей таблице перечислены сведения о конфигурации прокси-сервера и брандмауэра для Azure Китая 21Vianet.

|Ресурс агента|Порт |Описание | 
|--------------|------|-------------|
| `*.ods.opinsights.azure.cn` | 443 | Прием данных |
| `*.oms.opinsights.azure.cn` | 443 | Адаптация OMS |
| `dc.services.visualstudio.com` | 443 | Для телеметрии агентов, использующих общедоступное облако Azure Application Insights |

В следующей таблице перечислены сведения о конфигурации прокси-сервера и брандмауэра для государственных организаций США.

|Ресурс агента|Порт |Описание | 
|--------------|------|-------------|
| `*.ods.opinsights.azure.us` | 443 | Прием данных |
| `*.oms.opinsights.azure.us` | 443 | Адаптация OMS |
| `dc.services.visualstudio.com` | 443 | Для телеметрии агентов, использующих общедоступное облако Azure Application Insights |

## <a name="components"></a>Компоненты

Возможность мониторинга производительности зависит от контейнера Log Analytics агента для Linux, специально разработанного для аналитики контейнеров. Этот специализированный агент собирает данные о производительности и событиях со всех узлов кластера, а потом автоматически развертывается и регистрируется в указанной рабочей области Log Analytics во время развертывания. 

Версия агента — Microsoft/OMS: ciprod04202018 или более поздняя и представленная датой в следующем формате: *ммддииии*.

>[!NOTE]
>Благодаря общедоступной поддержке Windows Server для AKS кластер AKS с узлами Windows Server имеет предварительную версию агента предварительного просмотра, установленную на каждом отдельном узле Windows Server для получения журналов и пересылки их в Log Analytics. Для метрик производительности узел Linux, автоматически развернутый в кластере в рамках стандартного развертывания, собирает и пересылает данные Azure Monitor от имени всех узлов Windows в кластере.

При выпуске новой версии агента она автоматически обновляется в управляемых кластерах Kubernetes, размещенных в службе Kubernetes Azure (AKS). Сведения о том, какие версии выпускаются, см. в разделе [извещения о выпуске агента](https://github.com/microsoft/docker-provider/tree/ci_feature_prod).

> [!NOTE]
> Если вы уже развернули кластер AKS, вы включили наблюдение с помощью Azure CLI или предоставленного шаблона Azure Resource Manager, как показано далее в этой статье. Нельзя использовать `kubectl` для обновления, удаления, повторного развертывания или развертывания агента.
>
> Развертывание шаблона должно проходить в той же группе ресурсов, что и у кластера.

Чтобы включить аналитику контейнера, используйте один из методов, описанных в следующей таблице.

| Состояние развертывания | Метод | Описание |
|------------------|--------|-------------|
| Новый кластер Kubernetes | [Создание кластера AKS с помощью Azure CLI](../../aks/kubernetes-walkthrough.md#create-aks-cluster)| Вы можете включить мониторинг для нового кластера AKS, созданного с помощью Azure CLI. |
| | [Создание кластера AKS с помощью terraform](container-insights-enable-new-cluster.md#enable-using-terraform)| Вы можете включить мониторинг для нового кластера AKS, созданного с помощью средства с открытым кодом terraform. |
| | [Создание кластера OpenShift с помощью шаблона Azure Resource Manager](container-insights-azure-redhat-setup.md#enable-for-a-new-cluster-using-an-azure-resource-manager-template) | Вы можете включить мониторинг для нового кластера OpenShift, созданного с помощью предварительно настроенного шаблона Azure Resource Manager. |
| | [Создание кластера OpenShift с помощью Azure CLI](/cli/azure/openshift#az-openshift-create) | Мониторинг можно включить при развертывании нового кластера OpenShift с помощью Azure CLI. |
| Существующий кластер Kubernetes | [Включение мониторинга кластера AKS с помощью Azure CLI](container-insights-enable-existing-clusters.md#enable-using-azure-cli) | Вы можете включить мониторинг для кластера AKS, который уже развернут с помощью Azure CLI. |
| |[Включение для кластера AKS с помощью terraform](container-insights-enable-existing-clusters.md#enable-using-terraform) | Вы можете включить мониторинг для кластера AKS, который уже развернут с помощью средства с открытым кодом terraform. |
| | [Включить для кластера AKS из Azure Monitor](container-insights-enable-existing-clusters.md#enable-from-azure-monitor-in-the-portal)| Вы можете включить мониторинг для одного или нескольких кластеров AKS, которые уже развернуты на странице с несколькими кластерами в Azure Monitor. |
| | [Включить из кластера AKS](container-insights-enable-existing-clusters.md#enable-directly-from-aks-cluster-in-the-portal)| Мониторинг можно включить непосредственно из кластера AKS в портал Azure. |
| | [Включение для кластера AKS с помощью шаблона Azure Resource Manager](container-insights-enable-existing-clusters.md#enable-using-an-azure-resource-manager-template)| Вы можете включить мониторинг для кластера AKS с помощью предварительно настроенного шаблона Azure Resource Manager. |
| | [Включить для гибридного кластера Kubernetes](container-insights-hybrid-setup.md) | Вы можете включить мониторинг для подсистемы AKS, размещенной на Azure Stack или для кластера Kubernetes, размещенного в локальной среде. |
| | [Включить кластер Kubernetes с поддержкой Arc](container-insights-enable-arc-enabled-clusters.md). | Вы можете включить мониторинг для кластеров Kubernetes, размещенных за пределами Azure и включенных с помощью дуги Azure. |
| | [Включение для кластера OpenShift с помощью шаблона Azure Resource Manager](container-insights-azure-redhat-setup.md#enable-using-an-azure-resource-manager-template) | Вы можете включить мониторинг для существующего кластера OpenShift с помощью предварительно настроенного шаблона Azure Resource Manager. |
| | [Включить для кластера OpenShift из Azure Monitor](container-insights-azure-redhat-setup.md#from-the-azure-portal) | Вы можете включить мониторинг для одного или нескольких кластеров OpenShift, которые уже развернуты на многокластерной странице в Azure Monitor. |

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы включили мониторинг, можно начать анализ производительности кластеров Kubernetes, размещенных в службе Kubernetes Azure (AKS), Azure Stack или другой среде. Чтобы узнать, как использовать аналитику контейнера, ознакомьтесь со статьей [Просмотр производительности кластера Kubernetes](container-insights-analyze.md).