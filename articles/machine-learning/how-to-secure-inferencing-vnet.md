---
title: Защита сред с помощью виртуальных сетей
titleSuffix: Azure Machine Learning
description: Используйте изолированную виртуальную сеть Azure, чтобы защитить среду Машинное обучение Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: peterlu
author: peterclu
ms.date: 10/23/2020
ms.custom: contperf-fy20q4, tracking-python, contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: 1a1a9158c06a12caaeb5702f2fdf7da3c801c143
ms.sourcegitcommit: 87a6587e1a0e242c2cfbbc51103e19ec47b49910
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103573444"
---
# <a name="secure-an-azure-machine-learning-inferencing-environment-with-virtual-networks"></a>Обеспечение безопасности Машинное обучение Azure окружения с помощью виртуальных сетей

Из этой статьи вы узнаете, как защитить окружения с помощью виртуальной сети в Машинное обучение Azure.

Эта статья состоит из четырех статей, посвященных обеспечению безопасности Машинное обучение Azure рабочего процесса. Мы настоятельно рекомендуем прочесть первую [часть: обзор виртуальной сети](how-to-network-security-overview.md) , чтобы понять общую архитектуру. 

См. Другие статьи в этой серии:

[1. Общие сведения о виртуальной сети](how-to-network-security-overview.md)  >  [Обеспечьте безопасность рабочей области](how-to-secure-workspace-vnet.md)  >  [3. Обеспечьте безопасность среды обучения](how-to-secure-training-vnet.md)  >  **4. Обеспечьте безопасность окружения**  >  [5. Включить функциональные возможности студии](how-to-enable-studio-virtual-network.md)

В этой статье вы узнаете, как защитить следующие ресурсы в виртуальной сети.
> [!div class="checklist"]
> - Кластер Azure Kubernetes Service (AKS) по умолчанию
> - Частный кластер AKS
> - Кластер AKS с закрытой ссылкой
> - Экземпляры контейнеров Azure (ACI)

## <a name="prerequisites"></a>Предварительные требования

+ Ознакомьтесь со статьей [Обзор сетевой безопасности](how-to-network-security-overview.md) , чтобы ознакомиться с общими сценариями виртуальной сети и общей архитектурой виртуальной сети.

+ Существующая виртуальная сеть и подсеть для использования с ресурсами вычислений.

