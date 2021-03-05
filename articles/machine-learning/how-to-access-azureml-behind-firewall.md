---
title: Использование брандмауэра
titleSuffix: Azure Machine Learning
description: Контролируйте доступ к рабочим областям Машинное обучение Azure с помощью брандмауэров Azure. Сведения об узлах, которые необходимо разрешить через брандмауэр.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 11/18/2020
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 295228e9eaa3529b05055869bd46f9aefc938a6f
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102212779"
---
# <a name="use-workspace-behind-a-firewall-for-azure-machine-learning"></a>Использование рабочей области за брандмауэром для Машинное обучение Azure

Из этой статьи вы узнаете, как настроить брандмауэр Azure для управления доступом к рабочей области Машинное обучение Azure и к общедоступному Интернету. Дополнительные сведения о защите Машинное обучение Azure см. в разделе [Корпоративная безопасность для машинное обучение Azure](concept-enterprise-security.md).

> [!WARNING]
> Доступ к хранилищу данных за брандмауэром поддерживается только в первом интерфейсе кода. Использование [машинное обучение Azure Studio](overview-what-is-machine-learning-studio.md) для доступа к данным за брандмауэром не поддерживается. Для работы с хранилищем данных в частной сети с помощью студии необходимо сначала [настроить виртуальную сеть](../virtual-network/quick-create-portal.md) и [предоставить среде доступ к данным, хранящимся в виртуальной сети](how-to-enable-studio-virtual-network.md).

## <a name="azure-firewall"></a>Брандмауэр Azure

