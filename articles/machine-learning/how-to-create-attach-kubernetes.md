---
title: Создание и присоединение службы Kubernetes Azure
titleSuffix: Azure Machine Learning
description: Узнайте, как создать новый кластер службы Azure Kubernetes с помощью Машинное обучение Azure или подключить существующий кластер AKS к рабочей области.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 03/11/2021
ms.openlocfilehash: bc8f7aa6827ce251799acd0673d43344c0833c3a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103149330"
---
# <a name="create-and-attach-an-azure-kubernetes-service-cluster"></a>Создание и подключение кластера службы Kubernetes Azure

Машинное обучение Azure можете развернуть обученные модели машинного обучения в службе Azure Kubernetes. Однако сначала необходимо __создать__ кластер Azure Kubernetes Service (AKS) из рабочей области машинного обучения Azure или __ПОДКЛЮЧИТЬ__ существующий кластер AKS. В этой статье приводятся сведения о создании кластера и подключении к нему.

## <a name="prerequisites"></a>Предварительные требования

- Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

- [Расширение Azure CLI для службы машинное обучение](reference-azure-machine-learning-cli.md), [машинное обучение Azure пакет SDK для Python](/python/api/overview/azure/ml/intro)или [расширение машинное обучение Azure Visual Studio Code](tutorial-setup-vscode-extension.md).

- Если вы планируете использовать виртуальную сеть Azure для защиты обмена данными между рабочей областью машинного обучения Azure и кластером AKS, прочтите статью [Сетевая изоляция во время обучения &](./how-to-network-security-overview.md) получения информации о выводе.

## <a name="limitations"></a>Ограничения

- Если вам требуется **Load Balancer (цен. Категория "Стандартный") (SLB)** , развернутый в кластере вместо базового Load Balancer (блб), создайте кластер на ПОРТАЛе AKS, CLI или SDK, а затем **Подключите** его к рабочей области AML.

- Если у вас есть политика Azure, которая ограничит создание общедоступных IP-адресов, то создание кластера AKS завершится сбоем. Для AKS требуется общедоступный IP-адрес для [исходящего трафика](../aks/limit-egress-traffic.md). В статье исходящий трафик также приводятся рекомендации по блокированию исходящего трафика из кластера через общедоступный IP-адрес, за исключением нескольких полных доменных имен. Существует два способа включения общедоступного IP-адреса:
    - Кластер может использовать общедоступный IP-адрес, созданный по умолчанию с помощью БЛБ или SLB, или
    - Кластер можно создать без общедоступного IP-адреса, а затем настроить общедоступный IP-адрес брандмауэром с определяемым пользователем маршрутом. Дополнительные сведения см. в статье Настройка исходящего [трафика кластера с помощью определяемого пользователем маршрута](../aks/egress-outboundtype.md).
    
    Плоскость управления AML не взаимодействуют с этим общедоступным IP-адресом. Он обращается к плоскости управления AKS для развертываний. 