+ Чтобы развернуть ресурсы в виртуальной сети или подсети, учетная запись пользователя должна иметь разрешения на доступ к следующим действиям в управлении доступом на основе ролей в Azure (Azure RBAC):

    - "Microsoft. Network/virtualNetworks/соединение/действие" для ресурса виртуальной сети.
    - "Microsoft. Network/virtualNetworks/подсеть/соединение/действие" в ресурсе подсети.

    Дополнительные сведения об Azure RBAC с сетью см. в разделе [встроенные сетевые роли](../role-based-access-control/built-in-roles.md#networking) .

<a id="aksvnet"></a>

## <a name="azure-kubernetes-service"></a>Служба Azure Kubernetes

Чтобы использовать кластер AKS в виртуальной сети, необходимо соблюдать следующие требования к сети.

> [!div class="checklist"]
> * Выполните необходимые условия в статье [Настройка расширенной сети в службе Azure Kubernetes (AKS)](../aks/configure-azure-cni.md#prerequisites).
> * Экземпляр AKS и виртуальная сеть должны находиться в одном регионе. Если вы защищаете учетные записи хранения Azure, используемые рабочей областью в виртуальной сети, они должны быть в той же виртуальной сети, что и экземпляр AKS.

Чтобы добавить AKS в виртуальную сеть в рабочую область, выполните следующие действия.

1. Войдите в [студию Машинного обучения Azure](https://ml.azure.com/), выберите свою подписку и рабочую область.

1. Слева выберите пункт __Вычисление__.

1. Выберите __Кластеры аналитики__ в центре, а затем — __+__ .

1. В диалоговом окне __Создание кластера аналитики__ выберите __Дополнительно__ в разделе __Конфигурация сети__.

1. Чтобы настроить этот виртуальный ресурс для использования виртуальной сети, выполните следующие действия.

    1. В раскрывающемся списке __Группа ресурсов__ выберите группу ресурсов, которая содержит виртуальную сеть.
    1. В раскрывающемся списке __Виртуальная сеть__ выберите виртуальную сеть, которая содержит подсеть.
    1. В раскрывающемся списке __Подсеть__ выберите подсеть.
    1. В поле __Диапазон адресов службы Kubernetes__ введите диапазон адресов службы Kubernetes. Этот диапазон адресов использует диапазон IP-адресов нотации CIDR для определения IP-адресов, которые доступны этому кластеру. Он не должен пересекаться с диапазонами IP-адресов подсети (например, 10.0.0.0/16).
    1. В поле __IP-адрес службы DNS Kubernetes__ введите IP-адрес службы DNS Kubernetes. Это IP-адрес, назначенный службе DNS Kubernetes. Он должен находиться в пределах диапазона адресов службы Kubernetes (например, 10.0.0.10).
    1. В поле __Адрес моста Docker__ введите адрес моста Docker. Этот IP-адрес, назначенный мосту Docker. Он не должен пересекаться с диапазоном IP-адресов, выделенным для любой подсети или службы Kubernetes (например, 172.17.0.1/16).

   ![Машинное обучение Azure: Параметры виртуальной сети для Вычислительной среды Машинного обучения](./media/how-to-enable-virtual-network/aks-virtual-network-screen.png)

1. При развертывании модели в качестве веб-службы для AKS создается конечная точка оценки для обработки запросов, запрашивающих запросы. Убедитесь, что в группе NSG, управляющей виртуальной сетью, включено правило безопасности для входящего трафика для IP-адреса конечной точки оценки, если вы хотите вызвать ее извне из виртуальной сети.

    Чтобы найти IP-адрес конечной точки оценки, просмотрите код URI оценки для развернутой службы. Сведения о просмотре URI оценки см. в разделе [Использование модели, развернутой в качестве веб-службы](how-to-consume-web-service.md#connection-information).

   > [!IMPORTANT]
   > Сохраните правила по умолчанию для исходящего трафика для группы безопасности сети. Дополнительные сведения см. в описании стандартных правил безопасности в статье [о группах безопасности](../virtual-network/network-security-groups-overview.md#default-security-rules).

   [![Правило безопасности для входящего трафика](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png)](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png#lightbox)

    > [!IMPORTANT]
    > IP-адрес, показанный в образе для конечной точки оценки, будет отличаться для ваших развертываний. Хотя один и тот же IP-адрес совместно используется всеми развертываниями в одном кластере AKS, каждый кластер AKS будет иметь разные IP-адреса.

Вы также можете использовать пакет SDK для Машинного обучения Azure, чтобы добавить в виртуальную сеть службу Azure Kubernetes. Если в вашей виртуальной сети уже есть кластер AKS, можно подключить его к рабочей области, следуя инструкциям в разделе [Развертывание в AKS](how-to-deploy-and-where.md). Следующий код создает новый экземпляр AKS в подсети `default` виртуальной сети с именем `mynetwork`.

```python
from azureml.core.compute import ComputeTarget, AksCompute

# Create the compute configuration and set virtual network information
config = AksCompute.provisioning_configuration(location="eastus2")
config.vnet_resourcegroup_name = "mygroup"
config.vnet_name = "mynetwork"
config.subnet_name = "default"
config.service_cidr = "10.0.0.0/16"
config.dns_service_ip = "10.0.0.10"
config.docker_bridge_cidr = "172.17.0.1/16"

# Create the compute target
aks_target = ComputeTarget.create(workspace=ws,
                                  name="myaks",
                                  provisioning_configuration=config)
```

Когда завершится процесс создания, вы сможете выполнять анализ или оценку моделей в кластере AKS за виртуальной сетью. Дополнительные сведения см. в статье [о развертывании в Службе Azure Kubernetes](how-to-deploy-and-where.md).

Дополнительные сведения об использовании Role-Based управления доступом с помощью Kubernetes см. в статье [Использование Azure RBAC для авторизации Kubernetes](../aks/manage-azure-rbac.md).

## <a name="network-contributor-role"></a>Роль "участник сети"

> [!IMPORTANT]
> Если вы создаете или подключаете кластер AKS, предоставив ранее созданную виртуальную сеть, необходимо предоставить субъекту-службе (SP) или управляемому удостоверению для кластера AKS роль " _участник сети_ " для группы ресурсов, содержащей виртуальную сеть.
>
> Чтобы добавить удостоверение в качестве участника сети, выполните следующие действия.

1. Чтобы найти субъект-службу или идентификатор управляемого удостоверения для AKS, используйте следующие Azure CLI команды. Замените `<aks-cluster-name>` именем кластера. Замените `<resource-group-name>` именем группы ресурсов, которая _содержит кластер AKS_:

    ```azurecli-interactive
    az aks show -n <aks-cluster-name> --resource-group <resource-group-name> --query servicePrincipalProfile.clientId
    ``` 

    Если эта команда возвращает значение `msi` , используйте следующую команду, чтобы указать идентификатор субъекта для управляемого удостоверения:

    ```azurecli-interactive
    az aks show -n <aks-cluster-name> --resource-group <resource-group-name> --query identity.principalId
    ```

1. Чтобы узнать идентификатор группы ресурсов, содержащей виртуальную сеть, используйте следующую команду. Замените `<resource-group-name>` именем группы ресурсов, _содержащей виртуальную сеть_.

    ```azurecli-interactive
    az group show -n <resource-group-name> --query id
    ```

1. Чтобы добавить субъект-службу или управляемое удостоверение в качестве участника сети, используйте следующую команду. Замените `<SP-or-managed-identity>` идентификатором, возвращенным для субъекта-службы или управляемого удостоверения. Замените `<resource-group-id>` идентификатором, возвращенным для группы ресурсов, содержащей виртуальную сеть.

    ```azurecli-interactive
    az role assignment create --assignee <SP-or-managed-identity> --role 'Network Contributor' --scope <resource-group-id>
    ```
Дополнительные сведения об использовании внутреннего балансировщика нагрузки с AKS см. в разделе [Использование внутреннего балансировщика нагрузки со службой Azure Kubernetes](../aks/internal-lb.md).

## <a name="secure-vnet-traffic"></a>Безопасный трафик виртуальной сети

Существует два подхода к изоляции трафика от кластера AKS к виртуальной сети.

* __Частный кластер AKS__. Этот подход использует частную связь Azure для защиты обмена данными с кластером для операций развертывания и управления.
* __Внутренняя подсистема балансировки нагрузки AKS__. Этот подход настраивает конечную точку для развертываний в AKS для использования частного IP-адреса в виртуальной сети.

> [!WARNING]
> Внутренняя подсистема балансировки нагрузки не работает с кластером AKS, использующим кубенет. Если вы хотите одновременно использовать внутреннюю подсистему балансировки нагрузки и частный кластер AKS, настройте частный кластер AKS с сетевым интерфейсом контейнера Azure (CNI). Дополнительные сведения см. [в статье configure Azure CNI Networking in Azure Kubernetes Service](../aks/configure-azure-cni.md).

### <a name="private-aks-cluster"></a>Частный кластер AKS

По умолчанию кластеры AKS имеют плоскость управления или сервер API с общедоступными IP-адресами. Вы можете настроить AKS для использования закрытой плоскости управления, создав частный кластер AKS. Дополнительные сведения см. [в статье Создание частного кластера службы Kubernetes Azure](../aks/private-clusters.md).

После создания частного кластера AKS [Подключите кластер к виртуальной сети](how-to-create-attach-kubernetes.md) для использования с машинное обучение Azure.

> [!IMPORTANT]
> Прежде чем использовать AKS кластер с включенной частной связью с Машинное обучение Azure, необходимо открыть обращение в службу поддержки, чтобы включить эту функцию. Дополнительные сведения см. в статье [Управление квотами и их увеличение](how-to-manage-quotas.md#private-endpoint-and-private-dns-quota-increases).

### <a name="internal-aks-load-balancer"></a>Внутренняя подсистема балансировки нагрузки AKS

По умолчанию в развертываниях AKS используется [общедоступная подсистема балансировки нагрузки](../aks/load-balancer-standard.md). В этом разделе вы узнаете, как настроить AKS для использования внутренней подсистемы балансировки нагрузки. Используется внутренняя подсистема балансировки нагрузки (или частная), где только частные IP-адреса разрешены в качестве внешнего интерфейса. Внутренние подсистемы балансировки нагрузки используются для балансировки нагрузки трафика внутри виртуальной сети.

Частный балансировщик нагрузки включается путем настройки AKS для использования _внутренней подсистемы балансировки нагрузки_. 

#### <a name="enable-private-load-balancer"></a>Включить частную подсистему балансировки нагрузки

> [!IMPORTANT]
> Вы не можете включить частный IP-адрес при создании кластера службы Kubernetes Azure в Машинное обучение Azure Studio. Вы можете создать его с внутренней подсистемой балансировки нагрузки при использовании пакета SDK для Python или расширения Azure CLI для машинного обучения.

В следующих примерах показано, как __создать новый кластер AKS с частным IP-или внутренней подсистемой балансировки нагрузки__ с помощью пакета SDK и CLI:

# <a name="python"></a>[Python](#tab/python)

```python
import azureml.core
from azureml.core.compute import AksCompute, ComputeTarget

# Verify that cluster does not exist already
try:
    aks_target = AksCompute(workspace=ws, name=aks_cluster_name)
    print("Found existing aks cluster")

except:
    print("Creating new aks cluster")

    # Subnet to use for AKS
    subnet_name = "default"
    # Create AKS configuration
    prov_config=AksCompute.provisioning_configuration(load_balancer_type="InternalLoadBalancer")
    # Set info for existing virtual network to create the cluster in
    prov_config.vnet_resourcegroup_name = "myvnetresourcegroup"
    prov_config.vnet_name = "myvnetname"
    prov_config.service_cidr = "10.0.0.0/16"
    prov_config.dns_service_ip = "10.0.0.10"
    prov_config.subnet_name = subnet_name
    prov_config.load_balancer_subnet = subnet_name
    prov_config.docker_bridge_cidr = "172.17.0.1/16"

    # Create compute target
    aks_target = ComputeTarget.create(workspace = ws, name = "myaks", provisioning_configuration = prov_config)
    # Wait for the operation to complete
    aks_target.wait_for_completion(show_output = True)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az ml computetarget create aks -n myaks --load-balancer-type InternalLoadBalancer
```

> [!IMPORTANT]
> С помощью интерфейса командной строки можно создать только кластер AKS с внутренней подсистемой балансировки нагрузки. Не существует команды AZ ml для обновления существующего кластера для использования внутренней подсистемы балансировки нагрузки.

Дополнительные сведения см. в справке по команде [AZ ML computetarget Create AKS](/cli/azure/ext/azure-cli-ml/ml/computetarget/create#ext-azure-cli-ml-az-ml-computetarget-create-aks) .

---

При __присоединении существующего кластера__ к рабочей области необходимо подождать, пока не будет выполнена операция присоединения для настройки балансировщика нагрузки. Сведения о присоединении кластера см. в разделе [Подключение существующего кластера AKS](how-to-create-attach-kubernetes.md).

После присоединения существующего кластера можно обновить кластер, чтобы использовать внутренний балансировщик нагрузки или частный IP-адрес:

```python
import azureml.core
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute import AksCompute

# ws = workspace object. Creation not shown in this snippet
aks_target = AksCompute(ws,"myaks")

# Change to the name of the subnet that contains AKS
subnet_name = "default"
# Update AKS configuration to use an internal load balancer
update_config = AksUpdateConfiguration(None, "InternalLoadBalancer", subnet_name)
aks_target.update(update_config)
# Wait for the operation to complete
aks_target.wait_for_completion(show_output = True)
```

## <a name="enable-azure-container-instances-aci"></a>Включение службы "экземпляры контейнеров Azure" (ACI)

Экземпляры контейнеров Azure динамически создаются при развертывании модели. Чтобы Машинное обучение Azure могло создать ACI в виртуальной сети, необходимо включить __делегирование подсети__ для подсети, используемой в развертывании.

> [!WARNING]
> При использовании экземпляров контейнеров Azure в виртуальной сети виртуальная сеть должна быть:
> * В той же группе ресурсов, что и Рабочая область Машинное обучение Azure.
> * Если в рабочей области есть __Частная конечная точка__, то виртуальная сеть, используемая для экземпляров контейнеров Azure, должна совпадать с той, которая используется в частной конечной точке рабочей области.
>
> При использовании экземпляров контейнеров Azure в виртуальной сети реестр контейнеров Azure (запись контроля доступа) для рабочей области не может находиться в виртуальной сети.

Чтобы использовать ACI в виртуальной сети своей рабочей области, выполните следующие действия.

1. Чтобы включить делегирование подсети в виртуальной сети, используйте сведения, приведенные в статье [Добавление или удаление делегирования подсети](../virtual-network/manage-subnet-delegation.md). Делегирование можно включить при создании виртуальной сети или добавить в существующую сеть.

    > [!IMPORTANT]
    > При включении делегирования используйте `Microsoft.ContainerInstance/containerGroups` в качестве значения __Делегировать подсеть службе__.

2. Разверните модель с помощью [AciWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none--vnet-name-none--subnet-name-none-), используйте параметры `vnet_name` и `subnet_name`. Задайте для этих параметров имя виртуальной сети и подсеть, в которой включено делегирование.

## <a name="limit-outbound-connectivity-from-the-virtual-network"></a>Ограничение исходящих подключений из виртуальной сети

Если вы не хотите использовать правила исходящего трафика по умолчанию и вы хотите ограничить исходящий доступ к виртуальной сети, необходимо разрешить доступ к реестру контейнеров Azure. Например, убедитесь, что группы безопасности сети (NSG) содержат правило, которое разрешает доступ к тегу службы __азуреконтаинеррегистри. RegionName__ , где "{RegionName}" — это имя региона Azure.

## <a name="next-steps"></a>Дальнейшие действия

Эта статья состоит из четырех частей серии виртуальных сетей из пяти частей. Ознакомьтесь с остальными статьями, чтобы узнать, как защитить виртуальную сеть.

* [Часть 1. Обзор виртуальной сети](how-to-network-security-overview.md)
* [Часть 2. Защита ресурсов рабочей области](how-to-secure-workspace-vnet.md)
* [Часть 3. Защита среды обучения](how-to-secure-training-vnet.md)
* [Часть 5. Включение функций Studio](how-to-enable-studio-virtual-network.md)

См. также статью об использовании [ПОЛЬЗОВАТЕЛЬСКОГО DNS](how-to-custom-dns.md) для разрешения имен.