При использовании брандмауэра Azure используйте __Преобразование сетевых адресов назначения (ДНАТ)__ для создания правил NAT для входящего трафика. Для исходящего трафика создайте правила __сети__ или __приложения__ . Эти коллекции правил более подробно описаны в разделе [что такое некоторые понятия брандмауэра Azure](../firewall/firewall-faq.yml#what-are-some-azure-firewall-concepts).

### <a name="inbound-configuration"></a>Конфигурация входящего трафика

Если вы используете Машинное обучение Azure __вычислительный экземпляр__ или __вычислительный кластер__, добавьте [определяемые пользователем маршруты (определяемые пользователем маршруты)](../virtual-network/virtual-networks-udr-overview.md) для подсети, содержащей ресурсы машинное обучение Azure. Этот маршрут направляет трафик __с__ IP-адресов `BatchNodeManagement` `AzureMachineLearning` ресурсов и на общедоступный IP-адрес вычислительного экземпляра и вычислительного кластера.

UDR позволяют пакетной службе взаимодействовать с вычислительными узлами с целью планирования задач. Также добавьте IP-адрес для службы Машинное обучение Azure, так как это требуется для доступа к экземплярам вычислений. При добавлении IP-адреса для службы Машинное обучение Azure необходимо добавить IP-адрес как для __основного, так и для дополнительного__ региона Azure. Основной регион, в котором находится рабочая область.

Чтобы найти дополнительный регион, см. статью [обеспечение непрерывности бизнес-процессов & аварийное восстановление с помощью парных регионов Azure](../best-practices-availability-paired-regions.md#azure-regional-pairs). Например, если служба Машинное обучение Azure находится в восточной части США 2, дополнительный регион — Центральная часть США. 

Чтобы получить список IP-адресов пакетной службы и службы Машинного обучения Azure, используйте один из следующих методов.

* Скачайте [диапазоны IP-адресов Azure и теги службы](https://www.microsoft.com/download/details.aspx?id=56519) и найдите в файле `BatchNodeManagement.<region>` и `AzureMachineLearning.<region>`, где `<region>` — ваш регион Azure.

* Для загрузки информации используйте [интерфейс командной строки Azure](/cli/azure/install-azure-cli). В следующем примере загружаются сведения об IP-адресе и отфильтровываются сведения о регионе "Восточная часть США 2" (основной) и центральном регионе США (дополнительный):

    ```azurecli-interactive
    az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'Batch')] | [?properties.region=='eastus2']"
    # Get primary region IPs
    az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'AzureMachineLearning')] | [?properties.region=='eastus2']"
    # Get secondary region IPs
    az network list-service-tags -l "Central US" --query "values[?starts_with(id, 'AzureMachineLearning')] | [?properties.region=='centralus']"
    ```

    > [!TIP]
    > Если вы используете регионы US-Виргиния, US-Arizona regions или Китае-Восток-2, эти команды не возвращают IP-адресов. Вместо этого воспользуйтесь одной из следующих ссылок, чтобы скачать список IP-адресов:
    >
    > * [Диапазоны IP-адресов и теги службы Azure для Azure для государственных организаций](https://www.microsoft.com/download/details.aspx?id=57063)
    > * [Диапазоны IP-адресов и теги службы Azure для Китая в Azure](https://www.microsoft.com//download/details.aspx?id=57062)

При добавлении определяемого пользователем маршрута укажите маршрут для каждого связанного префикса IP-адреса пакетной службы, а также задайте для параметра __Тип следующего прыжка__ значение __Интернет__. На следующем изображении показан пример этого UDR на портале Azure.

![Пример UDR для префикса адреса](./media/how-to-enable-virtual-network/user-defined-route.png)

> [!IMPORTANT]
> С течением времени IP-адреса могут меняться.

См. дополнительные сведения в разделе [Создание пула пакетной службы Azure в виртуальной сети](../batch/batch-virtual-network.md#user-defined-routes-for-forced-tunneling).

### <a name="outbound-configuration"></a>Конфигурация исходящего трафика

1. Добавьте __правила сети__, разрешив __входящий и__ исходящий трафик __из__ следующих тегов службы:

    * AzureActiveDirectory
    * AzureMachineLearning
    * AzureResourceManager
    * Хранилище. регион
    * KeyVault. регион
    * ContainerRegistry. регион

    Если вы планируете использовать образы DOCKER по умолчанию, предоставляемые корпорацией Майкрософт, и включаете управляемые пользователем зависимости, необходимо также добавить следующие теги служб:

    * Микрософтконтаинеррегистри. регион
    * AzureFrontDoor.FirstParty

    Для записей, содержащих `region` , замените используемым регионом Azure. Например, `keyvault.westus`.

    В качестве __протокола__ выберите `TCP` . Для исходных и целевых __портов__ выберите `*` .

1. Добавьте __правила приложения__ для следующих узлов:

    > [!NOTE]
    > Это не полный список узлов, необходимых для всех ресурсов Python в Интернете, наиболее часто используемый. Например, если необходим доступ к репозиторию GitHub или другому узлу, необходимо указать и добавить необходимые узлы для этого сценария.

    | **Имя узла** | **Назначение** |
    | ---- | ---- |
    | **graph.windows.net** | Используется Машинное обучение Azure вычислительным экземпляром или кластером. |
    | **anaconda.com**</br>**\*. anaconda.com** | Используется для установки пакетов по умолчанию. |
    | **\*. anaconda.org** | Используется для получения данных репозитория. |
    | **pypi.org** | Используется для перечисления зависимостей из индекса по умолчанию, если таковые имеются, а индекс не перезаписывается параметрами пользователя. Если индекс перезаписывается, необходимо также разрешить **\* . pythonhosted.org**. |
    | **cloud.r-project.org** | Используется при установке пакетов CRAN для разработки R. |
    | **\*pytorch.org** | Используется в некоторых примерах на основе PyTorch. |
    | **\*. tensorflow.org** | Используется в некоторых примерах на основе Tensorflow. |

    В качестве __протокола: порт__ выберите использовать __http, HTTPS__.

    Дополнительные сведения о настройке правил приложений см. в статье [развертывание и настройка брандмауэра Azure](../firewall/tutorial-firewall-deploy-portal.md#configure-an-application-rule).

1. Чтобы ограничить доступ к моделям, развернутым в службе Azure Kubernetes Service (AKS), см. раздел [ограничение трафика исходящих данных в службе Azure Kubernetes](../aks/limit-egress-traffic.md).

## <a name="other-firewalls"></a>Другие брандмауэры

Рекомендации в этом разделе являются универсальными, так как каждый брандмауэр имеет собственную терминологию и конкретные конфигурации. Если у вас есть вопросы о том, как разрешить обмен данными через брандмауэр, обратитесь к документации по используемому брандмауэру.

Если настройка не настроена должным образом, брандмауэр может вызвать проблемы с использованием рабочей области. Существует множество имен узлов, которые используются в рабочей области Машинное обучение Azure. В следующих разделах перечислены узлы, необходимые для Машинное обучение Azure.

### <a name="microsoft-hosts"></a>Узлы Майкрософт

Узлы в этом разделе принадлежат корпорации Майкрософт и предоставляют службы, необходимые для нормальной работы рабочей области. В следующих таблицах перечислены имена узлов для регионов Azure для общего пользования, Azure для государственных организаций и Azure для Китая.

**Общие узлы Azure**

| **Обязательно для:** | **общедоступный пиринг Azure;** | **Azure для государственных организаций** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Azure Active Directory | login.microsoftonline.com | login.microsoftonline.us | login.chinacloudapi.cn |
| Портал Azure | management.azure.com | management.azure.us | management.azure.cn |
| Azure Resource Manager | management.azure.com | management.usgovcloudapi.net | management.chinacloudapi.cn |

**Узлы Машинное обучение Azure**

| **Обязательно для:** | **общедоступный пиринг Azure;** | **Azure для государственных организаций** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Студия машинного обучения Azure. | ml.azure.com | ml.azure.us | studio.ml.azure.cn |
| API |\*. azureml.ms | \*. ml.azure.us | \*. ml.azure.cn |
| Эксперименты, журнал, устройство, метки | \*. experiments.azureml.net | \*. ml.azure.us | \*. ml.azure.cn |
| управление моделью; | \*. modelmanagement.azureml.net | \*. ml.azure.us | \*. ml.azure.cn |
| Конвейер | \*. aether.ms | \*. ml.azure.us | \*. ml.azure.cn |
| Конструктор (служба Studio) | \*. studioservice.azureml.com | \*. ml.azure.us | \*. ml.azure.cn |
| Интегрированная Записная книжка | \*. notebooks.azure.net | \*. notebooks.usgovcloudapi.net |\*. notebooks.chinacloudapi.cn |
| Интегрированная Записная книжка | \*. file.core.windows.net | \*. file.core.usgovcloudapi.net | \*. file.core.chinacloudapi.cn |
| Интегрированная Записная книжка | \*.dfs.core.windows.net | \*. dfs.core.usgovcloudapi.net | \*. dfs.core.chinacloudapi.cn |
| Интегрированная Записная книжка | \*.blob.core.windows.net | \*.blob.core.usgovcloudapi.net | \*. blob.core.chinacloudapi.cn |
| Интегрированная Записная книжка | graph.microsoft.com | graph.microsoft.us | graph.chinacloudapi.cn |
| Интегрированная Записная книжка | \*. aznbcontent.net |  | |

**Машинное обучение Azure вычислительных экземпляров и узлов вычислительных кластеров**

| **Обязательно для:** | **общедоступный пиринг Azure;** | **Azure для государственных организаций** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Вычислительный кластер/экземпляр | \*. batchai.core.windows.net | \*. batchai.core.usgovcloudapi.net |\*. batchai.ml.azure.cn |
| Вычислительный кластер/экземпляр | graph.windows.net | graph.windows.net | graph.chinacloudapi.cn |
| Вычислительная операция | \*. instances.azureml.net | \*. instances.azureml.us | \*. instances.azureml.cn |
| Вычислительная операция | \*. instances.azureml.ms |  |  |

**Связанные ресурсы, используемые Машинное обучение Azure**

| **Обязательно для:** | **общедоступный пиринг Azure;** | **Azure для государственных организаций** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Учетная запись хранения Azure | core.windows.net | core.usgovcloudapi.net | core.chinacloudapi.cn |
| Azure Key Vault | vault.azure.net | vault.usgovcloudapi.net | vault.azure.cn |
| Реестр контейнеров Azure | azurecr.io | azurecr.us | azurecr.cn |
| Реестр контейнеров Майкрософт | mcr.microsoft.com | mcr.microsoft.com | mcr.microsoft.com |


> [!TIP]
> Если вы планируете использовать федеративное удостоверение, следуйте рекомендациям [по обеспечению безопасности службы федерации Active Directory (AD FS)](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs) статье.

Кроме того, используйте сведения в [принудительном туннелировании](how-to-secure-training-vnet.md#forced-tunneling) , чтобы добавить IP-адреса для `BatchNodeManagement` и `AzureMachineLearning` .

Сведения об ограничении доступа к моделям, развернутым в службе Azure Kubernetes Service (AKS), см. [в статье ограничение трафика исходящих данных в службе Azure Kubernetes](../aks/limit-egress-traffic.md).

### <a name="python-hosts"></a>Узлы Python

Узлы в этом разделе используются для установки пакетов Python. Они необходимы во время разработки, обучения и развертывания. 

> [!NOTE]
> Это не полный список узлов, необходимых для всех ресурсов Python в Интернете, наиболее часто используемый. Например, если необходим доступ к репозиторию GitHub или другому узлу, необходимо указать и добавить необходимые узлы для этого сценария.

| **Имя узла** | **Назначение** |
| ---- | ---- |
| **anaconda.com**</br>**\*. anaconda.com** | Используется для установки пакетов по умолчанию. |
| **\*. anaconda.org** | Используется для получения данных репозитория. |
| **pypi.org** | Используется для перечисления зависимостей из индекса по умолчанию, если таковые имеются, а индекс не перезаписывается параметрами пользователя. Если индекс перезаписывается, необходимо также разрешить **\* . pythonhosted.org**. |
| **\*pytorch.org** | Используется в некоторых примерах на основе PyTorch. |
| **\*. tensorflow.org** | Используется в некоторых примерах на основе Tensorflow. |

### <a name="r-hosts"></a>Узлы R

Узлы в этом разделе используются для установки пакетов R. Они необходимы во время разработки, обучения и развертывания.

> [!NOTE]
> Это не полный список узлов, необходимых для всех ресурсов R в Интернете, наиболее часто используемый. Например, если необходим доступ к репозиторию GitHub или другому узлу, необходимо указать и добавить необходимые узлы для этого сценария.

| **Имя узла** | **Назначение** |
| ---- | ---- |
| **cloud.r-project.org** | Используется при установке пакетов CRAN. |

> [!IMPORTANT]
> На внутреннем уровне пакет SDK R для Машинное обучение Azure использует пакеты Python. Поэтому необходимо также разрешить узлы Python через брандмауэр.
## <a name="next-steps"></a>Дальнейшие действия

* [Руководство. развертыванию и настройке службы "Брандмауэр Azure" с помощью портала Azure](../firewall/tutorial-firewall-deploy-portal.md)
* [Защита заданий экспериментирования и вывода Машинного обучения Azure в виртуальной сети Azure](how-to-network-security-overview.md)