- Если вы **подключаете** кластер AKS с [разрешенным диапазоном IP-адресов для доступа к серверу API](../aks/api-server-authorized-ip-ranges.md), включите диапазоны IP-адресов плоскости управления AML для кластера AKS. Плоскость управления AML развертывается в парных регионах и развертывает модули определения вывода в кластере AKS. Без доступа к серверу API невозможно развернуть модули не выводу. Используйте [диапазоны IP-адресов](https://www.microsoft.com/download/confirmation.aspx?id=56519) для [парных регионов](../best-practices-availability-paired-regions.md) при включении диапазонов IP-адресов в кластере AKS.

    IP-диапазоны авторизации работают только с Load Balancer (цен. категория "Стандартный").

- При **присоединении** кластера AKS он должен находиться в той же подписке Azure, что и рабочая область машинное обучение Azure.

- Если вы хотите использовать частный кластер AKS (с помощью частного канала Azure), сначала необходимо создать кластер, а затем **подключить** его к рабочей области. Дополнительные сведения см. [в статье Создание частного кластера службы Kubernetes Azure](../aks/private-clusters.md).

- Имя вычислений для кластера AKS должно быть уникальным в пределах рабочей области машинного обучения Azure.
    - Имя является обязательным и должно иметь длину от 3 до 24 символов.
    - Допустимыми символами являются буквы верхнего и нижнего регистра, цифры и символ-знак.
    - Имя должно начинаться с буквы.
    - Имя должно быть уникальным среди всех существующих вычислений в регионе Azure. Вы увидите оповещение, если выбранное имя не является уникальным.
   
 - Если вы хотите развернуть модели на узлах **GPU** или узлах **FPGA** (или на любом конкретном номере SKU), необходимо создать кластер с конкретным номером SKU. Создание пула вторичных узлов в существующем кластере и развертывание моделей в пуле вторичных узлов не поддерживаются.
 
- При создании или присоединении кластера можно выбрать, следует ли создать кластер для разработки, __тестирования__ или __производства__. Если вы хотите создать кластер AKS для __разработки__, __проверки__ и __тестирования__ вместо рабочей среды, задайте для параметра __назначение кластера__ значение __Разработка и тестирование__. Если назначение кластера не указано, создается __Рабочий__ кластер. 

    > [!IMPORTANT]
    > Кластер для __разработки и тестирования__ не подходит для трафика уровня рабочей среды и может увеличить время вывода. Кластеры для разработки и тестирования также не гарантируют отказоустойчивость.

- При создании или присоединении кластера, если кластер будет использоваться для __рабочей среды__, он должен содержать не менее 12 __виртуальных ЦП__. Количество виртуальных ЦП можно вычислить, умножив __количество узлов__ в кластере на __число ядер__ , предоставляемых выбранным размером виртуальной машины. Например, если вы используете размер виртуальной машины "Standard_D3_v2" с 4 виртуальными ядрами, то в качестве числа узлов следует выбрать значение 3 или больше.

    Для кластера с __поддержкой разработки и тестирования__ мы переcommand по крайней мере два виртуальных ЦП.

- Пакет SDK для Машинное обучение Azure не обеспечивает поддержку масштабирования кластера AKS. Чтобы масштабировать узлы в кластере, используйте пользовательский интерфейс для кластера AKS в Машинное обучение Azure Studio. Можно изменить только число узлов, а не размер виртуальной машины кластера. Дополнительные сведения о масштабировании узлов в кластере AKS см. в следующих статьях:

    - [Ручное масштабирование числа узлов в кластере AKS](../aks/scale-cluster.md)
    - [Настройка автомасштабирования кластера в AKS](../aks/cluster-autoscaler.md)

- __Не обновляйте кластер напрямую с помощью конфигурации YAML__. Хотя службы Azure Kubernetes Services поддерживают обновления через конфигурацию YAML, Машинное обучение Azure развертывания будут переопределять изменения. Только два поля YAML, которые не будут перезаписаны, — это __ограничения запросов__ , а также __ЦП и память__.

## <a name="azure-kubernetes-service-version"></a>Версия Службы Azure Kubernetes (AKS)

Служба Kubernetes Azure позволяет создавать кластеры с помощью различных версий Kubernetes. Дополнительные сведения о доступных версиях см. [в разделе Supported Kubernetes Versions in Azure Kubernetes Service](../aks/supported-kubernetes-versions.md).

При **создании** кластера службы Kubernetes Azure с помощью одного из следующих методов у вас нет *выбора в* создаваемой версии кластера:

* Машинное обучение Azure Studio или раздел Машинное обучение Azure портал Azure.
* Расширение Машинное обучение для Azure CLI.
* Пакет SDK для Машинное обучение Azure.

Эти методы создания кластера AKS используют версию кластера __по умолчанию__ . *Версия по умолчанию меняется со временем* , так как становятся доступны новые версии Kubernetes.

При **присоединении** существующего кластера AKS поддерживаются все версии AKS, поддерживаемые в настоящее время.

> [!NOTE]
> Могут возникнуть граничные случаи, когда у вас более старая версия кластера, которая больше не поддерживается. В этом случае операция присоединения возвратит ошибку и перечислит версии, поддерживаемые в настоящее время.
>
> Вы можете присоединить **Предварительные** версии. Функции предварительной версии предоставляются без соглашения об уровне обслуживания и не рекомендуются для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Поддержка использования предварительных версий может быть ограничена. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="available-and-default-versions"></a>Доступные и версии по умолчанию

Для поиска доступных версий AKS и по умолчанию используйте команду [Azure CLI](/cli/azure/install-azure-cli) [AZ AKS get-versions](/cli/azure/aks#az_aks_get_versions). Например, следующая команда возвращает версии, доступные в регионе "Западная часть США":

```azurecli-interactive
az aks get-versions -l westus -o table
```

После выполнения этой команды вы должны увидеть текст, аналогичный приведенному ниже.

```text
KubernetesVersion    Upgrades
-------------------  ----------------------------------------
1.18.6(preview)      None available
1.18.4(preview)      1.18.6(preview)
1.17.9               1.18.4(preview), 1.18.6(preview)
1.17.7               1.17.9, 1.18.4(preview), 1.18.6(preview)
1.16.13              1.17.7, 1.17.9
1.16.10              1.16.13, 1.17.7, 1.17.9
1.15.12              1.16.10, 1.16.13
1.15.11              1.15.12, 1.16.10, 1.16.13
```

Чтобы найти версию по умолчанию, используемую при **создании** кластера с помощью машинное обучение Azure, можно использовать `--query` параметр, чтобы выбрать версию по умолчанию:

```azurecli-interactive
az aks get-versions -l westus --query "orchestrators[?default == `true`].orchestratorVersion" -o table
```

После выполнения этой команды вы должны увидеть текст, аналогичный приведенному ниже.

```text
Result
--------
1.16.13
```

Если вы хотите **программно проверить доступные версии**, используйте REST APIные в [списке клиентов службы контейнеров](/rest/api/container-service/container%20service%20client/listorchestrators) . Чтобы найти доступные версии, просмотрите записи, в которых `orchestratorType` есть `Kubernetes` . Связанные `orchestrationVersion` записи содержат доступные версии, которые можно **подключить** к рабочей области.

Чтобы найти версию по умолчанию, используемую при **создании** кластера с помощью машинное обучение Azure, найдите запись, где `orchestratorType` имеет значение `Kubernetes` , а `default` — `true` . Связанное `orchestratorVersion` значение является версией по умолчанию. В следующем фрагменте кода JSON показан пример записи:

```json
...
 {
        "orchestratorType": "Kubernetes",
        "orchestratorVersion": "1.16.13",
        "default": true,
        "upgrades": [
          {
            "orchestratorType": "",
            "orchestratorVersion": "1.17.7",
            "isPreview": false
          }
        ]
      },
...
```

## <a name="create-a-new-aks-cluster"></a>Создание кластера AKS.

**Оценка времени**: приблизительно 10 минут.

Создание или присоединение кластера AKS выполняется один раз для рабочей области. Один кластер можно использовать для нескольких развертываний. При удалении кластера или группы ресурсов, содержащей этот кластер, необходимо создать новый кластер в следующий раз, когда потребуется выполнить развертывание. К рабочей области можно подключить несколько кластеров AKS.

В следующем примере показано, как создать новый кластер AKS с помощью пакета SDK и интерфейса командной строки.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (you can also provide parameters to customize this).
# For example, to create a dev/test cluster, use:
# prov_config = AksCompute.provisioning_configuration(cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
prov_config = AksCompute.provisioning_configuration()

# Example configuration to use an existing virtual network
# prov_config.vnet_name = "mynetwork"
# prov_config.vnet_resourcegroup_name = "mygroup"
# prov_config.subnet_name = "default"
# prov_config.service_cidr = "10.0.0.0/16"
# prov_config.dns_service_ip = "10.0.0.10"
# prov_config.docker_bridge_cidr = "172.17.0.1/16"

aks_name = 'myaks'
# Create the cluster
aks_target = ComputeTarget.create(workspace = ws,
                                    name = aks_name,
                                    provisioning_configuration = prov_config)

# Wait for the create process to complete
aks_target.wait_for_completion(show_output = True)
```

Дополнительные сведения о классах, методах и параметрах, используемых в этом примере, см. в следующих справочных документах:

* [Акскомпуте. Клустерпурпосе](/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose)
* [AksCompute.provisioning_configuration](/python/api/azureml-core/azureml.core.compute.akscompute#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [ComputeTarget. Create](/python/api/azureml-core/azureml.core.compute.computetarget#create-workspace--name--provisioning-configuration-)
* [ComputeTarget.wait_for_completion](/python/api/azureml-core/azureml.core.compute.computetarget#wait-for-completion-show-output-false-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az ml computetarget create aks -n myaks
```

Дополнительные сведения см. в справке по команде [AZ ML computetarget Create AKS](/cli/azure/ext/azure-cli-ml/ml/computetarget/create#ext-azure-cli-ml-az-ml-computetarget-create-aks) .

# <a name="portal"></a>[Портал](#tab/azure-portal)

Сведения о создании кластера AKS на портале см. в статье [Создание целевых объектов вычислений в машинное обучение Azure Studio](how-to-create-attach-compute-studio.md#inference-clusters).

---

## <a name="attach-an-existing-aks-cluster"></a>Подключение существующего кластера AKS

**Оценка времени:** Приблизительно 5 минут.

Если у вас уже есть кластер AKS в подписке Azure, его можно использовать в рабочей области.

> [!TIP]
> Существующий кластер AKS может находиться в регионе Azure, отличном от Машинное обучение Azure рабочей области.


> [!WARNING]
> Не создавайте несколько одновременных вложений в одном кластере AKS из рабочей области. Например, можно подключить один кластер AKS к рабочей области, используя два разных имени. Каждое новое вложение приведет к нарушению предыдущих существующих вложений.
>
> Если требуется повторно подключить кластер AKS, например для изменения настроек TLS или другого кластера, необходимо сначала удалить существующее вложение с помощью [акскомпуте. Detach ()](/python/api/azureml-core/azureml.core.compute.akscompute#detach--).

Дополнительные сведения о создании кластера AKS с помощью Azure CLI или портала см. в следующих статьях:

* [Создание кластера AKS с помощью CLI](/cli/azure/aks?bc=%2fazure%2fbread%2ftoc.json&toc=%2fazure%2faks%2fTOC.json#az-aks-create)
* [Создание кластера AKS (портал)](../aks/kubernetes-walkthrough-portal.md)
* [Создание кластера AKS (шаблон ARM в шаблонах быстрого запуска Azure)](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aks-azml-targetcompute)

В следующем примере показано, как подключить существующий кластер AKS к рабочей области.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.compute import AksCompute, ComputeTarget
# Set the resource group that contains the AKS cluster and the cluster name
resource_group = 'myresourcegroup'
cluster_name = 'myexistingcluster'

# Attach the cluster to your workgroup. If the cluster has less than 12 virtual CPUs, use the following instead:
# attach_config = AksCompute.attach_configuration(resource_group = resource_group,
#                                         cluster_name = cluster_name,
#                                         cluster_purpose = AksCompute.ClusterPurpose.DEV_TEST)
attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
aks_target = ComputeTarget.attach(ws, 'myaks', attach_config)

# Wait for the attach process to complete
aks_target.wait_for_completion(show_output = True)
```

Дополнительные сведения о классах, методах и параметрах, используемых в этом примере, см. в следующих справочных документах:

* [AksCompute.attach_configuration ()](/python/api/azureml-core/azureml.core.compute.akscompute#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)
* [Акскомпуте. Клустерпурпосе](/python/api/azureml-core/azureml.core.compute.aks.akscompute.clusterpurpose)
* [Акскомпуте. Attach](/python/api/azureml-core/azureml.core.compute.computetarget#attach-workspace--name--attach-configuration-)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы подключить существующий кластер с помощью интерфейса командной строки, необходимо получить идентификатор ресурса существующего кластера. Чтобы получить это значение, используйте следующую команду. Замените `myexistingcluster` именем кластера AKS. Замените на `myresourcegroup` группу ресурсов, содержащую кластер:

```azurecli
az aks show -n myexistingcluster -g myresourcegroup --query id
```

Эта команда возвращает значение следующего вида:

```text
/subscriptions/{GUID}/resourcegroups/{myresourcegroup}/providers/Microsoft.ContainerService/managedClusters/{myexistingcluster}
```

Чтобы подключить существующий кластер к рабочей области, используйте следующую команду. Замените на `aksresourceid` значение, возвращенное предыдущей командой. Замените `myresourcegroup` группой ресурсов, содержащей рабочую область. Замените `myworkspace` именем рабочей области.

```azurecli
az ml computetarget attach aks -n myaks -i aksresourceid -g myresourcegroup -w myworkspace
```

Дополнительные сведения см. в справочнике по команде [AZ ML computetarget Attach AKS](/cli/azure/ext/azure-cli-ml/ml/computetarget/attach#ext-azure-cli-ml-az-ml-computetarget-attach-aks) .

# <a name="portal"></a>[Портал](#tab/azure-portal)

Сведения о присоединении кластера AKS на портале см. в статье [Создание целевых объектов вычислений в машинное обучение Azure Studio](how-to-create-attach-compute-studio.md#inference-clusters).

---

## <a name="create-or-attach-an-aks-cluster-with-tls-termination"></a>Создание или подключение кластера AKS с завершением TLS
При [создании или присоединении кластера AKS](how-to-create-attach-kubernetes.md)можно включить завершение TLS с помощью объектов конфигурации **[AksCompute.provisioning_configuration ()](/python/api/azureml-core/azureml.core.compute.akscompute#provisioning-configuration-agent-count-none--vm-size-none--ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--location-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--service-cidr-none--dns-service-ip-none--docker-bridge-cidr-none--cluster-purpose-none--load-balancer-type-none--load-balancer-subnet-none-)** и **[AksCompute.attach_configuration ()](/python/api/azureml-core/azureml.core.compute.akscompute#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)** . Оба метода возвращают объект конфигурации, имеющий метод **enable_ssl** , и для включения TLS можно использовать метод **enable_ssl** .

В следующем примере показано, как включить завершение TLS с автоматическим созданием и настройкой сертификата TLS с помощью сертификата Майкрософт.
```python
   from azureml.core.compute import AksCompute, ComputeTarget
   
   # Enable TLS termination when you create an AKS cluster by using provisioning_config object enable_ssl method

   # Leaf domain label generates a name using the formula
   # "<leaf-domain-label>######.<azure-region>.cloudapp.azure.com"
   # where "######" is a random series of characters
   provisioning_config.enable_ssl(leaf_domain_label = "contoso")
   
   # Enable TLS termination when you attach an AKS cluster by using attach_config object enable_ssl method

   # Leaf domain label generates a name using the formula
   # "<leaf-domain-label>######.<azure-region>.cloudapp.azure.com"
   # where "######" is a random series of characters
   attach_config.enable_ssl(leaf_domain_label = "contoso")


```
В следующем примере показано, как включить завершение TLS с пользовательским сертификатом и пользовательским именем домена. При использовании личного домена и сертификата необходимо обновить запись DNS, чтобы она указывала на IP-адрес конечной точки оценки, см. статью [обновление DNS](how-to-secure-web-service.md#update-your-dns) .

```python
   from azureml.core.compute import AksCompute, ComputeTarget

   # Enable TLS termination with custom certificate and custom domain when creating an AKS cluster
   
   provisioning_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    
   # Enable TLS termination with custom certificate and custom domain when attaching an AKS cluster

   attach_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")


```
>[!NOTE]
> Дополнительные сведения о защите развертывания моделей в кластере AKS см. [в статье Использование TLS для защиты веб-службы с помощью машинное обучение Azure](how-to-secure-web-service.md)

## <a name="create-or-attach-an-aks-cluster-to-use-internal-load-balancer-with-private-ip"></a>Создание или подключение кластера AKS для использования внутренних Load Balancer с частным IP-адресом
При создании или присоединении кластера AKS можно настроить использование внутреннего Load Balancer в кластере. При внутреннем Load Balancer конечные точки оценки для развертываний в AKS будут использовать частный IP-адрес в виртуальной сети. В следующих фрагментах кода показано, как настроить внутреннее Load Balancer для кластера AKS.
```python
   
   from azureml.core.compute.aks import AksUpdateConfiguration
   from azureml.core.compute import AksCompute, ComputeTarget
   
   # When you create an AKS cluster, you can specify Internal Load Balancer to be created with provisioning_config object
   provisioning_config = AksCompute.provisioning_configuration(load_balancer_type = 'InternalLoadBalancer')

   # when you attach an AKS cluster, you can update the cluster to use internal load balancer after attach
   aks_target = AksCompute(ws,"myaks")

   # Change to the name of the subnet that contains AKS
   subnet_name = "default"
   # Update AKS configuration to use an internal load balancer
   update_config = AksUpdateConfiguration(None, "InternalLoadBalancer", subnet_name)
   aks_target.update(update_config)
   # Wait for the operation to complete
   aks_target.wait_for_completion(show_output = True)
   
   
```
>[!IMPORTANT]
> Машинное обучение Azure не поддерживает завершение TLS с внутренними Load Balancer. Внутренний Load Balancer имеет частный IP-адрес, а частный IP-адрес может быть в другой сети, и сертификат может быть рекусед. 

>[!NOTE]
> Дополнительные сведения о том, как обеспечить безопасность окружения, см. в статье [защита машинное обучение Azure в окружении](how-to-secure-inferencing-vnet.md)

## <a name="detach-an-aks-cluster"></a>Отсоединение кластера AKS

Чтобы отключить кластер из рабочей области, используйте один из следующих методов.

> [!WARNING]
> Использование Машинное обучение Azure Studio, пакета SDK или расширения Azure CLI для отключения кластера AKS не **приводит к удалению кластера AKS**. Сведения об удалении кластера см. [в разделе использование Azure CLI с AKS](../aks/kubernetes-walkthrough.md#delete-the-cluster).

# <a name="python"></a>[Python](#tab/python)

```python
aks_target.detach()
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы отсоединить существующий кластер от рабочей области, используйте следующую команду. Замените `myaks` именем, к которому присоединен кластер AKS в рабочей области. Замените `myresourcegroup` группой ресурсов, содержащей рабочую область. Замените `myworkspace` именем рабочей области.

```azurecli
az ml computetarget detach -n myaks -g myresourcegroup -w myworkspace
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

В Машинное обучение Azure Studio выберите __вычисления__, __кластеры вывода__ и кластер, который нужно удалить. Используйте ссылку __отсоединить__ , чтобы отсоединить кластер.

---

## <a name="troubleshooting"></a>Устранение неполадок
### <a name="update-the-cluster"></a>Обновление кластера

Обновления компонентов Машинное обучение Azure, установленных в кластере службы Azure Kubernetes, необходимо применять вручную. 

Эти обновления можно применить, отсоединив кластер от рабочей области Машинное обучение Azure, а затем повторно присоединив кластер к рабочей области. Если в кластере включен протокол TLS, то при повторном присоединении кластера необходимо предоставить сертификат TLS/SSL и закрытый ключ. 

```python
compute_target = ComputeTarget(workspace=ws, name=clusterWorkspaceName)
compute_target.detach()
compute_target.wait_for_completion(show_output=True)

attach_config = AksCompute.attach_configuration(resource_group=resourceGroup, cluster_name=kubernetesClusterName)

## If SSL is enabled.
attach_config.enable_ssl(
    ssl_cert_pem_file="cert.pem",
    ssl_key_pem_file="key.pem",
    ssl_cname=sslCname)

attach_config.validate_configuration()

compute_target = ComputeTarget.attach(workspace=ws, name=args.clusterWorkspaceName, attach_configuration=attach_config)
compute_target.wait_for_completion(show_output=True)
```

Если у вас больше нет сертификата TLS/SSL и закрытого ключа или вы используете сертификат, созданный Машинное обучение Azure, вы можете получить файлы до отключения кластера, подключившись к кластеру с помощью `kubectl` и извлекая секретный код `azuremlfessl` .

```bash
kubectl get secret/azuremlfessl -o yaml
```

>[!Note]
>Kubernetes сохраняет секреты в формате в кодировке Base-64. Необходимо сначала декодировать компоненты и для этих секретов в Base-64, `cert.pem` `key.pem` прежде чем предоставлять их в `attach_config.enable_ssl` . 

### <a name="webservice-failures"></a>Сбои WebService

Многие сбои WebService в AKS можно отладить, подключившись к кластеру с помощью `kubectl` . Вы можете получить `kubeconfig.json` для кластера AKS, выполнив

```azurecli-interactive
az aks get-credentials -g <rg> -n <aks cluster name>
```

## <a name="next-steps"></a>Следующие шаги

* [Использование Azure RBAC для авторизации Kubernetes](../aks/manage-azure-rbac.md)
* [Как и где развертывается модель](how-to-deploy-and-where.md)
* [Развертывание модели в кластере службы Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md)